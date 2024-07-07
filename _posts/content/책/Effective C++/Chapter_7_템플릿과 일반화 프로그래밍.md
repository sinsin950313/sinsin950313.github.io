# 개요
템플릿은 처음에는 범용 컨테이너를 지원할 용도로 제작되었지만 최근에는 튜링 완전하다는 것까지 밝혀지면서 템플릿 메타 프로그래밍이라는 영역까지 생겨났다.
<br>
최근에는 템플릿에서 컨테이너가 차치자는 부분이 매우 작음.
<br>
그러나 템플릿 기반 프로그래밍의 몇 개 되지않는 핵심 아이디어는 견고히 자리를 하고 있음.

# 항목 41 : 템플릿 프로그래밍의 천릿길도 암시적 인터페이스와 컴파일 타임 다형성부터
객체 지향 프로그래밍의 세계를 작동시키는 축이 명시적 인터페이스와 런타임 다형성이라면 템플릿 프로그래밍의 세계를 작동시키는 축은 암시적 인터페이스와 컴파일 타임 다형성이다.

```cpp
class Widget
{
public:
    Widget();
    virtual ~Widget();

    virtual std::size_t size() const;
    virtual void normalize();
    void swap(Widget& other);
};

void doProcessing(Widget& w)
{
    if(w.size() > 10 && w != someNastyWidget)
    {
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```
w는 Widget 타입으로 선언되었기 때문에, w는 Widget 인터페이스를 지원해야 함. 이 인터페이스를 소스 코드(header 파일)에서 찾으면 어떤 형태인지 확인할 수 있으므로, 이런 인터페이스를 가리켜 명시적 인터페이스라 함. 즉, 소스 코드에 명시적으로 드러나는 인터페이스를 의미
<br>
Widget의 멤버 함수 중 몇 개는 가상 함수이므로, 이 가상 함수에 대한 호출은 런타임 다형성에 의해 이루어짐. 즉, 특정한 함수에 대한 실제 호출은 w의 동적 타입을 기반으로 런타임에 결정됨.

```cpp
template<typename T>
void doProcess(T& w)
{
    if(w.size() > 10 && w != someNastyWidget)
    {
        T temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```
w가 지원해야 하는 인터페이스는 템플릿 안에서 w에 대해 실행되는 연산이 결정. 이 템플릿이 제대로 컴파일 되기 위해서는 몇 개의 표현식이 유효해야하는데, 이 표현식들은 바로 T가 지원해야 하는 암시적 인터페이스를 의미.
<br>
w가 수반되는 함수 호출이 일어날 때, 해당 호출을 성공시키기 위해 템플릿의 인스턴스화가 일어남. 이러한 인스턴스화가 일어나는 시점은 컴파일 타임으로 이를 가리켜 컴파일 타임 다형성이라고 함.

명시적 인터페이스란 함수의 시그니처로 이루어짐. 함수의 이름, 매개변수 타입, 반환 타입 typedef 등을 통칭. public 데이터 멤버는 포함되지 않음.
<br>
암시적 인터페이스란 함수 시그니처가 아닌 유효 표현식에 기반함. 이는 함수 수행 과정에서 암시적 casting이 되더라도 유효하기만 하면 상관 없음.

## 정리
클래스 및 템플릿은 모두 인터페이스와 다형성을 지원합니다.
<br>
클래스의 경우, 인터페이슨느 명시적이며 함수의 시그니처를 중심으로 구성되어 있습니다. 다형성은 프로그램 실행 중에 가상 함수를 통해 나타납니다.
<br>
템플릿 매개변수의 경우, 인터페이스는 암시적이며 유효 표현식에 기반을 두어 구성됩니다. 다형성은 컴파일 중에 템플릿 인스턴스화와 함수 오버로딩 모호성 해결을 통해 나타납니다.

# 항목 42 : typename의 두 가지 의미를 제대로 파악하자
```cpp
template<class T> class Widget;
template<typename T> class Widget;
```
위 두 코드는 동일하다.

