# 개요
생성자는 객체의 메모리를 만드느 데 필요한 과정을 제어하고 객체의 초기화를 맡는 함수
<br>
소멸자는 객체를 없앰과 동시에 그 객체가 메모리에서 적절히 사라질 수 있도록 하는 과정을 제어하는 함수
<br>
대입 연산자는 기존의 객체에 다른 객체의 값을 줄 때 사용하는 함수

# 항목 5 : C++가 은근슬쩍 만들어 호출해버리는 함수들에 촉각을 세우자
생성자, 복사 생성자, 복사 대입 연산자, 소멸자는 프로그래머가 직접 선언하지 않으면 컴파일러가 저절로 선언한다.
<br>
public, inline 함수이다. 소멸자는 기본 클래스가 non-virtual이라면 non-virtual 소멸자이다.

```cpp
template<typename T>
class NamedObject
{
public:
    NamedObject(std::string& name, const T& value);

private:
    std::string& nameValue;
    const T ObjectValue;
};

std::string newDog("Persephone");
std::string oldDog("Satch");

NamedObject<int> p(newDog, 2);
NamedObject<int> s(oldDog, 36);

p = s;
```
복사 대입 연산자는 legal 해야하고 reasonalbe 해야한다.
<br>
위 경우 컴파일러는 컴파일을 거부한다.
<br>
또한 기본 클래스에서 private으로 선언된 복사 대입 연산자의 경우 파생 클래스에서는 암시적 복사 대입 연산자를 가질 수 없음.

## 정리
컴파일러는 경우에 따라 클래스에 대해 기본 생성자, 복사 생성자, 복사 대입 연산자, 소멸자를 암시적으로 만들어 놓을 수 있습니다.

# 항목 6 : 컴파일러가 만들어낸 함수가 필요 없으면 확실히 이들의 사용을 금지해 버리자.
```cpp
class HomeForSale
{
public:
    ...

private:
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeForSale&);
};
```
어떤 클래스에서 특정한 종류의 기능을 지원하지 않았으면 하는 의도를 반영하는 방법은 그런 기능을 제공하는 함수를 선언하지 않는 것.
<br>
하지만 복사 생성자와 복사 대입 연산자는 컴파일러가 대신 선언함으로 이런 방식을 불가능.
<br>
이 방식은 컴파일러가 기본 선언하는 생성자는 public으로 선언된다는 점에서 착안한 방식으로 복사 생성자 및 복사 대입 연산자를 private으로 선언하면 외부로부터의 호출을 차단할 수 있음.
<br>
또한, friend class로부터의 호출도 막기 위해 정의도 하지 않아야 함.

```cpp
class Uncopyable
{
protected:
    Uncopyable() { }
    ~Uncopyable() { }

private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale : private Uncopyable { ... }
```
복사 생성자와 복사 대입 연산자를 private으로 선언한 별도의 기본 클래스에 선언하고, 이를 상속받는 구조로 구현할 경우 링크 시점 에러를 컴파일 시점 에러로 당길 수 있음.
<br>
특이한 점은 public 상속이 아니라 private 상속으로 해도 된다는 점.
<br>
소멸자가 가상 소멸자가 아니어도 된다는 점.
<br>
공백 기본 클래스 최적화(empty base class optimization) 기법의 경우 Uncopyable을 사용하면 다중 상속으로 가게 되면서 최적화가 적용되지 않을 수도 있음.(항목 39, 40 참조)

## 정리
컴파일러에서 자동으로 제공하는 기능을 허용치 않으려면, 대응되는 멤버 함수를 private으로 선언한 후에 구현은 하지 않은 채로 두십시오. Uncopyable과 비슷한 기본 클래스를 쓰는 것도 한 방법입니다.
<br>
modern c++에서는 그냥 delete를 사용.

