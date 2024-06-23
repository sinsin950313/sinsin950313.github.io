# 항목 26 : 변수 정의는 늦출 수 있는 데까지 늦추는 근성을 발휘하자
```cpp
std::string encryptPasssward(const std::string& password)
{
    using namespace std;

    string encrypted;

    if(password.length() < MinimumPasswordLength)
    {
        throw logic_error("Password is too short");
    }
    ...

    return encrypted;
}

std::string encryptPasssward(const std::string& password)
{
    using namespace std;

    if(password.length() < MinimumPasswordLength)
    {
        throw logic_error("Password is too short");
    }
    ...

    string encrypted(password);

    return encrypted;
}
```
생성자, 소멸자가 있는 변수의 타입은 프로그램 제어 흐름이 변수의 정의에 닿을 때마다 생성자와 소멸자가 호출되는 비용이 부가된다. 이는 해당 변수가 정의는 됐으나 사용하지 않더라도 부과된다.
<br>
또한 기본 생성자를 호출한 후 값을 대입하는 것보다 필요한 매개변수를 생성자 시점에 넣는 것이 더 효율적이다.
<br>
즉, 실제로 해당 객체가 필요하고 해당 객체를 초기화하는데 필요한 모든 인자를 얻을 때가지 변수 정의를 늦추는 것이 좋다.

```cpp
Widget w;
for(int i = 0; i < n; ++i)
{
    w = i;
    ...
}
```
loop구문의 경우 대입 비용이 생성-소멸보다 싸다면 루프 밖에 정의하는 것이 좋지만 아니라면 루프 안에 정의하는 것이 좋음.
<br>
대신, 루프 밖에 정의할 경우 해당 변수의 유효 범위가 늘어나기 때문에 프로그램의 이해도와 유지보수성이 안좋아질 수 있음.
<br>
즉, 대입이 생성-소멸보다 비용이 덜 들고, 전체 코드에서 수행 성능에 민감한 부분을 건드는 중이 아니라면 루프 안에 선언하는 것이 좋음.
<br>
최근에는 이 문제도 최적화가 되었다고 알고 있음.

## 정리
변수 정의는 늦출 수 있을 때까지 늦춥시다. 프로그램이 더 깔끔해지며 효율도 좋아집니다.

# 항목 27 : 캐스팅은 절약, 또 절약! 잊지 말자
C++는 일단 컴파일만 되면 타입 에러가 생기지 않도록 보장한다는 철학을 바탕으로 설계되어있음.
<br>
다만, 캐스팅의 무분별한 사용은 이 타입시스템의 보장을 위협함.

```cpp
// C Style Casting
(T) 표현식
T(표현식)

// C++ Style Casting
const_cast<T>(표현식)
dynamic_cast<T>(표현식)
reinterpret_cast<T>(표현식)
static_cast<T>(표현식)
```
## const_cast
객체의 상수성을 없애는 용도로 사용.

## dynamic_cast
안전한 다운캐스팅을 할 때 사용하는 연산자.
<br>
주어진 객체가 어떤 클래스 상속 계통에 속한 특정 타입인지 아닌지를 결정하는 작업에 사용.
<br>
런타임 비용이 높음.

## reinterpret_cast
하부 수준 캐스팅을 위해 만들어진 연산자.
<br>
구현환경에 의존적이므로 이식성이 없음.
<br>
하부 수준 코드 외에는 거의 사용하지 말아야 함.

## static_cast
암시적 변환을 강제로 진행할 때 사용.(int를 double로 바꾸는 변환)
<br>
타입 변환을 거꾸로 수행하는 용도(void*를 일반 포인터로 변환)
<br>
상수 객체를 비상수 객체로 변환은 불가능.

캐스팅 사용 목적을 좁혀서 지정하기 때문에 컴파일러에서 에러를 진단하기 쉬움.