```cpp
template<typename C>
void print2nd(const C& container)
{
    if(container.size() >= 2)
    {
        typename C::const_iterator iter(container.begin());

        ++iter;
        int value = *iter;
        std::cout << value;
    }
}
```
C::const_iterator는 템플릿 매개변수인 C에 따라 달라지는 타입으로 이렇게 템플릿 매개변수에 종속된 것을 의존 이름(Dependent Name)이라고 한다.
<br>
value는 int 타입으로 이를 비의존 이름(Non-Dependent Name)이라고 한다.
<br>
문제는 C::const_iterator라는 static 변수일 가능성도 존재한다는 것.
<br>
typename을 붙임으로써 컴파일러에게 C::const_iterator가 타입이라는 것을 알림. 이 부분이 class와 typename의 차이.

```cpp
template<typename T>
class Derived : public Base<T>::Nested
{
public:
    explicit Derived(int x) : Base<T>::Nested(x)
    {
        typename Base<T>::Nested temp;
    }
};
```
단, 중첩 의존 타입이 기본 클래스의 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로서 있을 경우에는 typename을 붙이면 안됨.

```cpp
template<typename IterT>
void workWithIterator(IterT iter)
{
    typename std::iterator_traits<IterT>::value_type temp(*iter);

    typedef typename std::iterator_traits<IterT>::value_type value_type;
    value_type temp(*iter);
}
```
C++ 표준의 특성 정보(traits)를 이용하여 IterT 타입이 가리키는 타입의 객체로 가리키는 대상의 타입을 임시 변수로 초기화.
<br>
특성정보 클래스에 속한 value_type같은 멤버 이름에 대한 typedef 이름을 만들 때는 그 멤버 이름과 똑같이 짓는 것이 관례
<br>
마지막으로 컴파일러에 따라 typename을 받아들이는 방식이 차이가 있을 수 있음. 즉, 프로그램 이식 과정에서 골치가 아플 수 있음.

## 정리
템플릿 매개변수를 선언할 때, class 및 typename은 서로 바꾸어 써도 무방합니다.
<br>
중첩 의존 타입 이름을 식별하는 용도에는 반드시 typename을 사용합니다. 단, 중첩 의존 이름이 기본 클래스 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우에는 예외입니다.

# 항목 43 : 템플릿으로 만들어진 기본 클래스 안의 이름에 접근하는 방법을 알아두자
```cpp
class CompanyA
{
public:
    void sendClearText(const std::string& msg);
    void sendEncrypted(const std::string& msg);
};

class CompanyB
{
public:
    void sendClearText(const std::string& msg);
    void sendEncrypted(const std::string& msg);
};

class MsgInfo { ... };

template<typename Company>
class MessageSender
{
public:
    void sendClear(const MsgInfo& info)
    {
        std::string msg;

        ... // info로부터 msg 생성

        Company c;
        c.sendClearText(msg);
    }

    void sendSecret(const MsgInfo& info) { ... }
}

template<typename Company>
class LoggingMsgSender : public MsgSender<Company>
{
public:
    void sendClearMsg(const MsgInfo& info)
    {
        ... // 메시지 전송 전 정보를 로그에 기록

        sendClear(info);

        ... // 메시지 전송 후 정보를 로그에 기록
    }
};
```
위 코드는 Company가 무엇인지 알 수 없는 상황이므로 MsgSender<Company\>가 어떤 형태인지 알 수 없기 때문에 어떤 클래스로부터 상속을 받는지 알 수 없으므로 sendClear 함수를 찾을 수 없음.

```cpp
class CompanyZ
{
public:
    void sendEncrypted(const std::string& msg);
};

template<>
class MsgSender<CompanyZ>
{
public:
    void sendSecret(const std::string& info);
};

template<typename Company>
class LoggingMsgSender : public MsgSender<Company>
{
public:
    void sendClearMsg(const MsgInfo& info)
    {
        ... // 메시지 전송 전 정보를 로그에 기록

        sendClear(info);

        ... // 메시지 전송 후 정보를 로그에 기록
    }
}
```
template<\>는 템플릿 특수화로 템플릿 매개변수가 CompanyZ일 때만 쓸 수 있는 특수한 버전.
<br>
즉, 위 코드는 CompanyZ에 대해서 사용되는 구체적인 타입이 결정된 상태.
<br>
그러나 LoggingMsgSender는 여전히 Company가 CompanyA, B인지 CompanyZ인지는 알 수 없음.
<br>
즉, 템플릿은 언제나 특수화될 수 있고, 특수화 버전 인터페이스가 항상 일반형 템플릿과 같으리란 법이 없으므로 C++ 컴파일러는 템플릿으로 만들어진 기본 클래스를 뒤져서 상속된 이름을 찾는 것을 거부함.

