___
# makefile 동작 분석

```shell
all: hello

    @echo "build finished!"

  

hello: main.o func.o

    @ls

    aarch64-linux-gnu-gcc -o hello main.o func.o

  

main.o: main.c

    @ls

    aarch64-linux-gnu-gcc -c main.c

  

func.o: func.c

    @ls

    aarch64-linux-gnu-gcc -c func.c

  

clean:

    rm -f hello main.o func.o

    @echo "clean finished!"
```

- `make hello`를 실행했을 때 명령의 순서를 분석

![](../../../assets/Pasted%20image%2020260918143250.png)
```shell
aarch64-linux-gnu-gcc -c main.c

aarch64-linux-gnu-gcc -c func.c

aarch64-linux-gnu-gcc -o hello main.o func.o
```

- `hello: main.o func.o`
	- `main.o: main.c`
		- main.o가 존재하지 않으므로 `aarch64-linux-gnu-gcc -c main.c` 실행
	- `func.o: func.c`
		- func.o가 존재하지 않으므로 `aarch64-linux-gnu-gcc -c func.c` 실행
- `hello: main.o func.o`
	- hello가 존재하지 않으므로 `aarch64-linux-gnu-gcc -o hello main.o func.o` 실행

## 관련 노트

- [[Linux/과제/0918/ls 명령어의 출력|ls 명령어의 출력]]
- [[Linux/개념/GNU 툴체인과 빌드 과정|GNU 툴체인과 빌드 과정]]


