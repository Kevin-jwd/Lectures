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

- <b>`cin.getline()`</b>
	- Enter 입력까지 한 문장을 받음
	- 이전 Enter로 인해 입력을 못 받는 경우가 생김

<br>

- <b>`cin.ignore()`</b>
	- 입력 버퍼에 남은 글자들을 <b>flush</b>시킴