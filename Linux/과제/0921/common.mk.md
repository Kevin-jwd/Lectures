___
# `common.mk` 전체 분석
  
## 용도

여러 예제 디렉터리에서 공통으로 사용하는 C 프로그램 빌드 규칙이다. 각 디렉터리의 `Makefile`이 `TARGET1`~`TARGET4`를 지정한 뒤 `include ../common.mk`로 불러온다.

관련 개념: [[Linux/개념/GNU 툴체인과 빌드 과정|GNU 툴체인과 빌드 과정]]

관련 과제: [[Linux/과제/0921/EX01-20|EX01-20]]

## 전체 코드

```makefile
CC = aarch64-linux-gnu-gcc
CFLAGS = -c -Wall

TARGETS = $(TARGET1) $(TARGET2) $(TARGET3) $(TARGET4)
all: $(TARGETS)

ifdef TARGET1
TARGET1_OBJS = $(TARGET1).o
$(TARGET1): $(TARGET1_OBJS)
	$(CC) -o $@ $(TARGET1_OBJS) $(LFLAGS)
endif

ifdef TARGET2
TARGET2_OBJS = $(TARGET2).o
$(TARGET2): $(TARGET2_OBJS)
	$(CC) -o $@ $(TARGET2_OBJS) $(LFLAGS)
endif

ifdef TARGET3
TARGET3_OBJS = $(TARGET3).o
$(TARGET3): $(TARGET3_OBJS)
	$(CC) -o $@ $(TARGET3_OBJS) $(LFLAGS)
endif

ifdef TARGET4
TARGET4_OBJS = $(TARGET4).o
$(TARGET4): $(TARGET4_OBJS)
	$(CC) -o $@ $(TARGET4_OBJS) $(LFLAGS)
endif

%.o: %.c
	$(CC) $(CFLAGS) $<

clean:
	rm -f $(TARGETS) *.o
```


```make

TARGET1 = 실행파일명

include ../common.mk

```  

## 주요 설정

| 항목 | 의미 |
|---|---|
| `CC` | ARM64용 크로스 컴파일러 `aarch64-linux-gnu-gcc` |
| `CFLAGS` | 컴파일만 수행(`-c`)하고 일반 경고를 표시(`-Wall`) |
| `TARGETS` | `TARGET1`~`TARGET4`를 하나의 빌드 목록으로 구성 |
| `LFLAGS` | 링크 단계에 추가할 옵션 또는 라이브러리. 개별 `Makefile`에서 지정 가능 |

## 빌드 흐름

1. `make`를 실행하면 기본 타깃 `all`이 선택된다.
2. 정의된 `TARGET1`~`TARGET4`만 조건부로 빌드된다.
3. 패턴 규칙 `%.o: %.c`가 각 C 소스를 오브젝트 파일로 컴파일한다.
4. 해당 오브젝트 파일을 링크해 같은 이름의 실행 파일을 만든다.

자동 변수는 `$<`가 첫 번째 의존 파일(소스), `$@`가 현재 생성할 타깃을 뜻한다.  

## 코드별 분석

### 컴파일러와 옵션

```makefile
CC = aarch64-linux-gnu-gcc
CFLAGS = -c -Wall
```

- `CC`: AArch64 Linux용 크로스 컴파일러
- `-c`: 링크하지 않고 목적 파일 `.o`까지만 생성
- `-Wall`: 주요 컴파일 경고 활성화

### 전체 타깃 구성

```makefile
TARGETS = $(TARGET1) $(TARGET2) $(TARGET3) $(TARGET4)
all: $(TARGETS)
```

`TARGET1`~`TARGET4` 중 개별 `Makefile`에서 정의된 이름이 `all`의 의존 타깃이 된다. `make`에서 타깃을 생략하면 첫 번째 규칙인 `all`이 실행된다.

### 조건부 타깃 규칙

```makefile
ifdef TARGET1
TARGET1_OBJS = $(TARGET1).o
$(TARGET1): $(TARGET1_OBJS)
	$(CC) -o $@ $(TARGET1_OBJS) $(LFLAGS)
endif
```

- `TARGET1`이 정의된 경우에만 해당 규칙을 사용한다.
- 실행 파일은 같은 이름의 목적 파일 하나에 의존한다.
- `$@`는 현재 타깃인 실행 파일명을 의미한다.
- `$(LFLAGS)`는 개별 Makefile에서 추가할 링크 옵션이나 라이브러리이다.
- `TARGET2`~`TARGET4`도 같은 방식으로 동작한다.

### 패턴 규칙

```makefile
%.o: %.c
	$(CC) $(CFLAGS) $<
```

모든 `.c` 파일을 같은 이름의 `.o` 파일로 컴파일하는 공통 규칙이다.

- `$<`: 첫 번째 의존 파일인 `.c` 파일
- 생성할 `.o` 파일명은 패턴 `%.o`로 결정되므로 명령에 `-o`가 없어도 GCC가 같은 기본 이름을 사용한다.

### 정리 규칙

```makefile
clean:
	rm -f $(TARGETS) *.o
```

정의된 실행 파일과 현재 디렉터리의 모든 목적 파일을 삭제한다. `-f` 때문에 파일이 없어도 오류를 출력하지 않는다.

## `EX01-20` 빌드 예시

개별 Makefile이 다음과 같다고 가정한다.

```makefile
TARGET1 = EX01-20
include ../common.mk
```

변수와 규칙은 다음처럼 확장된다.

```text
TARGETS      = EX01-20
TARGET1_OBJS = EX01-20.o

EX01-20: EX01-20.o
```

처음 빌드할 때 실행되는 명령은 다음과 같다.

```bash
aarch64-linux-gnu-gcc -c -Wall EX01-20.c
aarch64-linux-gnu-gcc -o EX01-20 EX01-20.o
```

`EX01-20.c`가 변경되면 목적 파일을 다시 만들고 실행 파일을 다시 링크한다. 모든 결과물이 최신이면 명령을 실행하지 않는다.

## 기타

- `make clean`은 생성된 실행 파일과 현재 디렉터리의 모든 `.o` 파일을 삭제한다.
- 하나의 실행 파일이 같은 이름의 C 파일 하나로 구성된다는 전제의 규칙이다.
- `all`, `clean`에 `.PHONY` 선언이 없어 같은 이름의 파일이 있으면 명령이 실행되지 않을 수 있다.
- 일반적으로 링크 옵션 변수는 `LDFLAGS`/`LDLIBS`를 사용하지만, 이 파일은 `LFLAGS`라는 자체 이름을 사용한다.
- `ifdef` 블록이 반복되므로 지원 가능한 실행 파일은 최대 네 개이다.
- 여러 소스 파일로 하나의 실행 파일을 만들려면 `TARGETn_OBJS`에 목적 파일을 추가하도록 규칙을 확장해야 한다.
