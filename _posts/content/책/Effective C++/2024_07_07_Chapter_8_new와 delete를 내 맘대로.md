# 개요
이번 장은 new 처리자와 메모리 할당 및 해제 루틴에 대한 이야기를 다룸
<br>
힙은 다중 스레드 시스템에서 수정 가능한 전역 자원으로 분류된다.
<br>
수정 가능한 정적 데이터는 적절한 동기화를 걸지 않으면 스레드 잠금, 동시 접근 방지 등이 효과가 없어진다.

# 항목 49 : new 처리자의 동작 원리를 제대로 이해하자.
사용자가 보낸 메모리 할당 요청을 operator new 함수가 맞추어 주지 못할 경우에(즉, 할당할 메모리가 없는 경우)는 예외를 던지거나 nullptr을 반환하기로 되어있음.

```cpp
namespace std
{
    typedef void (*new_handler)();

    new_handler set_new_handler(new_handler p) throw();
}
```
new가 예외를 던지기 전에 사용자 쪽에서 지정할 수 있는 예외 처리 함수를 new 처리자(new handler, 할당 에러 처리자)라고 한다.
<br>
set_new_handler가 받아들이는 new_handler타입의 매개변수는 요구된 메모리를 operator new가 할당하지 못했을 때 operator new가 호출할 함수의 포인터.
<br>
사용자가 부탁한 만큼의 메모리를 할당해 주지 못하면, operator new는 충분한 메모리를 찾아낼 때까지 new 처리자를 되풀이해서 호출.

## new_handler가 처리해야 하는 작업
### 사용할 수 있는 메모리를 더 많이 확보합니다.
operator new가 시도하는 이후의 메모리 확보가 성공할 수 있도록 하는 방법.
<br>
프로그램 시작 시 메모리 블록클 크게 하나 할당해 놓았다가 new 처리자가 가장 처음 호출될 때 그 메모리를 쓸 수 있도록 허용하는 방법이 있음

### 다른 new 처리자를 설치합니다
현재 new handler 내부에서 set_new_handler를 호출하여 다른 new_handler를 등록하는 방법.
<br>
현재 new 처리자에서 가용 메모리를 확보할 수 없을 경우 사용.
<br>
new 처리자의 동작을 조정하는 데이터를 정적 데이터 또는 네임스페이스 유효 범위 안의 데이터, 전역 데이터로 마련해둔 후 new 처리자가 이 데이터를 수정하게 만드는 방법.

### new 처리자 설치를 제거합니다
set_new_handler에 nullptr을 넘기는 방법.
<br>
operator new는 예외를 던질 것이다.

### 예외를 던진다
bad_alloc 또는 bad_alloc에서 파생된 타입의 예외를 던진다.

### 복귀하지 않습니다.
abort 또는 exit을 호출

## 할당되는 객체의 클래스 타입에 따라 메모리 할당 실패에 대한 다른 처리를 구현하는 방법.
```cpp
class X
{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw()
    {
        std::new_handler oldHandler = currentHandler;
        currentHandler = p;
        return oldHandler;
    }
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void outOfMemory();

private:
    static std::new_handler currentHandler;
};

class Y
{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void outOfMemory();

private:
    static std::new_handler currentHandler;
};

X* p1 = new X;  // 예외 발생 시 X::outOfMemory 호출
Y* p2 = new Y;  // 예외 발생 시 Y::outOfMemory 호출
```

### operator new가 해야 할 일
```cpp
class NewHandlerHolder
{
public:
    explicit NewHandlerHolder(std::newHandler nh) : handler(nh) { }
    ~NewHandlerHolder() { std::set_new_handler(handler); }

private:
    std::new_handler handler;
};

void* Widget::operator new(std::size_t size) throw(std::bad_alloc)
{
    NewHandlerHolder h(std::set_new_handler(currentHandler);)
    return ::operator new(size);
}
```
1. 표준 set_new_handler 함수에 Widget의 new 처리자를 넘겨서 호출. 즉, 전역 new 처리자로서 Widget의 new 처리자를 설치
2. 전역 operator new를 호출하여 실제 메모리 할당 수행. 설치한 new handler가 실패하면 bad_alloc 예외를 던짐. 전역 new handler를 이전의 new handler로 복구.
3. 전역 operator new를 호출하여 성공하면 할당된 메모리가 반환되고, 전역 new 처리자를 관리하는 객체의 소멸자가 호출되면서 Widget의 operator new가 호출되기 전에 쓰이고 있던 전역 new 처리자를 복원.