```cpp
class Base { ... }
class Derived : public Base { ... }

Derived d;
Base* pb = &d;
```
캐스팅은 단순히 어떤 타입을 다른 타입으로 처리하라고 컴파일러에게 알려주는 것 외에도 타입 변환으로 인해 런타임에 실행되는 코드가 만들어지기도 함.
<br>
위 코드는 포인터의 변위를 Derived*에 적용하여 Base*의 실제 포인터 값을 구하는 동작이 런타임에 이루어진다.
<br>
위 코드가 단일 상속이라 아마 차이가 없겠지만, 다중 상속이라면 변위를 계산하는 과정이 복잡하게 발생.
<br>
여튼, C++를 쓸때는 데이터가 어떤 식으로 메모리에 박혀 있을 것이라는 섣부른 가정을 피해야 한다.

```cpp
class Window
{
public:
    virtual void onResize() { ... }
    ...
};

class SpecialWindow : public Window
{
public:
    virtual void onResize()
    {
        static_cast<Window>(*this).onResize();
    }
    ...
};
```
캐스팅이 들어가면 보기엔 맞는 것 같지만 실제로는 틀린 코드도 존재.
<br>
위 호출은 캐스팅이 일어나면서 *this에 대한 사본이 암시적으로 만들어지므로 함수 호출이 이루어지는 객체는 현재 객체가 아니다.
<br>
캐스팅 대신 Window::onResize()를 명시적으로 호출해야 한다.

dynamic_cast는 정말 느리다. 수행 성능이 중요하다면 사용 시 주의를 놓지 말아야 한다.

```cpp
class Window
{
public:
    virtual void blink() { }
};

class SpecialWindow : public Window
{
public:
    virtual void blink() override { ... }
};

std::vector<std::shared_ptr<SpecialWindow>> VPSW;
```
dynamic_cast를 호출하고 싶은 상황은 기본 클래스의 포인터가 담고 있는 객체가 파생 클래스임이 확실한 상황에서 파생 클래스의 함수를 호출하고 싶을 때 이다.
<br>
위처럼 파생 클래스를 포인터를 담은 컨테이너를 사용하던가 아무것도 하지 않는 기본 구현 가상 함수를 사용하는 방법도 있다.

```cpp
class Window { ... };
class SpecialWindow1 : public Window { ... }
class SpecialWindow2 : public Window { ... }
...

typedef std::vector<std::shared_ptr<Window>> VPW;
VPW winPtrs;

for(VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter)
{
    if(SpecialWindow1* psw1 = dynamic_cast<SpecialWindow1*>(iter->get())) { ... }
    else if(SpecialWindow2* psw2 = dynamic_cast<SpecialWindow2*>(iter->get())) { ... }
    ...
}
```
정말 피해야 하는 설계는 폭포식(cascading) dynamic_cast 구조이다.
<br>
크기가 비대하면서 아름답지 않고, 속도도 둔한데다, 망가지기도 쉽다.
<br>
window 계통 구조가 변경되면 지속적인 검토의 대상이 된다.

잘 작성된 C++코드는 캐스팅을 거의 쓰지 않는다.
<br>
캐스팅을 해야하는 코드를 내부 함수에 몰아놓고, 그 안에서 일어나는 일들을 함수 외부에서 알 수 없도록 인터페이스로 막아두는 것이 좋다.

## 정리
다른 방법이 가능하다면 캐스팅은 피하십시오. 특시 수행 성능에 민감한 코드에서 dynamic_cast는 몇 번이고 다시 생각해보십시오. 설계 중에 캐스팅이 필요해졌다면, 캐스팅을 쓰지 않는 다른 방법을 시도해 보십시오.
<br>
캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길 수 있도록 해보십시오. 이렇게 하면 최소한 사용자는 자신의 코드에 캐스팅을 넣지 않고 이 함수를 호출할 수 있게 됩니다.
<br>
구형 스타일의 캐스트를 쓰려거든 C++ 스타일의 캐스트를 선호하십시오. 발견하기도 쉽고, 설계자가 어떤 역할을 의도했는지가 더 자세하게 드러납니다.

