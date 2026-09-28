# Linux 스레드

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]], [[Linux/개념/Linux 세마포어|Linux 세마포어]]

시험 요약: [[Linux/시험/스레드 동기화와 pthread_create|스레드 동기화와 pthread_create]]

함수 안전성 시험 요약: [[Linux/시험/재진입 가능 함수와 MT-safe 함수|재진입 가능 함수와 MT-safe 함수]]

## 프로세스·스레드·태스크

- **프로세스**: 독립된 주소 공간과 자원을 가진 실행 단위.
- **스레드**: 프로세스 안에서 코드를 실행하는 흐름. 같은 프로세스의 스레드는 주소 공간과 파일 디스크립터 등을 공유하며, 각자 스택과 실행 상태를 가진다.
- **POSIX 스레드(Pthread)**: 유저 영역의 프로그램에서 스레드를 만들고 관리할 때 사용하는 표준 API.
- **태스크(task)**: Linux 커널이 스케줄링하는 실행 단위. Linux에서는 같은 프로세스에 속한 스레드도 각각 커널 태스크로 관리된다.

```text
유저 영역:  프로세스의 주소 공간 ─┬─ Pthread A (자기 스택·실행 상태)
                              └─ Pthread B (자기 스택·실행 상태)
                    공통: 코드·전역 변수·힙·파일 디스크립터
커널 영역:                     ├─ 태스크 A
                              └─ 태스크 B
```

## 스레드 동작과 동기화 *시험 출제*

같은 프로세스의 스레드는 자원을 공유하므로 여러 스레드가 공유 데이터를 동시에 읽고 수정할 수 있다. 수정 순서에 따라 결과가 달라지는 **경쟁 상태**를 막으려면 공유 데이터에 접근하는 임계영역에 **동기화 기법이 필요하다**. 한 번에 한 스레드만 접근하도록 뮤텍스로 보호하거나, 세마포어로 접근 수를 제어할 수 있다.

## 장단점

| 장점 | 단점 |
|---|---|
| 같은 프로세스의 자원 공유가 쉬움 | 공유 자원에서 동시성 문제 발생 가능 |
| 한 스레드가 입출력 등을 기다리는 동안 다른 스레드가 작업하기 용이함 | 실행 순서가 달라져 디버깅이 어려움 |
| 별도 프로세스 생성보다 생성·전환 비용이 적음 | 스레드가 너무 많으면 전환·관리 비용으로 성능 저하 |
| | 한 스레드의 심각한 오류가 프로세스 전체에 영향 가능 |

## `pthread_create()`

새 스레드를 생성한다.

```c
#include <pthread.h>

int pthread_create(pthread_t *thread,
                   const pthread_attr_t *attr,
                   void *(*start_routine)(void *),
                   void *arg);
```

| 인자 | 의미 |
|---|---|
| `thread` | 생성된 스레드의 식별자를 저장할 곳 |
| `attr` | 스레드 속성. 기본 속성이면 `NULL` |
| `start_routine` | **새 스레드가 실행을 시작할 함수**를 가리킴 |
| `arg` | 시작 함수에 전달할 인자 |

`start_routine` 함수가 반환하면 **해당 스레드의 실행이 종료**된다. 이때 반환한 포인터는 스레드의 종료값이 된다. `pthread_create()`는 성공 시 `0`, 실패 시 오류 번호를 반환한다.

## 스레드 종료 조건

- `start_routine`에서 `return`: 해당 스레드가 종료된다.
- `pthread_exit()` 호출: 호출한 스레드가 종료된다.
- 다른 스레드가 취소를 요청하고 취소가 적용됨: 해당 스레드가 종료된다.
- 프로세스가 종료됨: 프로세스 안의 모든 스레드가 종료된다. 특히 `main()`에서 `return`하거나 어느 스레드에서든 `exit()`을 호출하면 프로세스 전체가 종료된다.

## `pthread_exit()`와 `pthread_join()`

```c
void pthread_exit(void *retval);
int pthread_join(pthread_t thread, void **retval);
```

- `pthread_exit(retval)`: **호출한 스레드만** 종료하고 `retval`을 종료값으로 남긴다. 다른 스레드가 계속 실행 중이면 프로세스는 유지된다.
- `pthread_join(thread, &retval)`: 지정한 **join 가능한 스레드의 종료를 기다리고** 종료값을 받는다. 종료값이 필요 없으면 두 번째 인자에 `NULL`을 줄 수 있다.
- `pthread_join()` 자체는 호출한 스레드를 종료시키지 않는다. 성공하면 `0`, 실패하면 오류 번호를 반환한다.

## Pthread 뮤텍스

**뮤텍스(mutex)**는 한 번에 하나의 스레드만 임계영역에 들어가도록 하는 잠금이다. 공유 데이터를 사용하는 스레드들이 **같은 뮤텍스**를 잠가야 보호 효과가 있다.

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *mutex,
                       const pthread_mutexattr_t *attr);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

| 함수 | 역할 |
|---|---|
| `pthread_mutex_init()` | 뮤텍스를 초기화한다. 기본 속성이면 `attr`에 `NULL`을 지정한다. |
| `pthread_mutex_lock()` | 잠금을 획득한다. 다른 스레드가 사용 중이면 해제될 때까지 기다린다. |
| `pthread_mutex_unlock()` | 현재 스레드가 획득한 잠금을 해제한다. |
| `pthread_mutex_destroy()` | 사용을 마친 뮤텍스를 정리한다. 잠겨 있거나 다른 스레드가 사용하는 동안 호출하면 안 된다. |

```text
pthread_mutex_init() → pthread_mutex_lock() → 공유 데이터 접근
                     → pthread_mutex_unlock() → pthread_mutex_destroy()
```

`init()`은 사용 전에 한 번, `destroy()`는 모든 스레드가 사용을 마친 뒤 한 번 호출한다. `lock()`과 `unlock()`은 공유 데이터에 접근할 때마다 짝을 이뤄 호출한다. 네 함수 모두 성공 시 `0`, 실패 시 오류 번호를 반환한다.

## 재진입 가능 함수와 MT-safe 함수 *시험 출제*

| 구분 | 의미 |
|---|---|
| **재진입 가능 함수(reentrant)** | 함수 실행 도중 같은 함수가 다시 호출되어도 각 호출의 데이터가 서로 섞이지 않고 올바르게 동작하는 함수. 공유하는 가변 상태에 의존하지 않는 방식으로 작성한다. |
| **MT-safe 함수(thread-safe)** | 여러 스레드가 동시에 호출해도 올바르게 동작하는 함수. 내부 잠금 등으로 공유 상태를 보호하는 방법도 가능하다. |

**재진입 가능하면 일반적으로 MT-safe이지만, MT-safe라고 반드시 재진입 가능한 것은 아니다.** 예를 들어 내부 뮤텍스로 동시 호출을 막는 함수는 스레드 간 호출에는 안전할 수 있으나, 실행 중 같은 스레드가 다시 호출하면 잠금에서 멈출 수 있다.
