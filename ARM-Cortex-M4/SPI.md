___
- Serial Peripheral Interface 버스
<br>

- <img src="../assets/SPI%20pin%20map.png"/>

<br>

- 근거리의 주변장치를 4개의 신호로 고속 접속이 가능하여 널리 사용됨
- `CLK`은 mater에서 제공하고 master/slave 간 데이터가 동시에 상대방으로 전달 (Full Duplex, 전이중)
<br><br>
- <b>4 신호</b>
	- `CLK`: 데이터에 동기되어 전달되는 클럭 신호
	- `MOSI(Master Out Slave In)`: CLK에 따라 Master가 전달하는 데이터
	- `MISO(Master In Slave Out)`: CLK에 따라 Slave가 전달하는 데이터
	- `CS(Chip Select)`: multi master로 운영될 경우 다른 SPI가 master로 사용됨

<br>

___
### 프로토콜
<br>

<img src="../assets/SPI%20protocol.png"/>

<br>

- CPOL (Clock Polarity), CPHA (Clock Phase)의 선택에 의해 4가지 인터페이스가 가능
