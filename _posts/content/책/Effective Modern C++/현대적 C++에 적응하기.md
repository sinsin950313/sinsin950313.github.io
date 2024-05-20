# 항목 7 : 객체 생성 시 괄호와 중괄호를 구분하라
## 장점
```cpp
int x(0);
int x = 0;
int x{0};
int x = { 0 };
```
C++11에서는 초기화 값을 괄호, 등호, 중괄호로 지정할 수 있음
<br>
또한 중괄호와 등호를 함께 사용할수도 있는데, 이는 중괄호만 사용한 구문과 동일하게 판정

```cpp
Widget w1;      // 기본 생성자 호출
Widget w2 = w1; // 배정이 아님. 복사 생성자 호출
w1 = w2;        // 배정. 복사 배정 연산자 호출
```
초기화와 배정(assignment)는 다른 함수를 호출하므로 둘을 구분하는 것이 중요함
<br>
이런 점에서 등호를 사용한 초기화 구문은 배정이 일어난다고 오해할 수 있음

```cpp
class Widget
{
    private:
        int x{0};   // x의 기본값은 0
        int y = 0;  // y의 기본값은 0
        int z(0);   // 사용 불가능
}
```
C++98에서는 서로 다른 임의의 값을 이용한 초기화(std::vector<int> v {1, 3, 5};)같은 모든 가능한 초기화 시나리오가 가능하지 않았음.
<br>
이를 해결하기 위해 C++11에서는 균일 초기화(Uniform Initialization)을 도입
<br>
균일 초기화 구문은 중괄호를 사용. 중괄호 초기화라고 표현.
<br>
또한 비정적 자료 멤버의 초기화에서도 사용 가능

```cpp
std::atomic<int> ai1{0};
std::atomic<int> ai2(0);
std::atomic<int> ai3 = 0;   // 사용 불가능
```
복사할 수 없는 객체는 중괄호나 괄호로는 초기화가 가능하지만, 등호로는 초기화 불가능

즉, 중괄호 초기화는 모든 상황에서 사용 가능

```cpp
double x, y, z;
int sum1{x + y + z};    // double의 합을 int로 표현할 경우 narrow conversion이 발생하므로 오류 발생
int sum2 = x + y + z;   // narrow conversion이 발생해도 문제없이 컴파일
int sum3(x + y + z);    // narrow conversion이 발생해도 문제없이 컴파일
```
중괄호 초기화는 Narrowing Conversion을 방지함

```cpp
Widget w1(10);  // 인수를 10으로 하는 Widget의 생성자 호출
Widget w2();    // w2라는 이름의 함수 선언
Widget w3{};    // 인수가 없는 Widget 생성자 호출
```
중괄호 초기화는 '선언으로 해석할 수 있는 것은 항상 선언으로 해석해야 한다'는 C++의 규칙에서 자유로움

## 단점
auto와 중괄호 초기화를 함께 사용한다면 std::initializer_List 형식으로 연역될 위험이 있음.

```cpp
class Widget
{
public:
    Widget(int i, bool b);
    Widget(int i, double d);
    Widget(std::initializer_list<long double> il);

    operator float() const;
};

Widget w1(10, true);        // 첫번째 생성자 호출
Widget w2{10, true};        // 3번째 생성자가 없다면 1번째 생성자 호출. 3번째 생성자가 있다면 3번째 생성자 호출

Widget w3(10, 0.5f);        // 두번째 생성자 호출
Widget w4{10, 0.5f};        // 3번째 생성자가 없다면 2번째 생성자 호출. 3번째 생성자가 있다면 3번째 생성자 호출

Widget w5(w4);              // 복사 생성자 호출
Widget w6{w4};              // w4가 float으로 변환되고 float이 long double로 변환되어 std::initiailizer_list 호출

Widget w7(std::move(w4));   // 이동 생성자 호출
Widget w8{std::move(w4)};   // w4가 float으로 변환되고 float이 long double로 변환되어 std::initiailizer_list 호출
```

```cpp
class Widget
{
public:
    Widget(int i, bool b);
    Widget(int i, double d);
    Widget(std::initializer_list<bool> b);
};

Widget w9{10, 0.5f};        // narrow conversion이 발생해야 하므로 error 발생
```
중괄호 생성자는 std::initializer_list를 매개변수로 받는 생성자가 선언되어 있을 때, 해당 생성자로 해석할 여지가 있다면 std::initializer_list를 매개변수로 하는 생성자를 선택한다.
<br>
심지어 해당 해석보다 더 부합하는 생성자가 있어도 std::initializer_list가 우선순위가 더 높음

```cpp
class Widget
{
public:
    Widget();
    Widget(std::initializer_list<int> i);
};

Widget w1;      // 기본 생성자 호출
Widget w2{};    // 기본 생성자 호출
Widget w3();    // 함수 선언
Widget w4{{}}   // std::initializer_list 생성자 호출
Widget w5({})   // std::initializer_list 생성자 호출
```
기본생성자와 std::initializer_list를 매개변수로 받는 생성자를 모두 선언한 상황에서 아무 인자가 없이 객체를 생성하면 기본 생성자를 호출
<br>
즉, 빈 중괄호 쌍은 인수 없음을 의미

```cpp
std::vector<int> v1(10, 20);    // non-std::initiailier_list 생성자 사용. 모든 element의 값이 20인 element를 10개 갖고있는 vector 생성
std::vector<int> v2{10, 20};    // std::initializer_list 생성자 사용. element로 10, 20을 갖는다
```
container class의 경우 중괄호 초기화자로 인한 규칙에 민감하게 사용될 수 있음

즉, 생성자 overloading에서는 std::initializer_list를 매개변수로 받는 생성자를 가급적 만들지 말아야 한다.
<br>
클래스 사용자는 객체를 생성할 때 중괄호와 괄호를 세심하게 선택해서 사용해야 한다.

### 중괄호 초기화자의 장점
1. 다양한 문맥에서 적용 가능
2. narrow conversion 방지
3. 함수로 해석하는 구문 문제에서 자유로움
-> 주어진 크기와 초기 요소 값으로 std::vector를 생성하는 경우같은 상황에서는 괄호를 사용해야 한다

### 괄호 초기화자의 장점
1. 전통적인 C++의 구문과 일관적
2. auto가 std::initializer_list로 연역하는 문제가 없음
3. 객체 생성 시 의도치 않게 std::initializer_list 생성자가 호출되지 않음
-> 구체적인 값들로 컨테이너를 생성하는 경우같은 상황에서는 중괄호를 사용해야 한다