```cpp
template<typename Company>
class LoggingMsgSender : public MsgSender<Company>
{
public:
    using MsgSender<Company>::sendClear;

    void sendClearMsg(const MsgInfo& info)
    {
        ...
        this->sendClear(info);
        MsgSender<Company>::sendClear(info);
        ...
    }
}
```
해결책은 명시적으로 함수를 호출하는 것.
<br>
첫번째는 this를 사용하여 호출하는 방법.
<br>
두번째는 using을 사용하여 기본 클래스의 유효 범위를 찾아보라고 알려주는 방법
<br>
세번째는 호출할 함수가 기본 클래스의 함수라는 점을 명시적으로 지정하는 방법. 단, 이 경우 호출되는 함수가 가상 함수라면 가상 함수 바인딩이 무시되는 단점이 있음.

```cpp
LoggingMsgSender<CompanyZ> zMsgSender;
MsgInfo msgData;
...
zMsgSender.sendClearMsg(msgData);
```
단, 실제 인스턴스화 과정에서 문제가 발생한다면 컴파일이 불가능.

## 정리
파생 클래스 템플릿에서 기본 클래스 템플릿의 이름을 참조할 때는, "this->"를 접두사로 붙이거나 기본 클래스 한정문을 명시적으로 써 주는 것으로 해결합시다.

# 항목 44 : 매개변수에 독립적인 코드는 템플릿으로부터 분리시키자
템플릿이 유용하고 편리한 것은 맞지만 아무 생각 없이 사용하다보면 코드 비대화가 초래될 수 있음.

## 공통성 및 가변성 분석
함수에서 공통적인 부분을 별도로 빼내어 다른 함수로 만들고 해당 함수를 호출하는 방식으로 제작하는 방식.
<br>
또는 공통 클래스를 새로운 클래스로 옮긴 후 해당 클래스를 상속 또는 합성하는 방식.
<br>
단, 템플릿은 코드 중복이 암시적이지만, 소스코드는 코드 중복이 명시적이라는 것에 유의해야 한다.

```cpp
template<typename T, std::size_t n>
class SquareMatrix
{
public:
    void invert();
};

SqaureMatrix<double, 5> sm1;
sm1.invert();

SquareMatrix<double, 10> sm2;
sm2.invert();
```
두 객체는 행렬 크기를 나타내는 상수만 제외하면 동일한 함수.

```cpp
template<typename T>
class SquareMatrixBase
{
protected:
    void invert(std::size_t matrixsize);
};

template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T>
{
private:
    using SquareMatrixBase<T>::invert;

public:
    void invert() { this->invert(n); }
};
```
행렬의 크기를 매개변수로 받도록 invert 함수가 분리되어 변경됨.
<br>
같은 타입의 객체를 원소로 갖는 모든 정방행렬이 동일한 SquareMatrixBase 클래스를 공유.
<br>
또한, 코드 복제를 피하는 목적이므로 public이 아니라 protected로 선언.
<br>
기본 클래스를 사용한 것은 파생 클래스의 구현을 돕기 위한 것이므로 private 상속을 한 것에 주의.
<br>
즉, is-a관계가 아님.

```cpp
template<typename T>
class SquareMatrixBase
{
protected:
    SquareMatrixBase(std::size_t n, T* pMem) : size(n), pData(pMem) { ... }

    void setDataPtr(T* ptr) { pData = ptr; }

private:
    std::size_t size;
    T* pData;
};

template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T>
{
public:
    SquareMatrix() : SquareMatrixBase<T>(n, data) { ... }

private:
    T data[n * n];
};

template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T>
{
public:
    SquareMatrix() : SquareMatrixBase<T>(n, 0), pData(new T[n * n]) { this->setDataPtr(pData.get()); }

private:
    boost::scoped_array<T> pData;
};
```
행렬 값들을 담는 메모리에 대한 포인터를 저장하는 방식
<br>
행렬의 크기가 미리 저장된 상태로 정의된 방식은 최적화 등에서 좋은 효과를 볼 수 있음.
<br>
행렬의 크기를 매개변수로 넘기거나 객체에 저장된 형태로 다른 파생 클래스들에서 공유하는 방식. 이 방식의 경우 실행 코드의 크기가 작아지면서 작업 세트 크기가 줄어들고, 이로 인해 명령어 캐시 내의 참조 지역성이 향상될 수 있음.