CRTP(Curiously Recurring Template Pattern)을 사용하여 템플릿을 이용한 구현도 가능.

```cpp
class Wiget { ... }
Widget* pw1 = new Widget;   // 할당 실패 시 bad_alloc 예외를 던짐

if(pw1 == 0) ...    // 할당 실패 시 코드 점검 실패

Widget* pw2 = new(std::nothrow) Widget; // 할당 실패 시 NULL 반환
if(pw2 == 0) ...    // 할당 실패 시 코드 점검 성공
```
bad_alloc 말고도 할당 실패 시 NULL을 반환하는 new도 있음.
<br>
다만, Widget 생성자 내부에서 new가 호출되는 상황에서까지 NULL 반환을 수행하는 것은 아님.
<br>
즉, std::nothrow new를 사용하더라도 예외가 발생할 수 있음.

## 정리
set_new_handler 함수를 쓰면 메모리 할당 요청이 만족되지 못했을 때 호출되는 함수를 지정할 수 있습니다.
<br>
예외불가(nothrow) new는 영향력이 제한되어 있습니다. 메모리 할당 자체에만 적용되기 때문입니다. 이후에 호출되는 생성자에서는 얼마든지 예외를 던질 수 있습니다.

# 항목 50 : new 및 delete를 언제 바꿔야 좋은 소리를 들을지를 파악해두자
## operator new와 operator delete를 수정하는 이유
### 잘못된 힙 사용을 탐지하기 위해
할당된 메모리 주소를 operator new가 유지하고 operator delete가 목록으로부터 주소를 제거한다거나, 데이터 오버런(할당 메모리보다 더 쓰는 것), 데이터 언더런(할당된 메모리 블록 앞에 기록하는 것) 등을 방지하기 위해 바이트 패턴을 만든다던가 등의 용도로 사용 가능

### 효율을 향상시키기 위해
컴파일러 제공 함수는 일반적인 쓰임새에 맞추어 설계된 것.
<br>
사용자 정의 버전은 컴파일러 제공 버전보다 더 빠르게 최적화될 수 있음.
<br>
속도가 빠르고, 메모리가 차지하는 공간이 더 적게 사용함으로써 new와 delete를 재정의하는 것을 통해 프로그램의 괄목할만한 성능을 뽑아낼 수 있음.

### 동적 할담 메로리의 실제 사용에 관한 통계 정보를 수집하기 위해
할당된 메모리 블록의 크기 분포, 사용 기간, 할당 및 해제 순서, 시간에 따른 패턴 변화, 최대 할당량 등의 정보를 수집할 수 있음.

## 바이트 정렬
대부분의 컴퓨터는 아키텍처적으로 특정 타입의 데이터가 특정 종류의 메모리 주소를 시작 주소로 하여 저장될 것을 요구사항으로 둠. 즉, 4의 배수에 해당하는 주소에 맞추어 저장되는 4바이트 정렬이나 8의 배수에 해당하는 주소에 맞추어 저장되는 8바이트 정렬이 요구됨.
<br>
아키텍처에 따라서 바이트 정렬 제약을 따르지 않을 경우 하드웨어 예외를 일으키기도 함.
<br>
또한 바이트 정렬을 만족했을 때 더 나은 성능을 제공함.
<br>
custom operator new 역시 바이트 정렬을 만족하는 포인터를 반환해야 함.
<br>
바이트 정렬 등 세세한 문제를 어떻게 다루느냐에 따라 메모리 관리자가 달라지므로 꼭 만들어 쓸 필요가 없다면 굳이 새로 만드는 것을 추천하지는 않음.

