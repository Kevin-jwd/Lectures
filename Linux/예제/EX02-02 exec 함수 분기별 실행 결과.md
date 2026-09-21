# EX02-02 exec 함수 분기별 실행 결과

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]]

## 문제

`execl()`과 `execlp()`의 실행 파일 검색 방식과 `argv[0]`의 의미를 구분하고, A~D 분기별 출력 결과를 확인한다.

`#if` 뒤의 값을 `1`로 설정한 분기만 컴파일된다.

## 코드

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

pid_t pid;

int main(int argc, char **argv)
{
    int ret;

    if (argc != 1) {
        printf("usage: %s\n", argv[0]);
        return EXIT_FAILURE;
    }
    printf("[%d] running %s\n", pid = getpid(), argv[0]);

#if 0 /* A */
    ret = execl("/bin/ls", "ls", "-l", NULL);
#endif

#if 0 /* B */
    ret = execl("/bin/ls", "xxx", "-l", NULL);
#endif

#if 0 /* C */
    ret = execl("ls", "ls", "-l", NULL);
#endif

#if 1 /* D */
    ret = execlp("ls", "ls", "-l", NULL);
#endif

    printf("[%d] ret = %d\n", pid, ret);
    printf("[%d] terminted\n", pid);

    return EXIT_FAILURE;
}
```

## 분기별 결과

다음 출력은 프로그램을 터미널에서 실행한 경우를 기준으로 한다.

### A: 절대 경로로 `execl()` 실행

```c
ret = execl("/bin/ls", "ls", "-l", NULL);
```

- `/bin/ls`가 존재하므로 `exec`에 성공한다.
- 현재 프로세스가 `ls -l` 프로그램으로 교체되며 PID는 유지된다.
- 디렉터리의 상세 목록이 출력된다.
- 성공한 `exec`는 돌아오지 않으므로 `ret`과 `terminted`는 출력되지 않는다.

```text
[PID] running 실행파일명
total ...
... ls -l 결과 ...
```

### B: `argv[0]`을 다른 이름으로 전달

```c
ret = execl("/bin/ls", "xxx", "-l", NULL);
```

- 실행 파일 경로는 여전히 `/bin/ls`이므로 `exec`에 성공한다.
- `"xxx"`는 실행할 파일명이 아니라 새 프로그램에 전달되는 `argv[0]`이다.
- 정상적인 `ls -l` 목록이 출력된다.
- 성공했으므로 `ret`과 `terminted`는 출력되지 않는다.

```text
[PID] running 실행파일명
total ...
... ls -l 결과 ...
```

> `argv[0]`은 관례적으로 프로그램 이름을 전달하지만, 실행할 파일은 `execl()`의 첫 번째 인수로 결정된다.

### C: 경로 검색 없이 `execl()` 실행

```c
ret = execl("ls", "ls", "-l", NULL);
```

- `execl()`은 `PATH`에서 실행 파일을 검색하지 않는다.
- 현재 디렉터리에 `ls`라는 실행 파일이 없다면 실행에 실패하고 `-1`을 반환한다.
- 기존 프로세스가 계속 실행되므로 뒤의 두 `printf()`가 실행된다.

```text
[PID] running 실행파일명
[PID] ret = -1
[PID] terminted
```

### D: `PATH`를 검색하는 `execlp()` 실행

```c
ret = execlp("ls", "ls", "-l", NULL);
```

- `execlp()`의 `p`는 `PATH` 환경 변수에서 실행 파일을 검색한다는 의미이다.
- `PATH`에서 `ls`를 찾아 실행하므로 일반적인 환경에서는 성공한다.
- 현재 프로세스가 `ls -l`로 교체되고 PID는 유지된다.
- 성공했으므로 `ret`과 `terminted`는 출력되지 않는다.

```text
[PID] running 실행파일명
total ...
... ls -l 결과 ...
```

## 결과 비교

| 분기 | 실행 파일 결정 | 결과 | 이후 `printf()` |
|---|---|---|---|
| A | 절대 경로 `/bin/ls` | 성공 | 실행되지 않음 |
| B | 절대 경로 `/bin/ls` | 성공 | 실행되지 않음 |
| C | 현재 디렉터리의 `ls` | 일반적으로 실패 | 실행됨 |
| D | `PATH`에서 `ls` 검색 | 성공 | 실행되지 않음 |

## 핵심 포인트

- `execl()`은 첫 번째 인수에 지정된 경로를 그대로 사용한다.
- `execlp()`는 `PATH`에서 실행 파일을 검색한다.
- 두 번째 인수는 새 프로그램에 전달되는 `argv[0]`이다.
- `exec()`가 성공하면 호출 이후 코드는 실행되지 않는다.
- `exec()`가 실패하면 `-1`을 반환하고 기존 프로그램이 계속 실행된다.