또한 invert 비슷하게 독립형 버전 함수를 아무 생각 없이 기본 클래스로 옮기다가는 기본 클래스의 크기가 커지는 문제가 있음.
<br>
또는 int와 long형의 동일성으로 인해 불필요한 코드 비대화가 발생할수도 있음.
<br>
이러한 문제를 해결하기 위해 void*를 사용하는 방법도 있음.

## 정리
템플릿을 사용하면 비슷비슷한 클래스와 함수가 여러 벌 만들어집니다. 따라서 템플릿 매개변수에 종속되지 않은 템플릿 코드는 비대화의 원인이 됩니다.
<br>
비타입 템플릿 매개변수로 생기는 코드 비대화의 경우, 템플릿 매개변수를 함수 매개변수 혹은 클래스 데이터 멤버로 대체함으로써 비대화를 종종 없앨 수 있습니다.
<br>
타입 매개변수로 생기는 코드 비대화의 경우, 동일한 이진 표현구조를 가지고 인스턴스화되는 타입들이 한가지 함수 구현을 공유하게 만듦으로써 비대화를 감소시킬 수 있습니다.

# 항목 45 : 호환되는 모든 타입을 받아들이는 데는 멤버 함수 템플릿이 직방
```cpp
template<typename T>
class SmartPtr
{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other) : heldPtr(other.get()) { ... }

    T* get() const { return heldPtr; }

    SmartPtr(const SmartPtr<T>& r); // 복사 생성자

    template<typename U>
    SmartPtr(const SmartPtr<U>& r); // 일반화 복사 생성자

    SmartPtr<T>& operator=(const SmartPtr& r);  // 복사 대입 연산자

    template<typename U>
    SmartPtr& operator=(const SmartPtr<U>& r);  // 일반화 복사 대입 연산자

private:
    T* heldPtr;
};
```
스마트 포인터는 일반 포인터에서는 지원하지 않는 다양한 기능을 지원하지만, 반대로 일반 포인터도 스마트포인터에서 지원하지 않는 기능을 많이 지원함. 대표적으로 암시적 변환이 있음.
<br>
포인터간의 계층간 암시적 변환은 가능하지만, 스마트 포인터간의 계층간 암시적 변환은 불가능.
<br>
이를 지원하기 위해 멤버 함수 템플릿을 통해 계층간의 생성자 함수를 만들어냄.
<br>
위 코드는 모든 T 타입 및 모든 U 타입에 대해서 SmartPtr<T\> 객체가 SmartPtr<U\>로부터 생성될 수 있다는 것을 의미. 이러한 꼴의 생성자를 일반화 복사 생성자(Generalized Copy Constructor)라고 부름.
<br>
T* 타입의 포인터를 U*의 포인터로 초기화함으로써 암시적 변환이 가능할 때만 컴파일 에러가 발생하지 않음.
<br>
단, 프로그래머가 선언하지 않는 생성자 중 컴파일러가 자동으로 생성하는 규칙은 유효하므로 복사 대입 연산자와 일반화 복사 생성자도 명시적으로 선언해야 한다.


## 정리
호환되는 모든 타입을 받아들이는 멤버 함수를 만들려면 멤버 함수 템플릿을 사용합시다.
<br>
일반화된 복사 생성 연산과 일반화된 대입 연산을 위해 멤버 함수 템플릿을 선언했다 하더라도, 보통으 ㅣ복사 생성자와 복사 대입 연산자는 여전히 직접 선언해야 합니다.

# 항목 46 : 타입 변환이 바람직할 경우에는 비멤버 함수를 클래스 템플릿 안에 정의해두자.
```cpp
template<typename T>
class Rational
{
public:
    Rational(const T& numerator = 0, const T& denominator = 0);

    const T numerator() const;
    const T denominator() const;
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs);

Rational<int> oneHalf(1, 2);
Rational<int> result = oneHalf * 2; // 컴파일 에러
```
켐플릿 인자 추론 과정에서는 암시적 타입 변환이 고려되지 않기 때문에 마지막 구문은 컴파일 에러를 발생시킨다.

