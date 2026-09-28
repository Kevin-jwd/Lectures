# 스레드가 동시에 `errno`를 사용하면?

관련 개념: [[Linux/개념/Linux 스레드|Linux 스레드]], [[Linux/시험/재진입 가능 함수와 MT-safe 함수|재진입 가능 함수와 MT-safe 함수]]

## 질문

여러 스레드에서 동시에 오류가 발생하면 `errno` 값이 서로 덮어써지는가?

## 답

**POSIX 스레드 환경에서 `errno`는 스레드별로 관리된다.** 코드에는 하나의 이름처럼 보이지만, 각 스레드는 자기 `errno`에 접근한다. 따라서 스레드 A와 B에서 서로 다른 오류가 발생해도 B의 오류가 A의 `errno`를 덮어쓰지 않는다.

```text
스레드 A: 함수 실패 → A의 errno = ENOENT
스레드 B: 함수 실패 → B의 errno = EACCES
                       서로의 errno 값은 변경하지 않음
```

만약 모든 스레드가 **하나의 전역 오류 변수**를 공유한다면 나중에 오류가 난 스레드가 값을 덮어써 원인을 잘못 읽을 수 있다. `errno`가 스레드별로 관리되는 이유가 바로 이 문제를 막기 위해서다.

## 실제 주의할 점

- **오류를 반환한 함수의 결과를 먼저 확인**하고, 실패한 경우에만 `errno`를 읽는다. 성공한 함수가 `errno`를 `0`으로 초기화해 준다고 가정하면 안 된다.
- 같은 스레드에서 다른 함수를 호출하면 `errno`가 달라질 수 있으므로, 필요한 오류 번호는 실패 직후 저장한다.
- `pthread_create()`, `pthread_join()`, `pthread_mutex_lock()` 같은 Pthread 함수는 보통 실패 시 **오류 번호를 반환**한다. 이때 반환값을 확인하며 `errno`로 원인을 판단하지 않는다.

```c
int err = pthread_create(&thread, NULL, worker, NULL);
if (err != 0) {
    /* err가 오류 번호이며, errno를 읽지 않는다. */
}
```

## 핵심

**서로 다른 스레드의 `errno`는 충돌하지 않는다.** 다만 자기 스레드에서 발생한 오류도 다른 함수 호출 전에 확인하거나 저장해야 한다.

___

```c
errno is defined by the ISO C standard to be  a  modifiable  lvalue  of
       type  int,  and  must not be explicitly declared; errno may be a macro.
       errno is thread-local; setting it in one thread  does  not  affect  its
       value in any other thread.
```