# 항목 28 : 내부에서 사용하는 객체에 대한 핸들을 반환하는 코드는 되도록 피하자.
```cpp
class Point
{
public:
    Point(int x, int y);
    void setX(int newVal);
    void setY(int newVal);
};

struct RectData
{
    Point ulhc;
    Point lrhs;
};

class Rectangle
{
public:
    const Point& upperLeft() const { return pData->ulhc; }
    const Point& lowerRight() const { return pData->lrhs; }
    ...
private:
    std:shared_ptr<RectData> pData;
};

Point coord1(0, 0);
Point coord2(100, 100);

const Rectangle rec(coord1, coord2);
rec.upperLeft().setX(50);
```
const로 선언된 함수를 반환했지만 내부 데이터에 대한 참조자를 반환함으로써 외부에서 해당 객체에 대한 조작 가능성이 발생한다.
<br>
즉, 클래스 데이터 멤버는 아무리 숨겨봤자 그 멤버의 참조자가 반환하는 함수들의 최대 접근도에 따라 캡슐화의 정도가 정해진다.
<br>
즉, 내부 데이터 객체의 핸들을 반환하면 해당 객체의 캡슐화가 무너질 수 있음.
<br>
내부 요소는 데이터 멤버 뿐만 아니라 private으로 선언된 내부 함수도 포함.
<br>
반환 값에 const를 붙여줌으로써 외부에 핸들을 제공하더라도 조작이 불가능해진다.
<br>
이를 통해 사용자에게 의도적으로 부분적으로 제한을 완화할 수 있음.

```cpp
class GUIObject { ... };

const Rectangle boundingBox(const GUIObject& obj);

GUIObject* pgo;
...
const Point* pUpperLeft = &((*pgo).upperLeft());
```
내부 핸들을 반환한다는 것은 무효 참조 핸들 문제가 발생할 수 있음.
<br>
밖으로 나간 핸들은 pointer, reference, const 여부에 상관 없이 객체보다 더 오래 살 위험이 있음.

## 정리
어떤 객체의 내부요소에 대한 핸들을 반환하는 것은 되도록 피하세요. 캡슐화 정돌르 높이고, 상수 멤버 함수가 객체의 상수성을 유지한 채로 동작할 수 있도록 하며, 무효참조 핸들이 생기는 경우를 최소화할 수 있습니다.

# 항목 29 : 예외 안전성이 확보되는 그날 위해 싸우고 또 싸우자!
```cpp
class PrettyMenu
{
public:
    void changeBackground(std::istream& imgSrc);

private:
    Mutex mutex;
    Image* bgImage;
    int imgaeChanges;
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    lock(&mutex);

    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);

    unlock(&mutex);
}
```

예외 안정성을 위해서는 두가지 요구사항을 맞춰야 한다.
## 자원이 새도록 만들지 않습니다.
위 코드는 자원이 샌다.
<br>
new에서 예외가 발생하면 unlock이 수행되지 않는다.
<br>
자원 관리 객체를 이용하여 해결할 수 있음.

## 자료구조가 더럽혀지는 것을 허용하지 않습니다.
new Image가 예외를 던지면 bgImage가 가리키는 객체는 이미 삭제된 후이고, 새 이미지가 깔리지도 않으면서 imageChange가 증가한다.
<br>
이 부분이 문제

## 예외 안정성을 갖춘 함수의 보장
예외 안정성을 갖는 함수는 아래 3가지 중 1개를 보장한다

### 기본적인 보장
함수 동작 중 예외가 발생하면 실행 중인 프로그램에 관련된 모든 것들을 유효한 상태로 유지하겠다는 보장.

### 강력한 보장
함수 동작 중에 예외가 발생하면, 프로그램의 상태를 절대로 변경하지 않겠다는 보장.
<br>
원자적인 동작을 보장
<br>
예측할 수 있는 프로그램의 상태가 1개 뿐이므로 기본 보장보다 쓰기 쉬움.

