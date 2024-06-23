# 항목 18 : 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자
```cpp
class Date
{
public:
    Date(int month, int day, int year);
};

Date d(30, 19, 12345);  // 무의미한 숫자가 들어올 수 있음.

// ============================

struct Day { ... }
struct Month { ... }
struct Year { ... }

class Date
{
public:
    Date(const Month& m, const Day& d, const Year& y);
};
```
단순한 타입으로 인터페이스의 매개변수를 줄 경우 사용자의 실수로 인해 무의미한 값이 들어올 수 있음.
<br>
대신 타입을 명시하여 사용자의 실수를 줄일 수 있음.

## 특별한 이유가 없다면 사용자 정의 타입은 기본제공 타입처럼 동작하게 만들어라.
사용자를 위해 만드는 타입은 사용자가 직관적으로 이해할 성질 그대로 지원해야 한다.
<br>
이를 통해 일관성 있는 인터페이스를 제공할 수 있음. STL은 모두 size 멤버 함수를 지원함.

```cpp
Investment* createInvestment();                 // 사용자가 포인터를 삭제해야 한다는 것을 기억해야 함.
std::shared_ptr<Investment> createInvestment(); // 사용자가 외워야 할 것이 없음
```
사용자는 외워야 제대로 쓸 수 있는 인터페이스는 잘 못 쓰기 쉬움.
<br>
사용자가 유발할 수 있는 대부분의 문제는 std::shared_ptr을 통해 예방할 수 있음.

## 정리
좋은 인터페이스는 제대로 쓰기에 쉬우며 엉터리로 쓰기에 어렵습니다. 인터페이스를 만들때는 이 특성을 지닐 수 있도록 고민하고 또 고민합시다.
<br>
인터페이스의 올바른 사용을 이끄는 방법으로는 인터페이스 사이의 일관성 잡아주기, 그리고 기본제공 타입과의 동작 호환성 유지하기가 있습니다.
<br>
사용자의 실수를 방지하는 방법으로는 새로운 타입 만들기, 타입에 대한 연산을 제한하기, 객체의 값에 대해 제약 걸기, 자원 관리 작업을 사용자 책임으로 놓지 않기가 있습니다.
<br>
std::shared_ptr은 사용자 정의 삭제자를 지원합니다. 이 특징 때문에 std::shared_ptr은 교차 DLL 문제를 막아주며, 뮤텍스 등을 자동으로 잠금 해제하는데 쓸 수 있습니다.

# 항목 19 : 클래스 설계는 타입 설계와 똑같이 취급하자.
C++에서 새로운 클래스를 정의한다는 것은 새로운 타입을 정의하는 것과 같다.
<br>
즉, 프로그래머는 클래스 설계자이면서 타입 설계자이다.
<br>
좋은 타입은 문법(syntax)가 자연스럽고, 의미구조(semantics)가 직관적이며, 효율적인 구현이 가능해야 한다.

## 효과적인 클래스를 설계하기 위해 고려해야 하는 사항
### 새로 정의한 타입의 객체 생성 및 소멸은 어떻게 이루어져야 하는가?
이에 따라 생성자와 소멸자의 설계가 바뀐다.
<br>
또한 메모리 할당 함수(operator new, operator delete, operator new[], operator delete[])의 custom 구현에도 영향을 미친다.

### 객체 초기화는 객체 대입과 어떻게 달라야 하는가?
생성자와 대입 연산자의 동작 및 둘 사이의 차이점을 결정짓는 요소.

### 새로운 타입으로 만든 객체가 값에 의해 전달되는 경우에 어떤 의미를 줄 것인가?
어떤 타입에 대해 '값에 의한 전달'을 구현하는 쪽은 복사 생성자이다. 즉, 복사 생성자를 어떻게 구현할 것인가?

### 새로운 타입이 가질 수 있는 적법한 값에 대한 제약은 무엇으로 잡을 것인가?
클래스의 데이터 멤버 몇 가지 조합 값만은 반드시 유효해야 하는데, 이를 클래스의 불변속성(invariant)라 하며, 클래스 차원에서 지켜줘야 하는 부분.
<br>
이 불변속성에 따라 클래스 멤버 함수 안에서 해 주어야 할 에러 점검 루틴이 좌우된다. 특히, 생성자, 대입 연산자, setter 함수.
<br>
또한 예외에도 영향을 준다.