# 항목 7 : 다형성을 가진 기본 클래스에서는 소멸자를 반드시 가상 소멸자로 선언하자
```cpp
class TimeKeeper
{
public:
    TimeKeeper();
    ~TimeKeeper();

    ...
};

class AtomicClock : public TimeKeeper { ... }
class WaterClock : public TimeKeeper { ... }

TimeKeeper* getTimeKeeper();

TimeKeeper* ptk = getTimeKeeper();
...
delete ptk;
```
getTimeKeeper함수가 반환하는 포인터는 파생 클래스 객체에 대한 포인터이고, 이 포인터가 가리키는 객체가 삭제될 때는 기본 클래스 포인터를 통해 삭제가 된다.
<br>
결정적으로 기본 클래스에 들어있는 소멸자는 non-virtual 소멸자인데 이는 정의되지 않은 행동을 수행한다.

```cpp
class TimeKeeper
{
public:
    TimeKeeper();
    virtual ~TimeKeeper();

    ...
};

class AtomicClock : public TimeKeeper { ... }
class WaterClock : public TimeKeeper { ... }

TimeKeeper* getTimeKeeper();

TimeKeeper* ptk = getTimeKeeper();
...
delete ptk;
```
기본 클래스의 소멸자를 virtual로 선언한다면 이 문제를 해결할 수 있음.
<br>
가상 소멸자를 갖고 있지 않는 클래스는 기본 클래스로 쓰일 의도가 없는 클래스.
<br>
즉, 모든 클래스를 virtual 소멸자로 선언해서도 안된다.
<br>
가상 소멸자로 선언하는 것은 그 클래스에 가상 함수가 하나라도 들어있는 경우에만 한정하는 것이 좋음.
<br>
STL 컨테이너나 std::string은 비가상 소멸자를 갖는 클래스.

```cpp
class AWOV
{
public:
    virtual ~AWOV() = 0;
};

AWOV::~AWOV() { }
```
임의의 클래스가 추상 클래스이길 원하지만 마땅히 넣을 만한 순수 가상 함수가 없는 경우 순수 가상 소멸자를 선언하는 것이 좋음.
<br>
순수 가상 소멸자의 경우 정의가 꼭 필요함.

## 정리
다형성을 가진 기본 클래스에는 반드시 가상 소멸자를 선언해야 합니다. 즉, 어떤 클래스가 가상 함수를 하나라도 갖고 있으면, 이 클래스의 소멸자도 가상 소멸자이어야 합니다.
<br>
기본 클래스로 설계되지 않았거나 다형성을 갖도록 설계되지 않은 클래스에는 가상 소멸자를 선언하지 말아야 합니다.

# 항목 8 : 예외가 소멸자를 떠나지 못하도록 붙들어 놓자.
```cpp
class DBConnection
{
public:
    static DBConnection create();
    void close();
}

class DBConn
{
public:
    ~DBConn()
    {
        db.close();
    }

private:
    DBConnection db;
};
```
만약 소멸자에서 db.close()를 수행하는 과정에서 예외가 발생한다면 어떻게 처리해야 하는가?

### 프로그램을 바로 종료한다.
```cpp
DBConn::~DBconn()
{
    try
    {
        db.close();
    }
    catch(...)
    {
        ... // close 호출이 실패했다는 로그
        std::abort();
    }

}
```
예외를 그대로 흘려보내다가 정의되지 않은 동작을 하기보다는 프로그램을 종료시킨다.

### 예외를 삼킨다
```cpp
DBConn::~DBConn
{
    try
    {
        db.close();
    }
    catch (...)
    {
        ... // close 호출이 실패했다는 로그
    }
}
```
무엇이 잘못되었는지를 알려주는 정보가 묻혀 버리므로 별로 좋은 선택은 아님.
<br>
대신 프로그램이 신뢰성 있게 실행을 지속할 수 있어야 함.

