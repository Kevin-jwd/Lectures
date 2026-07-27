___
## Interrupt

- HW 제어
- 이벤트가 발생하면 HW가 신호를 줌
- 인터럽트는 Exception(고장 등의 예외 상황) 의 한 종류임
- **IRQ (Interrupt Request)**
- **ISR (Interrupt Service Routine)**
- 우선순위가 같은 IRQ가 동시에 일어나면 <u>낮은 position부터 먼저 처리</u>
<br>

- ***crt0.s*** : boot code
	- 지정된 ISR 함수 이름들이 있으며 ISR은 같은 이름으로 함수만 생성


## Interrupt Controller

- 대부분의 프로세서는 1개의 IRQ핀을 가짐
- 하지만 주변장치들의 인터럽트 소스는 여러 개
- 많은 인터럽트가 하나의 인터럽트를 공유하도록 하는 장치가 Interrupt Controller
<br>

- **인터럽트 발생 허용/금지(Interrupt enable/masking) 여부 결정**
- 요청이 발생(Interrupt Pending)한 **인터럽트 소스를 확인**
- 동시에 들어온 인터럽트에 대해 Arbiter logic에 의해 **우선순위 확인**
<br>

- 인터럽트에 연결된 주변장치(Subsource) 내부에서도 여러 인터럽트가 존재
<br>

## 외부 인터럽트의 개념

- GPIO 핀이 많으므로 개별로 IRQ 번호를 배정하지 않고, **Grouping**
- 