### 기존 클래스 상속 계통망에 맞출 것인가?
기존 클래스로부터 상속을 받는다면 설계는 기존 클래스에 의해 제약을 받는다.
<br>
멤버 함수의 virtual 여부, 소멸자의 virtual 여부 등

### 어떤 종류의 타입 변환을 허용할 것인가?
새로 만든 타입이 기존 타입들 중 어떤 타입과 변환을 지원할 것인가.
<br>
암시적 변환(T1::operator T2, non expicit 생성자), 명시적 변환(explcit 생성자)

### 어떤 연산자와 함수를 두어야 의미가 있을까?
어떤 것은 멤버 함수로 적합하고, 어떤것은 그렇지 않다.

### 표준 함수들 중 어떤 것을 허용하지 말 것인가?
private, public 여부를 결정한다.

### 새로운 타입의 멤버에 대한 접근권한을 어느 쪽에 줄 것인가?
멤버 변수의 public, protected, private여부를 결정한다.
<br>
또한 friend를 줄 클래스 및 함수를 결정할 수 있음.
<br>
한 클래스를 다른 클래스에 중첩시킬 것인가도 결정

### 선언되지 않은 인터페이스로 무엇을 둘 것인가?
프로그래머가 만들 타입이 제공할 보장이 어떤 종류인가에 대한 질문.
<br>
보장할 수 있는 부분은 수행 성능 및 예외 안전성, 자원 사용.
<br>
보장하겠다고 결정한 결과는 클래스 구현에 제약으로 작용

### 새로 만드는 타입이 얼마나 일반적인가?
만약 새로 만드는 타입이 동일 계열의 타입군이라면 클래스를 만드는 것이 아니라 템플릿을 정의해야 한다.

### 정말로 꼭 필요한 타입인가?
기존 클래스에 대한 기능 몇 개가 아쉬워 파생 클래스를 새로 뽑는다면, 차라리 간단하게 비멤버 함수라든지 템플릿을 몇 개 더 정의하는 편이 좋음.

## 정리
클래스 설계는 타입 설계입니다. 새로운 타입을 정의하기 전에, 이번 항목에 나온 모든 고려사항을 빠짐없이 점검해보십시오.

# 항목 20 : '값에 의한 전달'보다는 '상수 객체 참조자에 의한 전달' 방식을 택하는 편이 대개 낫다.
기본적으로 C++는 함수로부터 객체를 전달받거나 함수에 객체를 전달할 때 '값에 의한 전달' 방식을 사용한다. 즉, 인자의 '사본'을 통해 초기화되며, '사본'을 돌려준다.

```cpp
class Person
{
public:
    Person();
    virtual ~Person();

private:
    std::string name;
    std::string address;
};

class Student : public Person
{
public:
    Student();
    ~Student();

private:
    std::string schoolName;
    std::string schoolAddress;
};

bool validateStudent(Student s);

Student plato;
bool platoIsOk = validateSudent(plato);

bool validateStudent(const Student& s);
```
위 함수는 호출 과정에서 불필요한 생성과 소멸을 수행한다.
<br>
또한 std::string의 복사와 삭제도 4번씩 수행된다.
<br>
만약 참조자로 전달할 경우 불필요한 생성과 소멸을 수행할 필요가 없다.
<br>
대신 const로 전달한 점에 주의해야 한다. 기존 함수는 사본으로 보냈기 때문에 Student 객체에 변화가 생겨도 외부에서는 문제가 없었지만 참조로 전달한다면 함수 내부에서 원본이 수정될 가능성이 있음.

```cpp
class Window
{
public:
    ...
    std::string name() const;
    virtual void display() const;
};

class WindowWithScrollBars : public Window
{
public:
    ...
    virtual void display() const override;
};

void printNameAndDisplay(const Window w)
{
    std::cout << w.name();
    w.display();
}

WindowWithScrollBar wwsb;
printNameAndDisplay(wwsb);
```
참조에 의한 전달은 복사 손실 문제도 없어짐.
<br>
값에 의한 전달에서 복사 손실 문제를 해결하기 위해서는 const를 추가해야 함.

