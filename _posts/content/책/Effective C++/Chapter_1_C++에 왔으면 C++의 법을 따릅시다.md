# 0장 개요
## 용어 사용
### 선언
```cpp
extern int x;

std::size_t numDisgits(int number);

class Widget;

template<typename T>
class GraphNode;
```
코드에 사용되는 '어떤 대상의 이름과 타입을 컴파일러에게 알려주는 것

### 시그니처
함수의 매개변수 리스트와 반환 타입(비공식) 정보. 함수의 시그니처가 함수의 타입이다.

### 정의
```cpp
int x;

std::size_t numDigits(int number)
{
    ...
}

class Widget
{
    ...
};

template<typename T>
class GraphNode
{
    ...
};
```
선언에서 빠진 구체적인 세부사항을 컴파일러에게 제공하는 것

### 초기화
객체에 최초의 값을 부여하는 과정
<br>
사용자 정의 타입은 생성자에서 초기화가 이루어짐

### 기본 생성자
```cpp
class A
{
public:
    A();    // 기본 생성자
};

class B 
{
public:
    explicit B(int x = 0, bool b = true);   // 기본 생성자, 묵시적 형변환이 안됨
};
```
어떤 인자도 주어지지 않은 채로 호출될 수 있는 생성자

## 복사 생성자
어떤 객체의 초기화를 위해 그와 같은 타입의 객체로부터 초기화할 때 호출되는 함수

## 복사 대입 연산자
```cpp
class Widget
{
public:
    Widget();                               // 기본 생성자
    Widget(const Widget& rhs);              // 복사 생성자
    Widget& operator=(const Widget& rhs);   // 복사 대입 연산자
};

Widget w1;      // 기본 생성자 호출
Widget w2(w1);  // 복사 생성자 호출
w2 = w2;        // 복사 대입 연산자 호출
```
같은 타입의 다른 객체에 어떤 객체의 값을 복사하는 용도로 쓰이는 함수

```cpp
bool hasAcceptableQuality(Widget w);

Widget aWidget;
if(hasAcceptableQuality(aWidget)) { ... }
```
값에 의한 객체 전달을 정의해주는 함수

### STL
표준 템플릿 라이브러리
<br>
컨테이너(std::vector, std::set 등), 반복자(std::vector<iterator> 등), 알고리즘(for_each, find 등) 을 지원

### 미정의 동작
```cpp
int *p = nullptr;
std::cout << *p;

char name[] = "Darla";
char c = name[10];
```
결과의 예측이 불가능.

### 인터페이스
C++에서는 공식적으로 지원하지 않음
<br>
이 책에서는 함수의 시그니처, 접근 가능 요소, 템플릿의 타입 매개변수로서 유효해야 하는 표현식 등을 지칭
<br>
즉, 설계 아이디어로서의 인터페이스를 의미

### 사용자
여러분이 만든 코드를 사용하는 사람
<br>
코드를 호출하는 사람, 코드를 유지보수하는 사람

## 이름 짓기에 대한 규약
lhs : Left Hand Side
<br>
rhs : Right Hand Side
<br>
Widget은 아무 의미 없는 클래스를 의미
<br>
포인터를 의미하는 p 접두사 사용
<br>
참조자를 의미하는 r 접두사 사용
<br>
멤버 변수를 의미하는 m 접두사 사용

## 스레딩에 대한 고려사항
C++에서는 언어 차원에서 스레드에 대한 개념이 없음(지금은 지원 중)
<br>
대신 스레드 환경에서 문제를 일으킬 만한 것들을 지적해서 알려주는 방식으로 설명

## TR1과 Boost
### TR1
C++ 표준 라이브러리에 새로 추가되는 기능들에 대한 명세
<br>
클래스 및 함수 템플릿 위주로 담김
<br>
해시 테이블, 참조 차운팅 방식 스마트 포인터, 정규 표현식 등
<br>
std::tr1 네임스페이스에 있음(현재는 modern의 표준으로 들어와있음)

### 부스트
오픈소스 C++ 라이브러리를 제공하는 단체 또는 웹사이트
<br>
실제로 modern의 대부분이 Boost를 수용한 것으로 알고있음.

