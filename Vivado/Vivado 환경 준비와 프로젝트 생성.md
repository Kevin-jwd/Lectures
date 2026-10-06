# Vivado 환경 준비와 프로젝트 생성

출처: PDF 20–32쪽 · [[Vivado/00 Vivado 디지털 회로 설계 길잡이|길잡이]]

## 준비 파일의 차이

자료에서는 Vitis 설치에 Vivado가 포함되는 구성을 안내한다. 다운로드·설치 중 PC가 절전 상태로 들어가지 않게 설정한다.

| 준비 항목 | 역할 |
|---|---|
| Digilent Board 파일 | 프로젝트 생성 시 Basys3 보드를 선택하도록 보드 정보를 제공 |
| Basys-3-Master.xdc | 스위치·LED·클록 등의 핀 정보를 제공하는 제약 파일 |

**Board 파일을 설치하는 것과 XDC에서 회로 포트를 매핑하는 것은 별개**이다.

자료는 XilinxBoardStore의 Digilent Board 파일을 내려받아 설치 폴더에 복사하도록 안내한다. 2022.2 기준 경로 예시는 다음과 같다.

```text
C:\Xilinx\Vivado\2022.2\data\boards\board_files\Digilent
```

설치 위치가 다르면 해당 설치 경로를 사용한다. XDC는 별도 작업 폴더에 보관하고 프로젝트에 추가한다. 상세 매핑은 [[Vivado/XDC 핀 제약과 포트 매핑]]에서 확인한다.

## 프로젝트 생성

1. `Create Project`에서 프로젝트 이름과 저장 위치를 지정한다.
2. `RTL Project`를 선택한다.
3. 자료의 진행 순서에서는 소스·제약을 나중에 추가한다.
4. 보드 선택 화면에서 **Basys3**를 선택한다.
5. 프로젝트를 생성하고 `Add Sources`로 회로 소스를 추가한다.
6. `Add or Create Design Sources` → `Create File` → Verilog 파일을 생성한다.
7. 모듈 이름과 포트를 정의하고 회로 코드를 작성한다.

Basys3가 검색되지 않으면 Board 파일 설치 상태와 목록 새로 고침을 확인한다.

## 프로젝트에서 파일을 구분하는 기준

| 소스 영역 | 들어갈 파일 |
|---|---|
| Design Sources | 실제 회로를 기술한 Verilog 모듈 |
| Constraints | 핀·타이밍 등의 조건을 지정한 XDC |
| Simulation Sources | 검증용 입력을 만드는 테스트벤치 |

회로 작성은 [[Vivado/Verilog 모듈과 게이트 회로]], 검증 소스 작성은 [[Vivado/테스트벤치와 시뮬레이션]]으로 이어진다. 편집·화면 전환은 [[Vivado/Vivado 2022.2 유용한 단축키]]를 함께 참고한다.
