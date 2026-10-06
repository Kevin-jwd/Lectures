# Vivado 디지털 회로 설계 길잡이

출처: [01.vivado디지털회로설계_v1.1.pdf](<D:/강의/강의자료/FPGA/01.vivado디지털회로설계_v1.1.pdf>) · PDF 3–61쪽

**Vivado 2022.2 수업 기준**으로 복습할 핵심 내용을 정리한다. PDF 화면에는 다른 버전도 포함되어 있으므로 설치 경로는 2022.2 기준으로 표기했다. 쪽수는 PDF 페이지 기준이다.

## 주제별 목차

| 주제 | 확인할 내용 |
|---|---|
| [[Vivado/FPGA와 RTL 설계]] | FPGA, HDL, 레지스터 간 데이터 흐름 |
| [[Vivado/Basys3 보드와 FPGA 구성]] | 보드 자원, 100 MHz 클록, 구성 저장 방식 |
| [[Vivado/Vivado 환경 준비와 프로젝트 생성]] | Board 파일, RTL Project, 소스 종류 |
| [[Vivado/Verilog 모듈과 게이트 회로]] | 포트, assign, 모듈 계층, 블로킹·논블로킹 |
| [[Vivado/XDC 핀 제약과 포트 매핑]] | HDL 포트와 스위치·LED 핀 연결 |
| [[Vivado/합성과 구현]] | Netlist, Synthesis, Place·Route |
| [[Vivado/비트스트림과 보드 프로그래밍]] | .bit 생성, JTAG 다운로드, 구성 유지 |
| [[Vivado/테스트벤치와 시뮬레이션]] | DUT, initial, timescale, 입력 조합과 파형 |

## 실습 흐름

```text
RTL 프로젝트 생성 → Verilog 회로 작성 → Elaborated Design 확인
                         ├→ 테스트벤치 → Behavioral Simulation → 파형 검증
                         └→ XDC 핀 제약 → 합성 → 구현 → .bit 생성 → 보드 검증
```

시뮬레이션은 입력에 대한 논리 동작을 확인하고, 보드 실습은 실제 핀에 연결된 스위치·LED로 동작을 확인한다.

## 함께 볼 노트

- 이론 출발점: [[디지털 논리 회로/00 디지털 논리 회로 길잡이]]
- 회로 구분: [[디지털 논리 회로/조합논리와 순차논리]]
- 화면 전환·파형 조작: [[Vivado/Vivado 2022.2 유용한 단축키]]