참조는 기본적으로 포인터를 통해 구현되기 때문에 기본 제공 타입의 경우 값에 의한 전달이 더 효율적일수도 있음. 또한 STL의 Iterator나 함수 객체에 대해서도 값에 의한 전달이 더 효율적임.
<br>
참고로 Iterator와 함수 객체를 구현할 때 주의해야 할 점이 1. 복사 효율을 높일 것, 2. 복사 손실 문제에 노출되지 않도록 만드는 것 이다.
<br>
기본 타입이 값으로 전달하는 것이 효율적인 이유는 단순히 크기가 작기 때문은 아니다.
<br>
포인터 멤버 하나만 있는 클래스를 복사하는 과정은 포인터가 가리키는 객체도 복사해야 한다는 점을 유의해야 한다.
<br>
또한, 객체의 크기가 작고 복사 생성자의 비용이 비싸지 않더라도, 컴파일러 중에는 기본제공 타입이 아닌 경우에는 최적화를 다르게 하는 경우가 있음.
<br>
마지막으로 사용자 정의 타입은 언제든지 크기가 변화될 수 있음.

즉, 일반적으로 값에 의한 전달이 저비용이라고 가정해도 좋은 유일한 타입은 기본 제공 타입, STL 반복자, 함수 객체 타입 3개 뿐이다. 

## 정리
값에 의한 전달보다는 상수 객체 참조자에 의한 전달을 선호합시다. 대체적으로 효율적일 뿐만 아니라 복사손실 문제까지 막아줍니다.
<br>
이번 항목에서 다룬 법칙은 기본제공 타입 및 STL 반복자, 함수 객체 타입에는 맞지 않습니다. 이들에 대해서는 값에 의한 전달이 더 적절합니다.

# 항목 21 : 함수에서 객체를 반환해야 할 경우에는 참조자를 반환하려고 들지 말자.
```cpp
class Rational
{
public:
    Rational(int numerator = 0, int denominator = 1);

private:
    int n, d;

friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};
```
함수에서 새로운 객체를 만드는 방법은 스택에 만드는 방법과 힙에 만드는 방법이 있음.
<br>
Rational operator*를 참조를 반환하고싶다면 어디선가 객체를 만들어서 반환해야 함.

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```
스택에 객체를 만드는 방법
<br>
참조로 반환하고자 하는 것은 생성, 소멸을 줄이기 위해서인데 기본적으로 생성이 이미 호출되고 있음.
<br>
또한, result는 지역 객체로 함수가 끝나면 알아서 소멸됨. 미정의 동작을 유발함.

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}

Rational w, x, y, z;
w = x * y * z;
```
힙에 객체를 만드는 방법.
<br>
이번에도 여전히 생성자가 호출되며 사용자가 delete를 사용해야한다는 것을 기억해야 한다.
<br>
심지어 위 사용법의 경우 메모리 누수를 막을 수 없음.

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    static Rational result;
    result = ...;
    return result;
}

bool operator==(const Rational& lhs, const Rational& rhs);

Rational a, b, c, d;

