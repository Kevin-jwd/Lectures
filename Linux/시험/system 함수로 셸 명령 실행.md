# system 함수로 셸 명령 실행

관련 개념:
- [[Linux/개념/Linux 프로세스|Linux 프로세스]]
- [[Linux/개념/Linux 파이프와 grep|Linux 파이프와 grep]]

관련 예제: [[Linux/예제/EX02-01 system 함수로 명령 실행|EX02-01 system 함수로 명령 실행]]

## 시험 핵심

```c
int ret;

ret = system("ls /etc | grep init > init.txt");
```

`system()`은 전달받은 문자열을 셸 명령으로 실행하고, 명령이 종료될 때까지 기다린다.

셸 내부의 처리 순서는 다음과 같다.

```text
ls /etc
   ↓ 표준 출력
grep init
   ↓ init이 포함된 결과
init.txt에 저장
```

| 구성 | 의미 |
|---|---|
| `ls /etc` | `/etc`의 목록 출력 |
| `\| grep init` | 앞 명령의 출력에서 `init`이 포함된 행 선택 |
| `> init.txt` | 결과를 `init.txt`에 저장하며 기존 내용은 덮어씀 |

`system()`의 반환값에는 명령의 종료 상태가 인코딩되어 있다.

```c
if (ret == -1) {
    // system() 실행 실패
} else if (WIFEXITED(ret)) {
    int exit_code = WEXITSTATUS(ret);
} else if (WIFSIGNALED(ret)) {
    int signal_number = WTERMSIG(ret);
}
```

- `WIFEXITED(ret)`: 명령이 정상적인 방식으로 종료되었는지 확인
- `WEXITSTATUS(ret)`: 정상 종료한 명령의 종료 코드 추출
- `WIFSIGNALED(ret)`: 명령이 시그널에 의해 종료되었는지 확인
- `WTERMSIG(ret)`: 종료 원인이 된 시그널 번호 추출
- 종료 코드 `0`: 일반적으로 명령 실행 성공

```text
정상 종료   → exit code 확인
시그널 종료 → signal number 확인
```

**exit code와 시그널 번호 중 하나만 유효하다.**

| 종료 원인 | 유효한 값 |
|---|---|
| 프로세스가 스스로 정상 종료 | `WEXITSTATUS(ret)`로 얻은 exit code |
| 프로세스가 시그널로 강제 종료 | `WTERMSIG(ret)`로 얻은 시그널 번호 |

반드시 `WIFEXITED(ret)` 또는 `WIFSIGNALED(ret)`로 종료 원인을 먼저 확인한다.

종료 코드와 시그널 번호는 `ret`의 서로 다른 비트 영역에 인코딩되므로 원시 반환값을 그대로 종료 코드로 사용하면 안 된다.
