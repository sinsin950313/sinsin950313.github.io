# 항목 7 : 객체 생성 시 괄호와 중괄호를 구분하라
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