### 소멸자가 아닌 시점에서 사용자가 직접 제어할 수 있도록 한다
```cpp
class DBConn
{
public:
    void close()
    {
        db.close();
        closed = true;
    }

    ~DBConn()
    {
        if(!closed)
        {
            try
            {
                db.close();
            }
            catch(...)
            {
                ... // close 호출이 실패했다는 로그
                ...
            }
        }
    }

private:
    DBConnection db;
    bool closed;
};
```
어떤 동작이 예외를 일으키면서 실패할 가능성이 있고, 또 그 예외를 처리해야 할 필요가 있따면 그 예외는 소멸자가 아닌 다른 함수에서 비롯된 것이어야 한다.

## 정리
소멸자에서는 예외가 빠져나가면 안됩니다. 만약 소멸자 안에서 호출된 함수가 예외를 던질 가능성이 있다면, 어떤 예외이든지 소멸자에서 모두 받아낸 후에 삼켜 버리든지 프로그램을 끝내든지 해야합니다.
<br>
어떤 클래스의 연산이 진행되다가 던진 예외에 대해 사용자가 반응해야 할 필요가 있다면, 해당 연산을 제공하는 함수는 반드시 보통의 함수(즉, 소멸자가 아닌 함수)이어야 합니다.

# 항목 9 : 객체 생성 및 소멸 과정 중에는 절대로 가상 함수를 호출하지 말자.
기본 클래스 생성자는 파생 클래스 생성자보다 먼저 실행되기 때문에, 기본 클래스 생성자가 돌아가고 있는 시점에 파생 클래스 데이터 멤버는 아직 초기화 되지 않았음.
<br>
기본 클래스 생성자에서 호출된 가상 함수가 파생 클래스쪽으로 내려간다면 정의되지 않은 행동을 함.

파생 클래스 객체의 기본 클래스 부분이 생성되는 동안에는, 그 객체의 타입은 기본 클래스. 호출되는 가상 함수, 런타임 타입 정보 모두 기본 클래스로 취급.

```cpp
class Transaction
{
public:
    Transaction() { init(); }

    virtual void logTransaction() const = 0;

private:
    void init()
    {
        ...
        logTransaction();
    }
};
```
위와 같이 간접적으로 virtual 함수를 생성자에서 호출하지 않는지에 대해 주의해야 함.
<br>
생성자와 소멸자에서 가상 함수 호출을 피하고 필요한 정보는 매개변수를 통해 올려줘야 함.
<br>
파생 클래스에서 static helper function을 이용하여 매개변수를 올려준다면 파생 클래스의 미초기화된 데이터를 건드릴 위험도 줄어든다.

## 정리
생성자 혹은 소멸자 안에서 가상 함수를 호출하지 마세요. 가상 함수라고 해도, 지금 실행중인 생성자나 소멸자에 해당하는 클래스의 파생 클래스 쪽으로 내려가지 않으니까요.

# 항목 10 : 대입 연산자는 *this의 참조자를 반환하게 하자.
```cpp
int x, y, z;
x = y = z = 15;
```
대입 연산은 여러개가 사슬처럼 엮일 수 있는데, 우측 연관 연산이다. 즉, 오른쪽부터 대입이 수행된다.
<br>
위 코드처럼 수행되기 위해서는 대입 연산자는 좌변 인자에 대한 참조자를 반환해야 한다.

```cpp
class Widget
{
public:
    Widget& operator=(const Widget& rhs)
    {
        ...
        return *this;
    }
    Widget& operator+=(const Widget& rhs)
    {
        ...
        return *this;
    }
    Widget& operator=(int rhs)
    {
        ...
        return *this;
    }
};
```
물론 관례같은 거라 지키지 않더라도 컴파일이 안되거나 하지는 않는다.

## 정리
대입 연산자는 *this의 참조자를 반환하도록 만드세요.

