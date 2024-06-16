# 개요
자원이란 사용을 마치고 난 후 시스템에 돌려주어야 하는 모든 것
<br>
동적 할당 메모리, file descriptor, mutex lock, GUI의 font와 brush, 데이터베이스 연결, 네트워크 소켓 등
<br>
이를 수작업으로 해제하는 것은 쉽지 않음. 예외 발생, 수많은 return, 코드를 이해하지 못한 개발자의 유지보수 등의 이유로 문제가 발생할 수 있음.
<br>
객체 기반 방식의 자원 관리 기법을 설명. 생성자, 소멸자, 객체 복사 함수를 사용하는 방법

# 항목 13 : 자원 관리에는 객체가 적합
```cpp
class Investment { ... };

Investment* createInvestment();

void f()
{
    Investment* pInv = createInvestment();
    ... // return 이 있을수도 있고, 예외가 발생할 수 도 있음
    delete pInv;
}
```
...에서 문제가 발생할 경우 delete 까지 도달할 수 없음
<br>
자원이 항상 해제되도록 만드는 방법은 자원을 객체에 넣고 그 자원 해제를 소멸자가 맡도록 하며, 그 소멸자는 실행 제어가 f를 떠날 때 호출되도록 만드는 것
<br>
책에서는 auto_ptr의 사용을 권장하는데 modern에서는 auto_ptr이 deprecated된 것으로 알고 있음. 대신 unique_ptr, shared_ptr을 사용

## 자원 관리에 객체를 사용하는 방법의 중요한 특징
### 자원을 획득한 후에 자원 관리 객체에게 넘깁니다.
```cpp
std::shared_ptr sp = std::make_shared(createInvestment());
```
RAII(Resource Acquisition Is Initialization) 원칙에 따라 객체 생성 시점에 함수가 만들어준 자원을 관리 객체에게 바로 넘겨준다.

### 자원 관리 객체는 자신의 소멸자를 사용해서 자원이 확실히 해제되도록 합니다.
소멸자는 객체가 소멸될 때 자동으로 호출되기 때문에 실행 제어 경로가 어떤 경위로 블록을 떠났는가에 상관없이 자원 해제가 제대로 이루어질 수 있음

참고로 std::shared_ptr은 내부에서 delete 연산을 함으로 배열에 대해서는 사용하기 좋지 않음.
<br>
대신 std:vector를 사용하는 것을 추천(물론 반복문 돌면서 해제해야 함)

또한 createInvestment가 포인터를 반환하는 것은 사용자가 delete 호출을 까먹을 위험이 있음.
<br>
이에 대한 해결법은 항목 18을 참조

## 정리
자원 누출을 막기 위해, 생성자 안에서 자원을 획득하고 소멸자에서 그것을 해제하는 RAII 객체를 사용합시다.
<br>
일반적으로 널리 쓰이는 RAII 클래스는 std::shared_ptr, std::unique_ptr입니다. std::shared_ptr은 복사 시의 동작이 직관적이기 때문에 이해하기 좋습니다. std::unique_ptr은 복사되는 원본을 nullptr로 만듭니다.

# 항목 14 : 자원 관리 클래스의 복사 동작에 대해 지지하게 고찰하자
관리해야 할 자원이 항상 힙에서 생기지만은 않는다. mutex같이 힙이 아님에도 생성되는 자원이 있음. 이러한 자원들은 스마트 포인터로 관리할 수 없음.
<br>
이 경우에도 RAII 원칙에 따라 생성 시에 자원을 획득하고 소멸 시에 자원을 해제하는 방향으로 클래스를 구현하면 됨.

```cpp
void lock(Mutex* pm);
void unlock(Mutex* pm);

class Lock
{
public:
    explicit Lock(Mutex* pm) : mutexPtr(pm) { Lock(mutexPtr); }
    ~Lock() { unlock(mutexPtr); }

private:
    Mutex* mutexPtr;
};

Mutex m;
...
{
    ...
    Lock m1(&m);
}
```
만약 Lock 객체가 복사된다면 어떻게 해야 하는가?
<br>
만약 RAII 객체가 복사된다면 어떤 동작이 이루어져야 하는가?

## 선택지
### 복사 금지
RAII 객체가 복사되도록 놔두는 것 자체가 말이 안되는 경우가 많음.
<br>
스레드 동기화 객체가 사본을 가질 수는 없음.
<br>
즉, 복사 연산자를 private 멤버 함수(modern c++에서는 delete 추천)로 만드는 것.

### 관리하고 있는 자원에 대해 참조 카운팅을 수행
자원을 사용하고 있는 마지막 객체가 소멸될 때까지 자원을 해제하지 않는 방법.
<br>
자원을 참조하는 객체의 개수에 대한 카운트를 증가시키는 방식으로 RAII 객체 복사 동작 수행
<br>
std::shared_ptr의 기법

```cpp
class Lock
{
public:
    explicit Lock(Mutex* pm) : mutexPtr(pm, unlock) { lock(mutexPtr.get()); }

private:
    std::shared_ptr<Mutex> mutexPtr;
};
```
std::shared_ptr에서 custom delete를 추가하는 방식으로 mutex관리 가능

### 관리하는 자원을 진짜로 복사
자원을 다 썼을 때 각각의 사본을 해제하는 방식
<br>
깊은 복사를 수행하는 방식

