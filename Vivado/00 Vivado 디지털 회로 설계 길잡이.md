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
| [[Vivado/테스트벤치 작성 지침]] | 작성 순서와 실행 전 체크리스트 |

## 실습 흐름

### 빠르게 따라 하는 5단계

| 순서 | Vivado에서 할 일 | 목적·확인할 것 |
|---|---|---|
| 1. Source 추가 | `Add Sources` → `Add or Create Design Sources` → `Add Files` 또는 `Create File` → `Finish` | Verilog 회로 파일을 추가하거나 생성한다. 최상위 모듈이 맞는지 확인한다. |
| 2. Constraint 추가 | `Add Sources` → `Add or Create Constraints` → XDC 파일 추가 → `Finish` | 사용할 핀의 주석을 해제하고 `get_ports` 이름을 회로 포트와 맞춘다. 핀 위치·입출력 규격 등을 지정한다. |
| 3. Schematic 확인 | `RTL Analysis` → `Open Elaborated Design` → `Schematic` | 코드가 어떤 회로와 연결로 해석됐는지 확인한다. **아직 합성·배치·배선 결과는 아니다.** |
| 4. Synthesis 실행 | `Run Synthesis` → 완료 후 `Open Synthesized Design` | 논리를 최적화하고 FPGA의 LUT·플립플롭 등의 자원으로 변환한다. 오류와 자원 사용량을 확인한다. |
| 5. Implementation 실행 | `Run Implementation` → 완료 후 `Open Implemented Design` | 실제 FPGA 안에서 자원을 **배치(Place)하고 배선(Route)**한다. 타이밍 조건 충족 여부를 확인한다. |

**Source는 회로 내용, Constraint는 구현 조건, Schematic은 회로를 보는 화면, Synthesis는 자원 변환, Implementation은 배치·배선**으로 기억한다.

상세 내용: [[Vivado/Vivado 환경 준비와 프로젝트 생성|소스 추가]], [[Vivado/XDC 핀 제약과 포트 매핑|제약 설정]], [[Vivado/합성과 구현|합성·구현]]. 이후 보드에 넣으려면 [[Vivado/비트스트림과 보드 프로그래밍|Generate Bitstream]]으로 이어간다.

### 전체 흐름

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
