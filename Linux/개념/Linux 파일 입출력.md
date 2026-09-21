# Linux 파일 입출력

## 저수준 파일 입출력

### 개념

Linux의 저수준 파일 입출력은 `open()`, `read()`, `write()`, `close()` 등의 **시스템 호출**을 사용한다. 파일을 열어 얻은 **파일 디스크립터(File Descriptor, FD)**를 통해 파일과 입출력 자원에 접근한다.

```text
open() → FD 획득 → read() / write() → close()
```

필요한 헤더는 다음과 같다.

```c
#include <sys/types.h> // mode_t, ssize_t
#include <sys/stat.h>  // S_IRUSR 등의 권한 매크로
#include <fcntl.h>     // open(), O_* 플래그
#include <unistd.h>    // read(), write(), close()
```

### 파일 디스크립터

파일 디스크립터는 프로세스가 열어 놓은 파일이나 입출력 자원을 구분하기 위해 사용하는 **0 이상의 정수 번호**이다. 운영체제는 프로세스마다 독립적인 FD 테이블을 관리한다.

- 같은 숫자의 FD라도 프로세스가 다르면 서로 다른 파일을 가리킬 수 있다.
- 한 프로세스가 같은 파일을 여러 번 열면 서로 다른 FD가 할당될 수 있다.
- 여러 프로세스가 하나의 파일을 동시에 열 수 있다.
- FD 번호는 시스템 전체가 아니라 해당 프로세스 안에서만 유효하다.

#### 표준 파일 디스크립터

프로세스가 시작될 때 일반적으로 다음 세 FD가 미리 열린다.

| FD | 이름 | 기본 연결 대상 |
|---:|---|---|
| `0` | 표준 입력(stdin) | 키보드 또는 터미널 입력 |
| `1` | 표준 출력(stdout) | 모니터 또는 터미널 출력 |
| `2` | 표준 오류(stderr) | 모니터 또는 터미널 출력 |

따라서 새 파일을 열면 보통 사용 중이지 않은 가장 작은 번호인 `3`부터 할당된다. 표준 입출력과 리다이렉션은 [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]]에서 확인할 수 있다.

#### 프로세스마다 다른 FD

| FD | 프로세스 A | 프로세스 B |
|---:|---|---|
| `0` | 표준 입력 | 표준 입력 |
| `1` | 표준 출력 | 표준 출력 |
| `2` | 표준 오류 | 표준 오류 |
| `3` | `a.txt` | `c.bmp` |
| `4` | `b.bin` | 사용하지 않음 |
| `5` | `b.bin` | `b.bin` |

프로세스 A의 FD `3`과 프로세스 B의 FD `3`은 서로 다른 파일을 가리킨다. 반대로 두 프로세스의 FD 번호가 달라도 같은 파일을 가리킬 수 있다.

### 주요 함수

| 함수 | 역할 | 성공 시 반환값 | 실패 시 반환값 |
|---|---|---|---|
| `open()` | 파일을 열거나 생성 | 파일 디스크립터 | `-1` |
| `read()` | FD에서 데이터 읽기 | 읽은 바이트 수 | `-1` |
| `write()` | FD에 데이터 쓰기 | 쓴 바이트 수 | `-1` |
| `close()` | FD 닫기 | `0` | `-1` |

### 파일 열기: `open()`

기존 파일을 열 때는 다음 형식을 사용한다.

```c
int open(const char *pathname, int flags);
```

새 파일을 생성할 때는 권한을 지정하는 `mode` 인수가 추가된다.

```c
int open(const char *pathname, int flags, mode_t mode);
```

> C에서 `open()`이 오버로딩된 것은 아니다. 실제 헤더에서는 가변 인자 형태로 선언되며, `O_CREAT`로 파일을 생성할 때 세 번째 인수 `mode`를 전달하는 방식이다.

- `pathname`: 열거나 생성할 파일의 경로
- `flags`: 파일 접근 방식과 동작 옵션
- `mode`: 새로 생성하는 파일의 접근 권한

#### 주요 `flags`

| 플래그 | 의미 |
|---|---|
| `O_RDONLY` | 읽기 전용으로 열기 |
| `O_WRONLY` | 쓰기 전용으로 열기 |
| `O_RDWR` | 읽기와 쓰기용으로 열기 |
| `O_CREAT` | 파일이 없으면 새로 생성 |
| `O_TRUNC` | 기존 파일 내용을 비움 |
| `O_APPEND` | 데이터를 파일 끝에 추가 |