### 예외불가 보장
예외를 절대로 던지지 않겠다는 보장.
<br>
기본 타입의 모든 연산은 예외불가를 보장.
```cpp
int doSomething() throw();
```
어떤 예외를 던지지 않게끔 예외 지정된 함수가 예외불가를 보장하지는 않음.
<br>
위 코드는 doSomething이 절대로 예외를 던지지 않겠다는 의미가 아니라, doSomething에 예외가 발생하면 매우 심각한 예외가 발생한 것이므로 unexpected 함수가 호출되어야한다는 것을 의미.

```cpp
class PrettyMenu
{
    ...
    std::shared_ptr<Image> bgImage;
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    Lock m1(&mutex);

    bgImage.reset(new Image(imgSrc));

    ++imageChanges;
}
```
즉, 프로그래머는 위 3개의 보장 중 어떤 보장을 제공할 것인지를 결정해야 한다.
<br>
이전 함수에서 기본적인 보장을 제공하기 위해서는 bgImage를 객체로 지원해야 한다.
<br>
또한, 함수 내의 문장을 재배치해서 배경 그림이 진짜로 바뀌기 전에는 imageChange가 변경되지 않도록 해야한다.

```cpp
struct PMImpl
{
    std::shared_ptr<Image> bgImage;
    int imageChanges;
};

class PrettyMenu
{
    ...

private:
    Mutex mutex;
    std::shared_ptr<PMImpl> pImpl;
};

void PrettyMenu::changeBackground(std::istream& imgSrc)
{
    using std::map;

    Lock m1(&mutex);
    std::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));

    pNew->bgImage.reset(new Image(imgSrc));
    ++pNew->imageChanges;

    swap(pImpl1, pNew);
}
```
강력한 예외 안정성을 보장하기 위한 한가지 방법은 Copy and Swap 방식이 있음.
<br>
객체를 수정하고 싶다면 원본을 복사한 다음, 사본을 수정하고, 예외가 던져지지 않았다면 교체를 하는 작업.
<br>
다만 Copy and Swap은 복사에 시간과 공간이 소모됨.

```cpp
void someFunc()
{
    ...
    f1();
    f2();
    ...
}
```
일반적으로 함수 전체가 강력한 예외 안전성을 갖도록 보장하지는 않음.
<br>
함수 내부에서 호출하는 다른 함수들이 강력한 예외 안전성을 보장하지 못한다면 불가능.

## 예외 안전성을 아예 제공하지 않는 함수
소프트웨어 시스템은 예외 안전성을 갖추었거나 예외 안전성을 갖추지 못한 2가지만 존재한다.
<br>
예외 안전성이 보장되지 않는 함수를 사용하면 시스템 전부 예외로부터 안전하지 않다.
<br>
레거시 코드를 사용하는 경우 많은 부분에서 예외 안전적이지 못하지만, 작성하는 코드 자체는 최대한 예외로부터 안전할 수 있도록 작성해야 하며, 이 부분에 대해 문서로 남겨 나중에도 쉽게 이해할 수 있도록 해야한다.

## 정리
예외 안전성을 갖춤 함수는 실행 중 예외가 발생되더라도 자원을 누출시키지 않으며, 자료구조를 더럽힌 채로 내버려두지 않습니다. 이런 함수들이 제공할 수 있는 예외 안전성 보장은 기본적인 보장, 강력한 보장, 예외 금지 보장이 있습니다.
<br>
강력한 예외 안전성 보장은 'Copy and Swap' 방법을 써서 구현할 수 있지만, 모든 함수에 대해 강력한 보장이 실용적인 것은 아닙니다.
<br>
어떤 함수가 제공하는 예외 안전성 보장의 강도는, 그 함수가 내부적으로 호출하는 함수들이 제공하는 가장 약한 보장을 넘지 않습니다.

# 항목 30 : 인라인 함수는 미주알고주알 따져서 이해해 두자
인라인 함수의 숨겨진 이점 중 하나는 컴파일러의 최적화가 대부분 이어지는 구간에 적용되도록 설계되어있기 때문에 함수 본문에 대해 문맥별 최적화를 걸기 용이해진다는 점.

단, 목적 코드의 크기가 커져서 메모리가 제한된 환경에서는 프로그램의 크기가 컴퓨터에 허용된 공간보다 커질 수 있음. 이는 가상 메모리를 쓰는 환경에서 성능의 걸림돌이 될 수 있음.(페이징, 캐시 적중률 등)

