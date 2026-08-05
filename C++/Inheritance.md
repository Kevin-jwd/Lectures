___
#### Class로부터 새로운 Class의 생성
<br>

- 기존 클래스의 성질을 상속받는 새로운 클래스의 생성
- <b>기존 클래스</b>
	- base class or parent class
- <b>새로운 클래스</b>
	- derived class or child class
	- base 자산을 그대로 사용, 변형, 추가하여 사용 가능
<br>

- <b>Pros</b>
	- <b>Reusability</b>: 기존 클래스를 상속받아 유사한 속성의 클래스를 생성 가능
	- <b>Polymorphism</b>: 동일한 인터페이스로 instance에 따른 다른 멤버 함수 호출 가능
	- <b>Maintenance ability</b>: 상위 클래스 변경 시 하위 클래스도 모두 변경된 내용 적용 가능
	- <b>Operator overloading</b>: 새로 생성한 클래스에 대한 기존 연산자 동작 재정의 가능
<br>

#### 상속에서의 Access Modifier
<br>

| 접근 지정자 | class 내부 | child class | class 외부 | 
| --- | --- | --- | --- | 
| `public` | O | O | O | 
| `protected` | O | O | X | 
| `private` | O | X | X | 
<br>

