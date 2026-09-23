# Linux 세마포어

관련 개념: [[Linux/개념/Linux 프로세스|Linux 프로세스]], [[Linux/개념/Linux 공유 메모리|Linux 공유 메모리]]

## 개념

**세마포어(Semaphore)**는 여러 프로세스가 공유 자원에 동시에 접근할 때 실행 순서를 제어하는 동기화 도구이다.

공유 메모리는 여러 프로세스가 같은 데이터를 볼 수 있게 하지만, 동시에 수정하면 **경쟁 상태(Race Condition)**가 발생할 수 있다. System V 세마포어는 공유 메모리와 같은 공유 자원에 접근하는 순서를 제어한다.

## System V 세마포어 함수

### `semget()`

세마포어 집합을 생성하거나 기존 세마포어 집합의 식별자를 얻는다.

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);
```

| 인자 | 의미 |
|---|---|
| `key` | 세마포어 집합을 구분하는 키 |
| `nsems` | 집합에 포함할 세마포어 개수 |
| `semflg` | 생성 옵션과 접근 권한 |

세마포어 배열을 두 칸 사용할 경우 `nsems`에 `2`를 지정한다.

- 성공: 세마포어 식별자 `semid` 반환
- 실패: `-1` 반환

### `semctl()`

세마포어의 초깃값을 설정하거나 값을 조회하고, 세마포어 집합을 제어한다.

```c
int semctl(int semid, int semnum, int cmd, ...);
```

| 인자 | 의미 |
|---|---|
| `semid` | `semget()`으로 얻은 세마포어 식별자 |
| `semnum` | 제어할 세마포어의 배열 번호 |
| `cmd` | 수행할 제어 명령 |

주요 명령은 다음과 같다.

| 명령 | 의미 |
|---|---|
| `SETVAL` | 세마포어 하나의 값 설정 |
| `SETALL` | 세마포어 배열 전체의 값 설정 |
| `GETVAL` | 세마포어 하나의 값 조회 |
| `IPC_RMID` | 세마포어 집합 제거 |

### `semop()`

세마포어 값을 변경하거나 특정 상태가 될 때까지 대기한다.

```c
int semop(int semid, struct sembuf *sops, size_t nsops);
```

```c
struct sembuf {
    unsigned short sem_num;  // 세마포어 배열 번호
    short          sem_op;   // 수행할 연산
    short          sem_flg;  // 동작 옵션
};
```

| `sem_op` | 동작 |
|---:|---|
| 양수 | 세마포어 값 증가 |
| 음수 | 값이 충분할 때 감소하고, 부족하면 대기 |
| `0` | 세마포어 값이 `0`이 될 때까지 대기 |

여러 개의 `sembuf` 연산을 한 번의 `semop()`에 전달하면 해당 연산들은 **원자적으로 한 묶음으로 처리**된다.

## Reader–Writer 세마포어 배열

하나의 파일에 Reader `R1`, `R2`, `R3`와 Writer `W1`, `W2`가 접근한다고 가정한다.

- 여러 Reader는 동시에 읽을 수 있다.
- Writer가 쓰는 동안에는 Reader와 다른 Writer가 접근할 수 없다.
- Reader가 한 명이라도 읽고 있으면 Writer는 접근할 수 없다.

세마포어 배열 두 칸을 다음과 같이 사용한다.

| 배열 | 초기값 | 의미 |
|---|---:|---|
| `sem[0]` | 0 | 현재 읽고 있는 Reader 수 |
| `sem[1]` | 0 | 현재 쓰고 있는 Writer 수: 없음 `0`, 있음 `1` |

이 구조에서는 세마포어를 단순한 이진 잠금으로 사용하지 않는다. 따라서 초기값이 `1`이 아니라 **Reader와 Writer가 아무도 없다는 의미의 `[0, 0]`**이다.

### Reader 진입

Reader는 Writer가 없는지 확인한 후 Reader 수를 증가시킨다.

```c
struct sembuf reader_lock[] = {
    {1,  0, 0},  // sem[1]이 0, 즉 Writer가 없을 때까지 대기
    {0, +1, 0}   // sem[0]의 Reader 수 증가
};

semop(semid, reader_lock, 2);
```

두 연산이 한 번의 `semop()`으로 실행되므로 Writer 확인과 Reader 수 증가가 원자적으로 처리된다.

```text
초기값       R1 진입       R2 진입       R3 진입
[0, 0]  →   [1, 0]   →   [2, 0]   →   [3, 0]
 Reader 수    Writer는 계속 0
```

### Reader 이탈

```c
struct sembuf reader_unlock = {0, -1, 0};

semop(semid, &reader_unlock, 1);
```

Reader가 나갈 때마다 `sem[0]`을 감소시킨다. 마지막 Reader가 나가면 값이 다시 `0`이 되어 Writer가 들어올 수 있다.

### Writer 진입

Writer는 Reader와 다른 Writer가 모두 없을 때까지 기다린 후 Writer 수를 증가시킨다.

```c
struct sembuf writer_lock[] = {
    {0,  0, 0},  // sem[0]이 0, 즉 Reader가 없을 때까지 대기
    {1,  0, 0},  // sem[1]이 0, 즉 다른 Writer가 없을 때까지 대기
    {1, +1, 0}   // sem[1]을 1로 변경
};

semop(semid, writer_lock, 3);
```

```text
초기값       W1 진입       W1 이탈
[0, 0]  →   [0, 1]   →   [0, 0]
              ↑
       Reader와 W2 진입 불가
```

### Writer 이탈

```c
struct sembuf writer_unlock = {1, -1, 0};

semop(semid, &writer_unlock, 1);
```

Writer가 작업을 끝내면 `sem[1]`을 `1 → 0`으로 감소시킨다.

## `1 → 0` 방식과의 차이

일반적인 이진 잠금에서는 사용 가능 상태를 `1`, 사용 중인 상태를 `0`으로 표현할 수 있다.

하지만 위의 System V 세마포어 배열은 값을 다음과 같이 사용한다.

- `sem[0]`: **현재 Reader 수**
- `sem[1]`: **현재 Writer 수**
- `0`: 현재 사용하는 프로세스가 없음
- 양수: 현재 사용하는 프로세스가 있음

따라서 이 구조에서는 Reader가 들어오면 `sem[0]`이 `0 → 1 → 2 → 3`으로 증가하고, Writer가 들어오면 `sem[1]`이 `0 → 1`로 증가한다.

## 핵심 정리

- `semget()`: 세마포어 집합 생성 또는 식별자 획득
- `semctl()`: 초깃값 설정, 값 조회, 세마포어 집합 제거
- `semop()`: 세마포어 값 변경 또는 값이 `0`이 될 때까지 대기
- Reader 진입: Writer가 없는지 확인하고 `sem[0]` 증가
- Writer 진입: Reader와 Writer가 모두 없는지 확인하고 `sem[1]` 증가
- 여러 연산을 하나의 `semop()`으로 전달해야 확인과 값 변경이 원자적으로 처리된다.
