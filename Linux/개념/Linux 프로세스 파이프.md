# Linux 프로세스 파이프

관련 개념: [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]], [[Linux/개념/Linux 파일 입출력|Linux 파일 입출력]]

## 개념

`popen()`은 자식 프로세스에서 셸 명령을 실행하고, 부모 프로세스와 자식 명령 사이에 **단방향 파이프**를 만든다. 파이프는 `FILE *` 스트림으로 반환되므로 [[Linux/개념/Linux 파일 입출력|고수준 파일 입출력]] 함수를 사용할 수 있다.

```c
#include <stdio.h>

FILE *popen(const char *command, const char *type);
int pclose(FILE *stream);
```

### 파이프 방향

| `type` | 호출한 프로세스의 동작 | 연결 대상 |
|---|---|---|
| `"w"` | 스트림에 데이터 쓰기 | 실행 명령의 표준 입력(stdin) |
| `"r"` | 스트림에서 데이터 읽기 | 실행 명령의 표준 출력(stdout) |

```text
popen(command, "w")
부모 fwrite() ──▶ 파이프 ──▶ 자식 명령의 stdin

popen(command, "r")
부모 fread() ◀── 파이프 ◀── 자식 명령의 stdout
```

`popen()`은 내부적으로 셸을 통해 명령을 실행하므로 파이프나 리다이렉션 같은 셸 문법을 명령 문자열에 사용할 수 있다.

### `pclose()`

`pclose()`는 파이프 스트림을 닫고 실행한 명령이 종료될 때까지 기다린다. 반환값은 자식 명령의 종료 상태가 인코딩된 값이므로 `WIFEXITED()`와 `WEXITSTATUS()` 등으로 해석한다.

> `popen()`으로 연 스트림은 `fclose()`가 아니라 `pclose()`로 닫아야 한다.

### 쓰기 방향: `tr` 명령에 문자열 전달

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

pid_t pid;

int main(int argc, char **argv)
{
    int ret, len;
    FILE *fp_w;

    if (argc != 2) {
        printf("usage: %s {string}\n", argv[0]);
        return EXIT_FAILURE;
    }
    printf("[%d] running %s %s\n", pid = getpid(), argv[0], argv[1]);

    fp_w = popen("tr a-z A-Z", "w");
    if (fp_w == NULL) {
        printf("[%d] error: %s (%d)\n", pid, strerror(errno), __LINE__);
        return EXIT_FAILURE;
    }

    len = strlen(argv[1]) + 1;
    fwrite(argv[1], 1, len, fp_w);

    ret = pclose(fp_w);
    printf("\n[%d] ret = 0x%x\n", pid, ret);

    printf("[%d] terminted\n", pid);

    return EXIT_SUCCESS;
}
```

#### 동작

```text
argv[1]
   ↓ fwrite()
부모의 FILE * 스트림
   ↓ 파이프
tr 명령의 표준 입력
   ↓ 소문자를 대문자로 변환
tr 명령의 표준 출력
   ↓
터미널 화면
```

1. `popen("tr a-z A-Z", "w")`가 `tr` 명령을 실행하고 쓰기용 파이프 스트림을 반환한다.
2. `fwrite()`가 `argv[1]`의 문자열을 파이프로 전달한다.
3. `tr`은 소문자 `a-z`를 대문자 `A-Z`로 변환한다.
4. `"w"` 모드는 `tr`의 표준 입력만 파이프에 연결하므로 변환 결과는 기존 표준 출력인 화면에 표시된다.
5. `pclose()`가 쓰기 파이프를 닫으면 `tr`은 EOF를 받고 종료하며, 부모는 종료 상태를 회수한다.

`strlen(argv[1]) + 1`은 문자열 끝의 널 문자까지 파이프로 전달한다. 텍스트 내용만 전달하려면 일반적으로 `strlen(argv[1])`바이트만 쓰면 된다.

### 읽기 방향: `ls` 명령의 결과 가져오기

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

pid_t pid;

#define BUF_SIZE 32

int main(int argc, char **argv)
{
    int ret;
    size_t ulen;
    FILE *fp_r;
    char buf[BUF_SIZE];
    char cmd[BUF_SIZE];

    if (argc != 2) {
        printf("usage: %s {directory}\n", argv[0]);
        return EXIT_FAILURE;
    }
    printf("[%d] running %s %s\n", pid = getpid(), argv[0], argv[1]);

    sprintf(cmd, "ls %s", argv[1]);
    fp_r = popen(cmd, "r");
    if (fp_r == NULL) {
        printf("[%d] error: %s (%d)\n", pid, strerror(errno), __LINE__);
        return EXIT_FAILURE;
    }

    for (;;) {
        ulen = fread(buf, 1, BUF_SIZE - 1, fp_r);
        if (ulen == 0)
            break;

        buf[ulen] = '\0';
        printf("%s", buf);
    }

    ret = pclose(fp_r);
    printf("[%d] ret = 0x%x\n", pid, ret);

    printf("[%d] terminted\n", pid);

    return EXIT_SUCCESS;
}
```

#### 동작

```text
argv[1]
   ↓
"ls 디렉터리" 명령 문자열 생성
   ↓ popen(command, "r")
자식 ls 명령 실행
   ↓ 표준 출력
파이프
   ↓ fread()
부모의 buf
   ↓ printf()
터미널 화면
```

1. `sprintf()`가 사용자 인수를 이용해 `ls 디렉터리` 명령 문자열을 만든다.
2. `popen(cmd, "r")`이 명령을 실행하고 명령의 표준 출력을 읽는 스트림을 반환한다.
3. `fread()`가 한 번에 최대 `BUF_SIZE - 1`바이트를 읽는다.
4. 읽은 데이터 뒤에 널 문자(`\0`)를 추가해 문자열로 만든 뒤 `printf()`로 출력한다.
5. `fread()`가 `0`을 반환하면 읽기를 끝내고 `pclose()`로 자식 명령의 종료 상태를 회수한다.

`cmd`의 크기가 32바이트이므로 긴 디렉터리 이름을 `sprintf()`로 결합하면 버퍼를 넘을 수 있다. 또한 사용자 입력이 셸 명령에 그대로 포함되므로 실제 프로그램에서는 길이를 제한하고 셸 특수 문자를 주의해야 한다.

## 핵심 정리

- `popen()`은 셸 명령을 실행하면서 부모와 자식 명령 사이에 **단방향 파이프**를 만든다.
- `"r"`은 명령의 출력을 `fread()`로 읽고, `"w"`는 명령의 입력으로 `fwrite()`한다.
- 반환값이 `FILE *`이므로 고수준 파일 입출력 함수를 사용한다.
- `popen()`으로 연 스트림은 `pclose()`로 닫는다.