if((A * b) == (c * d)) { ... }
```
생성자 호출을 없애는 방법.
<br>
스레드 안전성 문제가 있음.
<br>
심지어 위 조건문은 항상 참을 반환함.

```cpp
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```
즉, 그냥 값으로 반환하는 것이 가장 좋다.
<br>
또한, C++ 컴파일러는 몇 가지 조건(RVO)에 의해 반환되는 값에 대하여 최적화를 수행해놓았다.

## 정리
지역 스택 객체에 대한 포인터나 참조자를 반환하는 일, 혹은 힙에 할당된 객체에 대한 참조자를 반환하는 일, 또는 지역 정적 객체에 대한 포인터나 참조자를 반환하는 일은 그런 객체가 두 개 이상 필요해질 가능성이 있다면 절대로 하지 마세요.

# 항목 22 : 데이터 멤버가 선언될 곳은 private 영역임을 명심하자
## public을 쓰면 안되는 이유
함수만 지원함으로써 문법적 일관성을 지원할 수 있음.
<br>
함수를 통해 데이터 멤버의 접근성에 대해 훨씬 정교한 제어흘 할 수 있음. 읽기 전용, 쓰기 전용 접근 구현이 가능함.

```cpp
class SpeedDataCollection
{
public:
    void addValue(int speed);
    double averageSoFar() const;
}
```
캡슐화를 통해 내부의 변화를 외부에 알리지 않을 수 있음.
<br>
arverageSoFar함수의 구현을 위해 변수를 저장할수도 있고 매번 계산할 수도 있음. 이는 함수를 사용함으로써 프로그램 사용 용도에 따라 선택적으로 적용 가능함.
<Br>
그 외에 알림 처리, 불변속성 및 사전조건-사후조건 검증, 스레딩 환경에서 동기화 등 절차를 통해 구현상의 융퉁성을 누릴 수 있음.
<br>
public이라는 말은 캡슐화되지 않았다는 뜻이머 이는 바꿀 수 없다는 것을 의미.

protected 역시 마찬가지로 캡슐화에 큰 영향을 주지 못한다.
<br>
즉, 캡슐화의 관점에서 쓸모 있는 접근 수준은 private과 non-private 둘뿐이다.

## 정리
데이터 멤버는 private 멤버로 선언합시다. 이를 통해 클래스 제작자는 문법적으로 일관성 있는 데이터 접근 통로를 제공할 수 있고, 필요에 따라서는 세밀한 접근 제어도 가능하며, 클래스의 불변속성을 강화할 수 있을 뿐만 아니라, 내부 구현의 융퉁성도 발휘할 수 있습니다.
<br>
protected는 public보다 더 많이 보호받고 있는 것이 절대로 아닙니다.

# 항목 23 : 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자.
```cpp
// "WebBrowser.h    WebBrowser와 관련된 핵심 기능들
namespace WebBrowserStuff
{
    class WebBrowser
    {
    public:
        void clearCache();
        void clearHistory();
        void removeCookies();

        void clearEverything();
    };

    void clearEverything(WebBrowser& wb)
    {
        wb.clearCache();
        wb.clearHistory();
        wb.removeCookies();
    }
}

// "WebBrowserBookmark.h    즐겨찾기 관련 편의 함수들
namespace WebBrowserStuff { ... }
```
객체지향이 할수있는 만큼 데이터를 캡슐화하라고 하지만, 멤버 버전 clearEveryting은 비멤버 버전보다 캡슐화 정도가 좋지 않음.
<br>
오히려 비멤버 버전은 패키징 유연성이 높아지고, 컴파일 의존도를 낮출 수 있고, WebBrowser의 확장성도 높일 수 있음.

## 캡슐화
어떤 것을 캡슐화하면 외부에서는 이것을 볼 수 없음. 캡슐화가 늘어날수록 밖에서 볼 수 있는 것들이 줄어듬. 밖에서 볼 수 있는 것이 줄어들수록 바꿀 때 필요한 유연성이 증가함.
<br>
즉, 이미 있는 코드를 바꾸더라도 제한된 사용자들밖에 영향을 주지 않는 융통성을 확보할 수 있음.
<br>
private 멤버로 되어있는 멤버는 멤버 함수 및 프렌드 함수만 접근할 수 있으므로 데이터에 접근할 수 있는 함수의 개수 예측이 쉬움.
<br>
동일한 기능을 제공한다면 비멤버 비프렌드 함수를 사용할 경우 private 멤버에 접근할 수 있는 함수의 개수를 늘리지 않으므로 데이터의 캡슐화 정도를 낮추지 않음.
<br>
같은 네임스페이스에 두는 것을 통해 utility 함수를 몰아놓아 컴파일 의존성을 줄일 수 있음. 클래스 멤버 함수는 기능을 쪼개는 것이 불가능. 하나의 클래스는 그 전체가 통으로 정의되어야 하고 여러 조각으로 나뉠 수 없음.
<br>
오히려 편의 함수 전체를 여러 개의 헤더 파일에 나누어놓으면 편의 함수 집합의 확장이 손쉬워짐.
<br>
새로운 기능 집합이 필요하다면 새 헤더 파일에 해당 네임스페이스에 비멤버 비프렌드 함수를 원하는만큼 추가하면 됨.
<br>
멤버 함수의 경우 사용자는 클래스 정의를 확장할 수 없음.

## 정리
멤버 함수보다는 비멤버 비프렌드 함수를 자주 쓰도록 합시다. 캡슐화 정도가 높아지고, 패키징 유연성도 커지며, 기능적인 확장성도 늘어납니다.

# 항목 24 : 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자.
```cpp
class Rational
{
public:
    Rational(int numerator = 0, int denominator = 1);

