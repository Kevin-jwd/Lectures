# Verilog 모듈과 게이트 회로

출처: PDF 12–13, 33–36쪽 · [[Vivado/00 Vivado 디지털 회로 설계 길잡이|길잡이]]

## 모듈과 포트

**모듈은 회로를 구성하는 기본 단위**이며, 내부 동작과 외부 연결 포트를 정의한다. 여러 모듈을 연결하여 큰 회로를 만들 수 있다.

| 문법 | 역할 |
|---|---|
| `module` / `endmodule` | 모듈 정의 시작·종료 |
| `input` | 입력 포트 |
| `output` | 출력 포트 |
| `inout` | 양방향 포트 |
| `assign` | 연속 할당으로 논리 연결을 기술 |
| `always` | 이벤트에 따라 반복 실행되는 동작 기술 |
| `initial` | 시뮬레이션 시작 시 한 번 실행되는 동작 기술 |
| `begin` / `end` | 여러 문장을 하나의 블록으로 묶음 |

키워드는 소문자로 작성한다. 버스 포트는 여러 비트를 묶은 포트이다.

```verilog
output [3:0] o_s;  // 4비트 출력 포트
```

## 게이트 실습 코드

PDF의 Gates 모듈은 두 입력과 여섯 출력을 갖는 [[디지털 논리 회로/조합논리와 순차논리|조합논리 회로]]이다.

```verilog
`timescale 1ns / 1ps

module Gates(
    input a,
    input b,
    output ld0,
    output ld1,
    output ld2,
    output ld3,
    output ld4,
    output ld5
);
    assign ld0 = a & b;     // AND
    assign ld1 = a | b;     // OR
    assign ld2 = ~(a & b);  // NAND
    assign ld3 = ~(a | b);  // NOR
    assign ld4 = a ^ b;     // XOR
    assign ld5 = ~a;        // NOT
endmodule
```

**`assign`은 프로그램의 한 줄을 한 번 수행하는 대입이 아니라, 입력에 따라 출력이 계속 결정되는 회로 연결**로 이해한다. 각 줄은 실행 순서를 정하는 것이 아니라 서로 다른 출력의 논리를 표현한다.

- AND·OR·NAND·NOR: [[디지털 논리 회로/AND OR NAND NOR]]
- XOR: [[디지털 논리 회로/XOR과 XNOR]]
- NOT: [[디지털 논리 회로/NOT과 버퍼]]

## 블로킹과 논블로킹 할당

절차 블록에서 쓰는 할당 연산자는 구분해서 읽는다. 위 코드의 `assign`과는 맥락이 다르다.

| 연산자 | 동작을 읽는 기준 |
|---|---|
| `=` 블로킹 | 문장 순서대로 할당되어 앞의 결과가 뒤 문장에 영향을 줌 |
| `<=` 논블로킹 | 해당 이벤트에서 오른쪽 값을 평가하고 왼쪽 갱신을 예약 |

자료의 클록 기반 예시는 다음과 같다.

```verilog
always @(posedge reset or posedge clock) begin
    if (reset) begin
        flop1 <= 0;
        flop2 <= 1;
    end else begin
        flop1 <= flop2;
        flop2 <= flop1;
    end
end
```

리셋 후 `(flop1, flop2) = (0, 1)`이라면, 다음 클록에서 **이전 값을 기준으로 서로 교환**하여 `(1, 0)`이 된다. 핵심은 레지스터를 갱신할 때 다른 레지스터의 이전 값을 참조한다는 점이다.

테스트벤치의 순차적 입력 설정에서는 블로킹 할당을 사용한다. 관련 내용은 [[Vivado/테스트벤치와 시뮬레이션]]에서 확인한다.

## 회로 구조 확인과 재사용

`Open Elaborated Design`의 Schematic에서 코드가 어떤 게이트와 연결 구조로 해석됐는지 확인한다. 이는 [[Vivado/합성과 구현|최종 배치·배선 결과]]와는 다르다.

기존 모듈을 상위 모듈 안에 인스턴스화하면 하위 회로로 재사용할 수 있다. Gates를 DUT로 연결하는 named instantiation 예시는 [[Vivado/테스트벤치와 시뮬레이션]]에 있다. 보드 연결은 [[Vivado/XDC 핀 제약과 포트 매핑]]으로 지정한다.