## 언제 new와 delete를 재정의해야 하는가
1. 잘못된 힙 사용을 탐지하기 위해.
2. 동적 할당 메모리의 실제 사용에 관한 통계 정보를 수집하기 위해.
3. 할당 및 해제 속도를 높이기 위해.
4. 기본 메모리 관리자의 공간 오버헤드를 줄이기 위해.
5. 적당히 타협한 기본 할당자의 바이트 정렬 동작을 보장하기 위해.
6. 임의의 관계를 맺고 있는 객체들을 한 군데에 나란히 모아 놓기 위해. placement new 및 위치지정 delete를 사용.
7. 그때 그때 원하는 동작을 수행하도록 하기 위해.

## 정리
개발자가 스스로 사용자 정의 new 및 delete를 작성하는 데는 여러 가지 나름대로 타당한 이유가 있습니다. 여기에는 수행 성능을 향상시키려는 목적, 힙 사용 시 에러를 디버깅하려는 목적, 힙 사용 정보를 수집하려는 목적 등이 포함됩니다.

# 항목 51 : new 및 delete를 작성할 때 따라야 할 기존 관례를 잘 알아두자
## operator new의 관례
### 반환 값이 제대로 되어 있어야 한다
요청된 메모리를 마련해 줄 수 있으면 그 메모리에 대한 포인터를 반환해야 한다.
<br>
메모리를 마련해 줄 수 없을 경우 new handler를 호출하고 그래도 불가능할 경우 bad_alloc 예외를 던진다.
<br>
operator new가 예외를 던지는 경우는 오직 new handler가 nullptr인 경우.

### 가용 메모리가 부족할 경우에는 new handler를 호출해야 한다

### 크기가 없는 메모리 요청에 대한 대비책을 갖추어야 한다
```cpp
void* operator new(std::size_t size) throw(std::bad_alloc)
{
    using namespace std;

    if(size == 0)
    {
        size = 1;
    }

    while(true)
    {
        if(할당 성공)
        {
            return 할당 메모리 포인터;
        }

        new_handler globalHandler = set_new_handler(0); // global handler를 직접적으로 받아올 수는 없음
        set_new_handler(globalHandler);

        if(globalHandler) { (*globalHandler)(); }
        else { throw std::bad_alloc; }
    }
}
```
0바이트가 요구되었을 때에도 operator new는 적법한 포인터를 반환해야 한다.
<br>
다중 스레드 환경에서는 new handler 처리 과정에서 스레드 안전성이 보장되어야 한다.
<br>
무한 루프를 빠져나오기 위해서는 메모리 할당이 성공하던가, new 처리자에서 가용 메모리를 늘리던가, 다른 new 처리자를 설치하던가, new 처리차의 설치를 제거하던가, bad_alloc 계열 예외를 던지는 것 말고는 불가능.
<br>
즉, new 처리자에서 올바른 처리를 하지 않을 경우 new 내부 루프를 빠져나올 수 없음.

### 기본 형태의 new가 가려지지 않도록 해야 한다
```cpp
class Base
{
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc)
    {
        if(size != sizeof(Base))
        {
            return ::operator new(size);
        }
        ...
    }
};

class Derived : public Base { ... }

Derived* p = new Derived;
```
operator new 멤버 함수는 파생 클래스를 통해 상속이 된다.
<br>
만약, 특정 클래스의 메모리 할당 최적화를 위해 operator new를 재정의했다면 파생 클래스는 이 최적화가 필요 없을 가능성이 높음.
<br>
가장 간단한 방법은 Base와 다른 크기의 메모리가 왔을 경우 기본 new를 호출하도록 변경하는 것.
<br>
C++의 모든 독립 구조 객체는 반드시 크기가 0을 넘어야 한다는 규칙에 의해 sizeof(Base)는 항상 0보다 크므로 size가 0일 경우 if문이 거짓이 되어 Empty Class도 정상적으로 처리됨.

operator new[]를 재정의할 경우 해당 구문 안에서 할 일은 단순히 원시 메모리 덩어리를 할당하는 것 외에 없다는 것을 기억해야 한다.
<br>
파생 클래스의 객체는 대부분 기본 클래스보다 크므로 몇 개의 객체가 해당 배열에 들어갈지 알 수 없고, 배열 앞에 배열 개수를 담는 자투리 공간 등으로 인해 정확하게 그 개수를 확정할 수 없음.