```cpp
class Person
{
public:
    int age() const { return theAge; }  // 암시적 inline 요청

private:
    int theAge;
};

template<typename T>
inline const T& std::max(const T& a, const T& b) { return a < b ? b : a; }  // 명시적 inline 요청
```
inline은 컴파일러에게 요청하는 것이지 명령하는 것이 아님.
<br>
이 요청은 inline이 없어도 자동으로 될 수도 있고, 명시적으로 할 수도 있음.

대부분의 인라인 작업은 컴파일 타임에 수행되므로 인라인 함수는 대체로 헤더 파일에 들어있는 것이 맞음.

inline은 컴파일러 선에서 무시할 수 있는 요청. 루프나 재귀 함수, 가상 함수는 인라인을 무시함.
<br>
인라인이 실패했을 경우 경고 메시지를 내주는 진단 수준 설정 기능이 대부분의 컴파일러에 있음.

```cpp
inline void f() { ... }
void (*pf)() = f;

f();    // inline 수행

pf();   // inline 미수행
```
인라인 함수를 포인터나 주소로 호출할 경우에도 인라인이 수행되지 않음. 즉, 인라인 함수는 어떻게 호출하느냐에 따라 인라인 여부가 결정되기도 함.

```cpp
class Base
{
public:
    ...

private:
    std::string bm1, bm2;
};

class Derived : public Base
{
public:
    Derived() { }

private:
    std::string dm1, dm2, dm3;
};
```
생성자와 소멸자는 인라인하기에 좋지 않음 함수.
<br>
위 Derived의 생성자는 비어있는 것처럼 보여서 인라인하기 좋아 보이지만, 컴파일러가 만드는 코드에 의해 데이터 멤버들의 생성자들이 추가될 수 도 있음.
<br>
마찬가지로 소멸자도 컴파일러에 의해 데이터 멤버들의 소멸자가 추가될 수 있음.

또한 라이브러리의 경우 inline으로 선언된 함수는 내부가 수정된다면 사용자는 자신의 코드를 다시 컴파일해야 함.
<br>
마지막으로 인라인은 디버그를 걸기 힘듬.

즉, 기본적으로 무엇도 인라인하지 말아라.
<br>
정말 단순한 함수만 인라인을 시작하라.
<br>
디버깅하고 싶은 부분에서 디버거를 제대로 쓸 수 있도록 만들어라.
<br>
정말 필요한 위치에 인라인을 추가하라.

## 정리
함수 인라인은 작고, 자주 호출되는 함수에 대해서만 하는 것으로 묶어둡시다. 이렇게하면 디버깅 및 라이브러리의 바이너리 업그레이드가 용이해지고, 자칫 생길 수 있는 코드 부풀림 현상이 최소화되며, 프로그램의 속력이 더 빨라질 수 있는 여지가 최고로 많아집니다.
<br>
함수 템플릿이 대개 헤더 파일에 들아간다는 일반적인 부분만 생각해서 이들을 inline으로 선언하면 안됩니다.

# 항목 31 : 파일 사이의 컴파일 의존성을 최대로 줄이자.
```cpp
class Person
{
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;

private:
    // 이 부분들은 전부 구현 세부사항
    std::string theName;
    Date theBirthDate;
    Address theAddress;
}
```
외부에 공개되지 않은 부분인데도 수정 후 빌드를 하면 다른 부분들이 컴파일 되는 상황이 있음.
<br>
이는 C++의 클래스 정의는 클래스 인터페이스만 지정하는 것이 아니라 구현 세부사항까지 상당히 많이 지정하기 때문에 발생하는 문제
<br>
위 구현 세부사항을 컴파일 하기위해서는 include가 필요함. 이를 통해 컴파일 의존성이 발생