즉, 두 초기화자 방법 중 1개를 일관적으로 사용하고 다른 사용법을 사용하는 경우에 대하여 잘 이해하고 있어야 한다.

```cpp
template<typename T, typename... Ts>
void doSomeWork(Ts&&... params)
{
    T localObject(std::forward<Ts>(params)...);     // 괄호를 사용. 그 결과 모든 element값이 20인 element가 10개 들어있는 vector를 인자로 사용
    T localObject{std::forward<Ts>(params)...};     // 중괄호를 사용. 그 결과 10과 20이 element로 들어있는 vector를 인자로 사용
}

doSomeWork<std::vector<int>>(10, 20);
```
템플릿 클래스의 작성자는 사용자가 괄호와 중괄호 중 어떤 생성 방식을 사용할지 알 수 없다.

## 정리
중괄호 초기화는 가장 광범위하게 적용할 수 있는 초기화 구문이며, narrow conversion을 방지하고, C++의 가장 성가신 구문 해석에서 자유롭다
<br>
생성자 중복적재 해소 과정에서 중괄호 초기화는 가능한 한 std::initializer_list 매개변수가 있는 생성자와 부합한다(심지어 겉으로 보기에 그보다 인수들에 더 잘 부합하는 생성자들이 있어도)
<br>
괄호와 중괄호의 선택이 의미 있는 차이를 만드는 예는 인수 두 개로 std::vector<Type>을 생성하는 것이다.
<br>
템플릿 안에서 객체를 생성할 때 괄호를 사용할 것인지 중괄호를 사용할 것인지 선택하기 어려울 수 있다.

# 항목 8 : 0과 NULL보다 nullptr를 선호하라
포인터만 사용할 수 있는 위치에서 0이 있으면 C++에서 그것을 null pointer로 해석하는 것이지 원칙적으로 Literal 0은 pointer가 아니다.
<br>
컴파일러에 따라 다르지만 NULL 역시 int 또는 이외의 정수 형식(long 등)의 0이지 pointer가 아니다.

```cpp
void f(int);
void f(bool);
void f(void*)

f(0);       // f(int) 호출
f(NULL);    // 컴파일 실패 or f(int). f(void*)가 호출되지는 않음
f(nullptr)  // f(void*) 호출
```
pointer와 int를 구분하지 않을 경우 함수 overloading에서 문제가 발생할 수 있음
<br>
Pointer type과 int type의 overloading을 피하는 것이 좋다.
<br>
nullptr은 std::nullptr_t type이고 std::nullptr_t의 정의는 'nullptr의 형식'으로 정의. std::nullptr_t는 모든것의 raw pointer type으로 변환되며 이는 모든 type의 pointer처럼 행동.

```cpp
auto result = findRecord(...);
if(result == 0) { ... };        // result가 0인지 NULL인지 알 수 없음
if(result == nullptr) { ... };  // result가 pointer type인 것을 알 수 있음
```
nullptr을 사용하므로써 ambiguity가 없음

```cpp
int f1(std::shared_ptr<Widget> spw);
double f2(std::unique_ptr<Widget> upw);
bool f3(Widget* pw);

std::mutex f1m, f2m, f3m;

using MuxGuard = std::lock_guard<std::mutex>;
{
    MuxGuard g(f1m);
    auto result = f1(0);        // nullptr의 의미로 0 사용
}
{
    MuxGuard g(f2m);
    auto result = f2(NULL);     // nullptr의 의미로 NULL 사용
}
{
    MuxGuard g(f3m);
    auto result = f3(nullptr);  // nullptr의 의미로 nullptr 사용
}

// 반복적인 소스 코드 중복을 방지하기 위해 template 사용
template<typename FuncType, typename MuxType, typename PtrType>
decltype(auto) lockAndCall(FuncType func, MuxType& mutex, PtrType ptr)
{
    using MuxGuard = std::lock_guard<MuxType>;

    MuxGuard g(mutex);
    return func(ptr);
}

auto result1 = lockAndCall(0);          // f1(int) 형식으로 연역되면서 컴파일 에러
auto result2 = lockAndCall(NULL);       // f2(정수) 형식으로 연역되면서 컴파일 에러
auto result3 = lockAndCall(nullptr);    // f3(std::nullptr_t)로 연역되고 f3(Widget*) 형식으로 연역되면서 컴파일 성공
```
nullptr의 의미로 0이나 NULL을 사용할 경우 템플릿에서 잘못된 연역을 유도할 수 있음

## 정리
0과 NULL보다 nullptr을 선호하라
<br>
정수 형식과 포인터 형식에 대한 중복 적재를 피하라

# 항목 9 : typedef보다 별칭 선언을 선호하라.
typedef와 alias declaration은 동일한 일을 수행한다.

```cpp
typedef void (*FP)(int, const std::string&);
using FP = void(*)(int, const std::string&);
```
typedef보다 alias declaration이 조금이라도 더 이해하기 쉽다.

```cpp
template<typename T>
struct MyAllocList
{
    typedef std::list<T, MyAlloc<T>> type;
};

MyAllocList<Widget>::type lw;           // ::type이라는 불필요해보이는 형식이 더 필요

template<typename T>
class Widget
{
private:
    typename MyAllocList<T>::type list; // MyAllocList는 T에 의존적이므로 typename이 필요
};

////////////////////////////////////////////////////////////////////////

template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;

template<typename T>
class Widget
{
private:
    MyAllocList<T> list;
};
```
typedef는 템플릿화 할 수 없지만, aliasing은 템플릿화 할 수 있음. 템플릿화 된 별칭을 alias templates라고 한다.
<br>
이를 이용하여 C++98에서 struct 안에서 typedef를 활용하던 편법 대신 C++11에서는 더 직접적으로 표현 가능

```cpp
class Wine { ... }

template<>
class MyAllocList<Wine>
{
private:
    enum class WineType { White, Red, Rose };

public:
    WineType type;
}

// MyAllocList<Wine>::type은 멤버 변수가 된다
```
C++에서는 의존적 형식의 이름 앞에 typename을 붙여야하는 규칙이 있는데, 템플릿 내부에서 템플릿 매개변수로 지정된 형식을 사용할 때 typedef를 사용한다면 의존적 형식(dependent type)이 된다.
<br>
typedef를 사용해서 정의한 MyAllocList<T>::type은 (특수화등으로 인해) Type임을 보장할 수 없기 때문에(멤버 변수 등) 컴파일러에 MyAllocList<T>::type이 형식의 일종이고 템플릿 매개변수에 의존적이다는 것을 알리기 위해 typename이라는 구문이 필요
<br>
aliasing의 경우 컴파일러는 MyAllocList<T>가 이미 type의 다른 이름이라는 것을 알고 있으므로 비의존적 형식이므로 typename이 필요하지 않다.

