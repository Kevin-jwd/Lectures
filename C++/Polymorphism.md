___
#### 멤버 함수 overloading과 polymorphism
<br>

- <b>Polymorphism</b>: 동질 다형성, 같은 interface로 다른 동작 구현
<br>

#### Overloading/Overriding
<br>

- <b>Function Overloading</b>: parameter 개수 or 타입 불일치
	- 호출 시 전달되는 인자에 의하여 호출할 함수 결정
- <b>Function Overriding</b>: parameter 개수 or 타입 완전 일치
	- 멤버 함수 재정의 시 child 함수가 우선
<br>

#### 추상 클래스
<br>

- <b>pure virtual function</b>: 내용을 구현하지 않은 함수
- <b>Abstract Class</b>: pure virtual function을 하나라도 포함한 클래스
	- 인스턴스 생성 불가능 (포인터 지정은 가능)
	- child class에서 꼭 정의해야 할 함수 지정 
	   <b>-> <u>자식 클래스에서 작성할 함수 가이드라인을 제공하는 의미</u></b>
<br>

```C+++
class A {
public:
	virtual void func() = 0;   // =0을 붙이지 않으면 외부에서 정의가 되는 것으로 착각
};

class B : public A {
public:
	virtual void func() override { cout << "hello" << endl; }
};
```