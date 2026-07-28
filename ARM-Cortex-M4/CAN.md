___
- 고전적인 방식: 주 제어기에 모든 제어 요소를 <u>전선</u>으로 연결
<br>

#### CAN
<br>

- Controller Area Network
- Main Controller가 집중 제어 -> 통신에 연결된 각 노드들로 분산제어
	- 각 노드가 MCU를 보유한 ECU(Electronics Control Unit)
- SW 변경 없이도 새로운 기능의 ECU 추가/삭제가 자유롭게
- 동작 중 특정 ECU가 고장나면 네트워크에서 배제시키고 동작 가능
<br>

___
#### CAN 특징 요약
<br>

- CSMA-CD/CR, Multi-Master, 메시지 기반 Filtering
- 최대 1Mbps, 장거리(40kbps로 최대 1km), 비동기 직렬 통신, 차동 신호 방식
