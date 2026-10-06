# Vivado 2022.2 유용한 단축키

Vivado 2022.2 IDE에서 수업 중 반복해서 쓰기 좋은 키만 추렸다. **현재 포커스가 있는 창에 따라 동작이 달라질 수 있다.**

## 먼저 익힐 단축키

| 단축키 | 동작 | 쓸 때 |
|---|---|---|
| `Ctrl+F` | 텍스트 찾기 | HDL·제약 파일에서 신호나 문자열 찾기 |
| `Ctrl+R` | 텍스트 바꾸기 | 파일의 이름·문자열 수정 |
| `Ctrl+Space` | 코드 완성 제안 표시 | Vivado 내장 텍스트 편집기에서 신호·식별자 입력 |
| `Ctrl+Tab` | 다음 작업 탭으로 이동 | 여러 소스·보고서 탭 전환 |
| `Ctrl+Shift+Tab` | 이전 작업 탭으로 이동 | 방금 지나친 탭으로 돌아가기 |

## 화면 정리

| 단축키 | 동작 | 쓸 때 |
|---|---|---|
| `Alt+-` | 현재 창 최대화/원래 크기로 복원 | 파형이나 타이밍 보고서를 크게 보기. 창 탭 더블클릭도 같은 용도 |
| `Ctrl+Q` | Flow Navigator 표시/숨기기 | 편집·파형 화면을 넓게 쓰기 |
| `F5` | 현재 레이아웃의 창 배치 초기화 | 패널을 옮기다가 화면 구성이 복잡해졌을 때 |

## 시뮬레이션·타이밍 화면에서

**파형은 시뮬레이션 화면에서, 타이밍 경로·보고서는 Timing Analysis 레이아웃에서 확인한다.** 레이아웃은 분석에 사용할 창을 보기 좋게 배치해 주며, 실제 시뮬레이션이나 분석 결과를 자동으로 생성하는 기능은 아니다.

| 작업 | 빠른 방법 |
|---|---|
| 시뮬레이션 파형 열기 | Flow Navigator의 `Run Simulation > Run Behavioral Simulation` |
| 파형의 시간축 확대·축소 | 파형 영역을 클릭한 뒤 `Ctrl+마우스 휠` |
| 파형·타이밍 보고서 크게 보기 | 해당 창을 선택한 뒤 `Alt+-` |
| 정적 타이밍 분석용 화면 배치 | 합성 또는 구현 설계를 연 뒤 상단 Layout Selector 또는 `Layout` 메뉴에서 `Timing Analysis` 선택 |
| 파형·보고서를 다른 모니터에 표시 | 창 탭 우클릭 → `Float` → 분리된 창을 다른 모니터로 이동 |

화면 전환 메뉴와 키보드 단축키를 구분해 기억한다. 자주 쓰는 메뉴 명령에 키를 지정하려면 `Tools > Settings > Shortcuts`에서 해당 명령의 현재 키를 확인하고 사용자 스키마에서 설정한다.

## Tcl Console에서

| 키 | 동작 |
|---|---|
| `↑` / `↓` + `Enter` | 자동 완성 후보를 선택 |
| `Tab` | 후보가 하나로 좁혀졌을 때 명령어·인자 완성 |

## 내 설정에서 확인하기

`Tools > Settings > Shortcuts`에서 현재 지정된 단축키를 확인할 수 있다. 기본 단축키를 바꾸고 싶다면 **Vivado Default 스키마를 복사해 새 단축키 스키마**를 만든 뒤 수정한다. 사용자 지정 단축키를 사용하는 환경이라면 위 표와 실제 키가 다를 수 있다.

## 출처

수업 내용은 [[Vivado/00 Vivado 디지털 회로 설계 길잡이]]에서 시작한다. 파형을 읽는 실습은 [[Vivado/테스트벤치와 시뮬레이션]], 타이밍 분석의 설계 단계는 [[Vivado/합성과 구현]]과 연결된다.

- [AMD UG893, Vivado IDE Tips (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Vivado-IDE-Tips): 찾기·바꾸기, 탭 전환, Flow Navigator, 배치 초기화
- [AMD UG893, Using the Text Editor (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Using-the-Text-Editor?contentId=j5GYcmeSEXqFBjIXPbK7qA): `Ctrl+Space` 코드 완성
- [AMD UG893, Using Auto-Complete (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Using-Auto-Complete?contentId=vnP3MiMbm52o8skDoY8ufw): Tcl Console 자동 완성
- [AMD UG893, Configuring Shortcut Keys (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Configuring-Shortcut-Keys?contentId=0Yy2nxY6VSeQFg20cFG_wQ): 단축키 조회·설정
- [AMD UG893, Layout Selector (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Layout-Selector): Timing Analysis 화면 배치
- [AMD UG893, Floating Windows (2022.2)](https://docs.amd.com/r/2022.2-English/ug893-vivado-ide/Floating-Windows): 창 분리와 다른 모니터로 이동
- [AMD UG900, Zooming with the Mouse Wheel (2022.2)](https://docs.amd.com/r/2022.2-English/ug900-vivado-logic-simulation/Zooming-with-the-Mouse-Wheel): 파형 확대·축소
- [AMD Vitis Tutorial, Use the Exported IP in RTL Design with Vivado Flow (2022.2)](https://docs.amd.com/r/2022.2-English/Vitis-Tutorials-Getting-Started/Use-the-Exported-IP-in-RTL-Design-with-Vivado-Flow): Run Behavioral Simulation과 파형 화면
