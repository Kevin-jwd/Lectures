# WS2812B RGB LED Driver (TIM2 CH2 PWM 기반)

STM32 (시스템 클럭 96MHz) 환경에서 TIM2 CH2 PWM을 이용해 WS2812B(NeoPixel) LED를 제어하는 드라이버 코드(`rgb_led.c`) 정리 문서. 코드가 왜 이렇게 짜여있는지, 값들이 어떻게 도출됐는지 이해하기 위한 목적.

---

## 1. WS2812B 프로토콜 요약

|항목|값|
|---|---|
|1비트 전체 주기|1.25us|
|T0H (0 비트 High 구간)|0.4us|
|T1H (1 비트 High 구간)|0.85us|
|Reset(Latch) 신호|라인을 50us 이상 Low 유지|
|데이터 순서|GRB, 각 바이트 MSB first|
|LED 1개당 비트 수|24bit (G 8 + R 8 + B 8)|

한 비트는 항상 **HIGH가 먼저, LOW가 나중**인 구조. HIGH 구간의 길이만 0.4us(0) / 0.85us(1)로 달라짐.

---

## 2. 타이머 클럭 계산 (PSC / ARR)

- 목표 PWM 주파수: `1 / 1.25us = 800kHz`
- TIM2 카운터 클럭: 96MHz (시스템 클럭 그대로 공급된다고 가정)
- 관계식: `(PSC+1) × (ARR+1) = TIMCLK / f_PWM = 96,000,000 / 800,000 = 120`

**선택: PSC = 0, ARR = 119**

- PSC=0인 이유: **분해능(tick 간격) 최대화** 목적
    - PSC=0 → 1tick = 1/96MHz ≈ **10.42ns**
    - T0H/T1H 차이가 0.45us밖에 안 됨 → tick 간격이 커지면(PSC↑) 두 값을 구분하는 CCR 격자가 성겨져서 오차 확대됨
- ARR=119 → 카운터가 0~119, 총 120tick 카운트 후 오버플로우 → `120 × 10.417ns = 1250ns = 1.25us` (목표 주기와 일치)

### CCR 값 (Duty)

|비트|목표 High 시간|계산|CCR 값|실제 High 시간|오차|
|---|---|---|---|---|---|
|0|0.4us|0.4us × 96MHz = 38.4|**38**|395.8ns|-4.2ns|
|1|0.85us|0.85us × 96MHz = 81.6|**82**|854.2ns|+4.2ns|

오차 ±5ns 수준. WS2812B 허용오차(수백 ns대) 대비 여유 충분.

---

## 3. 레지스터 설정 상세

### 3.1 GPIO (PA1, TIM2_CH2)

```c
Macro_Write_Block(GPIOA->MODER, 0x3, 0x2, WS2812_GPIO_PIN * 2U);   // AF 모드
Macro_Write_Block(GPIOA->AFR[0], 0xf, 0x1, WS2812_GPIO_PIN * 4U);  // AF01 = TIM2
Macro_Clear_Bit(GPIOA->OTYPER, WS2812_GPIO_PIN);                   // Push-Pull
Macro_Write_Block(GPIOA->OSPEEDR, 0x3, 0x3, WS2812_GPIO_PIN * 2U); // Very High Speed
```

- **MODER = AF(0x2)**: AF여야 TIM2 CH2 신호가 물리 핀까지 연결됨 (`0x1`은 GPIO 순수 출력 → 타이머 신호 핀에 전혀 안 나감, 과거 겪은 버그)
- **OTYPER = Push-Pull**: WS2812B는 짧은 시간 안에 확실한 전압 필요 → Open-Drain(풀업 의존, 느림)은 부적합
- **OSPEEDR = Very High Speed**: 슬루레이트 최대화 → rise/fall time 최소화 목적. T0H/T1H 마진이 좁아 느린 slew는 그대로 타이밍 오차로 반영됨
- **PUPDR**: Push-Pull 상태에서는 영향 거의 없음, 생략 가능 (리셋 디폴트가 No pull)

> **설계 전략**: GPIO를 AF ↔ GPIO Output으로 전환하지 않음. Init에서 AF로 한 번만 설정 후 이후 절대 미변경. Idle/Reset 상태도 GPIO 레벨 제어가 아니라 `CCR2 = 0`(PWM 0% duty)으로 표현. 모드 전환 과정에서 생기는 글리치를 원천 차단하기 위한 결정.

### 3.2 TIM2 기본 설정

