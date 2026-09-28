# 메시지 큐 `msgsnd()`와 `msgrcv()`

관련 개념: [[Linux/개념/Linux 메시지 큐|Linux 메시지 큐]]

## `msgsnd()`의 `msgflg`

```c
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
```

### `IPC_NOWAIT`를 사용하지 않은 경우

메시지 큐에 충분한 공간이 없으면 공간이 생길 때까지 **대기한다**.

### `IPC_NOWAIT`를 사용한 경우

```c
msgsnd(msqid, &msg, msgsz, IPC_NOWAIT);
```

메시지 큐에 충분한 공간이 없더라도 대기하지 않고 즉시 `-1`을 반환하며 `errno`는 `EAGAIN`이 된다.

## `msgrcv()`의 `msgtyp`

```c
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz,
               long msgtyp, int msgflg);
```

| `msgtyp` | 선택하는 메시지 |
|---:|---|
| `0` | 큐의 가장 앞에 있는 첫 번째 메시지 |
| `0` 초과 | `mtype`이 `msgtyp`과 정확히 같은 첫 번째 메시지 |
| `0` 미만 | `mtype <= |msgtyp|`인 메시지 중 **가장 작은 타입**의 첫 번째 메시지 |

예를 들어 큐에 타입 `1`, `3`, `5`의 메시지가 있을 때 다음과 같이 선택된다.

| 호출할 때의 `msgtyp` | 선택 결과 |
|---:|---|
| `0` | 큐의 맨 앞 메시지 |
| `3` | 타입이 `3`인 첫 번째 메시지 |
| `-4` | 타입이 `4` 이하인 것 중 가장 작은 타입인 `1` |

## `msgrcv()`의 `msgflg`

### `IPC_NOWAIT`를 사용하지 않은 경우

조건에 맞는 메시지가 없으면 메시지가 도착할 때까지 **대기한다**.

### `IPC_NOWAIT`를 사용한 경우

```c
msgrcv(msqid, &msg, msgsz, msgtyp, IPC_NOWAIT);
```

조건에 맞는 메시지가 없어도 대기하지 않고 즉시 `-1`을 반환하며 `errno`는 `ENOMSG`가 된다.

## 시험 핵심

- `msgsnd() + IPC_NOWAIT`: 큐가 가득 찼을 때 대기하지 않고 실패
- `msgrcv() + IPC_NOWAIT`: 조건에 맞는 메시지가 없을 때 대기하지 않고 실패
- `msgtyp == 0`: 큐의 첫 번째 메시지
- `msgtyp > 0`: 지정한 타입과 같은 첫 번째 메시지
- `msgtyp < 0`: 절댓값 이하의 타입 중 가장 작은 타입의 첫 번째 메시지
