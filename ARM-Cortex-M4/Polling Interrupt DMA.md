___

## Polling

- SW 제어
- 주기적으로 확인
- CPU 자원낭비가 심함
- **ex)**
-  ```c
	void Main(void)
	{
	    Sys_Init(115200);
	    printf("1sec LED toggle Test!!\n");
	    TIM4_Repeat(500);
	
	    for(;;)
	    {
	        if(TIM4_Check_Timeout)
	        {
	            Macro_Invert_Bit(GPIOA->ODR, 5);
	        }
	    }
	}
	```


## DMA ☆

- **Direct Memory Access**
- 인터럽트의 실행 횟수를 최소화
- 기억장치 접근을 위해 CPU의 시스템 버스 사용권을 뺏는 <u><b>사이클 스틸링 (Cycle Stealing)</b></u>
- 