여러 플래그는 비트 OR 연산자 `|`로 결합한다.

#### 파일 생성 권한: `mode` *시험 출제* — [[Linux/시험/open 파일 생성 권한|시험 요약]]

`mode`는 `O_CREAT`로 **새 파일을 생성할 때 부여할 접근 권한**을 지정한다. `O_CREAT` 없이 기존 파일을 열 때는 `mode`를 전달하지 않는다.

```c
int fd = open("a.txt", O_WRONLY | O_CREAT, 0644);
```

`mode`는 일반적으로 8진수로 작성하며, 맨 앞에 `0`을 붙인다. 뒤의 세 자리는 차례대로 소유자, 소유 그룹, 기타 사용자의 권한을 나타낸다.

| 권한 | 값 |
|---|---:|
| 읽기 `r` | `4` |
| 쓰기 `w` | `2` |
| 실행 `x` | `1` |

```text
0644
 ││└─ 기타 사용자: 4 = r--
 │└── 소유 그룹:   4 = r--
 └─── 소유자:      6 = rw-
```

따라서 `0644`는 `rw-r--r--` 권한을 의미한다.

권한 매크로를 비트 OR 연산자로 결합해서 같은 권한을 지정할 수도 있다.

```c
mode_t mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;
int fd = open("a.txt", O_WRONLY | O_CREAT, mode);
```

| 매크로 | 의미 | 값 |
|---|---|---:|
| `S_IRUSR` | 소유자 읽기 | `0400` |
| `S_IWUSR` | 소유자 쓰기 | `0200` |
| `S_IXUSR` | 소유자 실행 | `0100` |
| `S_IRGRP` | 소유 그룹 읽기 | `0040` |
| `S_IWGRP` | 소유 그룹 쓰기 | `0020` |
| `S_IXGRP` | 소유 그룹 실행 | `0010` |
| `S_IROTH` | 기타 사용자 읽기 | `0004` |
| `S_IWOTH` | 기타 사용자 쓰기 | `0002` |
| `S_IXOTH` | 기타 사용자 실행 | `0001` |

##### `mode`와 `umask`

`mode`에 지정한 권한이 그대로 적용되는 것은 아니다. 프로세스의 `umask`에서 제한한 권한 비트는 제거된다.

```text
실제 생성 권한 = mode & ~umask
```

예를 들어 `mode`가 `0666`이고 `umask`가 `0022`이면 실제 파일은 `0644`로 생성된다.

```text
0666 (rw-rw-rw-)
0022 (----w--w-) 제거
0644 (rw-r--r--)
```

> 파일이 이미 존재하면 `O_CREAT`를 사용해도 `mode`는 기존 파일의 권한을 변경하지 않는다. 기존 권한을 바꾸려면 `chmod()`를 사용한다.

기존 내용을 비우고 새로 작성할 파일을 열 때는 다음과 같이 플래그를 조합한다.

```c
int fd = open("a.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```

자세한 권한 체계는 [[Linux/개념/Linux 파일 접근 권한|Linux 파일 접근 권한]]에서 확인할 수 있다.

`open()`이 성공하면 FD를 반환한다. 실패하면 `-1`을 반환하므로 사용하기 전에 확인해야 한다.

```c
int fd = open("a.txt", O_RDONLY);

if (fd == -1) {
    // 파일 열기 실패
}
```

### 파일 읽기: `read()`

```c
ssize_t read(int fd, void *buf, size_t count);
```

- `fd`: 읽을 파일 디스크립터 (파일 식별자: 파일 고유 번호)
- `buf`: 읽은 데이터를 저장할 버퍼
- `count`: 최대한 읽을 바이트 수

반환값은 실제로 읽은 바이트 수이다.

| 반환값 | 의미 |
|---:|---|
| `1` 이상 | 실제로 읽은 바이트 수 |
| `0` | 파일 끝(EOF)에 도달 |
| `-1` | 읽기 실패 |

한 번의 호출에서 요청한 크기보다 적은 데이터가 읽힐 수 있으므로 반환값을 기준으로 처리해야 한다.

```c
char buffer[128];
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
```

### 파일 쓰기: `write()`

```c
ssize_t write(int fd, const void *buf, size_t count);
```

- `fd`: 데이터를 쓸 파일 디스크립터
- `buf`: 출력할 데이터가 저장된 버퍼
- `count`: 쓸 바이트 수