# 항목 1 : C++를 언어들의 연합체로 바라보는 안목은 필수
C++는 다중 패러다임 언어라고 불린다.
<br>
절차적 프로그래밍을 기본으로 하여 객체 지향, 함수식, 일반화 프로그래밍을 포함하며, 메타프로그래밍 개념까지 지원한다.
<br>
즉, C++는 여러 언어의 연합체로 봐야한다. 이후 각 언어에 관한 규칙을 따로 이해해야 한다.

## 지원하는 하위 언어
### C
C++는 C를 기본으로 한다.
<br>
블록, 문장, 선행 처리자, 기본제공 데이터타입, 배열, 포인터 등

### 객체 지향 개념의 C++
클래스, 캡슐화, 상속, 다형성, 가상 함수(동적 바인딩)

### 템플릿 C++
많은 영향력을 끼치며 템플릿 메타 프로그래밍 개념까지 등장함.

### STL
템플릿 라이브러리
<br>
컨테이너, 반복자, 알고리즘, 함수 객체

이 4가지 하위 언어들이 C++를 이루고 있으며 각 하위언어가 자신만의 규칙을 갖는다.

## 정리
C++를 사용한 효과적인 프로그래밍 규칙은 경우에 따라 달라집니다. 그 경우란 바로 C++의 어떤 부분을 사용하느냐입니다.

# 항목 2 : #define을 쓰려거든 const, enum, inline을 떠올리자
가급적 선행처리자보다 컴파일러를 가까이 하는 것이 좋다.

```cpp
#define ASPECT_RATIO 1.653
```
컴파일러에겐 ASPECT_RATIO라는 기호가 보이지 않는다. 컴파일러에게 넘어가기 전 ASPECT_RATIO라는 부분은 1.653 숫자 상수로 바꾸어버린다.
<br>
이로 인해 컴파일 에러를 이해하기 어려워진다.

```cpp
const double AspectRatio = 1.653;
```
매크로 대신 상수를 쓰는 것을 추천한다.

```cpp
const char* const authorName = "Scott Mayers";
```
상수 포인터를 정의하는 경우에는 헤더 파일에 넣는 것이 상례이고, 포인터는 꼭 const로 선언해줘야 하고, 포인터가 가리키는 대상까지 const로 선언해야 한다.

```cpp
// header
class GamePlayer
{
private:
    static const int NumTurns = 5;
    int scores[NumTurns];
    ...
};

// cpp
const int GamePlayer::NumTurns; // 주소를 쓸 경우. 정수 타입 상수는 값이 주어지지는 않음
```
어떤 상수의 유효범위를 클래스로 한정하고자 할 때, 즉 클래스 멤버로 상수를 정의하는 경우에는 static 멤버로 만들어야 한다.
<br>
위 NumTurns는 선언 된것이지 정의된것이 아니다.
<br>
정적 멤버로 만들어지는 정수류 타입(int, char, bool 등) 상수는 주소를 취하지 않는 한 선언 만으로도 문제가 없다.
<br>
클래스 상수의 정의에는 초기값이 있으면 안되는데, 클래스 상수의 초기값은 해당 상수가 선언된 시점에서 바로 주어지기 때문이다. 즉, NumTurns가 선언될 당시에 바로 초기화된다.
<br>
참고로 #define은 유효범위의 개념이 없으므로 클래스 상수로는 만들 수 없음.

```cpp
class GamePlayer
{
private:
    enum { NumTurns = 5; };

    int scores[NumTurns];
};
```
만약 컴파일 타임에 클래스 상수값이 필요하다면 enum을 이용하는 방법도 있는데 modern에서는 그냥 constexpr을 쓰면 된다.
<br>
Enum Hack 방식은 #define에 더 가까운 방식. 주소와 참조자를 취할 수 없음.

```cpp
#define CALL_WITH_MAX(a, b) f((a) > (b)) > (a) : (b)

int a = 5, b = 0;
CALL_WITH_MAX(++a, b);      // a는 2번 증가
CALL_WITH_MAX(++a, b + 10); // a는 1번 증가
```
매크로 함수는 오용 가능성이 있음.

