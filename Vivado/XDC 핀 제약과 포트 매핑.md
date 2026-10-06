# XDC 핀 제약과 포트 매핑

출처: PDF 22–24, 37–41쪽 · [[Vivado/00 Vivado 디지털 회로 설계 길잡이|길잡이]]

## 핀 제약의 역할

**핀 제약(Pin Constraints)은 HDL의 입출력 포트가 FPGA의 어느 물리 핀을 사용할지 지정하는 조건**이다. XDC는 이러한 설계 제약을 저장하는 파일이다.

```text
보드 스위치 → FPGA 물리 핀 → HDL 입력 a, b
HDL 출력 ld0~ld5 → FPGA 물리 핀 → 보드 LED
```

[[Vivado/Verilog 모듈과 게이트 회로|Verilog]]는 출력의 논리를 정하고, XDC는 그 신호가 보드에서 어디로 연결되는지 정한다.

## 파일 추가와 수정

1. Basys-3-Master.xdc 원본을 별도로 보관한다.
2. `Add Sources` → `Add or Create Constraints`에서 파일을 추가한다.
3. 사용할 스위치·LED 항목의 주석을 해제한다.
4. `get_ports` 뒤의 이름을 **최상위 모듈의 실제 포트 이름**으로 바꾼다.
5. 사용하지 않는 핀 항목은 주석 상태로 둔다.

Board 파일과 XDC의 차이는 [[Vivado/Vivado 환경 준비와 프로젝트 생성]]에서 확인한다.

## Gates 실습의 핀 연결

PDF 화면의 매핑을 정리하면 다음과 같다.

| HDL 포트 | FPGA 핀 | 보드 장치 |
|---|---|---|
| a | V17 | 스위치 0 |
| b | V16 | 스위치 1 |
| ld0 | U16 | LED 0 |
| ld1 | E19 | LED 1 |
| ld2 | U19 | LED 2 |
| ld3 | V19 | LED 3 |
| ld4 | W18 | LED 4 |
| ld5 | U15 | LED 5 |

```tcl
set_property -dict { PACKAGE_PIN V17 IOSTANDARD LVCMOS33 } [get_ports {a}]
set_property -dict { PACKAGE_PIN V16 IOSTANDARD LVCMOS33 } [get_ports {b}]

set_property -dict { PACKAGE_PIN U16 IOSTANDARD LVCMOS33 } [get_ports {ld0}]
set_property -dict { PACKAGE_PIN E19 IOSTANDARD LVCMOS33 } [get_ports {ld1}]
set_property -dict { PACKAGE_PIN U19 IOSTANDARD LVCMOS33 } [get_ports {ld2}]
set_property -dict { PACKAGE_PIN V19 IOSTANDARD LVCMOS33 } [get_ports {ld3}]
set_property -dict { PACKAGE_PIN W18 IOSTANDARD LVCMOS33 } [get_ports {ld4}]
set_property -dict { PACKAGE_PIN U15 IOSTANDARD LVCMOS33 } [get_ports {ld5}]
```

- `PACKAGE_PIN`: FPGA 패키지의 핀 위치.
- `IOSTANDARD`: 입출력 전기적 규격. 실습에서는 LVCMOS33을 사용한다.
- `get_ports`: 설정을 적용할 최상위 HDL 포트 선택.

**테스트벤치의 내부 신호 이름이 아니라 실제 설계의 최상위 포트 이름을 적는다.** 지정된 연결은 [[Vivado/합성과 구현]]을 거쳐 [[Vivado/비트스트림과 보드 프로그래밍|보드 구성]]에 반영된다.