책에는 위와 같이 적혀있는데 의존적의 정의가 애매해서 MS Document를 찾아보니 typename 키워드는 '알 수 없는 식별자가 Type이라는 힌트를 컴파일러에게 제공한다.'라고 적힌 것을 확인할 수 있음.
<br>
이는 결국 MyAllocList<T>::type의 경우 해당 식별자가 형식인지 변수인지를 컴파일러가 확신할 수 없으니 typename이라는 키워드를 작성하여 형식이라는 것을 확인시켜주는 작업이고, using을 사용한 구문에서는 이미 MyAllocList<T>가 형식이라는 것을 컴파일러가 알고 있기 때문에 typename이라는 구문이 필요하지 않은 것으로 보임.

## 정리
typedef는 템플릿화를 지원하지 않지만, Alias는 템플릿화를 지원한다
<br>
Alias template에서는 ::type 접미어를 붙일 필요가 없다. 템플릿 안에서 typedef를 지칭할 때에는 typename 접두어를 붙어야 하는 경우가 많다
<br>
C++14는 C++11의 모든 Type Trait 변환에 대한 Alias template을 지원한다.

# 항목 10 : 범위 없는 enum보다 범위 있는 enum을 선호하라
```cpp
enum Color { black, white, red };       // black, white, red는 Color가 속한 범위에 속함

auto white = false; // white는 범위 내에 이미 선언되어있음

////////////////////////////////////////////////////////

enum class Color { black, white, red }; // black, white, red는 Color의 범위에 속함

auto white = false;                     // 이 범위에 white라는 이름이 없음
Color c = white;                        // 이 범위에 white라는 이름이 없으므로 컴파일 오류
Color c = Color::white;                 // 컴파일 성공
auto c = Color::white;                  // 컴파일 성공
```
C++98 스타일의 enum은 unscope 하다. 이를 unscoped enum이라 부른다.
<br>
C++11 스타일의 scoped enum은 이름 누수가 발생하지 않는다. enum class라고 부르기도 한다.

```cpp
enum Color { black, white, red };

std::vector<std::size_t> primeFactors(std::size_t x);

Color c = red;

if(c < 14.5)
{
    auto factors = primeFactors(c); // Color의 소인수를 계산하는 의미상 불가능한 구조가 가능함
}
```
범위 없는 enum은 암묵적으로 정수 형식으로 변환되지만, 범위 있는 enum은 형식이 강력하게 적용된다.
<br>
범위 있는 enum은 암묵적으로 다른 형식으로 변환되지 않는다.

```cpp
enum Color;         // 오류. 전방 선언 불가능.
enum class Color;   // 전방 선언 가능
```
범위 있는 enum은 전방 선언이 가능. 즉, 열거자들을 지정하지 않고 열거형 이름만 미리 선언할 수 있음

```cpp
enum Status
{
    good = 0,
    failed = 1,
    incomplete = 100,
    corrupt = 200,
    // audited = 500,
    indeterminate = 0xFFFFFFFF
};
```
enum의 전방 선언은 생각보다 유효한데, 시스템 전반에서 쓰이는 Status enum같은 경우 공통 header 파일에 포함되어 사용할 수 있을텐데, 만약 해당 enum에 열거값이 변경된다면 대부분의 시스템을 다시 컴파일 해야하는 문제가 발생할 수 있음

```cpp
enum class Status;                  // 바탕 형식은 int
enum class Status : std::unit32_t;  // 바탕 형식은 std::unit32_t
enum class Status : std::unit32_t   // 정의에서도 바탕 형식 지정 가능
{
    good = 0,
    failed = 1,
    incomplete = 100,
    corrupt = 200,
    // audited = 500,
    indeterminate = 0xFFFFFFFF
}
```
범위 있는 enum의 기본적인 바탕 형식은 int이지만 명시적으로 지정할 수 있음
<br>
또한, 범위 없는 enum도 바탕 형식을 지정한다면 전방 선언이 가능

## 범위 없는 enum의 유용성
```cpp
// 사용자 이름, 이메일 주소, 평판을 튜플로 갖는 타입
using UserInfo = std::tuple<std::string, std::string, std::size_t>;

UserInfo uInfo;
auto val = std::get<1>(uInfo);  // tuple의 1번 필드의 값이 무엇인지 알 수 없음

enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
auto val = std::get<uiEmail>(uInfo);    // 암묵적 변환을 통해 tuple의 1번 필드의 값이 email 주소라는 것을 쉽게 알 수 있음

enum class UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo);  // 너무 장황하게 작성해야 함

// enum을 받아서 그에 해당하는 std::size_t값을 반환하는 함수를 만들 경우
template<typename E>
// std::get<>은 함수 템플릿으로 compile time에 결과를 반환해야 하므로 constexpr이 필요
// 함수가 결코 예외를 던지지 않을 것이라는 것을 알고 있을 경우 noexcept 선언
constexpr auto toUType(E enumerator) noexcept
{
    // 임의의 enum 바탕 형식에 대하여 값을 돌려줘야 하므로 std::underlying_type_t type trait 사용
    return static_cast<std::underlying_type_t<E>>(enumerator);
}

auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```
범위 없는 enum의 암묵적 변환을 이용하여 범위 있는 enum보다 간단하게 정보를 전달할 수 있음
<br>
범위 있는 enum을 조금 더 간단하게 정보 전달용으로 사용하기 위해서는 고유의 함수가 필요할 수도 있음

## 정리
C++98 스타일의 enum을 범위 없는 enum이라고 부른다
<br>
범위 있는 enum의 열거자들은 그 안에서만 보인다. 이 열거자들은 오직 캐스팅을 통해서만 다른 형식으로 변환된다.
<br>
범위 있는 enum과 범위 없는 enum 모두 바탕 형식 지정을 지원한다. 범위 있는 enum의 기본 바탕 형식은 int이다. 범위 없는 enum에는 기본 바탕 형식이 없다.
<br>
범위 있는 enum은 항상 전방 선언이 가능하다. 범위 없는 enum은 해당 선언에 바탕 형식을 지정하는 경우에만 전방 선언이 가능하다.