```c
TIM2->PSC = WS2812_TIM_PSC;   // 0
TIM2->ARR = WS2812_ARR;       // 119

Macro_Write_Block(TIM2->CCMR1, 0xff, 0x68, 8);   // CH2: PWM mode1(110) + Preload Enable(1)
TIM2->CCER = (0 << 5) | (1 << 4);                // CC2P=0(Active High), CC2E=1(출력 Enable)
TIM2->CCR2 = 0;                                  // Idle = 0% duty (Low)

Macro_Set_Bit(TIM2->EGR, 0);      // UG: Shadow reg → Active reg 강제 즉시 반영
Macro_Clear_Bit(TIM2->SR, 0);     // EGR로 인위 발생한 UIF 클리어

Macro_Set_Bit(TIM2->CR1, 0);      // Counter 상시 Run 시작 (Init에서 1회만, 이후 정지 없음)
```

**PWM mode1 동작 원리** (CCxP와 무관하게 먼저 결정):

```
CNT < CCR  →  OCxREF = 1 (Active)
CNT ≥ CCR  →  OCxREF = 0 (Inactive)
```

**CC2P(극성) 적용**:

|CC2P|의미|결과 파형|
|---|---|---|
|0|Active High|OCxREF 그대로 → **HIGH 먼저, LOW 나중** (WS2812B에 부합)|
|1|Active Low|OCxREF 반전 → LOW 먼저, HIGH 나중 (부적합)|

> ⚠️ "Active Low"라는 이름 때문에 "Low로 시작"으로 오해하기 쉬움. 실제로는 "Active 구간(CNT<CCR, 앞부분)을 Low로 표현"한다는 뜻이라 오히려 반대 파형이 나옴. **반드시 CC2P=0(Active High) 사용.**

**CCR2=0으로 카운터 시작 시**: 첫 주기는 `CNT<0`이 성립 안 해 통째로 Low로 지나감. 이 상태 자체가 Idle(Low)과 동일해 무해하나, 유효한 첫 비트값 반영을 위해 `EGR.UG`로 Shadow→Active 레지스터 강제 갱신 필요.

**타이머 상시 Run 유지 이유**: CR1.CEN을 매번 켜고 끄지 않고 Init에서 한 번만 켜서 계속 구동. Idle 상태에서도 CCR2=0이라 결과 파형은 그냥 Low로 보임. GPIO 모드/카운터 상태를 자주 전환하지 않아 안정성 확보.

---

## 4. 함수별 설명

### `RGB_LED_Init(void)`

GPIO(AF, Push-Pull, Very High Speed) + TIM2(PSC/ARR/CCMR/CCER) 설정 1회 수행, 카운터를 상시 Run 상태로 시작. 이후 GPIO MODER나 CR1.CEN 미변경.

### `RGB_LED_Enable(void)`

새 프레임 전송 전 상태 초기화. `CCR2=0` 세팅 후 `EGR.UG`로 즉시 반영해, 카운터 위상과 무관하게 다음 주기부터 0(Low)에서 깨끗하게 시작하도록 함.

### `WS2812_Send_Bit(int bit)`

```c
TIM2->CCR2 = bit ? WS2812_CODE_1 : WS2812_CODE_0;   // 82 또는 38
while (Macro_Check_Bit_Clear(TIM2->SR, 0));          // UIF(주기 완료) 대기 — Polling(Blocking)
Macro_Clear_Bit(TIM2->SR, 0);
```

CCR2에 값을 쓴 뒤, 해당 주기가 끝날 때까지(UIF=1) **CPU가 그 자리에서 대기**. 현재는 폴링 방식이라 함수 리턴까지 다른 작업 불가. (→ 인터럽트 기반 개선 방향은 8장 참고)

### `WS2812_Send_Byte(unsigned char data)`

1바이트를 MSB부터 LSB까지 `WS2812_Send_Bit()`로 8회 순차 전송.

### `RGB_LED_Send(r, g, b)`

GRB 순서로 3바이트(24bit) 전송, LED 1개 색상 세팅.

### `RGB_LED_Reset(void)`

`CCR2=0` 상태를 `WS2812_RESET_CNT`회의 주기(각 1.25us)만큼 반복 대기. `WS2812_RESET_CNT × 1.25us ≥ 50us` 성립하도록 값 설정 필요 (예: 40 이상, 여유 있게 45~50 권장).

### `RGB_LED_Disable(void)`

`CCR2=0`으로 복귀, Idle(Low) 상태 유지. GPIO 모드나 CR1은 미변경 (타이머는 계속 Run 상태 유지).

### `RGB_LED_Send_All(r, g, b)` / `RGB_LED_Send_One(index, r, g, b)`

