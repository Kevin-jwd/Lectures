# Linux 파이프와 grep

관련 문제: [[Linux/예제/EX01-03 파일 및 텍스트 검색|EX01-03 파일 및 텍스트 검색]]

## 표준 입출력

Linux 명령어는 일반적으로 다음과 같은 입출력 통로를 사용한다.

| 구분 | 번호 | 기본 대상 |
|---|---:|---|
| 표준 입력(stdin) | 0 | 키보드 |
| 표준 출력(stdout) | 1 | 화면 |
| 표준 오류(stderr) | 2 | 화면 |

## 파이프: `|`

파이프는 앞 명령어의 표준 출력을 뒤 명령어의 표준 입력으로 전달한다.

```bash
명령어1 | 명령어2
```

```bash
ls -l /etc | grep conf
```

1. `ls -l /etc`가 `/etc`의 목록을 출력한다.
2. 출력 결과가 `grep conf`의 입력으로 전달된다.
3. `conf`가 포함된 행만 화면에 출력된다.

파이프는 여러 개 연결할 수 있다.

```bash
ls -l /etc | grep conf | less
```

`conf`가 포함된 결과를 `less`로 페이지 단위로 확인한다.

## 문자열 검색: `grep`

`grep`은 입력에서 지정한 문자열이나 패턴이 포함된 행을 찾는다.

```bash
grep 패턴 파일명
```

```bash
cat /etc/passwd | grep sys
```

`/etc/passwd`에서 `sys`가 포함된 행을 출력한다.

파일을 직접 지정하면 불필요한 `cat`을 생략할 수 있다.

```bash
grep sys /etc/passwd
```

## 특정 문자열 제외: `grep -v`

`-v`는 지정한 패턴이 포함되지 않은 행을 출력한다.

```bash
cat /etc/passwd | grep sys | grep -v run
```

- `sys`가 포함된 행 선택
- 선택한 결과 중 `run`이 포함된 행 제외

## 여러 조건 연결

파이프로 `grep`을 연속해서 사용하면 모든 조건을 만족하는 행을 찾을 수 있다.

```bash
grep tcp /etc/services | grep daemon | grep -v service
```

- `tcp` 포함
- `daemon` 포함
- `service` 미포함

## 핵심 정리

- `|`: 앞 명령어의 출력을 뒤 명령어의 입력으로 전달
- `grep 패턴`: 패턴이 포함된 행 출력
- `grep -v 패턴`: 패턴이 포함되지 않은 행 출력
- 여러 파이프를 연결해 검색 조건을 단계적으로 적용할 수 있다.