## operator delete의 관례
### C++의 널 포인터에 대한 delete 적용은 항상 안전하도록 보장해야 한다.
```cpp
void Base::operator delete(void* rawMemory) throw()
{
    if(rawMemory == 0) return;

    if(size != sizeof(Base))
    {
        ::operator delete(rawMemory);
        return;
    }

    ...
}
```

## 정리
관례적으로 operator new 함수는 메모리 할당을 반복해서 시도하는 무한 루프를 가져야 하고, 메모리 할당 요구를 만족시킬 수 없을 때 new 처리자를 호출하며, 0 바이트에 대한 대책도 있어야 합니다. 클래스 전용 버전은 자신이 할당하기로 예정된 크기보다 더 큰 메모리 블록(파생 클래스)에 대한 요구도 처리해야 합니다.
<br>
operator delete 함수는 널 포인터가 들어왔을 때 아무 일도 하지 않아야 합니다. 클래스 전용 버전의 경우에는 예정 크기 보다 더 큰 블록(파생 클래스)을 처리해야 합니다.

# 항목 52 : placement new를 작성한다면 placement delete도 같이 준비하자
```cpp
Widget* pw = new Widget;
```
위 코드는 메모리 할당을 위한 operator new함수와 Widget의 기본 생성자까지 2번의 함수 호출이 이루어진다.
<br>
생성자 과정에서 예외가 발생한 경우 할당된 메모리는 회수되어야 한다. 하지만 사용자 코드에서 이를 처리할 방법은 없으므로 이는 런타임 시스템이 관리한다.
<br>
이 때 런타임 시스템은 operator new와 짝을 이루는 operator delete를 호출해야 하므로 비기본형 new를 사용한다면 이에 맞는 delete도 지정해야 한다.

```cpp
class Widget
{
public:
    static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);

    static void operator delete(void* pMemory, size_t size) throw();
};

Widget* pw = new (std::cerr)Widget;
delete pw;  // 기본형의 operator delete 호출
```
비기본형 new란 다른 매개변수를 추가로 갖는 new를 의미. 이를 placement new라고 부른다. 이 중 대표적인 구조가 객체를 생성시킬 메모리 위치를 나타내는 포인터를 매개변수로 받는 구조.
<br>
placement new를 통해 메모리를 할당하던 중 예외가 발생하면 런타임 시스템은 operator new가 받아들이는 매개변수의 개수 및 타입이 똑같은 버전의 operator delete를 찾아 호출한다. 이를 placement delete라고 한다.
<br>
만약 적절한 placement delete가 선언되지 않았을 경우 어떤 delete도 호출되지 않는다. 즉, 메모리 누수가 발생한다.
<br>
예외가 발생하지 않고 정상적으로 delete에 도달한다면 기본형 delete가 호출된다. 즉, placement delete가 호출되는 경우는 placement new에서 예외가 발생한 경우뿐이다.

```cpp
class Base
{
public:
    static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc) { ... }
};

Base* pb = new Base;    // 에러. 표준 new가 가려진다.
Base* pb = new (std::cerr)Base;
```
바깥쪽 유효범위에 있는 어떤 함수의 이름과 클래스 멤버 함수의 이름이 같으면 바깥쪽 유효범위의 함수가 가려지는 것을 고려하여 placement new가 클래스 전용 new를 가리지 않도록 신경써야 한다.
<br>
파생 클래스는 전역 new와 Base의 new도 가려버린다.

```cpp
void* operator new(std::size_t) throw(std::bad_alloc);
void* operator new(std::size_t, void*) throw(std::bad_alloc);   // placement new
void* operator new(std::size_t, const std::nothrow_t&) throw(std::bad_alloc);   // 예외불가 new
```
c++가 지원하는 operator new는 위 3가지를 기본적으로 지원한다.

## 정리
operator new 함수의 placement 버전을 만들 때는, 이 함수와 짝을 이루는 placement delete 함수도 꼭 만들어 주세요. 이 일을 빼먹었다가는, 찾아내기도 힘들며 또 생겼다가 안생겼다 하는 메모리 누출 현상을 경험하게 됩니다.
<br>
new 및 delete의 placement 버전을 선언할 때는 의도한 바도 아닌데 이들의 표준 버전이 가려지는 일이 생기지 않도록 주의해주세요.