```cpp
class PersonImpl;
class Date;
class Address;

class Person
{
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;

private:
std::shared_ptr<PersonImpl> pImpl;
};
```
pimpl 관용구를 이용하는 방법이 있음.
<br>
이는 정의부에 대한 의존성을 선언부에 대한 의존성으로 바꾸는 방식
<br>
즉, 실용적으로 의미를 갖는 한 자체조잘 형태로 만들고, 정 안되면 다른 파일에 대해 의존성을 갖도록 하되 정의부가 아닌 선언부에 대해 의존성을 갖도록 만드는 것.

### 객체 참조자 및 포인터로 충분한 경우에는 객체를 직접 쓰지 않습니다.
어떤 타입에 대한 참조자 및 포인터를 정의할 때는 그 타입의 선언부만 필요합니다. 반면, 어떤 타입의 객체를 정의할 때는 그 타입의 정의가 필요합니다.

### 할 수 있으면 클래스 정의 대신 클래스 선언에 최대한 의존하도록 만듭니다.
```cpp
class Date;

Date today();
void clearAppointments(Date d);
```
어떤 클래스를 사용하는 함수를 선언할 때는 그 클래스의 정의를 가져오지 않아도 됩니다. 심지어 그 클래스 객체를 값으로 전달하거나 반환하더라도 클래스 정의가 필요 없습니다.
<br>
위 함수는 사용자가 실제로 호출하지 않는다면 Date의 정의가 필요 없음.

### 선언부와 정의부에 대해 별도의 헤더 파일을 제공합니다.
```cpp
#include "datefwd.h

Date today();
void clearAppointments(Date d);
```
선언부를 위한 헤더파일과 정의부를 위한 헤더파일을 따로 분리한다.
<br>
라이브러리 사용자는 전방 선언 대신 선언부 헤더 파일을 include한다.
<br>
라이브러리 제작자는 헤더 파일 두개를 짝지어서 제공한다.

```cpp
#include "Person.h"
#include "PersonImpl.h"

Person::Person(const std::string& name, const Date& birthday, const Address& addr) : pImpl(new PersonImpl(name, birthday, addr)) { ... }

std::string Person::name() const { return pImpl->name(); }
```
pimpl 관용구를 사용하는 Person같은 클래스를 핸들 클래스라고 한다.
<br>
핸들 클래스에 대응되는 구현 클래스 쪽으로 그 함수의 호출을 전달해서 구현 클래스가 실제로 작업을 수행한다.

```cpp
class Person
{
public:
    static std::shared_ptr<Person> create(const std::string& name, const Date& birthday, const Address& addr);

    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
    virtual std::string address() const = 0;
};

class RealPerson : public Person { ... };

std::shared_ptr<Person> Person::create(const std::string& name, const Date& birthday, const Address& addr)
{
    return std::shared_ptr<Person>(new RealPerson(name, birthday, addr));
}

std::shared_ptr<Person> pp(Person::create(name, dateOfbirth, address));
```
인터페이스 클래스를 사용한느 방식도 있음.
<br>
팩토리 함수나 가상 생성자를 이용하여 파생 클래스의 생성자 역할을 대신하는 함수를 만들어 호출하는 방식을 사용.

다만, 실행 비용이 증가하고, 저장 공간이 늘어난다.
<br>
또한 인라인 함수를 적용하기 힘들다.
<br>
즉, 개발 도중에는 핸들 클래스 또는 인터페이스 클래스를 사용하고, 제품 출시가 다가오면 실행 속력이나 파일 크기로 인해 클래스 사이의 결합도를 높이는 방법밖에 없다면 그때 변경하는 방식을 적용해야 한다.

## 정리
컴파일 의존성을 최소화하는 작업의 배경이 되는 가장 기본적인 아이디어는 '정의' 대신에 '선언'에 의존하게 만들자는 것입니다. 이 아이디어에 기반한 두 가지 접근 방법은 핸들 클래스와 인터페이스 클래스입니다.
<br>
라이브러리 헤더는 그 자체로 모든 것을 갖추어야 하며 선언부만 갖고 있는 형태여야 합니다. 이 규칙은 템플릿이 쓰이거나 쓰이지 않거나 동일하게 적용합시다.
