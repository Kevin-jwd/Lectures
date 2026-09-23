# EX02-06 여러 grep 자식 프로세스 실행

관련 개념:
- [[Linux/개념/Linux 프로세스|Linux 프로세스]]
- [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]]

## 프로그램 목적

검색 경로와 여러 검색 패턴을 인수로 받아, **패턴마다 자식 프로세스를 하나씩 생성**한다. 각 자식은 `exec()`로 `grep -rn` 프로그램을 실행하고, 부모는 모든 자식의 종료 상태를 `wait()`로 회수한다.

```bash
./EX02-06 {path} {pattern...}
```

## 코드

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/wait.h>

pid_t pid;

int main(int argc, char **argv)
{
    pid_t pid_temp;
    int i, status;
    int num_of_child;

    if (argc < 3) {
        printf("usage: %s {path} {pattern...}\n", argv[0]);
        return EXIT_FAILURE;
    }

    printf("[%d] running %s", pid = getpid(), argv[0]);
    for (i = 1; i < argc; i++) {
        printf(" %s", argv[i]);
    }
    printf("\n");

    num_of_child = argc - 2;

    /* fork and exec */
    for (i = 0; i < num_of_child; i++) {
        pid_temp = fork();
        if (pid_temp == 0) {
            execlp("grep", "grep", "-rn", argv[i + 2], argv[1], NULL);
            return EXIT_FAILURE;
        }
    }

    /* wait */
    for (i = 0; i < num_of_child; i++) {
        pid_temp = wait(&status);
        printf("[%d] child %d terminated with status %#x\n",
               pid, pid_temp, status);
    }

    printf("[%d] terminted\n", pid);

    return EXIT_SUCCESS;
}
```

## 인수 구성

| 인수 | 의미 |
|---|---|
| `argv[0]` | 실행 파일명 |
| `argv[1]` | 검색을 시작할 경로 |
| `argv[2]` 이후 | 각각의 검색 패턴 |

검색 패턴의 개수만큼 자식이 필요하므로 다음과 같이 계산한다.

```c
num_of_child = argc - 2;
```

## 자식 프로세스 생성

```c
for (i = 0; i < num_of_child; i++) {
    pid_temp = fork();

    if (pid_temp == 0) {
        execlp("grep", "grep", "-rn", argv[i + 2], argv[1], NULL);
        return EXIT_FAILURE;
    }
}
```

부모는 반복문을 계속 실행하며 검색 패턴마다 자식을 하나씩 만든다.

```text
부모
 ├─ fork() → 자식 1 → grep -rn 패턴1 경로
 ├─ fork() → 자식 2 → grep -rn 패턴2 경로
 ├─ ...
 └─ fork() → 자식 N → grep -rn 패턴N 경로
```

자식에서 `execlp()`가 성공하면 현재 자식 프로세스가 `grep`으로 교체되므로 다음 반복으로 진행하지 않는다. `execlp()`가 실패해도 바로 `return EXIT_FAILURE`로 종료한다. 따라서 자식이 다시 `fork()`를 반복하지 않고, 생성되는 자식 수는 정확히 `num_of_child`개이다.

## `grep` 실행

```c
execlp("grep", "grep", "-rn", argv[i + 2], argv[1], NULL);
```

실행되는 명령은 다음과 같다.

```bash
grep -rn 패턴 검색경로
```

| 항목 | 의미 |
|---|---|
| `execlp()` | `PATH`에서 `grep` 실행 파일 검색 |
| `-r` | 하위 디렉터리까지 재귀 검색 |
| `-n` | 검색 결과에 행 번호 출력 |
| `argv[i + 2]` | 현재 자식이 검색할 패턴 |
| `argv[1]` | 검색 경로 |

각 자식은 동시에 실행될 수 있으므로 여러 `grep`의 출력 순서는 일정하지 않으며 서로 섞여 나타날 수 있다.

## 자식 프로세스 회수

```c
for (i = 0; i < num_of_child; i++) {
    pid_temp = wait(&status);
    printf("[%d] child %d terminated with status %#x\n",
           pid, pid_temp, status);
}
```

부모는 생성한 자식 수만큼 `wait()`를 호출한다.

- `wait()`는 종료된 자식 하나의 PID를 반환한다.
- 자식은 생성 순서가 아니라 **종료된 순서**로 회수된다.
- `status`에는 자식의 종료 상태가 인코딩되어 저장된다.
- 모든 자식을 회수하므로 종료된 자식이 좀비 프로세스로 남지 않는다.

## 종료 상태

`grep`의 일반적인 종료 코드는 다음과 같다.

| 종료 코드 | 의미 | 원시 `status` 예시 |
|---:|---|---:|
| `0` | 패턴을 찾음 | `0x0000` |
| `1` | 패턴을 찾지 못함 | `0x0100` |
| `2` | 파일 접근 등 오류 발생 | `0x0200` |

현재 코드는 원시 상태를 `%#x`로 출력한다. 실제 종료 코드를 확인하려면 다음과 같이 해석한다.

```c
if (WIFEXITED(status)) {
    int exit_code = WEXITSTATUS(status);
}
```

## 실행 예시

```bash
./EX02-06 /etc init ssh
```

```text
num_of_child = 2

자식 1 → grep -rn init /etc
자식 2 → grep -rn ssh /etc
부모   → wait() 두 번 호출
```

검색 결과와 자식 종료 메시지는 실행 시점과 검색량에 따라 순서가 달라질 수 있다.

## 주의할 점

- 부모는 `fork()`가 `-1`을 반환하는 실패 상황을 별도로 처리하지 않는다.
- `fork()`에 실패해도 부모가 원래 계산한 횟수만큼 `wait()`를 호출하면 `wait()`가 `-1`을 반환할 수 있다.
- 자식에서 `execlp()`가 실패한 원인을 출력하지 않으므로 `perror()` 등을 추가하면 원인 확인이 쉽다.
- `wait()`의 반환값이 `-1`인지 확인한 뒤 종료 상태를 해석해야 한다.

## 핵심 정리

- 검색 패턴 하나당 자식 프로세스 하나를 생성한다.
- 자식은 `exec()` 성공 또는 실패 후 종료하므로 자식이 추가 자식을 만들지 않는다.
- `execlp()`는 `PATH`에서 `grep`을 찾아 실행한다.
- 여러 자식의 실행과 출력 순서는 보장되지 않는다.
- 부모는 자식 수만큼 `wait()`를 호출하여 모든 종료 상태를 회수한다.
