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
- <b>기본 형태</b>: `class 자식_클래스_이름 : 접근_지정자 부모_클래스_이름`
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

#### Child Class의 상속 대상 접근성 지정
<br>

- 부모 클래스의 상속 대상의 속성을 변경하여 상속받음
- <b>더 좁은 범위</b>의 접근 지정자를 따름
<br>

#### Friend
<br>

- 특정 함수, 클래스 내 private, protected 멤버 접근 권한 부여 (<u>단방향</u>)
```C++
friend class B;        // class B 에게 private, protected 접근 권한 부여
```
```C++
friend void func();    // 함수 func 에게 private, protected 접근 권한 부여
```
<br>

#### 클래스 생성자, 소멸자
<br>

- <b>생성</b>: 부모 -> 자식
- <b>소멸</b>: 자식 -> 부모
<br>

- 부모 클래스의 생성자가 default가 아닐 경우
  자식 클래스에서 반드시 부모 클래스의 생성자 호출 필요
```C++
class A {
public: 
		A(int n) {}
}
class B : public A {
public:
	B(int x) : A(x + 1) {}
};
```
<br>

#### Upcasting / Downcasting
<br>

- <b>Upcasting</b>: 부모 클래스 포인터는 모든 자식 포인터 타입을 대입받을 수 있음
- <b>Downcasting</b>: 자식 포인터 멤버를 접근하고자 하면 부모 클래스 포인터를 자식 클래스 포인터 타입으로 변환
<br>

#### 다중 상속
<br>

- 상속받은 순서대로 생성/소멸
```C++
class A {};
class B {};
class C : public A, public B {};   // 상속 순서 : A->B

// 생성 : A -> B -> C
// 소멸 : ~C -> ~B -> ~A
```
```C++
class A {};
class B {};
class C : public B, public A {};   // 상속 순서 : B->A

// 생성 : B -> A -> C
// 소멸 : ~C -> ~A -> ~B
```
<br>

#### 가상 상속 (Virtual Inheritance)
<br>

- A를 상속받는 B1, B2
- B1, B2를 상속받는 C
- 이때 생성자 순서는 A->B1->A->B2->C
- 불필요한 생성자 2번 호출을 해결하기 위해 B1, B2에서 A를 가상 상속받도록 구현
- 바뀐 생성자 순서는 A->B1->B2->C
- 따라서 A의 멤버들의 중복이 제거됨
