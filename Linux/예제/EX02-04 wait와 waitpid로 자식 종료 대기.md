# EX02-04 wait와 waitpid로 자식 종료 대기

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]]

관련 시험: [[Linux/시험/fork 함수 부모 자식 분기|fork 함수 부모 자식 분기]]

## 문제

부모 프로세스가 `wait()` 또는 `waitpid()`를 사용하여 자식 프로세스의 종료를 기다리고 종료 상태를 회수하는 방식을 비교한다.

`#if` 뒤의 값을 `1`로 설정한 대기 방식만 컴파일된다.

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
    char *msg = "none";

    if (argc != 1) {
        printf("usage: %s\n", argv[0]);
        return EXIT_FAILURE;
    }
    printf("[%d] running %s\n", pid = getpid(), argv[0]);

    pid_temp = fork();

    if (pid_temp == -1) {
        printf("[%d] error: %s (%d)\n", pid, strerror(errno), __LINE__);
        return EXIT_FAILURE;
    }
    else if (pid_temp == 0) {
        pid = getpid();
        msg = "this is child";
        sleep(3);
    }
    else {
        pid_t pid_wait;
        int status;

        msg = "this is parent";
        printf("[%d] waiting child's termination\n", pid);

#if 0 /* using wait() */
        pid_wait = wait(&status);
#endif

#if 1 /* using waitpid() without WNOHANG */
        pid_wait = waitpid(pid_temp, &status, 0);
#endif

#if 0 /* using waitpid() with WNOHANG */
        for (;;) {
            pid_wait = waitpid(pid_temp, &status, WNOHANG);
            if (pid_wait)
                break;
        }
#endif

        printf("[%d] pid %d has been terminated with status %#x\n",
               pid, pid_wait, status);
    }

    printf("[%d] pid_temp = %d, msg = %s, ppid = %d\n",
           pid, pid_temp, msg, getppid());
    printf("[%d] terminted\n", pid);

    return 8;
}
```

## 공통 동작

```text
부모
 ├─ fork() → 자식 생성
 │            └─ sleep(3) → 정보 출력 → return 8
 │
 └─ 자식 종료 대기 → 종료 상태 회수 → 부모 정보 출력
```

- 자식의 `pid_temp`는 `0`이다.
- 부모의 `pid_temp`에는 자식 PID가 저장된다.
- 부모는 자식의 종료 상태를 회수한 뒤 다음 코드를 실행한다.
- 자식과 부모 모두 마지막에 `return 8`로 종료한다.

## 분기별 동작 방식

### A: `wait()` 사용

```c
pid_wait = wait(&status);
```

- 종료되는 자식 하나를 기다린다.
- 어떤 자식을 기다릴지 PID로 지정하지 않는다.
- 이 예제에는 자식이 하나뿐이므로 해당 자식이 종료될 때까지 부모가 차단된다.
- 자식이 종료되면 `pid_wait`에 자식 PID, `status`에 인코딩된 종료 상태가 저장된다.

### B: `waitpid()` 차단 방식

```c
pid_wait = waitpid(pid_temp, &status, 0);
```

- `pid_temp`로 지정한 자식이 종료될 때까지 부모가 차단된다.
- 세 번째 인수 `0`은 비차단 옵션을 사용하지 않는다는 의미이다.
- 현재 코드에서 활성화된 분기이다.
- 자식이 종료되면 `pid_wait`에 자식 PID, `status`에 종료 상태가 저장된다.

### C: `waitpid()` 비차단 방식

```c
for (;;) {
    pid_wait = waitpid(pid_temp, &status, WNOHANG);
    if (pid_wait)
        break;
}
```

- 자식이 아직 종료되지 않았다면 기다리지 않고 `0`을 반환한다.
- 반복문이 계속 자식 상태를 확인하다가 자식이 종료되면 자식 PID를 반환한다.
- 대기 중에도 부모가 다른 작업을 수행할 수 있는 방식이다.
- 이 코드처럼 다른 작업이나 대기 없이 반복하면 CPU를 계속 사용하는 **바쁜 대기(busy waiting)**가 된다.

## 종료 상태 해석

자식은 마지막에 `return 8`로 정상 종료한다.

```text
exit code       = 8
raw wait status = 8 << 8
                = 0x0800
```

따라서 부모의 다음 출력에서 `status`는 `0x800`으로 표시된다.

```c
WIFEXITED(status)   // 참
WEXITSTATUS(status) // 8
```

## 예상 출력

부모 PID가 `1000`, 자식 PID가 `1001`, 부모를 실행한 셸의 PID가 `900`이라고 가정한다.

```text
[1000] running 실행파일명
[1000] waiting child's termination
[1001] pid_temp = 0, msg = this is child, ppid = 1000
[1001] terminted
[1000] pid 1001 has been terminated with status 0x800
[1000] pid_temp = 1001, msg = this is parent, ppid = 900
[1000] terminted
```

부모의 `wait()` 또는 `waitpid()`가 자식 종료까지 기다리므로 부모의 종료 상태 출력은 자식의 출력 뒤에 나타난다.

## 핵심 포인트

- `wait()`는 종료된 자식 하나를 대상으로 한다.
- `waitpid()`는 기다릴 자식 PID와 대기 방식을 지정할 수 있다.
- `options`가 `0`이면 차단, `WNOHANG`이면 비차단 방식이다.
- 부모가 종료 상태를 회수하면 자식은 좀비 프로세스로 남지 않는다.
- `status`는 원시 종료 상태이므로 `WEXITSTATUS()`로 실제 exit code를 추출한다.