```cpp
template<typename T>
class Rational
{
public:
    friend Rational operator*(const Rational& lhs, const Rational& rhs)
    {
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }
};

// 시험 결과 이 부분 필요 없음.
// 또는 위 friend함수에서 이 함수를 호출하는 방식으로 변경

// https://godbolt.org/z/8Ge8va4T3

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs);
```
클래스 템플릿 안에 프렌드 함수를 넣어두면 함수 템플릿으로서의 성격을 주지 않고 특정한 함수 하나를 나타낼 수 있음.
<br>
위 코드의 마지막 구문은 oneHalf시점에 템플릿 인자 추론이 끝나서 함수 호출 시점이므로 암시적 타입 변환이 가능.
<br>
단, 선언부에 함수의 정의를 추가하지 않으면 링크 에러 발생.

## 정리
모든 매개변수에 대해 암시적 타입 변환을 지원하는 템플릿과 관계가 있는 함수를 제공하는 클래스 템플릿을 만들려고 한다면, 이런 함수는 클래스 템플릿 안에 프렌드 함수로서 정의합시다.

# 항목 47 : 타입에 대한 정보가 필요하다면 특성정보 클래스를 사용하자.
특성 정보란 C++에 미리 정의된 문법구조나 키워드가 아니라 C++ 프로그래머들이 따르는 구현 기법 또는 관례
<br>
특성 정보는 기본 제공 타입과 사용자 정의 타입에서 모두 작동해야 한다. 즉, 어떤 타입 내에 중첩된 정보를 넣는 방식으로는 구현할 수 없음.

```cpp
template<typename IterT>
struct iterator_traits
{
    typedef typename IterT::iterator_catetory iterator_category;
};

template<...>
class deque
{
public:
    typedef random_access_iterator_tag iteartor_category;
    ...
};
```
대신, 해당 특성 정보를 템플릿 및 그 템플리싀 1개 이상의 특수화 버전을 넣는 것.
<br>
위 구조체를 특성정보 클래스라고 부름
<br>
위 방식은 포인터에 대해서는 작동하지 않음.

```cpp
template<typename IterT>
struct iterator_traits<IterT*>
{
    // random_access_iterator_tag?
    typedef random_access_iterator_tag iterator_category;
};
```
포인터를 지원하기 위해 부분 템플릿 특수화 버전을 제공

## 특성 정보 클래스의 설계 및 구현 방법
1. 다른 사람이 사용하도록 열어주고 싶은 타입 관련 정보를 확인합니다.(iterator_category)
2. 그 정보를 식별하기 위한 이름을 선택합니다.(iterator_traits)
3. 지원하고자 하는 타입 관련 정보를 담은 템플릿 및 그 템플릿의 특수화 버전을 제공합니다.(iterator_traits)

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    if(typeid(typename std::iterator_traits<IterT>::iterator_category) == typeid(std::random_access_iterator_tag))
    { ... }
}
```
IterT는 컴파일 타임에 결정되지만 if구문은 런타임에 분기되므로 시간낭비가 발생한다.

```cpp
template<typename IterT, typename DistT>
void doAdvance(Iter& iter, DistT d, std::random_access_iterator_tag)
{
    iter += d;
}

template<typename IterT, typename DistT>
void doAdvance(Iter& iter, DistT d, std::bidirectional_iterator_tag)
{
    if(d >= 0)
    {
        while (d--)
        {
            ++iter;
        }
    }
    else
    {
        while(d++)
        {
            --iter;
        }
    }
}

