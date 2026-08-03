___
### 표준 입출력
<br>

- cout 과 stream insertion 연산자 `<<` 사용

```C++
std::cout << "i * i" << " = " << i * i << "\n";
std::cout << "i * i" << " = " << i * i << std::endl;
```

	- `::`: namespace를 지정하는 scope resolution 연산자 (C++ 표준은 모두 std 영역)
	- `<<`: stream insertion 연산자 -> 문자열로 모아줌 (pack)
	- `std:endl`: `\n`를 대신하는 기능 (속도가 더 느림)

<br>

#### namespace
<br>

- 중복된 이름(identifier)의 해석(resolution)을 위해 범위를 지정

```C++
int x, y;

  

namespace my

{

    int x, y;

  

    int func(int a, int b)

    {

        return a + b;

    }

}
```

- 사용: `my::x`, `my::y`, `my::func(x, y)`
<br>

#### using
<br>

- `::` 연산자를 사용하지 않고 적용될 namespace 지정
```C++
using namespace my;

x = 10, y = 20;
```
```C++
using my::x;
using my::y;

x = 10, y = 20;
```

- `my::x` 대신 `x`를 사용해도 global보다 my namespace가 우선시
- `using`으로 지정한 identifier가 먼저 사용됨
<br>

#### std 생략을 위한 using
<br>

- `std`도 namespace이기 때문에 using을 활용하여 생략할 수 있다
```C++
std::cout << "Hello" << std::endl;
```
```C++
using namespace std;

cout << "Hello" << endl;
```
<br>

#### 표준 입력 (cin)
<br>

```C++
cout << "입력: " ;
cin >> x;
```
	- cin과 stream extraction 연산자 `>>` 사용
<br>

- 여러 개 정수 입력
```C++
cin >> x >> y >> z;         // 입력 값은 순서대로 배정
```
<br>

#### 문자열 입력
<br>

```C++
cout << "실수 값 입력 ";

    cin >> d;

    cout << d << endl;

    cin.ignore();

  

    cout << "두 단어 입력(Hello World)" << endl;

    cin.getline(s, 20);

    cout << s << endl;
```
<br>

- <b>`cin`</b>: 공백을 구분하여 입력받음
<br>

- <b>`cin.getline(s, len)`</b>
	- 최대 `len`의 문자열을 받아 `s`에 저장
	- Enter 입력까지 한 문장을 받음
	- 이전 Enter로 인해 입력을 못 받는 경우가 생김

<br>

- <b>`cin.ignore()`</b>
	- 입력 버퍼에 남은 글자들을 <b>flush</b>시킴

#### 상수 표현, 변수 초기화
<br>

```C++
int b(20);
```

- `int class`를 이용한 instance 생성
- 결과적으로는 `int b = 20`과 동일하지만 다른 개념
<br>

```C++
int c{ 30 };
```
```C++
int d[2] = {10, 20, 30};
int e[2]{40, 50, 60};
```

- 단일 값을 갖는 변수에도 <b>aggregate</b> 타입 initializer 사용 가능
<br>

```
<br>
```C++
st g{ 30, 'A' };
```

- 구조체 선언 시 `struct` 생략 가능
- 인스턴스 생성 방법으로도 허용
<br>

#### 함수의 Default Parameter

- 일반 parameter들 뒤에만 올 수 있음
<br>
```C++
int f1(int x, int y = 10, char c = 'A')        // 선언
int f1(int xx, int yy, char cc) { //함수 내용 } // 정의
```
- 함수 선언과 정의를 구분하여 설계하는 경우, <u>반드시 선언에만 default 지정</u>
- 양쪽 모두 허용 시 모호성이 발생하기 때문
<br>

#### 함수 Overloading
<br>

- Paremeter와 return이 다르고 이름이 같은 함수를 여러 개 만들 수 있음
- <b>overloading</b>: 같은 이름의 함수 여러 개를 메모리에 적재하는 것
- <i>return만 다른 이름의 함수는 만들 수 없음</i>
- 컴파일러는 전달되는 argument의 타입, 개수를 판단하여 호출 함수 결정
<br>

#### Template
<br>

- Parameter의 타입을 인자의 타입에 맞추기 위함
```C++
template<typename T1, typename T2>
auto f1(T1 t1, T2 t2) { return t1 + t2; }

cout << f1<double, int>(5.8, 4) << endl;   // Template Type 지정 방식 (권장)
```

- argument의 타입에 맞춰지므로 리턴에만 타입 이름 사용은 불가
- `auto`: complier로 하여금 return값의 자료형을 유추하도록 만듦
<br>

#### 동적 메모리 할당
<br>

- `new`: 인스턴스를 만들고 초기화한 후 인스턴스 주소를 반환
	- new 뒤에 타입이 와야하며, 좌변 타입과 일치해야 함
	- `int *p = new int;`
	- `char *q = new char[4];`
	- `int (*r)[4] = new int[2][4];`
	- `st *x = new st[4];`
- `delete`: 인스턴스를 삭제 
	- `delete p;`
	- `delete q[];`
	- `delete r[];`
	- `delete x[];`
<br>

#### Call by Reference 
<br>

- <b>Reference &</b>: 변수의 이름의 별칭 (alias)를 만들어 변수를 공유
```C++
int a = 10;
int &b = a;            // b는 a의 별칭
int *p = &a;           // p는 a의 주소
```
<br>

- <b>Call by value</b>
- 값을 복사하여 전달 받음, 원본 수정 X
```C++
void f1(int x)
{
	x = 10;
}
```
<br>

- <b>Call by address</b>
- 주소 값을 전달받음, 원본을 고칠 수 있으나 간접연산 발생
```C++
void f2(int *p)
{
	p[1] = 10;
}
```
<br>

- <b>Call by reference</b>
- 원본 수정 가능, 간접 연산이 발생하지 않음
```C++
void f3(int &b)
{
	b = 10;
}
```
<br>

- 원본을 고치지 않고 참조만 하려면 `const`키워드 사용
<br>

#### Ranged for Loop
<br>

- 배열 or 컨테이너에서 item을 꺼내는 for loop
```C++
int a[4] = { 10, 20, 30, 40 };

for(int x : a) 
{ 
	cout << x << endl; 
}
```
<br>

- `const auto &a`: Ranger for에서 많이 사용
<br>

#### auto
<br>

- 컴파일러가 형식을 잡아주도록 하는 타입
- 단, 동적 타입, 배열 요소에는 사용 불가
```C++
auto f[] = { 1, 3, 5, 6 };        // 요소들의 타입만으로 판단 불가
```