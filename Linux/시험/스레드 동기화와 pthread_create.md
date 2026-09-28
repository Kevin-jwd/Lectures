# 스레드 동기화와 `pthread_create()`

관련 개념: [[Linux/개념/Linux 스레드|Linux 스레드]]

## 공유 자원과 동기화

같은 프로세스의 스레드는 주소 공간과 자원을 공유한다. **공유 데이터를 동시에 수정하면 경쟁 상태가 생길 수 있으므로 임계영역에 동기화 기법이 필요하다.**

## `pthread_create()`의 `start_routine`

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine)(void *), void *arg);
```

`start_routine`은 **새 스레드가 처음 실행할 함수**를 가리킨다. 이 함수가 `return`하면 해당 스레드가 종료되며, 반환값은 스레드의 종료값이 된다. 다른 스레드가 그 값을 받으려면 `pthread_join()`을 사용한다.
