# Linux 파일 관리 명령어

## 파일 및 디렉터리 삭제: `rm`

`rm`은 파일이나 디렉터리를 삭제하는 명령어이다.

```bash
rm MyDir_2/hello_2.c   # 파일 삭제
rm -r MyDir_2          # 디렉터리와 내부 파일을 재귀적으로 삭제
rm -f xxx              # 확인 없이 강제 삭제
rm -rf xxx             # 디렉터리를 강제로 재귀 삭제
```

- `-r`: 디렉터리 내부까지 재귀적으로 삭제
- `-f`: 대상이 없어도 오류 메시지를 출력하지 않고 강제로 처리
- `rm -rf`는 복구가 어려우므로 실행 전에 경로를 반드시 확인한다.

## 심볼릭 링크 생성: `ln`

`ln -s`는 원본 파일이나 디렉터리를 가리키는 [[Linux/심볼릭 링크]]를 생성한다.

```bash
ln -s /etc/issue myissue
```

위 명령은 `/etc/issue`를 가리키는 `myissue`라는 심볼릭 링크를 생성한다.

```bash
ls -l myissue
# myissue -> /etc/issue

cat myissue
# 링크를 통해 /etc/issue의 내용 출력
```

디렉터리를 가리키는 심볼릭 링크도 생성할 수 있다.

```bash
ln -s /etc myetc
```

> 심볼릭 링크를 삭제해도 원본 파일이나 디렉터리는 삭제되지 않는다.
