# Linux 이름 없는 파이프

관련 개념: [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]], [[Linux/개념/Linux 프로세스|Linux 프로세스]]

## 개념

**이름 없는 파이프(anonymous pipe)**는 파일 시스템에 이름을 만들지 않고, 주로 부모·자식 프로세스 사이에서 데이터를 전달한다.

```c
#include <unistd.h>

int pipe(int pipefd[2]);
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

`pipe(pipefd)`가 성공하면 두 개의 파일 디스크립터가 만들어진다.

| 파일 디스크립터 | 용도 |
|---|---|
| `pipefd[0]` | 읽기 끝 |
| `pipefd[1]` | 쓰기 끝 |

```text
프로세스 A: write(pipefd[1], ...)
                 │
                 ▼
              파이프
                 │
                 ▼
프로세스 B: read(pipefd[0], ...)
```

보통 `pipe()`를 먼저 호출한 뒤 `fork()`하여 부모와 자식이 파일 디스크립터를 공유한다. 각 프로세스는 사용하지 않는 파이프 끝을 `close()`해야 한다.

## 핵심 정리

- 생성: `pipe()`
- 읽기: `read(pipefd[0], ...)`
- 쓰기: `write(pipefd[1], ...)`
- 파일 시스템에 이름이 없으므로 일반적으로 관련 있는 프로세스 사이에서 사용한다.
- 기본적으로 한쪽에서 쓰고 다른 쪽에서 읽는 **단방향 통신**이다.