# 항목 11 : 정의되지 않은 비공개 함수보다 삭제된 함수를 선호하라
```cpp
template<class charT, class traits = char_traits<charT>>
class base_ios : public ios_base
{
private:
    basic_ios(const basic_ios&);                // not defined
    basic_ios& operator=(const basic_ios&);     // not defined
};

template<class charT, class traits = char_traits<charT>>
class basic_ios : public ios_base
{
public:
    basic_ios(const basic_ios&) = delete;
    basic_ios& operator=(const basic_ios&) = delete;
};
```
C++98에서는 사용하지 말아야할 함수를 private에서 선언을 하고 정의를 하지 않아 사용을 방지했다면 C++11에서는 delete 처리를 한다.
<br>
private에서 선언하는 경우 Link Time에서 문제가 발생하지만 delete처리를 하면 compile time에서 문제가 발생한다.
<br>
일부 컴파일러는 private과 delete가 함께 적혀있을 때 private이라는 점을 문제로 삼기 때문에 오해의 여지를 줄이기 위해 delete 함수는 public 선언하는 것이 좋음

```cpp
bool isLucky(int number);       // 정수만을 받기를 희망하는 함수
bool isLucky(char) = delete;    // 다른 값으로부터 정수로 캐스팅 되는 것을 막고 싶다면 delete 선언
bool isLucky(double) = delete;  // double과 float을 함께 배제
```
delete는 그 어떤 함수라도 삭제할 수 있기 때문에 멤버 함수가 아니어도 사용이 가능

```cpp
template<typename T>
void processPointer(T* ptr);

template<typename T>
void processPointer(void*) = delete;    // void* 타입의 템플릿 인스턴스 생성을 방지

class Widget
{
public:
    template<typename T>
    void processPointer(T* ptr) { ... }

private:
    template<typename T>
    void processPointer(void*) = delete;    // compile error. 특정한 type만 접근 수준을 다르게 지정하는 것은 불가능
};

template<>
void Widget::processPointer<void>(void*) = delete; // public 접근. 삭제되었음
```
원치않는 템플릿 인스턴스화를 방지할 수 도 있음
<br>
private 방식의 제한은 template에서는 불가능

## 정리
정의되지 않은 비공개 함수보다 삭제된 함수를 선호하라
<br>
비멤버 함수와 템플릿 인스턴스를 비롯한 그 어떤 함수도 삭제할 수 있다.

# 항목 12 : 재정의 함수들을 override로 선언하라
가상 함수 재정의는 파생 클래스 함수를 기반 클래스의 인터페이스를 통해서 호출할 수 있게 만드는 매커니즘
<br>
재정의 필수 조건
1. 기반 클래스 함수가 반드시 가상 함수이어야 한다
2. 기반 함수와 파생 함수의 이름이 반드시 동일해야 한다.(소멸자 예외)
3. 기반 함수와 파생 함수의 매개변수 형식들이 반드시 동일해야 한다.
4. 기반 함수와 파생 함수의 const성이 반드시 동일해야 한다.
5. 기반 함수와 파생 함수의 반환 형식과 예외 명세가 반드시 호환되어야 한다.

```cpp
class Widget
{
public:
    void doWork() &;    // *this가 lvalue 상황에서만 적용된다
    void doWork() &&;   // *this가 rvalue 상황에서만 적용된다
};

Widget makeWidget();    // Factory 함수. rvalue 반환
Widget w;               // 일반 객체. lvalue

w.doWork();             // lvalue용 doWork호출. 즉, doWork() & 호출
makeWidget().doWork();  // rvalue용 doWork 호출. 즉, doWork() && 호출
```

6. 멤버 함수들의 참조 한정사(reference qualifier)들이 반드시 동일해야 한다.
   1. 참조 한정사란 해당 함수를 lvalue, rvalue에서만 사용할 수 있도록 제한하는 것
   2. 파생 클래스에 동일한 참조 한정사가 없을 경우 선언된 함수는 파생 클래스에 존재하긴 하지만 재정의하지는 않는다.

재정의 실수는 프로그래머의 의도와 다른 행동을 유도하지만, 컴파일 오류를 발생하지는 않는다.
<br>
이를 막기 위해 override 키워드를 사용하면 재정의 조건과 맟지 않는 파생 함수가 있을 경우 컴파일러에서 오류를 발생한다.
<br>
또한, override 키워드가 모든 파생 함수에서 작성이 되어있다면 기반 함수에서 가상 함수의 서명을 수정하므로써 발생할 영향을 컴파일 타임에 미리 확인할 수 있음

## 참조 한정사
```cpp
class Widget
{
public:
    using DataType = std::vector<double>;

    DataType& data() { return values; }

private:
    DataType values;
};

Widget w;
auto vals1 = w.data();  // lvalue인 w에서 w.values를 복사하여 vals1 생성

Widget makeWidget();
auto vals2 = makeWidget().data();   // rvalue에서 values를 복사하여 vals2 생성. 비효율적

//////////////////////////////////////////////////////////////

class Widget
{
public:
    using DataType = std::vector<double>;

    DataType& data() & { return values; }

    DataType&& data() && { return std::move(values); }

private:
    DataType values;
};

auto vals1 = w.data();  // lvalue인 w에서 w.values를 복사하여 vals1 생성

Widget makeWidget();
auto vals2 = makeWidget().data();   // rvalue에서 values를 이동 생성
```
참조 한정사를 사용하여 성능상의 최적화를 이끌어낼 수 있음

## 정리
재정의 함수는 override로 선언하라
<br>
멤버 함수 참조 한정사를 이용하면 멤버 함수가 호출되는 객체(*this)의 lvalue 버전과 rvalue 버전을 다른 방식으로 처리할 수 있다

# 항목 13 : iterator보다 const_iterator를 선호하라
가능한 한 const를 사용하라는 표준 관행은 iterator에도 적용되며, const_iterator는 수정하면 안되는 값을 가리키는 iterator이다.

```cpp
std::vector<int> values;

auto it = std::find(values.cbegin(), values.cend(), 1983);
values.insert(it, 1998);
```
C++11에서는 const_iterator의 사용이 C++98보다 더 쉬워졌고, 삽입, 삭제를 목적으로 하는 STL 멤버 함수들은 const_iterator를 사용한다