    int numerator() const;
    int denominator() const;

    const Rational operator*(const Rational& rhs) const;

private:
    ...
};

Rational oneEight(1, 8);
Rational oneHalf(1, 2);

Rational result = oneEight * oneHalf;

result = oneHalf * 2;   // ok
result = 2 * oneHalf;   // error
```
oneHalf와 2를 곱하는 식이 가능한 이유는 컴파일러가 암시적 타입변환을 통해 Rational을 생성했기 때문이다.

```cpp
class Rational
{
public:
    Rational(int numerator = 0, int denominator = 1);

    int numerator() const;
    int denominator() const;

private:
    ...
};

const Rational operator*(const Rational& lhs, const Rational& rhs) { ... }
```
멤버 함수에 정의한 operator*를 지우고 비멤버 함수로 다시 정의한다.
<br>
이를 통해 다양한 형식의 인자를 암묵적 생성자를 통해 연산이 가능하다.

어떤 클래스와 연관 관계를 맺어 놓고는 싶은데 멤버 함수이면 안되는(모든 인자에 대하여 타입 변환이 필요 등) 함수에 대해 프렌드로 선언하기보다는 비멤버 함수로 선언해야 한다.

## 정리
어떤 함수에 들어가는 모든 매개변수(this 포인터가 가리키는 객체도 포함해서)에 대해 타입 변환을 해 줄 필요가 있따면, 그 함수는 비멤버이어야 한다.

# 항목 25 : 예외를 던지지 않는 swap에 대한 지원도 생각해보자.
```cpp
namespace std
{
    template<typename T>
    void swap(T& a, T& b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
```
표준에서 기본적으로 제공하는 swap은 복사(복사 생성자 및 복사 대입 연산자)만 제대로 지원하는 타입이기만 하면 어떤 타입의 객체이든 맞바꾸기 동작을 수행.
<br>
다만, 한 번 호출에서 복사가 세번 발생한다는 점이 단점.

```cpp
class WidgetImpl
{
public:
    ...

private:
    int a, b, c;
    std::vector<double> v;
};

class Widget
{
public:
    Widget(const Widget& rhs);

    Widget& operator=(const Widget& rhs)
    {
        ...
        *pImpl = *(rhs.pImpl);
        ...
    }

    void swap(Widget& other)
    {
        using std::swap;

        swap(pImpl, other.pImpl);
    }
    ...

private:
    WidgetImpl* pImpl;
};

namespace std
{
    template<>  // 완전 템플릿 특수화
    void swap<Widget>(Widget& a, Widget& b) // T가 Widget에 대한 특수화라는 것을 알림
    {
        a.swap(b);
    }
}
```
만약 pimpl 관용구같이 포인터를 주로 사용하는 객체라면 포인터를 변경하는 것 보다 표준 swap 연산의 비용이 매우 비쌈.
<br>
std::swap에 Widget 객체를 맞바꿀 때는 일반적인 방식 대신 pImpl 포인터만 바꾸라는 것을 알리기 위해 template 특수화를 할 수 있음.

```cpp
template<typename T>
class WidgetImpl { ... }

template<typename T>
class Widget { ... }

namespace std
{
    // 컴파일 에러
    template<typename T>
    void swap<Widget<T>>(Widget<T>& a, Widget<T>& b) { a.swap(b); }
}
```
템플릿 클래스는 템플릿 특수화를 할 수 없음.
<br>
이를 부분 특수화라 하는데 C++는 클래스에 대해서는 부분 특수화를 지원하지만 함수에 대해서는 허용하지 않음.

```cpp
namespace std
{
    // overload 버전 추가, 컴파일 가능. 정의되지 않은 동작
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) { a.swap(b); }
}
```
함수 템플릿을 부분적으로 특수화하고 싶을 경우 오버로드 버전을 추가하면 됨.
<br>
다만, std는 특별한 네임스페이스로 std 내의 템플릿에 대한 완전 특수화는 가능하지만, 새로운 템플릿을 추가하는 것은 불가능.

```cpp
namespace WidgetStuff
{
    template<typename T>
    class Widget { ... }

    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) { a.swap(b); }
}

