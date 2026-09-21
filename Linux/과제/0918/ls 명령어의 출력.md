___
# ls 명령어의 출력


```shell
DESCRIPTION
       List  information  about the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

       Mandatory arguments to long options are mandatory for short options too.
```

```shell
Exit status:
       0      if OK,

       1      if minor problems (e.g., cannot access subdirectory),

       2      if serious trouble (e.g., cannot access command-line argument).
```

- 현재 경로의 모든 파일의 정보를 알파벳 순으로 정렬
- |종료 상태|의미|
|---|---|
|`0`|정상적으로 실행됨|
|`1`|사소한 문제 발생. 예: 일부 하위 디렉터리에 접근하지 못함|
|`2`|심각한 문제 발생. 예: 명령행에서 지정한 파일이나 디렉터리에 접근하지 못함|

## 관련 노트

- [[Linux/과제/0918/명령 치환|명령 치환]]
- [[Linux/과제/0918/makefile 동작 분석|makefile 동작 분석]]
- [[Linux/개념/Linux 파일 관리 명령어|Linux 파일 관리 명령어]]
