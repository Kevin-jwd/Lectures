___

#### Open Drain 출력 용도


___

- 하나의 프로세서에 여러 IC를 <b>단 2선</b>만으로 연결 -> <u><b>매우 간단하고 저렴</b></u>
- 일반적으로 1:N 통신이 목적이지만 <u><b>multi master 방식도 지원</b></u>
- <b>`SCL`</b>: Clock (master가 제공, 동기 통신)
- <b>`SDA</b>: 양방향 데이터 (Half duplex, 동시에 주고받는 건 불가능)
<br>
___
### 16bit 통신 방식

- 확장 주소 방식
- 보편적으로 많이 사용됨
<br>

##### Write
- Start ->device 주소 (7bit + 1bit(R/W)) -> _ACK_ -> register 주소 (8bit) -> _ACK_ -> ... 
<br>
##### Read
- Start ->device 주소 (7bit + 1bit(R/W)) -> _ACK_ -> register 주소 (8bit) -> _ACK_ -> Repeat Start -> device 주소 -> register 주소 -> _DATA_ -> ACK
<br>