template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
}
```
대신 오버로딩을 통해 컴파일 타임에 결정할 수 있음.

## 특성 정보 클래스를 사용하는 방법
1. Worker 역할을 맡을 함수 혹은 함수 템플릿(doAdvance)을 특성정보 매개변수를 다르게 하여 오버로딩 합니다. 그리고 전달되는 해당 특성 정보에 맞추어 각 오버로드 버전을 구현합니다.
2. Worker를 호출하는 Master 역할을 맡을 함수 혹은 함수 템플릿(advance)을 만듭니다. 이때 특성정보 클래스에서 제공되는 정보를 넘겨서 작업자를 호출하도록 구현합니다.

iterator_traits이외에도 value_type, char_tratis, numeric_limits, is_fundamental<T>, is_array<T>, is_base_of<T1, T2\> 등이 있음

## 정리
특성정보 클래스는 컴파일 도중에 사용할 수 있는 타입 관련 정보를 만들어냅니다. 또한 특성정보 클래스는 템플릿 및 템플릿 특수 버전을 사용하여 구현합니다.
<br>
함수 오버로딩 기법과 결합하여 특성정보 클래스를 사용하면, 컴파일 타임에 결정되는 타입별 if, else 점검문을 구사할 수 있습니다.

# 항목 48 : 템플릿 메타 프로그래밍, 하지 않겠는가?
템플릿 메타 프로그래밍(TMP)란 컴파일 도중에 실행되는 템플릿 기반의 프로그램을 작성하는 일을 의미.
<br>
TMP 프로그램이 실행을 마친 결과로 나온 출력물을 컴파일하는 과정을 거침.
<br>
TMP는 다른 방법으로는 까다롭거나 불가능한 일을 쉽게 처리할 수 있고, 컴파일 타임동안 진행되기 때문에 런타임 영역에서 컴파일 타임 영역으로 작업을 전환할 수 있음(런타임 에러를 컴파일 타임에 잡기 등), 또한 실행 코드가 작아지고 실행 시간이 짧아지면서 메모리는 적게 잡아먹음(대신 컴파일 타임이 길어짐).

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    if(typeid(typename std::iterator_traits<IterT>::iterator_category) == typeid(std::random_access_iterator_tag))
    { ... }
}

template<typename IterT, typename DistT>
void doAdvance(Iter& iter, DistT d, std::random_access_iterator_tag)
{
    iter += d;
}

->

void advance(std::list<int>::iterator& iter, int d)
{
    if(typeid(typename std::iterator_traits<std::list<int>::iterator>::iterator_category) == typeid(std::random_access_iterator_tag))
    {
        iter += d;  // 컴파일 에러
    }
    ...
}

std::list<int>::iterator iter;
advance(iter, 10);
```
이전에 작성한 코드는 실제로는 해당 줄에 들어가지 않더라도 컴파일 과정에서 컴파일 에러가 발생한다.

```cpp
template<unsigned n>
struct Factorial
{
    enum { value = n * Factorial<n - 1>::value; }
};

template<>
struct Factorial<0>
{
    enum { value = 1; }
};

std::cout << Factorial<5>::value;
```
TMP는 자체적으로 튜링 완전성을 갖는다.
<br>
TMP의 루프는 재귀식 템플릿 인스턴스화를 통해 구현한다.
<br>
위 코드는 매개변수로 들어가는 값을 줄여가며 템플릿 인스턴스를 만든다.

## TMP로 할 수 있는 것들
### 치수 단위(dimensional unit)의 정확성 확인
프로그램 안에서 쓰이는 모든 치수 단위의 조합이 제대로 됐는지 컴파일 타임에 확인할 수 있음.

### 행렬 연산의 최적화
```cpp
typedef SquareMatrix<double, 10000> BigMatrix;
BigMatrix m1, m2, m3, m4, m5;

BigMatrix result = m1 * m2 * m3 * m4 * m5;
```
일반적인 코드는 위 구문의 비싼 연산을 런타임에 처리해야 하지만 TMP를 사용하면 컴파일 타임에 미리 연산을 처리할 수 있음.
<br>
또한 표현식 템플릿(expression template)을 사용한다면 임시 객체를 만들지 않고 루프도 합칠 수 있음.

### 맞춤식 디자인 패턴 구현의 생성
TMP를 사용한 프로그래밍 기술인 정책 기반 설계(policy-based design)이라는 거슬 사용하면 따로 마련된 설계상의 선택(policy)를 나타내는 템플릿을 만들어낼 수 있음.
<br>
이렇게 만들어진 정책 템플릿은 서로 임의대로 조합되어 사용자의 취향에 맞는 동작을 갖는 패턴으로 구현되는 곳에 쓰임.
<br>
예를 들어 스마트포인터 동작 정책을 하나씩 구현한 템플릿 타입을 만들고 사용자가 마음대로 조합하는 방식.
<Br>
생성식 프로그래밍(generated programming)

## 정리
템플릿 메타 프로그래밍은 기존 작업을 런타임에서 컴파일 타임으로 전환하는 효과를 냅니다. 따라서 TMP를 쓰면 선행 에러 탐지와 높은 런타임 효율을 손에 거머쥘 수 있습니다.
<br>
TMP는 정책 선택의 조합에 기반하여 사용자 정의 코드를 생성하는데 쓸 수 있으며, 또한 특정 타입에 대해 부적절한 코드가 만들어지는 것을 막는데도 쓸 수 있습니다.