`Enable → (Send × N) → Reset → Disable` 순서로 한 프레임 전송을 자동 처리하는 래퍼 함수. `Send_One`은 지정 index만 색을 넣고 나머지는 Off(0,0,0) 전송해 4개 LED 전체를 매 프레임 갱신 (WS2812B는 항상 체인 전체를 다시 보내야 함 → 한 개만 바꾸고 나머지 생략 불가).

---

## 5. 시간적 동작 (Send_All 기준)

```
Idle(Low, CCR2=0)
   │
RGB_LED_Send_All() 호출
   │
   ├─ Enable()                     : CCR2=0, EGR UG로 위상 정렬
   ├─ Send × 4                     : 4 × 24bit × 1.25us ≈ 120us (Blocking)
   ├─ Reset()                      : 50us+ Low 유지 (Blocking)
   └─ Disable()                    : CCR2=0 유지
   │
함수 리턴 (총 소요 시간 약 170~200us)
   │
Idle(Low, CCR2=0) — 다음 호출 전까지 계속
```

전체 파형은 `Send_All()`/`Send_One()` 호출 시점에 짧게(약 200us) 몰려서 출력됨. 스코프 관찰 시 **Rising Edge 트리거 + Single 모드** 필요 (Idle이 Low이므로 Falling Edge 트리거는 부적합).

---

## 6. 그동안 겪었던 주요 버그 (회고)

|증상|원인|해결|
|---|---|---|
|파형이 계속 Low, 트리거 안 잡힘|`GPIOA->MODER`가 AF(0x2)가 아닌 GPIO Output(0x1)로 설정|AF(0x2)로 수정|
|ISR 무한 재진입 / 멈춤|`TIM2->SR` UIF 클리어에 `Set_Bit`(1 write) 사용 — UIF는 rc_w0라 0을 써야 클리어됨|`Clear_Bit` 사용|
|LOW→HIGH로 파형이 뒤집혀 보임|CCR2=0 상태로 카운터 시작 → 첫 주기가 통째로 Low로 지나가는 걸 극성 문제로 오인|CC2P=0 유지, EGR UG로 첫 프레임 CCR2 값 강제 반영|
|Enable↔Send 사이 스퓨리어스 펄스|Enable()이 비동기로 리턴, 타이머 상태와 CPU 진행 사이 레이스 발생|Blocking(UIF polling) 구조로 전환해 CPU-타이머 동기화|
|GPIO 전환 이후 파형이 아예 안 나옴|`Disable()`에 남아있던 `MODER`를 GPIO Output으로 되돌리는 코드가 AF 복귀 없이 핀 고착|AF 고정 전략으로 전환, Enable/Disable에서 MODER 미조작|

---

## 7. 확인 필요한 파라미터 (매크로)

코드 내 아래 매크로들의 실제 값이 위 계산과 일치하는지 확인 필요:

```c
#define WS2812_TIM_PSC     0      // PSC
#define WS2812_ARR         119    // ARR (120tick = 1.25us)
#define WS2812_CODE_0       38    // T0H ≈ 0.4us
#define WS2812_CODE_1       82    // T1H ≈ 0.85us
#define WS2812_RESET_CNT   ≥40    // 50us / 1.25us, 여유 있게 45~50 권장
#define WS2812_NUM_LEDS      4
#define WS2812_BITS_PER_BYTE  8
#define WS2812_GPIO_PIN       1   // PA1
```

---

## 8. 다음 단계 (미완료): 인터럽트 기반 비동기 전송

현재 구조는 `WS2812_Send_Bit()`가 `while(UIF)` 폴링으로 CPU 점유 중. CPU를 자유롭게 하려면 아래 구조로 전환 필요:

1. `RGB_LED_Init()`에 `TIM2->DIER |= UIE`, `NVIC_EnableIRQ(TIM2_IRQn)` 추가
2. 전송할 24bit×N 데이터를 버퍼에 미리 채워둠
3. `RGB_LED_Send_All()`은 버퍼 세팅 + 상태 초기화 후 **즉시 리턴** (Blocking 없음)
4. `TIM2_IRQHandler()`가 매 주기마다:
    - DATA 상태: 버퍼에서 다음 비트를 꺼내 `CCR2`에 갱신
    - 데이터 소진 시: RESET 상태로 전환, `CCR2=0` 유지하며 카운트
    - Reset 카운트 충족 시: DONE 상태, 완료 플래그 세팅
5. 메인 루프는 완료 플래그를 필요할 때만 확인 (또는 무시 가능)

> 이 버전은 아직 코드 미반영. 필요 시 별도 구현 예정.

---

_본 문서는 `rgb_led.c` 구현 과정에서의 설계 결정과 근거를 정리한 초안. 매크로 실제값, 보드 배선(PA1 지정 여부) 등은 실제 프로젝트 설정에 맞춰 재확인 필요._