### 관리하고 있는 자원의 소유권을 옮기기
그 자원을 실제로 참조하는 RAII 객체가 딱 하나만 존재하도록 만들고 싶은 경우
<br>
std::unique_ptr의 복사 동작 방식

## 정리
RAII 객체의 복사는 그 객체가 관리하는 자원의 복사 문제를 안고 가기 때문에, 그 자원을 어떻게 복사하느냐에 따라 RAII 객체의 복사 동작이 결정됩니다.
<br>
RAII 클래스에 구현하는 복사 동작은 복사 금지, 참조 카운팅, 깊은 복사, 소유권 이전의 방식이 있음

# 항목 15 : 자원 관리 클래스에 관리되는 자원은 외부에서 접근할 수 있도록 하자.
현장의 수많은 API들은 자원을 직접 참조하도록 만들어져있다.
<br>
즉, 이런 함수들을 지원하기 위해서는 자원 관리 클래스는 관리하는 자원을 외부에 직접 제공할 수 있어야 한다.
<br>
스마트 포인터들은 get함수를 통해 명시적 변환을 수행한다. 또한 operator->를 오버로딩하여 암시적 변환도 지원한다.

```cpp
FontHandle getFont();

void releaseFont(FontHandle fh);

class Font
{
public:
    explicit Font(FontHandle fh) : f(fh) { }
    ~Font() { releaseFont(f); }

    FontHandle get() { return f; }
    operator FontHandle() const { return f; }

private:
    FontHandle f;
};
```
operator를 이용한 암시적 변환도 가능

```cpp
Font f1(getFont());
...
FontHandle f2 = f1; // 원 의도가 Font를 복사하는 것이었을 수 도 있음. 이로 인해 f1의 FontHandle이 외부로 노출되었음. f1이 자동으로 소멸된다면 f2는 dangling이 됨
```
문제는 실제로 Font를 사용하려던 상황에서 FontHandle이 들어갈 수 도 있다는 점이 있음.

즉, 명시적 변환은 실수를 줄여주지만 암시적 변환은 사용의 편리함을 제공함.

참고로 위 구조는 캡슐화를 깨는 구조이기는 하지만 이는 자원 해제가 목적이었기 때문에 자원 해제에 관련된 동작만 캡슐화한다면 문제가 되지 않음.
<br>
std::shared_ptr은 자원은 외부로 공개하지만 참조 카운팅과 관련해서는 캡슐화를 유지하고 있음.
<br>
즉, 사용자가 볼 필요가 없는 데이터는 가리고, 접근해야 하는 데이터는 열어주는 것이 캡슐화

## 정리
실제 자원을 직접 접근해야 하는 기존 API들도 많기 때문에, RAII 클래스를 만들 때는 그 클래스가 관리하는 자원을 얻을 수 있는 방법을 열어줘야 한다.
<br>
자원 접근은 명시적 변환, 암시적 변환을 통해 가능. 안정성만 따지면 명시적 변환이 대체적으로 더 낫지만, 고객 편의성을 보면 암시적 변환이 괜찮음.

# 항목 16 : new 및 delete를 사용할 때는 형태를 반드시 맞추자
```cpp
std::string* stringArray = new std::string[100];
...
delete stringArray;
```
단일 객체와 객체 배열은 메모리 구조가 다르다. 객체 배열의 경우 힙 메모리에 배열 원소의 개수 정보가 추가로 들어감.
<br>
delete[]를 통해 객체 배열을 해제해야지만 포인터가 배열을 가리킨다는 것을 알고 정확한 해제에 들어감.

즉, new를 통해 생성했다면 delete를 통해 해제해야 하고, new[]를 통해 생성했다면 delete[]를 통해 해제해야 한다.

```cpp
typedef std::string  AddressLines[4];
std::string* pal = new AddressLines;
delete[] pal;   // delete[]로 삭제해야 한다.
```
typedef를 통해 객체를 생성한다면 이 부분에 대해서 주의해야 한다.
<br>
즉 배열타입은 typedef를 사용하지 않는 것이 좋다.
<br>
대신 std::string이나 std::vector를 사용하는 것을 추천.

## 정리
new 표현식에 []를 썼으면, 대응되는 delete 표현식에도 []를 써야합니다. 마찬가지로 new 표현식에 []를 안 썼으면, 대응되는 delete 표현식에도 []를 쓰지 말아야 합니다.

# 항목 17 : new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자.
```cpp
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);

processWidget(std::shared_ptr<Widget>(new Widget), priority());
```
위 구문은 총 3가지 연산이 수행된다.
1. priority() 호출
2. new Widget 생성
3. std::shared_ptr 생성
이 과정에서 컴파일러에 의한 최적화가 발생하고 2번과 1번이 바뀐 상태에서 priority()에서 예외가 발생한다면 new Widget은 회수되지 않고 누수된다.

```cpp
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());

processWidget(std::make_shared<Widget>(), priority());  // modern c++에서는 이 방식 추천
```

## 정리
new로 생성한 객체를 스마트 포인터로 넣는 코드는 별도의 한 문장으로 만듭시다. 이것이 안되어있으면, 예외가 발생될 때 디버깅하기 힘든 자원 누출이 초래될 수 있습니다.