## Generic library code 작성 중에는
```cpp
template<class C>
// C가 const형이므로 non-member한 std::begin(const C& container)가 들어가도 const_iterator가 반환되고 const Type container[]가 들어가도 const Type*가 반환되므로 동일한 의마로 사용할 수 있음
decltype(auto) cbegin(const C& container)
{
    return std::begin(container);   // container.cbegin()이 아닌 non-member begin을 사용한 것에 주의
}
```
C++14와 달리 C++11에서는 non-member function으로 begin과 end는 표준에 추가되어있지만, cbegin, cend, rbegin, rend, crbegin, crend는 추가되어있지 않음
<br>
만약 프로그래머가 generic한 코드를 작성하고자 할 경우 non-member 함수들을 사용해야 유연하게 호환이 가능하다

## 정리
iterator보다 const_iterator를 선호하라
<br>
최대한 일반적인 코드에서는 begin, end, rbegin 등의 non-member 버전들을 해당 멤버 함수들보다 선호하라.

## 항목 14 : 예외를 방출하지 않을 함수는 noexcept로 선언하라
C++11에 들어서면서 함수의 예외 방출 행동에 관해 정말로 의미 있는 정보는 함수가 예외를 하나라도 방출하는지 여부라는 점에 대한 공감대가 형성됨
<br>
즉, 함수가 예외를 하나라도 던질 수 있는지 아니면 절대로 던지지 않는지라는 이분접적인 정보가 의미있다는 것
<br>
참고로 C++98 스타일의 예외 명세를 동적 예외 명세(dynamic exception specification)이라 부르며 deprecated 기능으로 분류되었으며 throw()(빈 예외 명세)도 동적 예외 명세에 속한다
<br>
C++11에서는 함수가 예외를 방출하지 않을 것임을 명시할 때에는 noexcept이라는 키워드를 사용
<br>
함수의 호출자는 함수의 noexcept여부를 조회할 수 있음

```cpp
int f(int x) throw();   // f는 예외를 방출하지 않음. C++98 방식. 최적화의 여지가 적음
int f(int x) noexcept;  // f는 예외를 방출하지 않음. C++11 방식. 최적화의 여지가 가장 큼
int f(int x);           // f는 예외를 방출할 수 있음. 최적화의 여지 매우 적음
```
noexcept를 선언하므로써 컴파일러는 더 나은 목적 코드를 생성할 수 있음
<br>
C++98방식은 예외가 발생하면 함수 f를 호출한 지점까지 stack unwinding이 발생하지만, noexcept을 선언하면 compiler optimizer가 실행 시점 stack unwinding 가능 상태를 유지할 필요가 없다.
<br>
또한 예외가 noexcept 함수를 벗어나더라도 noexcept 함수 내부의 객체들을 반드시 생성의 반대 순서로 파괴해야 할 필요가 없어져 최적화의 여지가 더 커진다. 

```cpp
std::vector<Widget> vw;
Widget w;
vw.push_back(w);
```
std::vector의 경우 충분한 공간이 없을 경우 새 공간을 할당한 다음 기존 객체들을 복사하는데, 이동 연산을 사용하면 성능상의 이점이 있지만 예외가 발생한 상황에서 원본 데이터를 복구할 수 없는 문제가 있음.
<br>
결국 '가능하면 이동하되 필요하면 복사한다'라는 전략을 사용하는데, 이동과 복사를 결정하는 조건으로 noexcept이 명시되어있는가가 사용

## 조건부 noexcept
```cpp
template<class T, size_t N>
void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap(*a, *b)));

template<class T1, class T2>
struct pair
{
    void swap(pair& p) noexcept(noexcept(swap(first, p.first)) && noexcept(swap(second, p.second)));
};
```
더 높은 수준의 자료구조들의 교환이 noexcept인지 여부는 해당 자료구조를 구성하는 더 낮은 수준의 구성요소들의 교환이 noexcept인가 여부에 의해 결정된다.

## 주의
함수에 noexcept을 선언할지 여부는 해당 함수의 구현이 예외를 방출하지 않는다는 성질을 오래 유지할 결심이 선 경우에만 선언해야 한다.
<br>
만약 함수를 noexcept으로 선언한 이후 나중에 마음을 바꾼다면 클라이언트 코드가 깨지는 것을 방지할 방법이 없다.

대부분의 함수는 예외 중립적(exception-neutral)하기 때문에 스스로 예외를 던지지 않더라도 예외를 던질 수 있는 함수를 호출할 수 있고, 이 경우 그 예외를 통과시키기 때문에 예외 중립적인 함수는 noexcept가 될 수 없음
<br>
또한, noexcept을 선언하기 위해 함수의 구현을 비틀어서 굳이 반환 값으로 함수의 상태를 표현하는 등의 불필요한 구현은 오히려 noexcept을 통한 최적화보다 더 안좋을 수 있고, 소스코드를 이해하고 유지보수하는데 더 어려움

메모리 해제 과정에서 예외를 방출하는 것은 나쁜 코딩 스타일로 간주되며 모든 메모리 해제 함수와 모든 소멸자는 암묵적으로 noexcept이므로 명시적으로 noexcept을 선언할 필요는 없음
<br>
noexcept(false)를 선언한 소멸자를 갖는 클래스 외에는 모든 소멸자는 암묵적으로 noexcept

넓은 계약을 갖는 함수들은 전제조건이 없는 함수를 의미하며, 그런 함수는 프로그램의 상태와 무관하게 호출될 수 있고, 호출자가 전달하는 인수들에 그 어떤 제약도 가하지 않는다.
<br>
넓은 계약이 아닌 함수들은 모두 좁은 계약을 가진 함수로 함수의 전제조건이 위반되면 그 결과는 미정의 행동이다.
<br>
넓은 계약의 함수에서는 noexcept을 선언하는 것이 쉽지만 좁은 계약에서는 전제 조건이 위반되었음을 알리기 위해 예욀르 던지는 것이 일반적이기 때문에 noexcept을 사용하기 어려움

noexcept 선언은 실제 함수 구현과 예외 명세 사이의 일관성을 보장하지 않음.
<br>
noexcept을 선언한 함수의 구현에서 noexcept을 선언한 함수를 사용하더라도 컴파일러는 어떠한 경고를 보여주지 않음.

