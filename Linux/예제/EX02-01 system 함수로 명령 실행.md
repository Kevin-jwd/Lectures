# EX02-01 system 함수로 명령 실행

관련 개념:
- [[Linux/개념/Linux 프로세스|Linux 프로세스]]
- [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]]

관련 시험: [[Linux/시험/system 함수로 셸 명령 실행|system 함수로 셸 명령 실행]]

## 문제

`system()`으로 Linux 명령을 실행하고 반환 상태를 확인한다. 파이프와 리다이렉션이 포함된 명령도 문자열로 전달하여 실행한다.

## 코드

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>

int main(int argc, char **argv)
{
    int ret;

    if (argc != 1) {
        printf("usage: %s\n", argv[0]);
        return EXIT_FAILURE;
    }
    printf("running %s\n", argv[0]);

    printf("command = ls -l\n");
    ret = system("ls -l");
    printf("-> ret = %#x (%#x)\n", ret, WEXITSTATUS(ret));

    printf("command = wrong command\n");
    ret = system("wrong command");
    printf("-> ret = %#x (%#x)\n", ret, WEXITSTATUS(ret));

    printf("command = sleep 3\n");
    ret = system("sleep 3");
    printf("-> ret = %#x (%#x)\n", ret, WEXITSTATUS(ret));

    printf("command = ls /etc | grep init > init.txt\n");
    ret = system("ls /etc | grep init > init.txt");
    printf("-> ret = %#x (%#x)\n", ret, WEXITSTATUS(ret));

    return EXIT_SUCCESS;
}
```

## 결과 / 원인

| 명령 | 동작 |
|---|---|
| `ls -l` | 현재 디렉터리의 상세 목록 출력 |
| `wrong command` | 존재하지 않는 명령이므로 셸에서 오류 발생 |
| `sleep 3` | 약 3초 동안 기다린 뒤 종료 |
| `ls /etc \| grep init > init.txt` | `/etc` 목록에서 `init`이 포함된 항목만 `init.txt`에 저장 |

`system()`은 명령을 셸에 전달하고 명령이 끝날 때까지 기다린다. 반환값 `ret`에는 자식 프로세스의 종료 상태가 인코딩되어 있으며, `WEXITSTATUS(ret)`로 실제 종료 코드를 확인한다.

> `WEXITSTATUS(ret)`는 프로세스가 정상적으로 종료되었을 때 사용한다. 정확히 확인하려면 먼저 `ret != -1`과 `WIFEXITED(ret)`를 검사한다.

## 핵심 포인트

- `system()`은 파이프와 리다이렉션이 포함된 셸 명령을 실행할 수 있다.
- 종료 코드 `0`은 일반적으로 명령이 성공했음을 의미한다.
- `system()`의 반환값과 명령의 실제 종료 코드는 구분해야 한다.
- `>`는 기존 파일 내용을 덮어쓴다.