성공하면 실제로 쓴 바이트 수를 반환하고, 실패하면 `-1`을 반환한다. 한 번의 호출에서 요청한 크기보다 적게 기록될 수도 있으므로 반환값을 확인해야 한다.

```c
const char message[] = "hello\n";
ssize_t bytes_written = write(fd, message, sizeof(message) - 1);
```

### 파일 닫기: `close()`

```c
int close(int fd);
```

`close()`는 사용이 끝난 FD를 닫아 운영체제 자원을 반환한다. 성공하면 `0`, 실패하면 `-1`을 반환한다.

```c
if (close(fd) == -1) {
    // 파일 닫기 실패
}
```

닫힌 FD를 다시 사용하면 안 된다.

### 기본 파일 출력 예제

```c
#include <fcntl.h>
#include <unistd.h>

int main(void)
{
    const char message[] = "hello\n";
    int fd = open("a.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);

    if (fd == -1)
        return 1;

    if (write(fd, message, sizeof(message) - 1) == -1) {
        close(fd);
        return 1;
    }

    if (close(fd) == -1)
        return 1;

    return 0;
}
```

### 파일 외의 입출력 자원

Linux에서는 일반 파일뿐 아니라 다음 자원도 FD로 다룬다.

- 장치 드라이버
- 파이프
- 소켓
- 터미널

따라서 `read()`와 `write()`를 사용해 여러 종류의 입출력 자원을 비슷한 방식으로 처리할 수 있다.

### 핵심 정리

- Linux 저수준 파일 입출력은 FD를 기준으로 동작한다.
- `open()`은 파일을 열고 FD를 반환한다.
- `O_CREAT`로 파일을 생성할 때는 `mode`를 함께 지정한다.
- `read()`와 `write()`는 실제 처리한 바이트 수를 반환하므로 반드시 확인한다.
- `read()`가 `0`을 반환하면 파일 끝에 도달한 것이다.
- 사용이 끝난 FD는 `close()`로 닫아야 한다.

## 고수준 파일 입출력

### 개념

고수준 파일 입출력은 C 표준 라이브러리 `<stdio.h>`가 제공하는 **스트림(stream)** 기반 입출력이다. 정수형 FD 대신 `FILE *`를 사용하며, 내부 버퍼를 통해 입출력 횟수를 줄이고 문자열·행·블록 단위 함수를 제공한다.

```c
#include <stdio.h>
```

```text
fopen() → FILE * 획득 → fread() / fwrite() / fgets() / fputs() → fclose()
```

### 주요 함수

| 함수 | 역할 | 성공 시 반환값 | 실패 또는 종료 |
|---|---|---|---|
| `fopen()` | 파일 스트림 열기 | `FILE *` | `NULL` |
| `fclose()` | 스트림 닫기 | `0` | `EOF` |
| `fread()` | 블록 단위 읽기 | 읽은 요소 수 | 요청 수보다 작은 값 |
| `fwrite()` | 블록 단위 쓰기 | 쓴 요소 수 | 요청 수보다 작은 값 |
| `fgets()` | 문자열 한 줄 읽기 | 버퍼 주소 | `NULL` |
| `fputs()` | 문자열 쓰기 | 음수가 아닌 값 | `EOF` |

### 파일 열기: `fopen()`

```c
FILE *fopen(const char *pathname, const char *mode);
```

- `pathname`: 열 파일의 경로
- `mode`: 읽기·쓰기·추가 등의 열기 방식

| 모드 | 의미 | 파일이 없을 때 | 기존 내용 |
|---|---|---|---|
| `"r"` | 읽기 | 실패 | 유지 |
| `"w"` | 쓰기 | 생성 | 삭제 |
| `"a"` | 이어 쓰기 | 생성 | 유지하고 끝에 추가 |
| `"r+"` | 읽기·쓰기 | 실패 | 유지 |
| `"w+"` | 읽기·쓰기 | 생성 | 삭제 |
| `"a+"` | 읽기·이어 쓰기 | 생성 | 유지하고 끝에 추가 |

> `fopen()`의 문자열 `mode`는 파일을 여는 방식을 뜻한다. `open()`의 세 번째 인수인 권한 값 `mode_t`와는 다른 개념이다.

바이너리 파일은 `"rb"`, `"wb"`, `"ab"`처럼 `b`를 붙여 표현할 수 있다.

```c
FILE *fp = fopen("data.txt", "r");

if (fp == NULL) {
    // 파일 열기 실패
}
```

