# Linux 시그널

## 개념

**시그널(signal)**은 프로세스에 특정 사건이 발생했음을 알리는 비동기 알림이다. 프로세스가 시그널에 대한 별도 처리 방법을 지정하지 않으면 해당 시그널의 **기본 동작(default action)**이 수행된다.

## 기본 동작 종류

| 표시 | 기본 동작 | 의미 |
|---|---|---|
| `Term` | 프로세스 종료 | 프로세스를 종료한다. |
| `Core` | 코어 덤프 후 종료 | 프로세스의 메모리 상태를 코어 덤프 파일로 남기고 종료한다. |
| `Stop` | 프로세스 정지 | 프로세스 실행을 일시 중단한다. |
| `Cont` | 프로세스 계속 실행 | 정지된 프로세스의 실행을 다시 시작한다. |
| `Ign` | 시그널 무시 | 시그널을 받아도 별도의 동작을 수행하지 않는다. |

> `Term`과 `Core`는 모두 프로세스를 종료하지만, `Core`는 오류 분석을 위한 코어 덤프를 남긴다는 차이가 있다.

## 시그널 동작 지정: `signal()`

`signal()`은 프로세스가 특정 시그널을 받았을 때 수행할 동작을 지정한다.

```c
#include <signal.h>

typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```

| 인수 | 의미 |
|---|---|
| `signum` | 동작을 지정할 시그널 번호 |
| `handler` | 시그널을 받았을 때 적용할 처리 방식 |

`handler`에는 다음 값을 지정할 수 있다.

| 값 | 동작 |
|---|---|
| 사용자 정의 함수 | 지정한 시그널 처리 함수 실행 |
| `SIG_DFL` | 해당 시그널의 기본 동작 수행 |
| `SIG_IGN` | 해당 시그널 무시 |

사용자 정의 시그널 처리 함수는 시그널 번호를 인수로 받는다.

```c
void handler(int signum)
{
    // 시그널 처리
}

signal(SIGINT, handler);
```

`signal()`이 성공하면 이전에 등록되어 있던 시그널 처리 방식을 반환하고, 실패하면 `SIG_ERR`를 반환한다.

> `SIGKILL`과 `SIGSTOP`은 사용자 정의 함수로 처리하거나 `SIG_IGN`으로 무시할 수 없다.

```text
시그널 수신
   ↓
등록된 동작 확인
   ├─ 사용자 함수 → handler(signum) 실행
   ├─ SIG_DFL     → 기본 동작 수행
   └─ SIG_IGN     → 무시
```

## Async-signal-safe

**Async-signal-safe 함수**는 프로그램이 어떤 작업을 수행하는 도중 시그널 처리 함수가 실행되어도 안전하게 호출할 수 있는 함수이다.

- 시그널 처리 함수에서는 async-signal-safe로 보장된 함수만 사용한다.
- `printf()`, `malloc()` 등 대부분의 표준 라이브러리 함수는 안전하지 않다.
- 간단한 출력에는 `write()`를 사용할 수 있다.
- 복잡한 작업은 처리 함수에서 직접 수행하지 않고 `volatile sig_atomic_t` 플래그만 변경한 뒤 기존 실행 흐름에서 처리한다.

```c
volatile sig_atomic_t signal_received = 0;

void handler(int signum)
{
    signal_received = 1;
}
```

## 시그널 전송 명령: `kill`

`kill` 명령은 지정한 PID의 프로세스에 시그널을 전송한다. 이름과 달리 프로세스를 종료하는 시그널만 보내는 명령은 아니다.

```bash
kill [옵션] PID
```

시그널을 생략하면 기본적으로 `SIGTERM`을 전송한다.

```bash
kill 1234
```

시그널 이름이나 번호를 직접 지정할 수 있다.

```bash
kill -TERM 1234
kill -9 1234
kill -STOP 1234
kill -CONT 1234
```

| 명령 | 의미 |
|---|---|
| `kill -TERM PID` | 프로세스에 정상적인 종료 요청 전송 |
| `kill -9 PID` | `SIGKILL`을 보내 프로세스 강제 종료 |
| `kill -STOP PID` | 프로세스 실행 정지 |
| `kill -CONT PID` | 정지된 프로세스 실행 재개 |

사용 가능한 시그널의 이름과 번호는 다음 명령으로 확인한다.

```bash
kill -l
```