## 정리
noexcept는 함수의 인터페이스의 일부이다. 이는 호출자가 noexcept여부에 의존할 수 있음을 의미한다.
<br>
noexcept 함수는 non-noexcept 함수보다 최적화의 여지가 크다
<br>
noexcept는 이동 연산들과 swap, 메모리 해제 함수들, 소멸자들에 특히 유용하다
<br>
대부분의 함수는 noexcept이 아니라 예외에 중립적이다.

# 항목 15 : 가능하면 항상 constexpr을 사용해라
constexpr은 어떠한 값이 단지 상수일 뿐만 아니라 컴파일 시점에 알려진다는 점을 의미
<br>
그런데 함수에 적용되면 반드시 const인 것을 보장할 수 없고 반드시 컴파일 시점에서 알려진다는 보장을 할수 없다?

```cpp
int sz;                             // non-constexpr 변수
constexpr auto arraySize1 = sz;     // sz의 값이 compile time에 알려지지 않았으므로 컴파일 오류
std::array<int, sz> data1;          // sz의 값이 compile time에 알려지지 않았으므로 컴파일 오류

const auto arraySize2 = sz;         // arraySize2는 sz의 const 복사본
std::array<int, arraySize2> data2;  // compile 시점에 arraySize2의 값이 알려져있지 않으므로 컴파일 오류

constexpr auto arraySize3 = 10;     // compile 시점에 알려져있는 상수
std::array<int, arraySize3> data3;  // 정상적으로 compile
```
컴파일 시점에 알려지는 값들은 Readonly Memory에 배치되는 등 특별한 권한이 있음
<br>
그 중 가장 중요한 점은 상수이자 컴파일 시점에 알려진 정수 값을 C++에서 정수 상수 표현식(integral constant expression -> 배열의 크기, 정수 템플릿 인수(std::array 객체의 길이), enum의 값, alignment 지정자 등)이 요구되는 문맥에서 사용할 수 있다는 점

```cpp
// pow는 base와 exp가 compile time에 상수일 경우에만 pow의 결과를 compile time에 상수로 사용할 수 있다
constexpr int pow(int base, int exp) noexcept { ... }

constexpr auto numConds = 5;
std::array<int, pow(3, numConds)> results;  // results는 3^numConds 개의 요소들을 담는다
```
constexpr 함수는 컴파일 시점 상수를 인수로해서 호출하는 경우에는 컴파일 시점 상수를 산출하고, 실행 시점 상수를 인수로해서 호출하는 경우에는 실행시점에 값을 산출한다.
1. 컴파일 시점 상수를 요구하는 문맥에서 constexpr 함수를 사용할 수 있다.
   1. 그런 문맥에서 만일 constexpr 함수에 넘겨주는 인수의 값이 컴파일 시점에 알려진다면, 함수의 결과는 컴파일 도중에 계산된다.
   2. 인수의 값이 컴파일 시점에 알려지지 않는다면 컴파일이 거부된다.
2. 컴파일 시점에서 알려지지 않는 하나 이상의 값들로 constexpr 함수를 호출하면 함수는 보통의 함수처럼 작동한다.
3. 즉, 컴파일 시점 상수를 위한 버전과 다른 모든 값을 위한 버전을 나누어서 구현할 필요가 없음을 의미한다.

```cpp
constexpr int pow(int base, int exp) noexcept
{
    return (exp == 0 ? 1 : base * pow(base, exp - 1));
}
```
C++11에서는 constexpr 함수가 실행 가능한 문장이 '많아야 1개'라는 조건이 있음
<br>
결국 return문에서 처리를 해야하는데, 표현의 확장을 위해 조건부 연산자(삼항 연산자)를 사용하거나 재귀 구문을 사용하는 방법이 있음

```cpp
// C++11
class Point
{
public:
    constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}

    constexpr double xValue() const noexcept { return x; }
    constexpr double yValue() const noexcept { return y; }

    void setX(double newX) { x = newX; }    // void 반환 형식은 C++11에서 literal type이 아님. 또한 constexpr 객체의 멤버 변수인 x는 암묵적으로 const로 선언됨
    void setY(double newY) { y = newY; }    // void 반환 형식은 C++11에서 literal type이 아님. 또한 constexpr 객체의 멤버 변수인 y는 암묵적으로 const로 선언됨

private:
    double x, y;
};

constexpr Point p1(9.4, 27.7);

constexpr Point midPoint(const Point& p1, const Point& p2) noexcept
{
    return { (p1.xValue() + p2.xValue()) / 2, (p1.yValue() + p2.yValue()) / 2 };
}

constexpr auto mid = midPoint(p1, p2);
```
constexpr 함수는 반드시 literal type을 받고 돌려주어야 한다. 즉, compile time에 값을 결정할 수 있는 형식을 의미
<br>
C++11에서는 void를 제외한 모든 내장 형식이 literal type. 생성자와 적절한 멤버 함수들이 constexpr인 사용자 형식도 literal 형식이 될 수 있음
<br>
Point의 생성자가 constexpr로 선언할 수 있는 이유는 주어지는 인수들이 compile time에 알려진다면 생성된 point 객체의 자료 멤버들의 값 역시 compile time에 알려질 수 있기 때문. 즉, Point 객체는 constexpr 객체가 될 수 있음
<br>
또한 getter 함수 역시 compile time에 조회가 가능하다면 constexpr이 될 수 있음
<br>
Point 조회 함수들을 호출한 결과들로 또 다른 constexpr 함수를 작성할 수 있음

```cpp
// C++13
class Point
{
public:
    constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}

    constexpr double xValue() const noexcept { return x; }
    constexpr double yValue() const noexcept { return y; }

    constexpr void setX(double newX) { x = newX; }    // C++14에서는 C++11의 void 제약과 constexpr객체의 const 멤버 변수 제약이 사라지므로써 constexpr setter가 가능
    constexpr void setY(double newY) { y = newY; }    // C++14에서는 C++11의 void 제약과 constexpr객체의 const 멤버 변수 제약이 사라지므로써 constexpr setter가 가능

private:
    double x, y;
};

constexpr Point p1(9.4, 27.7);

constexpr Point midPoint(const Point& p1, const Point& p2) noexcept
{
    return { (p1.xValue() + p2.xValue()) / 2, (p1.yValue() + p2.yValue()) / 2 };
}

constexpr auto mid = midPoint(p1, p2);

constexpr Point reflection(const Point& p) noexcept
{
    Point result;

    result.setX(-p.xValue());
    result.setY(-p.yValue());

    return result;
}

constexpr auto reflectedMid = reflection(mid);
```
C++14에서는 C++11에서 제한되었던 void type의 literal 판정과 constexpr 객체의 멤버 변수의 암묵적 const 조건이 사라지면서 setter함수 역시 constexpr을 붙일 수 있게 되었음

