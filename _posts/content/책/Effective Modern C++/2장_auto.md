# 항목 5 : 명시적 형식 선언보다는 auto를 선호하라
auto 변수의 형식은 항상 해당 초기치로부터 연역되므로, 반드시 초기치가 필요하다.
<br>
C++14에서는 Lambda의 Parameter에서도 auto를 사용할 수 있음
```cpp
auto derefLess = [](const auto& p1, const auto& p2) { return *p1 < *p2; }  // Pointer처럼 작동하는 것들이 가리키는 값을 비교하는 함수
```
std::function을 통해 Lambda을 지정할 수 있지만 매개변수의 형식이 너무 길어지는 문제가 발생할 수 있음
<br>
또한, auto 변수는 클로저에 요구되는 메모리만큼만 사용하지만 std::function 변수는 임의의 주어진 서명에 대하여 크기가 고정적이며, 때로는 클로저를 저장하기에 부족할 수도 있음
<br>
또한, std::function 객체는 auto 객체보다 호출이 느리다.

## 형식 단축 문제(type shortcut)

```cpp
std::vector<int> v;
unsigned sz = v.size();  // v.size()의 실제 형식은 std::vector<int>::size_type인데 32비트 운영체제와 64비트 운영체제에서 문제가 발생할 수 있음.
auto sz = v.size();      // auto를 이용하여 이식 중에 발생할 문제를 쉽게 해결할 수 있음
```
```cpp
std::unordered_map<std::string, int> m;
for(const std::pair<std::string, int>& p : m)
{ ... }
```
위 코드의 경우 std::unordered_map의 std::pair의 형식은 std::pair<const std::string, int>의 형식으로 위 코드로 작성할 경우 매 반복문마다 임시 객체를 생성하는 문제 발생.

즉, 명시적으로 형식을 지정함으로써 원치않는 암묵적 변환이 발생할 수 있음.

## 가독성(readability)
auto를 사용함으로써 객체의 형식을 직관적으로 파악하기는 힘들지만, 변수의 이름을 잘 작성했다면 추상적으로 짐작하는 형식 정보는 거의 바로 파악이 가능하다.
<br>
오히려 auto를 사용하므로써 리팩토링이 더 수월해지는 상황이 발생할 수도 있고, 정확성과 효율성에 더 도움이 되기도 한다.

## 정리
auto 변수는 반드시 초기화해야 하며, 이식성 또는 효율성 문제를 유발할 수 있는 형식 불일치가 발생하는 경우가 거의 없으며, 대체로 변수의 형식을 명시적으로 지정할 때보다 타자량도 더 적다.
<br>
auto로 형식을 지정한 변수는 항목 2와 항목 6에서 설명한 문제점들을 겪을 수 있다.

# 항목 6 : auto가 원치 않은 형식으로 연역될 때에는 명시적 형식의 초기치를 사용하라
```cpp
std::vector<bool> features(const Widget& w);

Widget w;

// 정상 작동
bool highPriority = features(w)[5];
processWidget(w, highPriority);

// 정의되지 않은 행동
auto highPriority = features(w)[5];
processWidget(w, highPriority);
```
std::vector<bool>은 std::vector<bool>::operator[]()에 대하여 std::vector<bool>::reference 형식을 반환하므로써 발생하는 문제
<br>
이는 std::vector<bool>이 자신의 bool들의 정보를 1비트의 압축된 형태로 표현하도록 명시되어있는데, C++에서는 비트에 대한 Reference가 불가능해서 bool&처럼 작동하는 객체를 반환하면서 발생하는 문제.
<br>
이후 std::vector<bool>::reference는 bool&가 아닌 bool을 암묵적으로 변환한다.

이처럼 Proxy Class를 사용하는 객체는 임시 객체를 반환하고 해당 임시 객체는 그 줄이 끝나면 파괴되는 것을 가정하기 때문에 auto를 이용할 경우 dangling pointer 문제가 발생할 수 있음.
<br>
이를 피하기 위해 해당 라이브러리의 문서를 통해 대리자를 사용하는지 확인하거나 헤더 파일을 통해 확인할 수 있음
```cpp
namespace std
{
  template<class Allocator>
  class vector<bool, Allocator>
  {
    public:
      class reference { ... }
      reference operator[](size_type n) { ... }
  }
}
```
위의 코드처럼 operator[]가 T&를 반환하지 않는 점같이 생소한 표현식을 통해 짐작할 수 있음.

## Proxy Class를 연역해야 하는 경우
auto가 다른 형식을 연역하도록 강제해야 하기 위해 형식 명시 초기치 관용구(Explicitly typed initialzer idiom)을 사용.
```cpp
auto hightPriority = static_cast<bool>(features(w)[5]);
```
즉, auto 변수를 사용하되 명시적으로 casting하여 값을 받는 방식.

```cpp
double calcEpsilon();

float ep = calcEpsilon();                      // 암묵적 변환이 발생하여 casting이 의도적인 것인지를 알 수 없음.
float ep = static_cast<float>(calcEpsilon());  // 명시적으로 casting하여 의도했다는 것을 알 수 있음.
```

## 정리
"보이지 않는" 대리자 형식 때문에 auto가 초기화 표현식의 형식을 잘못 연역할 수 있다.
<br>
형식 명시 초기치 관용구는 auto가 원하는 형식을 연역하도록 강제한다.
