___
#### Procedural or Structured Program
<br>

- 무분별한 분기를 지양하기 위한 방법론
- 반복적으로 수행되는 동작을 단위 동작으로 묶어서 <b>Procedure</b>라고 함

<br>

#### OOP: Object Oriented Programming
<br>

- 공통 Procedure를 호출하는 대신 <u>해당 동작을 하는 Object를 각자 소유</u>
- <b>Object</b>: 객체
- <b>Class</b>: 생성할 기능 모듈을 요약(abstraction, 추상화)
<br>
- <b>Pros</b>
	- 각 instance는 <b>독립</b>적으로 동작 가능
	- 각 instance는 <b>변형</b>하여 사용 가능
	- Class로 child class 생성 가능 (<b>확장성</b>)
	- 만든 class로 추가적인 instance 생성 가능 (instantiation)
- <b>Cons</b>
	- 메모리 낭비
	- instance 호출 시간이 더 길어짐
<br>

#### OOP 4대 특성
<br>

<b>1. 추상화 (Abstraction)</b>
<b>2. 상속성 (Inheritance)</b>
<b>3. 다형성 (Polymorphism)</b>
<b>4. 캡슐화 (Encapsulation)</b>: 정보 은닉 (ex. private, public, ...)

<br>

#### 구조체를 이용한 Object 생성
<br>

```C++
struct _st
{
private:
    double irate = 0.02;           // 멤버 초기화 가능

    void disp_total(void)          // 멤버 함수 가능
    {
        cout << total << endl;
    }
};                                 // 반드시 ; 사용
```
<br>

- <b>접근 지정자</b>
	- `private`: 해당 클래스에서만 접근 가능
	- `protected`: 상속 관계에서 접근 가능
	- `public`: 어디서든 접근 가능 (default)

<br>

#### 기본 Class, Object(Instance) 생성 및 사용
<br>

- 구조체와 다르게 default가 `private`
- 클래스의 함수는 메서드 (method)라고 함
```C++
class mart_calc
{
public:
    int total = 0;          // 멤버 변수

    void buy(int price)     // 멤버 함수 (메서드)
    {
        total += price;
    }
};
```

<br>

- 메서드는 내부에서 선언, 외부에 정의 가능
```C++
class mart_calc
{
public:
	void disp_total(void);              // 내부에서 선언
private:
	void buy(int price);
}

void mart_calc::disp_total(void) { }     // 외부에서 정의
void mart_calc::buy(int price) { }
```

<br>

#### this 포인터
<br>

- this 포인터를 이용하여 scope를 정확히 지정 가능
- this는 메서드 호출시 자동으로 전달되는 포인터 (타입: `class명 * const this`)
```C++
void mart_calc::change_tax(double tax)
{
	this->tax = tax;
	// mart_calc::tax = tax;
}
```
- <b>생성</b>
	- `void change_tax(mart_calc* const this, double tax)`
- <b>호출</b>
	- `calc1.change_tax(&calc1, 0.02)`
<br>

#### 생성자 (Constructor)
<br>

- 생성자에서 초기값을 삽입할 수 있음
<br>

- class와 같은 이름
- parameter 가능
- class 외부 정의 불가
- return 없음
- default parameter 가능
- function oveloading 가능
<br>

- <b>Default Constructor</b>
	- 변수처럼 생성 (ex. `buf x;`)
	- parameter가 없는 형식이며 반드시 `( )`없이 이름만으로 생성
	- 직접 만든 생성자도 parameter가 없다면 `( )`없이 이름만으로 생성
<br>

#### 소멸자 (Destructor)
<br>

- <b>기본 형식</b>: `~클래스 이름`
- parameter, return 없음
- Object를 전역으로 생성했다면, 프로그램이 종료될때까지 소멸되지 않음
- Object가 유효범위를 벗어나면 소멸자가 자동으로 호출됨
- 직접 호출해도 소멸과는 무관 (단순 함수 호출)
<br>

#### Static Member 변수 (Class 변수)
<br>

- 인스턴스가 아니어도 접근 가능한 변수 -> <b>scope</b> 연산자 사용
- <u>모든 인스턴스에서 공통으로 사용하는 변수</u>로 사용
<br>

- `static` 선언 시 초기화는 금지
- `static`은 반드시 외부에서 정의
<br>

#### Static Member 함수 (Class 함수)
<br>

- instance와 무관한 함수
- class함수에서는 class 변수(static), global 변수, 함수만 접근 가능
<br>

#### 초기화 List
<br>

- <b>기본 형식</b>: `생성자:변수(값) {};`
<br>

```C++
A(int a, int b)
{
	int x = a;
	int y = b;
}
```
```C++
A(int a, int b): x(a), y(b) { };
```
- `const`는 대입이 안되고 초기화밖에 안되므로 이런 방식을 주로 사용
<br>

#### 상수 멤버 함수 (Const Member Function)
<br>


- member data를 <i>read-only</i>로만 접근할 수 있는 함수
- <b>Const instance</b>
	- 멤버 데이터 수정 금지
	- 멤버 함수 호출 금지

<br>

- <b>Const Member Function</b>은 절대 멤버 데이터를 변경하지 않는다고 확신하기 때문에
  <b>Const Instance</b>에서 호출해도 확신 가능
<br>

#### Generic Class
<br>

- <b>Generic Function</b>: 입력되는 인자의 type에 따라 동작하는 함수
- <b>Generic Class</b>: Generic Function과 유사하게 생성된 클래스
<br>

- instance 생성 시 반드시 T 타입을 전달해야 한다 (ex. `<int>`)
- instance 생성 시 정의된 parameter는 변하지 않음