즉, constexpr을 사용하므로써 더 넓은 문맥에서 사용이 가능하기 때문에 '가능한 한' 항상 constexpr을 사용하는 것을 추천
<br>
constexpr은 객체나 함수의 인터페이스의 일부. 즉, constexpr을 지정한다는 것은 '이 함수(또는 객체)를 C++이 상수 표현식을 요구하는 문맥에서 사용할 수 있다'는 것을 의미하며 클라이언트는 해당 구문을 믿고 그 객체나 함수를 그런 문맥에서 사용할 것이다.
<br>
'가능한 한'이란 constexpr을 가능하게 하기 위해 객체나 함수를 적용한 제약들을 최대한 오래 유지하겠다는 의지를 의미

## 정리
constexpr 객체는 const이며, 컴파일 도중에 알려지는 값들로 초기화된다.
<br>
constexpr 함수는 그 값이 컴파일 도중에 알려지는 인수들로 호출하는 경웨는 컴파일 시점 결과를 산출한다.
<br>
constexpr 객체나 함수는 non-constexpr 객체나 함수보다 광범위한 문맥에서 사용할 수 있다.
<br>
constexpr은 객체나 함수의 인터페이스의 일부이다.

# 항목 16 : const member function을 Thread Safe하게 작성하라
```cpp
class Polynomial
{
public:
    using RootsType = std::vector<double>;

    RootsType roots() const
    {
        if(!rootsAreVaild)
        {
            ...
            rootsAreValid = true;
        }
        return rootVals;
    }

private:
    mutable bool rootsAreValid { false };
    mutable RootsType rootVals { };
};
```
임의의 다항식을 나타내는 클래스가 존재할 때 다항식의 근을 제공하는 함수를 제공하는 클래스를 구현한다.
<br>
이 때 근을 계산하는 함수를 임의의 캐시에 저장한 후 캐시에 있는 값을 반환하는 형태로 구현한다고 가정한다.
<br>
roots함수는 다항식을 변형하지 않으므로 const member function이어야 하지만, roots함수에서 사용되는 rootsAreVaild나 rootVals는 변형 되어야 하므로 mutable해야 한다.

```cpp
// Thread 1
auto rootsOfP = p.roots();
// Thread 2
auto valsGivingZero = p.roots();
```
root함수는 const member function으로 읽기 연산을 의미한다.
<br>
즉, const member function으로써 다중 스레드에서 제공이 가능해야 한다.
<br>
하지만 위 방식으로 구현된 함수는 thread safe하지 못하다.
<br>
이 문제를 해결하기 위해 mutex를 사용하거나 std::atomic을 사용해야 한다
<br>
하지만 mutex와 std::atomic을 사용하는 순간 복사와 이동 연산은 불가능하다.

```cpp
class Widget
{
public:
    int magicValue() const
    {
        if(cacheValid)
        {
            return cachedValue;
        }
        else
        {
            auto val1 = expensiveComputation1();
            auto val2 = expensiveComputation2();
            cachedValue = val1 + val2;
            cachedValid = true;
            return cachedValue;
        }
    }

private:
    mutable std::atomic<bool> cacheValid { false };
    mutable std::atomic<int> cachedValue;
};
```
std::atomic이 mutex보다 비용이 싸지만 동기화가 필요한 변수가 둘 이상인 경우에는 문제가 발생할 수 있음.
<br>
둘 이상의 변수의 동기화를 관리하려면 mutex를 사용하는 것이 좋음
<br>
const member function을 호출하는 스레드가 많아야 1개라는 점을 보장할 수 있다면 mutex나 std::atomic을 사용하지 않는 편이 비용을 절약할 수 있음
<br>
하지만 그런 상황은 매우 희귀하므로 cosnt member function은 항상 Thread safe하게 작성해야 한다.

## 정리
동시적 문맥에서 쓰이지 않을 것이 확실한 경우가 아니라면, const member function은 Thread safe하게 작성해야 한다
<br>
std::atomic 변수는 mutex에 비해 성능상의 이점이 있지만, 하나의 변수 또는 메모리 장소를 다룰 때에만 적합하다.

# 항목 17 : 특수 멤버 함수들의 자동 작성 조건을 숙지하라
특수 멤버 함수(Special member function)이란 C++이 스스로 작성하는 멤버 함수를 의미
<br>
C++98에서는 기본 생성자, 소멸자, 복사 생성자, 복사 배정 연산자가 존재
<br>
이 함수들은 클래스에 명시적으로 선언되어있지 않지만, 이 함수를 사용하는 클라이언트 코드가 존재할 때에만 작성된다
<br>
인수를 받는 생성자가 명시적으로 선언된 클래스는 기본 생성자가 작성되지 않는다.
<br>
특수 멤버 함수는 암묵적으로 public, inline 하다
<br>
가상 소멸자를 상속하는 파생 클래스의 소멸자 역시 virtual이다.

```cpp
class Widget
{
public:
    Widget(Widget&& rhs);

    Widget& operator=(Widget&& rhs);
};
```
C++11은 이동 생성자, 이동 배정 연산자를 특수 멤버 함수로 추가
<br>
이동 생성자, 이동 배정 연산자는 비정적 멤버 변수를 멤버별로 이동 '요청'을 수행하고, std::move가 적용되지 않을 경우 복사 연산이 수행된다.

복사 생성자와 복사 배정 연산자는 독립적
- 둘 중 하나가 이미 선언되어 있고, 선언되지 않은 함수를 클라이언트에서 호출한다면 선언되어있지 않은 함수가 자동으로 생성됨

이동 생성자와 이동 배정 연산자는 독립적이지 않음
- 둘 중 하나를 선언하면 컴파일러는 다른 하나를 작성하지 않음
- 만약 둘 중 하나를 프로그래머가 선언했다면 이는 컴파일러에서 작성하는 이동 연산이 적합하지 않을 가능성이 크므로 남은 연산 역시 적합하지 않을 것이라는 판단 하에 작성하지 않는 것이 이유
- 마찬가지 이유로 복사 연산(생성, 배정)이 선언되었다면 이동 연산들은 자동으로 작성되지 않고, 반대로 이동 연산이 선언되었다면 복사 연산은 자동으로 생성되지 않음

