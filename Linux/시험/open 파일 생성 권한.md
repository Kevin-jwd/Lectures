# open 파일 생성 권한

관련 개념: [[Linux/개념/Linux 파일 입출력|Linux 파일 입출력]]

## 시험 핵심

```c
dst_fd = open(dst_name,
              O_WRONLY | O_EXCL | O_CREAT,
              S_IRUSR | S_IWUSR);
```

| 항목 | 의미 |
|---|---|
| `O_WRONLY` | 쓰기 전용으로 열기 |
| `O_CREAT` | 파일이 없으면 생성 |
| `O_EXCL` | `O_CREAT`와 함께 사용하며 파일이 이미 있으면 실패 |
| `S_IRUSR` | 소유자 읽기 권한 `0400` |
| `S_IWUSR` | 소유자 쓰기 권한 `0200` |

```text
S_IRUSR | S_IWUSR = 0600 = rw-------
```

- 세 번째 인수 `mode`는 새 파일에 부여할 기본 권한이다.
- 이미 존재하는 파일의 권한은 `mode`로 변경되지 않는다.
- 최종 생성 권한 계산은 [[Linux/시험/umask 권한 계산|umask 권한 계산]]에서 확인한다.

