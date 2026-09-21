# EX02-03 fork 함수의 부모 자식 분기

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]]

관련 시험: [[Linux/시험/fork 함수 부모 자식 분기|fork 함수 부모 자식 분기]]

## 문제

`fork()`로 자식 프로세스를 생성하고 반환값에 따라 부모와 자식이 실행하는 코드와 출력 결과를 구분한다.

## 코드

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

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
        msg = "this is parent";
        sleep(5);
    }

    printf("[%d] pid_temp = %d, msg = %s, ppid = %d\n",
           pid, pid_temp, msg, getppid());
    printf("[%d] terminted\n", pid);

    return EXIT_SUCCESS;
}
```

## 실행 흐름

```text
부모 프로세스
pid_temp = fork()
       │
       ├─ 자식: pid_temp = 0
       │         pid = 자식 PID
       │         msg = "this is child"
       │         sleep(3)
       │
       └─ 부모: pid_temp = 자식 PID
                 pid = 부모 PID
                 msg = "this is parent"
                 sleep(5)
```

## 출력 결과

PID가 다음과 같다고 가정한다.

```text
부모 PID: 1000
자식 PID: 1001
부모의 부모 PID: 900
```

터미널 실행 기준으로 다음과 같은 순서로 출력된다.

```text
[1000] running 실행파일명
[1001] pid_temp = 0, msg = this is child, ppid = 1000
[1001] terminted
[1000] pid_temp = 1001, msg = this is parent, ppid = 900
[1000] terminted
```

- `running`은 `fork()` 전에 실행되므로 한 번만 출력된다.
- 자식은 3초, 부모는 5초 동안 대기하므로 자식의 결과가 먼저 출력된다.
- 자식의 `pid_temp`는 `0`이고, 부모의 `pid_temp`는 자식 PID이다.
- 자식의 PPID는 부모 PID이고, 부모의 PPID는 부모를 실행한 셸 등의 PID이다.
- `fork()` 이후 부모와 자식은 각자의 `pid`와 `msg` 값을 독립적으로 변경한다.

## 핵심 포인트

- `fork()`는 한 번 호출되지만 부모와 자식에서 각각 반환된다.
- 부모와 자식은 `pid_temp` 반환값으로 구분한다.
- `fork()` 이후 두 프로세스는 다음 명령부터 각각 실행을 계속한다.
- 일반적으로 실행 순서는 스케줄링에 따라 달라질 수 있지만, 이 예제에서는 서로 다른 `sleep()` 시간 때문에 자식 출력이 먼저 나타난다.