template<typename T>
void doSomething(T& obj1, T& obj2)
{
    using std::swap;

    swap(a, b);
}
```
대신 std가 아닌 곳에서 특수화 버전이나 오버로딩 버전을 추가하면 됨.
<br>
함수의 이름을 찾기 위해 해당 타입의 인자가 위치한 네임스페이스 내부의 이름을 탐색해 들어간다는 쾨니그 탐색을 통해 WidgetStuff 네임스페이스 안에서 Widget 특수화 버전을 찾는다.
<br>
즉, 클래스와 동일한 네임스페이스 안에 비멤버 버전 swap을 만들어 넣는다.
<br>
사용자는 namespace를 명시함으로써 컴파일러가 사용할 적절한 swap 버전을 지정할 수 있음.
<br>
using 대신 직접 명시하는 방식은 추천하지 않는데, 이는 더 맞을 수 있는 T 전용 버전이 다른 곳에 있을 가능성을 무시하고 컴파일러를 구속한다.

표준에서 제공하는 swap이 클래스 및 클래스 템플릿에 대해 납득할 만한 효율을 보이면, 그냥 사용하라
<br>
표준 swap의 효율이 기대한 만큼 충분하지 않다면(또는 pImpl 구문과 비슷하게 만들어져있다면)
1. Custom Type으로 만들어진 두 객체의 값을 빨리 맞바꾸는 swap함수를 만들고 public으로 선언하라.
2. 클래스 혹은 템플릿이 들어있는 네임스페이스와 같은 네임스페이스에 비멤버 swap을 만들고, 1번에서 만든 swap 멤버 함수를 호출하라.
3. 템플릿이 아닌 클래스를 만들었다면 해당 클래스에 대한 std::swap 특수화 버전을 만들어라. 이 특수화버전에서도 멤버 swap 함수를 호출하라.
<br>
using을 사용하고 namespace 한정자를 붙이지 마라.

멤버 버전 swap은 절대로 예외를 던져서는 안된다.
<br>
이는 강력한 예외 안전성 보장(어떤 연산이 실행되다가 예외가 발생하면 그 연산이 시작되기 전의 상태로 돌릴 수 있다는 보장)을 제공하기 위해서이다.
<br>
성능이 좋은 swap함수는 거의 항상 기본 제공 타입을 사용한 연산으로 만들어지고, 기본 제공 타입 연산은 절대로 예외를 던지지 않는다.

## 정리
std::swap이 여러분의 타입에 대해 느리게 동작할 여지가 있다면 swap멤버 함수를 제공합시다. 이 멤버 swap은 예외를 던지지 않도록 만듭시다.
<br>
멤버 swap을 제공했으면, 이 멤버를 호출하는 비멤버 swap도 제공합니다. 클래스(템플릿이 아닌)에 대해서는 std::swap도 특수화해 둡시다.
<br>
사용자 입장에서 swap을 호출할 때는, std::swap에 대한 using 선언을 넣어 준 후에 네임스페이스 한정 없이 swap을 호출합시다.
<br>
사용자 정의 타입에 대한 std 템플릿을 완전 특수화하는 것은 가능합니다. 그러나 std에 어떤 것이라도 새로 '추가'하려고 들지는 마십시오.
