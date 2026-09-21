# fork 함수 부모 자식 분기

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]]

관련 예제:
- [[Linux/예제/EX02-03 fork 함수의 부모 자식 분기|EX02-03 fork 함수의 부모 자식 분기]]
- [[Linux/예제/EX02-04 wait와 waitpid로 자식 종료 대기|EX02-04 wait와 waitpid로 자식 종료 대기]]

## 시험 핵심

```c
pid_t pid_temp = fork();

if (pid_temp == -1) {
    // 생성 실패
} else if (pid_temp == 0) {
    // 자식 프로세스
} else {
    // 부모 프로세스
}
```

| 실행 위치 | `fork()` 반환값 | PID·PPID |
|---|---:|---|
| 자식 프로세스 | `0` | 새 PID를 가지며 PPID는 부모 PID |
| 부모 프로세스 | 자식 PID | 기존 PID와 PPID 유지 |
| 생성 실패 | `-1` | 자식 프로세스가 생성되지 않음 |

- `fork()`를 한 번 호출하면 성공 시 부모와 자식, 총 두 프로세스가 존재한다.
- 두 프로세스 모두 `fork()` 다음 명령부터 실행을 계속한다.
- 부모와 자식은 독립된 가상 메모리를 가지므로 변수 변경 결과가 서로 영향을 주지 않는다.
- 열린 FD, 현재 작업 디렉터리, 환경 변수는 자식에게 상속된다.
- 부모와 자식 중 어느 쪽이 먼저 실행되는지는 보장되지 않는다.

```text
자식: pid_temp = 0
부모: pid_temp = 자식 PID
```

## `wait()`와 `waitpid()`

```c
#include <sys/wait.h>

pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
```

| 함수 | 시험 핵심 |
|---|---|
| `wait(&status)` | 종료된 자식 하나를 기다리고 해당 자식 PID 반환 |
| `wait(NULL)` | 종료 상태를 저장하지 않고 자식 하나를 기다림 |
| `waitpid(child_pid, &status, 0)` | 지정한 자식이 종료될 때까지 대기 |
| `waitpid(child_pid, &status, WNOHANG)` | 기다리지 않고 자식 상태 확인 |

`waitpid()` 반환값:

| 반환값 | 의미 |
|---:|---|
| 자식 PID | 종료 상태 회수 성공 |
| `0` | `WNOHANG` 사용 시 종료된 자식이 없음 |
| `-1` | 오류 발생 |

```c
if (WIFEXITED(status)) {
    int exit_code = WEXITSTATUS(status);
} else if (WIFSIGNALED(status)) {
    int signal_number = WTERMSIG(status);
}
```

- 정상 종료에서는 exit code만 유효하다.
- 시그널 종료에서는 시그널 번호만 유효하다.
- 부모가 종료된 자식의 상태를 회수하지 않으면 자식은 좀비 프로세스로 남는다.