```cpp
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b;
}
```
대신 inline함수를 사용하여 함수 호출 오버헤드를 줄이는 것을 추천.

## 정리
단순한 상수를 쓸 때는, #define보다 const 객체 또는 enum을 우선 생각합시다.
<br>
함수처럼 쓰이는 매크로를 만들려면, #define 매크로보다 인라인 함수를 우선 생각합시다.

# 항목 3 : 낌새만 보이면 const를 들이대 보자
const는 '의미적인 제약'을 소스 코드 수준에서 붙인다는 점과 컴파일러가 이 제약을 단단히 지켜준다
<br>
클래스 밖에서 전역 혹은 네임스페이스 유효범위의 상수 선언, 파일-함수-블록 유효 범위에서 static 선언한 객체, 포인터 자체, 포인터가 가리키는 데이터 등에서 사용 가능

```cpp
char greeting[] = "Hello";
char* p = greeting;                 // 비상수 포인터, 비상수 데이터
const char* p = greeting;           // 비상수 포인터, 상수 데이터
char * const p = greeting;          // 상수 포인터, 비상수 데이터
const char * const p = greeting;    // 상수 포인터, 상수 데이터
```
const 위치에 따른 상수 처리의 차이

```cpp
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin();    // T* const 처럼 동작
*iter = 10;     // 적용 가능
++iter;         // 적용 불가능

std::vector<int>::const_iterator cIter = vec.begin();   // const T* 처럼 동작
*cIter = 10;    // 적용 불가능
++cIter;        // 적용 가능
```
STL 반복자는 포인터를 본뜬 것으로 T*와 흡사하게 동작