### 3의 법칙
복사 생성자, 복사 배정 연산자, 소멸자 중 하나라도 선언했다면 나머지 둘도 선언해야 한다
<br>
이는 해당 연산자를 선언하는 이유는 해당 클래스가 어떠한 의미로든 자원을 관리해야 하기 때문이다.
<br>
자동으로 작성된 복사 연산들은 자원 관리에 적합하지 않다.
<br>
반대로 클래스에 소멸자가 선언되어 있다면 해당 클래스는 자동으로 생성되는 복사 연산자가 적합하지 않을 가능성이 높다.
<br>
이런 추론을 바탕으로 C++11부터는 소멸자가 선언된 클래스에서는 자동으로 이동 연산자를 작성하지 않는다.(복사 연산자는 그대로 작성하는 이유는 단지 기존 코드와의 호환성 문제로 인한 것이다)

이동 연산자는 다음 세 조건이 모두 만족할 경우에 필요에 의해서 자동으로 작성된다
1. 클래스에 그 어떤 복사 연산도 선언되어 있지 않다.
2. 클래스에 그 어떤 이동 연산도 선언되어 있지 않다.
3. 클래스에 소멸자가 선언되어 있지 않다.

```cpp
class Widget
{
public:
    ~Widget();

    Widget(const Widget&) = default;

    Widget& operator=(const Widget&) = default;
};
```
복사 연산자에 대해서도 위 조건에 의해 명시적으로 선언하는 것을 추천하는데, 컴파일러가 작성하는 행동을 명시적으로 선언하고싶다면 default를 사용하면 된다

```cpp
// https://godbolt.org/z/KrxEYf4WG

#include <iostream>

class TestClass
{
public:
    TestClass() = default;
    TestClass(const TestClass&& r) { std::cout << "Move Constructor" << std::endl; }
    TestClass(const TestClass& r) { std::cout << "Copy constructor" << std::endl; }
    TestClass& operator=(const TestClass&& r) { std::cout << "Move Assignment" << std::endl; }
    TestClass& operator=(const TestClass& r) { std::cout << "Copy Assignment" << std::endl; }
};

class WrappingClass
{
public:
    WrappingClass() { std::cout << "Constructor" << std::endl; }

    ~WrappingClass() { std::cout << "Destructor" << std::endl; }

private:
    TestClass Test;
};

int main()
{
    StringTable origin;
    StringTable Copy = std::move(origin);

    return 0;
}

// Result without Destructor
// Constructor
// Move Constructor

// Result with Destructor
// Constructor
// Copy constructor
// Destructor
// Destructor
```
또한 default를 사용하는 것이 프로그래머의 의도를 명시적으로 알리는 것일 뿐만 아니라 미묘한 버그를 피하는데도 도움이 되므로, 컴파일러가 작성할 수 있는 연산자들도 명시적으로 default 선언을 해주는 것이 좋음
<br>
이동 요청은 std::move를 수행했을 때 수행이 불가능하다면 복사를 수행함
<br>
소멸자가 추가되므로써 WrappingClass의 이동 연산자는 자동으로 작성되지 않았고, default 상황에서는 이동 생성자로 생성되었을 클래스 멤버 변수가 복사 생성자로 생성되는 것을 확인할 수 있음

## 특수 멤버 함수 규칙
### 기본 생성자
클래스에 사용자 선언 생성자가 없으면 자동으로 작성

### 소멸자
소멸자가 기본적으로 noexcept이다.
<br>
기반 클래스 소멸자가 virtual일 경우 기본으로 작성되는 파생 클래스의 소멸자도 virtual이다.

### 복사 생성자
non-static 멤버 변수들을 멤버별로 복사 생성.
<br>
클래스에 사용자 선언 복사 생성자가 없을 경우에 자동으로 작성
<br>
클래스에 사용자 선언 이동 연산이 하나라도 선언되어있으면 delete 처리
<br>
사용자 선언 복사 배정 연산자나 소멸자가 있는 클래스에서 자동으로 작성되는 복사 생성자는 비권장 기능

### 복사 배정 연산자
non-static 멤버 변수들을 멤버별로 복사 배정
<br>
클래스에 사용자 선언 복사 배정 연산자가 없을 경우에 자동으로 작성
<br>
클래스에 사용자 선언 이동 연산이 하나라도 선언되어 있으면 delete 처리
<br>
사용자 선언 복사 생성자나 소멸자가 있는 클래스에서 자동으로 작성되는 복사 배정 연산자는 비권장 기능

### 이동 생성자와 이동 배정 연산자
각각 non-static 멤버 변수의 멤버별 이동을 수행
<br>
클래스에 사용자 선언 복사 연산들과 이동 연산들, 소멸자가 없을 경우에만 자동으로 작성

### 참고
```cpp
class Widget
{
public:
    template<typename T>
    Widget(const T& rhs);

    template<typename T>
    Widget& operator=(const T& rhs);
}
```
멤버 함수 템플릿이 존재하더라도 특수 멤버 함수는 조건만 성립하면 자동으로 작성된다.
<br>
위 코드의 경우 T가 Widget이면 서명이 일치하여 인스턴스화 될 수 있지만 Widget은 여전히 복사 연산들과 이동 연산들을 작성한다

## 정리
컴파일러가 스스로 작성할 수 있는 멤버 함수들, 즉 기본 생성자와 소멸자, 복사 연산들, 이동 연산들을 가리켜 특수 멤버 함수라고 부른다.
<br>
이동 연산들은 이동 연산들이나 복사 연산들, 소멸자가 명시적으로 선언되어 있지 않은 클래스에 대해서만 자동으로 작성된다.
<br>
복사 생성자는 복사 생성자가 명시적으로 선언되어 있지 않은 클래스에 대해서만 자동으로 작성되며, 만일 이동 연산이 하나라도 선언되어 있으면 삭제된다.
<br>
복사 배정 연산자는 복사 배정 연산자가 명시적으로 선언되어 있지 않은 클래스에 대해서만 자동으로 작성되며, 만일 이동 연산이 하나라도 선언되어 있으면 삭제된다.
<br>
소멸자가 명시적으로 선언된 클래스에서 복사 연산들이 자동으로 작성되는 기능은 비권장이다.
<br>
멤버 함수 템플릿 때문에 특수 멤버 함수의 자동 작성이 금지되는 경우는 없다.