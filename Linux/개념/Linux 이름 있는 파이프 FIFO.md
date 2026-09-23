# Linux 이름 있는 파이프 FIFO

관련 개념: [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]], [[Linux/개념/Linux 파일 입출력|Linux 파일 입출력]]

## 개념

**이름 있는 파이프(FIFO)**는 파일 시스템에 경로를 가지는 파이프이다. 부모·자식 관계가 없는 프로세스도 같은 FIFO 경로를 열어 통신할 수 있다.

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int mkfifo(const char *pathname, mode_t mode);
int open(const char *pathname, int flags);
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

```text
쓰기 프로세스                     읽기 프로세스
open(path, O_WRONLY)             open(path, O_RDONLY)
write(fd, ...) ───▶ FIFO ───▶ read(fd, ...)
```

1. `mkfifo()`로 FIFO 파일을 생성한다.
2. 각 프로세스가 같은 경로를 `open()`한다.
3. 쓰는 쪽은 `write()`, 읽는 쪽은 `read()`를 사용한다.
4. 사용이 끝나면 파일 디스크립터를 `close()`한다.

FIFO의 데이터는 일반 파일처럼 디스크에 저장되는 것이 아니라 커널의 파이프 버퍼를 통해 전달된다. 경로는 통신할 프로세스들이 FIFO를 찾기 위한 이름이다.

## 핵심 정리

- 생성: `mkfifo(pathname, mode)`
- 읽기: `read()`
- 쓰기: `write()`
- 파일 시스템에 이름이 있어 서로 관계없는 프로세스도 사용할 수 있다.
- FIFO는 **First In, First Out** 순서로 데이터를 전달한다.