```cpp
class Rational { ... }

const Rational operator*(const Rational& lhs, const Rational& rhs);

Rational a, b, c;
if(a * b = c) { ... }   // 방지 가능
```
함수 선언에 있어서 const는 함수 반환값, 매개변수, 멤버 함수 앞, 함수 전체에 대하여 붙을 수 있음
<br>
const를 쓰면 위와 같은 대입 연산으로 작성된 비교 연산 문제를 미연에 막을 수 있음.
<br>
[참조](https://godbolt.org/z/78rxbff1r)
<br>
문제는 rvalue를 const로 줄 경우 move 최적화에 문제가 발생할수도 있는 걸로 알고있음.

## 상수 멤버 함수
멤버 함수에 const 키워드의 역할은 "해당 멤버 함수가 상수 객체에 대해 호출될 함수이다"라는 사실을 알려줌.

이는 클래스의 인터페이스를 이해하기 좋게 함.
<br>
클래스로 만들어진 객체를 변경할 수 있는 함수가 무엇이고, 변경할 수 없는 함수가 무엇인가를 사용자가 알 수 있음.

이 키워드를 통해 상수 객체를 사용할 수 있음.
<br>
상수 객체에 대한 참조자로 전달 받은 객체를 조작 가능함.

```cpp
class TextBlock
{
public:
    const char& operator[](std::size_t position) const { return text[position]; }
    char& operator[](std::size_t position) { return text[position]; }

private:
    std::string text;
};

TextBlock tb("Hello");
std::cout << tb[0];
tb[0] = 'x';

const TextBlock ctb("World");
std::cout << ctb[0];
ctb[0] = 'x';   // 컴파일 에러
```
const 키워드 여부로 멤버 함수의 오버로딩이 가능함.

```cpp
class CTextBlock
{
public:
    char& operator[] (std::size_t position) const { return pText[position]; }

private:
    char* pText;
};

const CTextBlock cctb("Hello");
char* pc = &cctb[0];
*pc = 'J';
```
어떤 멤버 함수가 상수 멤버라는 것은 비트수준 상수성(bitwise constness, 물리적 상수성, physical constness)과 논리적 상수성(logical constness)를 의미.
<br>
비트 수준 상수성이란 객체를 구성하는 비트들 중 어떤 것도 바뀌지 않는다는 것을 의미.
<br>
위 코드는 const 함수 자체는 멤버 객체의 무엇도 수정하지 않으므로 비트 수준 상수성은 지키지만 그 사용법에 따라 데이터가 변형될 위험이 있음.
<br>
함수 반환값을 const로 선언해줘야 할 필요가 있음

```cpp
class CTextBlock
{
public:
    std::size_t length() const;

private:
    char *pText;
    mutable std::size_t textlength;
    mutable bool lengthIsVaild;
};

std::size_t CTextBlock::length() const
{
    if(!lengthIsVaild)
    {
        textLength = std::strlen(pText);
        lengthIsValid = true;
    }
    return textLength;
}
```
논리적 상수성이라 상수 멤버 함수라도 객체의 일부를 수정할 수 있되 사용자측에서 알아채지 못하면 상수 멤버의 자격이 있다는 것.
<br>
위 함수는 TextBlock의 길이를 조회하는 것으로 함수 자체가 const로서 선언되는 것이 옳음.
<br>
다만 textLength와 lengthIsVaild를 수정해야하므로 이를 mutable로 선언해야 함.

## 상수 멤버 및 비상수 멤버 함수에서 코드 중복 현상을 피하는 방법
```cpp
class TextBlock
{
public:
    const char& operator[](std::size_t position) const
    {
        ...
        return text[position];
    }
    char& operator[](std::size_t position)
    {
        return std::const_cast<char&>(std::static_cast<const TextBlock>(*this)[position]);  // static_cast로 casting 해줘야지 const 함수가 불린다는 것에 주의
    }
};
```
const 함수가 오버로딩 가능하다는 점은 동일한 기능의 코드 중복을 유발할 수 있음.
<br>
일반적으로 const casting은 추천하지 않지만 const가 아닌 함수가 호출되는 경우는 객체도 const하지 않다는 의미이기 때문에 const casting을 통해 코드 중복을 줄여도 안전함.
<br>
비멤버 함수의 구현에 상수 멤버 쌍둥이 함수를 사용하는 기법
<br>
단, 상수 멤버 함수가 비상수 멤버 함수를 호출하는 반대 방식은 추천하지 않음. 상수 멤버 함수가 호출하는 비상수 멤버 함수가 내부에서 값을 수정할 위험이 있음.

## 정리
const를 붙여 선언하면 컴파일러가 사용상의 에러를 잡아내는데 도움을 줍니다. const는 어떤 유효 범위에 있는 객체에도 붙을 수 있으며, 함수 매개변수 및 반환 타입에도 붙을 수 있으며, 멤버 함수에도 붙을 수 있습니다.
<br>
컴파일러 쪽에서 보면 비트수준 상수성을 지켜야 하지만, 여러분은 개념적인(논리적인) 상수성을 사용해서 프로그래밍 해야 합니다.
<br>
상수 멤버 및 비상수 멤버 함수가 기능적으로 서로 똑같이 구현되어 있을 경우에는 코드 중복을 피하는 것이 좋은데, 이 때 비상수 버전이 상수 버전을 호출하도록 만드세요.

# 항목 4 : 객체를 사용하기 전에 반드시 그 객체를 초기화하자.
C++의 C부분만 사용한다면 초기화를 보장할 수 없지만 그 외의 부분을 사용한다면 초기화가 되는 등 초기화 규칙은 조금 복잡하므로 명시적으로 초기화하는 것이 좋음.

```cpp
int x = 0;
const char* text = "A C-Style String";

double d;
std::cin >> d;
```
기본 제공 타입으로 만들어진 비멤버 객체는 손수 초기화를 해야한다.

```cpp
class PhoneNumber { ... };

class ABEntry
{
public:
    ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones);

private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};

ABEntry:ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
{
    theName = name;
    theAddress = address;
    thePhones = phones;
    numTimesConsulted = 0;
}
```
대입과 초기화는 구분해야 한다. 위 코드는 초기화하지 않고 대입하고 있다.
<br>
객체의 데이터 멤버는 생성자의 본문이 실행되기 전에 초기화되어야 한다. 멤버 초기화 리스트를 사용해야 한다.
<br>
대입을 사용할 경우 theName, theAddress, thePhones의 기본 생성자를 호출한 후 대입을 수행하지만 멤버 초기화 리스트를 사용하면 생성과 함께 초기화가 진행되어 더 효율적임.
<br>
상수이거나 참조자로 선언된 변수는 반드시 초기화되어야 함.

다양한 생성자를 지원하면서 초기화 코드가 반복되는 상황에서 해당 초기화 값을 파일이나 데이터베이스에서 읽어오는 상황이라면 초기화 함수를 따로 빼는 것도 나쁘지는 않지만 기본적으로는 멤버 초기화를 사용하는 것이 좋음.

## 초기화 순서
### 기본 클래스는 파생 클래스보다 먼저 초기화

### 클래스 데이터 멤버는 그들이 선언된 순서대로 초기화.
멤버 초기화 리스트 순서와 상관없이 선언된 순서대로 초기화가 진행된다. 스레드같은 곳에서 발생할 '무척이나' 찾기 힘든 버그를 방지하기 위해 멤버 초기화 리스트는 클래스 선언 순서와 맞춰주는 것이 좋음.

### 비지역 정적 객체의 초기화 순서는 개별 번역 단위에서 정해진다.
#### 정적 객체
전역 객체
<br>
네임스페이스 유효범위에서 선언된 객체
<br>
클래스 안에서 static으로 선언된 객체
<br>
함수 안에서 static으로 선언된 객체
<br>
파일 유효범위에서 static으로 정의된 객체

이 중 함수 안에서 static으로 선언된 객체는 지역 정적 객체라고 한다.

#### 번역 단위
컴파일을 통해 하나의 목적 파일을 만드는 바탕이 되는 소스 코드
<br>
소스 파일 하나와 include하는 파일들

```cpp
// A.cpp
class FileSystem
{
public:
    std::size_t numDisks() const;
};

extern FileSystem tfs;

// B.cpp
class Directory
{
public:
    Directory(params);
};

Directory::Directory(params)
{
    ...
    std::size_t disks = tfs.numDisks();
}

Directory tempDir(params);
```
즉, 한 쪽 번역단위에서 아직 초기화되지 않은 비정적 객체를 다른 쪽 번역 단위에서 사용하는 문제가 발생할 수 있음.
<br>
즉, 위 코드는 해결할 방법이 없는 정의되지 않은 행동을 수행함.

```cpp
class FileSystem { ... };

FileSystem& tfs()
{
    static FileSystem fs;
    return fs;
}

class Directory { ... };

Directory::Directory(params)
{
    ...
    std::size_t disk = tfs().numDisks();
    ...
}

Directory& tempDir()
{
    static Directory td;
    return td;
}
```
비지역 정적 객체를 지역 정적 객체로 바꾸면 문제를 해결할 수 있음.
<br>
이는 Singleton Pattern.
<br>
지역 정적 객체는 함수 호출 중에 그 객체의 정의에 처음 닿았을 때 초기화.

1. 멤버가 아닌 기본제공 타입 객체는 직접 초기화하라
2. 객체의 모든 부분에 대한 초기화에는 멤버 초기화 리스트를 사용하라
3. 별개의 번역 단위에 정의된 비지역 정적 객체에 영향을 미치는 불확실한 초기화 순서를 염두에 두고 이러한 불확실성을 피해서 프로그램을 설계하라

## 정리
기본제공 타입의 객체는 직접 손으로 초기화합니다. 경우에 따라 저절로 되기도 하고 안되기도 하기 때문입니다.
<br>
생성자에서는 데이터 멤버에 대한 대입문을 생성자 본문 내부에 넣는 방법으로 멤버를 초기화하지 말고 멤버 초기화 리스트를 즐겨 사용합시다. 그리고 초기화 리스트에 데이터 멤버를 나열할 때는 클래스에 각 데이터 멤버가 선언된 순서와 똑같이 나열합시다.
<br>
여러 번역 단위에 있는 비지역 정적 객체들의 초기화 순서 문제는 피해서 설계해야 합니다. 비지역 정적 객체를 지역 정적 객체로 바꾸면 됩니다.