# 항목 11 : operator=에서는 자기대입에 대한 처리가 빠지지 않도록 하자.
```cpp
class Wiget { ... }

Widget w;
...
w = w;

int i = 0;
...
int j = 0;
...
a[i] = a[j]

*px = *py;

class Base { ... }
class Derived : public Base { ... }
void doSomething(const Base& rb, Derived* pd);
```
같은 타입으로 만들어진 객체 여러개를 참조자 혹은 포인터로 물어놓고 동작하는 코드를 작성함으로써 다양한 부분에서 자기 대입의 가능성이 존재함.

```cpp
class Bitmap { ... }

class Widget
{
public:
    Widget& Widget::operator=(const Widget& rhs)
    {
        delete pb;
        pb = new Bitmap(*rhs.pb);
        return *this;
    }

private:
    Bitmap* pb;
}
```
위 경우 자기 대입으로 인해 해제된 객체를 참조할 수 있음.

```cpp
class Bitmap { ... }

class Widget
{
public:
    Widget& Widget::operator=(const Widget& rhs)
    {
        Bitmap* pOrig = pb;
        pb = new Bitmap(*rhs.pb);
        delete pOrig;

        return *this;
    }

private:
    Bitmap* pb;
}
```
간단한 일치성 검사를 통해서도 해결할 수 있지만, 위 코드는 예외가 발생할 수도 있기 떄문에 이 부분도 고려하여 작성.
<br>
swap을 이용하는 방식으로도 구현 가능

## 정리
operator=을 구현할 때, 어떤 객체가 그 자신에게 대입되는 경우를 제대로 처리하도록 만듭시다. 원본 객체와 복사대상 객체의 주소를 비교해도 되고, 문장의 순서를 적절히 조정할 수도 있으며, 복사 후 맞바꾸기 기법을 써도 됩니다.
<br>
두 개 이상의 객체에 대해 동작하는 함수가 있다면, 이 함수에 넘겨지는 객체들이 사실 같은 객체인 경우에 정확하게 동작하는지 확인해 보세요.

# 항목 12 : 객체의 모든 부분을 빠짐없이 복사하자.
```cpp
void logCall(const std::string& funcName);

class Customer
{
public:
    Customer(const Customer& rhs);
    Customer& operator=(const Customer& rhs);

private:
    std::string name;
};

Customer::Customer(const Customer& rhs) : name(rhs.name)
{
    logCall("Customer copy constructor");
}

Customer& Customer::operator=(const Customer& rhs)
{
    logCall("Customer copy assignment operator");

    name = rhs.name;

    return *this;
}
```
위 상황은 적절한 복사 연산을 구현했지만, 만약 이후에 변수가 추가되더라도 컴파일러는 이를 경고하지 않는다.
<br>
즉, 생성자, operator=를 모두 수동으로 갱신해야 한다.

```cpp
class PriorityCustomer : public Customer
{
public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);

private:
    int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs) 
: Customer(rhs)
, priority(rhs.priority)
{
    logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
    logCall("PriorityCustomer copy assignment operator");
    Customer::operator=(rhs);
    priority = rhs.priority;
    return *this;
}
```
파생 클래스의 복사 연산자는 기본 클래스의 복사 연산을 명시적으로 수행해야 한다.

즉, 해당 클래스의 데이터 멤버를 모두 복사하고, 이 클래스가 상속한 기본 클래스의 복사 함수도 명시적으로 호출해야 한다.
<br>
만약 두 함수의 내용이 반복된다면 init류의 private함수를 추가하여 코드를 옮기는 것을 추천.

## 정리
객체 복사 함수는 주어진 객체의 모든 데이터 멤버 및 모든 기본 클래스 부분을 빠뜨리지말고 복사해야 합니다.
<br>
클래스의 복사 함수 두 개를 구현할 때, 한쪽을 이용해서 다른 쪽을 구현하려는 시도는 절대로 하지 마세요. 그 대신, 공통된 동작을 제 3의 함수에다 분리해 놓고 양쪽에서 이것을 호출하게 만들어서 해결합시다.