### 파일 닫기: `fclose()`

```c
int fclose(FILE *stream);
```

`fclose()`는 버퍼에 남은 출력 데이터를 파일에 반영하고 스트림을 닫는다. 성공하면 `0`, 실패하면 `EOF`를 반환한다.

```c
if (fclose(fp) == EOF) {
    // 파일 닫기 실패
}
```

### 블록 읽기: `fread()`

```c
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
```

- `ptr`: 읽은 데이터를 저장할 버퍼
- `size`: 요소 하나의 크기
- `nmemb`: 읽을 요소 개수
- `stream`: 입력 스트림

최대 `size × nmemb`바이트를 읽고, 실제로 읽은 **요소 개수**를 반환한다.

```c
char buffer[128];
size_t count = fread(buffer, 1, sizeof(buffer), fp);
```

반환값이 `nmemb`보다 작으면 파일 끝 또는 오류가 발생한 것이다. 두 경우는 `feof()`와 `ferror()`로 구분한다.

### 블록 쓰기: `fwrite()`

```c
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

최대 `size × nmemb`바이트를 쓰고, 실제로 쓴 **요소 개수**를 반환한다.

```c
const char data[] = "hello\n";
size_t count = fwrite(data, 1, sizeof(data) - 1, fp);
```

반환값이 `nmemb`보다 작으면 요청한 데이터를 모두 쓰지 못한 것이다.

### 한 줄 읽기: `fgets()`

```c
char *fgets(char *s, int size, FILE *stream);
```

최대 `size - 1`개의 문자를 읽고 마지막에 널 문자(`\0`)를 추가한다. 줄바꿈 문자를 만나면 그 문자까지 버퍼에 저장한다.

```c
char line[128];

while (fgets(line, sizeof(line), fp) != NULL) {
    // line 처리
}
```

성공하면 `s`를 반환하고, 파일 끝에 도달하거나 오류가 발생하면 `NULL`을 반환한다.

### 문자열 쓰기: `fputs()`

```c
int fputs(const char *s, FILE *stream);
```

문자열을 스트림에 출력한다. `fputs()`는 문자열 끝에 줄바꿈 문자를 자동으로 추가하지 않는다.

```c
fputs("hello\n", fp);
```

성공하면 음수가 아닌 값을, 실패하면 `EOF`를 반환한다.

### 기본 텍스트 파일 예제

```c
#include <stdio.h>

int main(void)
{
    FILE *fp = fopen("message.txt", "w");

    if (fp == NULL)
        return 1;

    if (fputs("hello\n", fp) == EOF) {
        fclose(fp);
        return 1;
    }

    if (fclose(fp) == EOF)
        return 1;

    return 0;
}
```

### 핵심 정리

- 고수준 파일 입출력은 `FILE *` 스트림을 사용한다.
- `fopen()`은 스트림을 열고 `fclose()`는 버퍼를 반영한 뒤 스트림을 닫는다.
- `fread()`와 `fwrite()`의 반환값은 바이트 수가 아니라 처리한 **요소 개수**이다.
- `fgets()`는 널 문자를 포함할 공간을 남기므로 최대 `size - 1`문자를 읽는다.
- `fputs()`는 줄바꿈 문자를 자동으로 추가하지 않는다.

## 저수준과 고수준 비교

| 구분 | 저수준 파일 입출력 | 고수준 파일 입출력 |
|---|---|---|
| 기준 | POSIX 시스템 호출 | C 표준 라이브러리 |
| 파일 표현 | 정수형 FD | `FILE *` 스트림 |
| 주요 함수 | `open`, `read`, `write`, `close` | `fopen`, `fread`, `fwrite`, `fgets`, `fputs`, `fclose` |
| 버퍼링 | stdio 버퍼 없음 | 라이브러리 내부 버퍼 사용 |
| 속도 특성 | 호출할 때마다 시스템 호출이 발생 | 데이터를 내부 버퍼에 모아 시스템 호출 횟수를 줄임 |
| 특징 | FD와 입출력을 직접 제어 | 문자열·행·블록 단위 처리가 편리 |

고수준 파일 입출력은 여러 번의 작은 입출력을 내부 버퍼에 모아 처리하므로 시스템 호출 횟수가 줄어들어 일반적으로 효율적이다. 저수준 파일 입출력도 큰 단위로 묶어 호출하면 성능 차이가 작아질 수 있지만, `read()`와 `write()`는 호출할 때마다 시스템 호출이 발생한다.
