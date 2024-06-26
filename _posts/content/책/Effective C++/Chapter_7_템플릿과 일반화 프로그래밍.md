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
