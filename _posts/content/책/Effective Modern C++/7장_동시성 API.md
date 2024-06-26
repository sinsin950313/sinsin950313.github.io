# 항목 35 : 스레드 기반 프로그래밍보다 과제 기반 프로그래밍을 선호하라
```cpp
int doAsyncWork();

std::thread t(doAsyncWork);         // thread 기반 프로그래밍

auto fut = std::async(doAsyncWork); // task 기반 프로그래밍
int ret = fut.get();                // return 값에 접근 가능
```
스레드 기반 프로그래밍은 doAsyncWork의 반환값에 접근할 수 없으나, task 기반 프로그래밍은 future객체를 통해 반환값에 접근할 수 있음
<br>
특히 만약 doAsyncWork가 예외를 방출한다면 get을 통해 예외에 접근할 수 있음. 스레드 기반 프로그래밍은 예외 방출 시 std::terminate 호출로 인해 프로그램이 죽는다.

## Thread에 대한 3가지 의미
1. 실제 계산을 수행하는 스레드를 뜻하는 Hardware Thread. 현 세대의 컴퓨터 아키텍쳐는 CPU 코어당 하나 이상의 하드웨어 스레드를 제공
2. 운영체제가 하드웨어 스레드들에서 실행되는 모든 프로세서와 일정을 관리하는데 사용하는 Software Thread. OS 스레드나 시스템 스레드라고도 한다. 하드웨어 스레드보다 많은 소프트웨어 스레드를 생성할 수 있음. 한개의 소프트웨어 스레드가 block(입출력 대기, 뮤텍스 대기 등) 되어도 block되지 않은 다른 소프트웨어 스레드들을 실행함으로써 산출량을 향상.
3. C++ 표준 라이브러리의 std::thread. 하나의 C++ 프로세스 안에서 std::thread 객체는 Software Thread에 대한 Handle로 작동. 만약 std::thread 객체가 null Handle이라면 이는 Software Thread에 대응되지 않는다는 의미.(기본 생성자, move, join으로 인해 다른 thread와 통합, detach를 통해 탈착된 후)

```cpp
int doAsyncWork() noexcept;

std::thread() t(doAsyncWork);   // noexcept함수를 실행하지만 예외가 발생할 수 있음
```
Software Thread는 제한된 자원으로 시스템이 제공할 수 있는 것보다 많은 양을 생성하려고 한다면 std::system_error 예외가 발생
<br>
즉, thread가 실행하는 함수가 noexcept이더라도 예외가 발생할 수 있음

또는 OverSubscription으로 인해 문제가 발생할 수 있음
<br>
OverSubscription이란 non-block 상태의 Software Thread가 Hardware Thread보다 많은 경우 발생. Context Switching으로 인한 overhead 문제와, Context Switching 이전에 수행되던 Thread가 사용하던 Cache의 오염 문제가 발생할 수 있음.

이러한 문제는 프로그래머가 그 비율을 적절히 조절하더라도 하드웨어에 따라 달라질 수 있기 때문에 사실상 프로그래머에 의해 관리하는 것이 불가능.
<br>
대신 std::async를 이용하여 스레드 관리 부담을 표준 라이브러리에 떠넘기는 방식으로 해결.
<br>
std::async는 실제로 스레드를 생성하지 않고 doAsyncWork의 결과가 필요한 스레드에서 직접 실행하는 방식으로, 즉 future에 대해 get이나 wait를 호출하는 스레드에서 직접 실행하도록 스케줄러에게 요청할 수 있는 방식으로 문제를 해결.
<br>
단, 이 기법은 부하 불균형 문제가 발생할 수 있음. 그러나 마찬가지로 이 역시 프로그래머보다 스케줄러가 더 잘 관리할 가능성이 높음.

만약, 반응성이 좋아야 하는 스레드를 특별히 지정해야 한다면 std::launch::async라는 Launch Policy를 std::async에 넘겨주는 것이 좋음.
<br>
이 경우 실행하고자 하는 함수가 실제로 현재 스레드와는 다른 스레드에서 실행됨.

즉, std::thread를 직접 다루는 방식의 프로그래밍은 스레드 고갈, 과다구독, 부하 불균형을 프로그래머가 직접 처리해야 하는 문제가 있다.

## 스레드를 직접 다루는 것이 적합한 경우
1. Base Thread Library의 API에 접근해야 하는 경우
   1. Windows의 thread library나 Linux의 pThread 같은 곳에 직접 접근해야 하는 경우 흔히(없을수도 있음) native_handle 멤버 함수를 통해 접근할 수 있음
2. 응용 프로그램의 스레드 사용량을 최적화해야하는, 그리고 할 수 있어야 하는 경우
   1. 하드웨어 특성이 미리 정해진 컴퓨터에서 유일하게 의미있는 프로세스로 실행될 서버 소프트웨어를 개발하는 경우
3. C++ 동시성 API가 제공하는 것 이상의 스레드 적용 기술을 구현해야 하는 경우
   1. 독자가 사용하는 C++ 구현이 스레드 풀을 제공하지 않는 특정 플랫폼을 위해 스레드 풀을 직접 구현해야하는 경우

대부분 흔치 않은 경우이다.

## 정리
std::thread API에서는 비동기적으로 실행된 함수의 반환값을 직접 얻을 수 없으며, 만일 그런 함수가 예외를 던지면 프로그램이 종료된다.
<br>
Thread 기반 프로그래밍에서는 스레드 고갈, 과다 구독, 부하 균형화, 새 플랫폼으로의 이식 시 적응을 독자가 직접 처리해야 한다.
<br>
std::async와 기본 Launch 방침을 이용한 Task 기반 프로그램밍은 그런 대부분의 문제를 알아서 처리해준다.

# 항목 36 : 비동기성이 필수일 때에는 std::launch::async를 지정하라
```cpp
auto fut1 = std::async(f);
auto fut2 = std::async(std::launch::async | std::launch::deferred);  // 두 함수의 시동 방침은 동일하다
```
std::async는 std::launch 범위의 enum에 정의된 열거자들을 이용해 시동 방침을 결정할 수 있음.

## std::launch::async
함수 f는 반드시 비동기적으로, 즉, 다른 스레드에서 실행된다.

## std::launch::deffered
함수 f는 std::async가 돌려준 std::future 객체에 대해 get이나 wait가 호출될 때에만 실행될 수 있다. 즉, 함수 f는 그런 호출이 일어날 때까지 지연(Deferred)된다.
<br>
get이나 wait가 호출되면 f는 동기적으로 실행된다. 즉, 호출자는 f의 실행이 종료될 때까지 block 된다.
<br>
get이나 wait가 호출되지 않으면 f는 결코 실행되지 않는다.
<br>
위 시동 방침은 동일한 의미를 같는다. 즉, 기본 시동 방침은 f를 비동기적으로 실행할 수 도 있고, 동기적으로 실행할 수 도 있다.

```cpp
// t Thread
auto fut = std::async(f);
```
이와 같은 유연성은 이전 항목에서 지적한 스레드 관리에 대한 부담을 줄여주지만, 이로 인해 다양한 문제를 유발한다.
1. 위 코드의 경우 f는 지연 실행될 수 도 있으므로, f가 t와 동시에 실행될지 예측하는 것이 불가능하다.
2. f가 fut에 대해 get이나 wait를 호출하는 스레드와는 다른 스레드에서 실행될지 예측하는 것이 불가능하다.
3. 프로그램의 모든 가능한 경로에서 fut에 대한 get이나 wait가 호출될 것이라는 보장할 수 없으므로, f가 반드시 실행될 것인지 예측하는 것이 불가능하다.

기본 시동 방침은 thread_local 변수들의 사용과는 궁합이 잘 맞지 않는 경우도 많다.
<br>
f에서 thread-local storage(TLS)에 접근한다면, 그 코드가 어떤 스레드의 지역 변수에 접근할 지 예측할 수 없기 때문이다.
<br>
위 코드의 경우 f의 TLS가 독립적인 스레드의 것일 수도 있고, fut에 대해 get이나 wait를 호출하는 스레드의 것일 수도 있다.

```cpp
using namespace std::literals;

void f()
{
   std::this_thread::sleep_for(1s);
}

auto fut = std::async(f);

while(fut.wait_for(100ms) != std::future_status::ready) { ... }   // 무한 loop에 빠질 수 있음

// =============================================================================

if(fut.wait_for(0s) == std::future_status::deferred)  // 과제가 지연된 경우
{
   ... // fut에 wait나 get을 적용해서 f를 동기적으로 호출
}
else  // 과제가 지연되지 않음
{
   while(fut.wait_for(100ms) != std::future_status::ready)  // 무한 loop는 불가능
   {
      ... // 과제가 지연되지도, 준비되지도 않았으므로 준비될 때까지 동시적 작업을 수행
   } 
   ...   // fut는 이제 준비되었다.
}
```
wait기반 루프에도 영향을 미친다.
<br>
wait_for나 wait_until을 호출하면 std::future_status::deferred라는 값이 반환되는데 이로 인해 언젠가는 끝날 것처럼 보이는 루프가 실제로는 무한히 실행될 수도 있다.
<br>
위 코드의 경우 f가 std::async를 호출한 스레드와 동시에 실행된다면 문제가 없지만, f가 지연된다면 std::future_status::deferred라 반환되므로 루프는 절대 종료되지 않는다.
<br>
이런 문제의 해결책은 std::async 호출이 돌려준 std::future객체를 이용해 해당 task가 deferred되었는지 확인한 후, deferred되었다면 시간 만료 기반 루프에 진입하지 않도록 하면 된다.

## 기본 시동 방침으로 std::async를 사용하기 적합한 상황(deffered하게 수행되도 상관없는 상황)
1. Task가 get이나 wait를 호출하는 스레드와 반드시 동기적으로 실행되어야 하는 것은 아니다.
2. 여러 Thread 중 어떤 Thread의 Thread_local Variable 변수들을 읽고 쓰는지가 중요하지 않다.
3. std::async가 돌려준 future 객체에 대해 get이나 wait가 반드시 호출된다는 보장이 있거나, Task가 전혀 실행되지 않아도 괜찮다.
4. Task가 deferred 상태일 수 있다는 점이 wait_for나 wait_until을 사용하는 코드에 반영되어 있다.

```cpp
template<typename F, typename... Ts>
inline std::future<typename std::result_of<F(Ts...)>::type> reallyAsync(F&& f, Ts&&... params)
{
   return std::async(std::launch::async, std::forward<F>(f), std::forward<Ts>(parmas)...);
}
```
위 조건 중 하나라도 성립하지 않으면 std::async가 주어진 과제를 비동기적으로 실행하도록 강제할 필요가 있다.
<br>
그 방법은 std::launch::async를 첫 인수로 지정해서 std::async를 호출하는 것이다.

## 정리
std::async의 기본 시동 방침은 과제의 비동기적 실행과 동기적 실행을 모두 허용한다
<br>
그러나 이러한 유연성 때문에 thread_local 접근의 불확실성이 발생하고, 과제가 절대로 실행되지 않을 수도 있고, 시간 만료 기반 wait 호출에 대한 프로그램 논리에도 영향이 미친다.
<br>
과제를 반드시 비동기적으로 실행해야 한다면 std::launch::async를 지정하라.

# 항목 37 : std::thread들을 모든 경로에서 합류 불가능하게 만들어라.
모든 std::thread 객체는 joinable 상태 이거나 unjoinable 상태이다.

## 합류 가능 std::thread
바탕 실행 스레드 중 현재 실행 중 이거나 실행 중 상태로 전이할 수 있는 스레드
<br>
blocked 상태이거나 실행 일정을 기다리는 스레드
<br>
실행이 완료된 바탕 스레드

## 합류 불가능 std::thread
기본 생성된 스레드 : 실행할 함수가 없으므로 바탕 실행 스레드와 대응되지 않음
<br>
다른 std::thread 객체로 이동된 후의 std::thread 객체
<br>
join에 의해 합류된 std::thread
<br>
detach에 의해 탈착된 std::thread

합류 가능한 std::thread의 소멸자가 호출되면 프로그램 실행이 종료된다.
```cpp
constexpr auto tenMillition = 10'000'000; // C++14

// std::thread t가 먼저 수행되고 그 이후 doWork가 수행되는 우선순위를 갖기를 희망하는 코드
bool doWork(std::function<bool>(int) filter, int maxVal = tenMillion)
{
   std::vector<int> goodVals;
   std::thread t([&filter, maxVal, &goodVals]
                  {
                     for(auto i = 0; i <= maxVal; ++i)
                     {
                        if(filter(i))
                        {
                           goodVals.push_back(i);
                        }
                     }
                  });
   auto nh = t.native_handle();

   ...

   if(conditionsAreSatisfied())
   {
      t.join();
      perfomComputation(goodVals);
      return true;
   }
   return false;
}
```
위 코드는 만약 conditionsAreSatisfied가 실패하거나 예외를 던진다면 std::thread t의 소멸자가 호출되고, 이 때 t는 함류 가능하기 때문에, 이는 프로그램의 실행이 종료를 유발한다.

## joinable한 std::thread의 소멸자가 호출되면 프로그램이 종료되는 이유
1. 암묵적 join은 추적하기 어려운 성능 이상들이 나타날 수 있음. 예를 들어 위 코드에서 conditionsAreSatisfied == false인 상황에서 t가 join하기를 기다리는 것은 직관적이지 않다.
2. 암묵적 detach는 심각한 디버깅 문제를 유발한다. 예를 들어 위 코드에서 람다가 아직 종료되지 않은 상황에서 conditionsAreSatisfied == false가 되면 doWork의 지역변수인 goodVals를 참조하는 람다 함수들이 이미 해제된 메모리 위치를 수정하는 일이 벌어질 수 있다.

즉, joinable한 std::thread의 소멸자가 호출된 경우 프로그램을 종료하는 것이 가장 안정적인 결과를 불러온다.
<br>
따라서, std::thread 객체를 사용할 때 그 객체가 그것이 정의된 범위 바깥의 모든 경로(return, continue, break, goto 등으로 인해 만들어질 수 있는 모든 샛길 포함)에서 합류 불가능으로 만드는 것이 프로그래머의 책임이다.

한 범위의 바깥으로 나가는 모든 경로에서 어떤 동작이 반드시 수행되어야 할 때 흔히 사용하는 접근 방식은 그 동작을 지역 객체의 소멸자 안에 넣는 것.
<br>
그런 객체를 RAII 객체라 부르고, 그런 객체의 클래스를 RAII 클래스라고 부른다.(엄밀한 RAII의 정의는 아님)
<br>
이러한 RAII의 예제는 STL 컨테이너(각 컨테이너의 소멸자는 컨테이너의 내용을 파괴하고 메모리를 해제), 스마트 포인터(std::shared_ptr와 std::weak_ptr은 소멸자 시점에서 자신이 가리키는 객체의 참조 횟수를 감소시킴) 등이 있다.
<br>
std::thread의 RAII 클래스가 없는 이유는 해당 객체에 대한 표준적인 RAII 클래스가 어떤 것이어야 할지 파악할 수 없었기 때문

```cpp
class ThreadRAII
{
public:
   enum class DtorAction { join, detach };

   ThreadRAII(std::thread&& t, DtorAction a) : action(a), t(std::move(t)) { }
   ~ThreadRAII()
   {
      if(t.joinable())  // 합류 가능 여부 확인
      {
         if(action == DtorAction::join)
         {
            t.join();
         }
         else
         {
            t.detach();
         }
      }
   }

   std::thread&& get() { return t; }

private:
   DtorAction action;
   std::thread t;
};
```
RAII Thread를 직접 만드는 class의 작성은 위와 같다.
<br>
위 클래스의 특징은 다음과 같다.
<br>
1. std::thread는 복사할 수 없으므로 ThreadRAII는 rvalue만 생성자 매개변수로 받는다.
2. 생성자의 매개변수들은 호출자가 직관적으로 기억할 수 있는 순서로 선언하는 것이 좋지만, 멤버 초기화 순서는 자료 멤버의 의존 상태 순서대로 작성하는 것이 좋다. 특히 std::thread의 경우 초기화되자마자 해당 함수를 실행할 수 있으므로 제일 마지막에 선언하는 것이 좋다. 이를 통해 해당 비동기 스레드가 다른 모든 멤버에 안전하게 접근할 수 있다.
3. ThreadRAII는 바탕 std::thread 객체에 접근할 수 있는 get 함수를 제공한다. 이처럼 get함수를 제공함으로써 std::thread의 인터페이스 전체를 ThreadRAII에 복제할 필요가 없다.
4. ThreadRAII 소멸자는 std::thread t에 대해 멤버 함수를 호출하기 전에 먼저 t가 합류 가능인지부터 점검한다. 사용자는 get을 통해 접근한 std::thread 객체를 사용자가 직접 join이나 detach를 호출할 수 있고 만약 unjoinable한 객체에 join이나 detach할 경우 미정의 행동이 발생할 수 있다.
5. ```cpp
      if(t.joinable())
      {
         if(action == DtorAction::join)
         {
            t.join();
         }
         else
         {
            t.detach();
         }
      }
   ```
   위 구문은 소멸자에서 호출되므로 다수의 클라이언트에서 동시에 접근할 수 없는 상황이기 때문에 race condition을 부르지 않는다. 만약 일반 멤버 함수라면 race condition을 불러일으킬 수 있다. 일반 멤버 함수는 const 멤버 함수인 경우에만 다수의 클라이언트에서 동시에 호출하는 상황에 대해 안전하다.

```cpp
class ThreadRAII
{
public:
   enum class DtorAction { join, detach };

   ThreadRAII(std::thread&& t, DtorAction a) : action(a), t(std::move(t)) { }
   ~ThreadRAII()
   {
      if(t.joinable())  // 합류 가능 여부 확인
      {
         if(action == DtorAction::join)
         {
            t.join();
         }
         else
         {
            t.detach();
         }
      }
   }

   ThreadRAII(ThreadRAII&&) = default;
   ThreadRAII& operator=(ThreadRAII&&) = default;

   std::thread&& get() { return t; }

private:
   DtorAction action;
   std::thread t;
};

bool doWork(std::function<bool>(int) filter, int maxVal = tenMillion)
{
   std::vector<int> goodVals;

   ThreadRAII t(
      std::thread([&filter, maxVal, &goodVals]
      {
         for(auto i = 0; i <= maxVal; ++i)
         {
            if(filtedr(i))
            {
               goodVals.push_back(i);
            }
         }
      }, ThreadRAII::DtorAction::join)
   );

   auto nh = t.get().native_handle();

   ...

   if(conditionsAreSatisfied())
   {
      t.get().join();
      performComputation(goodVals);
      return true;
   }
   return false;
}
```
사용자는 해당 람다를 join으로 종료되도록 설정한다.
<br>
항목 39를 참조해보면 위 ThreadRAII가 프로그램이 멈추는 hang문제를 일으킬 수 있다. 이를 해결하기 위해서는 람다에게 더 이상 일할 필요가 없으니 일찍 반환하라고 알려줘야 하지만, C++11에서는 그런 가로챌 수 있는 스레드(Interruptible Thread)를 지원하지는 않는다.

## 정리
모든 경로에서 std::thread를 합류 불가능으로 만들어라.
<br>
소멸 시 join 방식은 디버깅하기 어려운 성능 이상으로 이어질 수 있다.
<br>
소멸 시 detach 방식은 디버깅하기 어려운 미정의 행동으로 이어질 수 있다.
<br>
자료 멤버 목록에서 std::thread 객체를 마지막에 선언하라.

# 항목 38 : 스레드 핸들 소멸자들의 다양한 행동 방식을 주의하라.
근본적으로 std::thread와 std::future는 모두 시스템 스레드에 대한 handle이라고 볼 수 있음.
<br>
하지만 joinable한 std::thread의 소멸자를 호출할 경우 프로그램이 종료되는 것과는 달리, std::future는 소멸자가 호출되면 어떨때는 join, 어떨때는 detach를 수행한 것과 같은 결과를 낸다. 물론 프로그램이 종료되지도 않는다.

std::future 객체는 피호출자가 자신이 수행한 결과를 호출자에게 전송하는 통신 채널이다.
<br>
피호출자는 자신의 계산 결과를 통신 채널에 기록한다.(std::promise 객체를 통해서)
<br>
호출자는 std::future 객체를 이용해서 그 결과를 읽는다.

이 때 std::promise 객체는 피호출자의 지역 범위에 있으므로, 피호출자가 완료되면 함께 사라진다.
<br>
std::future는 std::shared_future가 될 수 있으는데, 만약 연산의 결과가 복사를 지원하지 않는다면 해당 연산의 결과를 어디에 담아야 할지 불명확하다.
<br>
즉, 피호출자의 결과는 std::promise나 std::future 모두 저장하기에 적합하지 않다.
<br>
피호출자의 결과는 shared state에 저장된다.
<br>
이 때 std::future 객체 소멸자의 행동(join, detach)은 shared state가 결정한다.

1. std::async를 통해 시동된 non-deferred task에 대한 공유 상태를 참조하는 마지막 std::future 객체의 소멸자는 task가 완료될 때까지 차단된다. 즉, 암묵적으로 join된다.
2. 다른 모든 std::future 객체의 소멸자는 그냥 해당 std::future 객체를 파괴한다. 이는 비동기적으로 실행되고 있는 과제의 경우 바탕 스레드에 암묵적인 detach를 수행하는 것과 비슷하다.

결국 단 하나의 예외상황을 제외하고는 소멸자가 호출되는 시점에는 std::future 객체를 소멸시키는 정상적인 행동을 수행한다.
<br>
단 하나의 예외상황을 유발하는 조건은 다음과 같다
1. std::future 객체가 std::async 호출에 의해 생성된 shared state를 참조한다.
2. task의 launch policy가 std::launch::async이다. 이는 실행시점 시스템이 선택한 방식과 std::async에서 명시적으로 호출한 경우 모두 포함된다.
3. std::future 객체가 shared state를 참조하는 마지막 std::future 객체이다.

위 조건에 모두 부합하는 상황의 std::future의 소멸자는 std::async로 생성한 task를 실행하는 스레드에서 암묵적으로 join 작업을 수행한다.
<br>
간단하게 std::shared_ptr이 언제 메모리가 해제되는가와 비슷하게 이해하면 될 듯.

```cpp
std::vector<std::future<void>> futs;

class Widget
{
public:
   ...

private:
   std::shared_future<double> fut;
};
```
단, std::future 객체가 위 조건에 해당하는지 여부를 판단할 수 있는 수단은 제공되지 않는다.
<br>
위 코드는 컨테이너와 클래스에서 block이 걸릴지를 장담할 수 없다.

```cpp
int calcValue();

std::packaged_task<int()> pt(calcValue);

auto fut = pt.get_future();

std::thread t(std::move(pt));
...
```
std::packaged_task는 std::async가 아닌 호출에서 공유 상태를 만들어낸다.
<br>
이를 통해 만들어진 std::future객체는 std::async를 통해서 호출되지 않았으므로 소말자는 정상적으로 작동한다.
<br>
std::packaged_task는 복사할 수 없으므로 rvalue로 캐스팅해야 한다.

위 코드에서 ...의 부분에서 t에서 일어나는 일의 가능성은 3가지이다.
1. t에 아무 일도 일어나지 않는다.
<br>
이 경우, 범위의 끝에서 t는 joinable thread이다. 따라서 프로그램은 종료된다.
2. t에 대해 join을 수행한다.
<br>
호출코드에서 명시적으로 join을 수행하니 소멸자에서 join을 수행할 필요가 없으므로 block되지 않는다.
3. t에 대해 detach를 수행한다.
<br>
호출코드에서 명시적으로 detach를 수행하니 소멸자에서 detach를 수행할 필요가 없으므로 block되지 않는다.

즉, std::thread에 의해 결정된다.

## 정리
std::future 객체의 소멸자는 그냥 std::future 객체의 자료 멤버들을 파괴할 뿐이다.
<br>
std::async를 통해 시동된 non-deferred task에 대한 shared state를 참조하는 마지막 std::future 객체의 소멸자는 해당 task가 완료될 때까지 block 된다.

# 항목 39 : 단발성 사건 통신에는 void 미래 객체를 고려하라.
특정한 사건(event, 자료구조의 초기화, 계산 과정 중 특정 단계 완료, 센서 값 입력 등)가 일어나야만 작업을 진행할 수 있는 비동기 실행 과제에게 그 사건이 발생했음을 알려주는 또 다른 과제를 두는 것이 유용한 경우가 있다.

## 조건 변수 사용 방식
```cpp
std::condition_variable cv;
std::mutex m;

// 검출 과제 코드
...               // 사건 검출
cv.notify_one();  // 반응 과제에게 알림

// 반응 과제 코드
{                                      // 임계 영역 열기
   std::unique_lock<std::mutex> lk(m); // 뮤텍스를 잠근다
   cv.wait(lk);                        // 통지를 기다린다
   ...                                 // 사건에 반응, m은 잠겨있음
}                                      // 임계 영역 닫기, m을 해제
```
이를 해결하기 위한 한가지 명백한 접근 방식은 condition variable을 사용하는 것이다.
<br>
조건을 검출하는 과제를 검출 과제(detecting task)라 부르고, 조건에 반응하는 과제를 반응 과제(reacting task)라고 부를 때, 반응 과제는 하나의 조건 변수를 기다리고, 검출 과제는 사건이 발생하면 그 조건 변수를 통지하는 방식으로 구현한다.
<br>
이 때 일반적으로 조건 변수에 대한 대기를 수행하기 전에 std::unique_lock을 통해서 뮤텍스를 잠근다.

위 코드는 몇 가지 문제점을 갖고 있음.
1. 논리적으로 뮤텍스가 꼭 필요하지 않을 수도 있음.
2. 반응 과제가 wait를 실행하기 전에 검출 과제가 조건 변수를 통지하면 반응 과제가 멈춘다(hang).
3. wait 호출문은 가짜 기상을 고려하지 않는다.
   <br>
   조건 변수를 기다리는 코드가 조건 변수가 통지되지 않았는데도 깨어날 수 있다. 이 문제를 제대로 처리하려면 기다리던 조건이 정말로 발생했는지 확인해야 한다. 이를 위해서는 wait에 기다리던 조건을 판정하는 람다(또는 함수 객체)를 추가로 넘겨주면 된다.
   ```cpp
   cv.wait(lk, []{ return 사건 발생 여부; });
   ```
   문제는, 발생 여부를 검출하는 것은 검출 과제의 몫인데, 만약 검출 과제가 발생 여부를 검출할 수 있었다면 굳이 조건 변수를 기다릴 필요가 없다.

## 공유 bool flag의 사용 방식
```cpp
std::atomic<bool> flag(false);

// 검출 과제
...
flag = true;

// 반응 과제
...
while(!flag);
...
```
위 방식은 mutex를 사용하지도, 반응 과제가 폴링을 시작하기 전에 검출 과제가 플래그를 설정해도 문제가 없고, 가짜 기상 문제도 없다. 하지만 이러한 장점을 반응 과제의 폴링 비용이 깎아먹는 단점이 있다.
<br>
즉, 다른 과제가 유용하게 사용할 하드웨어 스레드를 점유하고 잇으며, 자신의 시간 조각의 시작이나 끝에서 문맥 전환 비용을 유발하고, 전원 절약을 위해 닫아도 될 코어를 계속해서 돌리게 된다.

## 조건 변수와 플래그를 함께 사용하는 방식
```cpp
std::condition_variable cv;
std::mutex m;

bool flag(false);    // atomic일 필요는 없음

// 검출 과제
...
{
   std::lock_guard<std::mutex> g(m);
   flag = true;
}
cv.notify_one();

// 반응 과제
{
   std::unique_lock<std::mutex> lk(m);
   cv.wait(lk, [] { return flag; });   // 가짜 기상을 방지하기 위해 람다 사용 

   ...
}
...
```
위 방식은 이전에 지적한 문제가 없지만 검출 과제가 기묘한 방식으로 반응 과제와 통신한다는 점이 꺼림칙하다.
<br>
검출 과제는 기다리던 사건이 발생했음을 알리기위해 조건변수와 플래그를 설정한다.
<br>
반응 과제는 통지를 받은 것으로는 사건 발생을 확신하지 못하므로 반드시 플래그를 확인해야 한다.
<br>
즉, 검출 과제가 플래그를 설정함으로써 반응 과제에게 알려주지만, 반응 과제가 그 플래그를 점검하기 위해서는 조건 변수를 통지해서 반응 과제를 깨워야한다.

## std::future 객체가 wait하는 방식
```cpp
std::promise<void> p;

// 검출 과제
...
p.set_value();

// 반응 과제
...
p.get_future().wait();
...
```
std::future 객체는 피호출자에서 호출자로의 통신 채널의 호출자 쪽 수신 단자이다. 여기서 문제는 검출 과제와 반응 과제가 호출자-피호출자 관계가 아니라는 점이다.
<br>
하지만, 전송 단자가 std::promise이고, 수신 단자가 std::future인 통신 채널이라는 점을 기억하면 해결할 수 있음.

또 다른 문제는 std::promise와 std::future는 형식 매개변수를 요구하는 템플릿으로 전송할 자료 형식을 결정해야 하지만 여기서는 전송할 자료 형식이 존재하지 않음.
<br>
반응 과제는 미래 객체가 설정되었는지만 알면 되고, 전달할 자료는 없다는 것을 의미하기 위해 void형을 사용하면 됨.

또 다른 고려 사항은 std::promise와 std::future가 공유하는 shared status는 힙 기반 할당 및 해제 비용을 유발한다는 점.
<br>
그보다 더 중요한 점은 std::promise를 한번만 설정할 수 있다는 점.
<br>
그러나 이 문제들이 그렇게 많은 제약을 주지는 않는다.

```cpp
std::promise<void> p;

void react();

void detect()
{
   auto sf = p.get_future().share();

   std::vector<std::thread> vt;

   for(int i = 0; i < threadsToRun; ++i>)
   {
      vt.emplace_back([sf]{ sf.wait(); react(); });
   }

   ...

   p.set_value();

   ...

   for(auto& t : vt)
   {
      t.join();
   }
}
```
std::shared_future를 이용하면 반응 과제 여러개를 유보하고 풀도록 확장할 수 있다.

## 정리
간단한 사건 통신을 수행할 때, 조건 변수 기반 설계에는 여분의 뮤텍스가 필요하고, 검출 과제와 반응 과제의 진행 순서에 제약이 있으며, 사건이 실제로 발생했는지를 반응 과제가 다시 확인해야 한다.
<br>
플래그 기반 설계를 사용하면 그런 단점들이 없지만, 대신 차단이 아니라 폴링이 일어난다는 단점이 있다.
<br>
조건 변수와 플래그를 조합할 수도 있으나, 그런 조합을 이용한 통신 매커니즘은 필요 이상으로 복잡하다.
<br>
std::promise와 std::future 객체를 사용하면 이러한 문제점들을 피할 수 있지만, shared status에 힙 메모리를 사용하며, 단발성 통신만 가능하다.

# 항목 40 : 동시성에는 std::atomic을 사용하고, volatile은 특별한 메모리에 사용하라.
```cpp
std::atomic<int> ai(0);

ai = 10;
std::cout << ai;
++ai;
--ai;
```
std::atomic은 다수의 스레드들이 반드시 원자적으로 인식하는 연산을 제공한다.
<br>
위 코드를 실행하는 동안 다른 스레드에서 ai를 읽을 경우 볼 수 있는 값은 오직 0, 10, 11 뿐이다.
<br>
위 코드에서 std::cout << ai가 보장하는 것은 ai의 읽기가 원자적이라는 것 뿐이다. 즉, 전체 문장이 원자적으로 처리된다는 보장을 하지는 않는다. ai의 값을 읽는 것과 operator<<()에서 값이 표준 출력에 기록되는 시점 사이에 다른 스레드가 ai값을 변경할 수 있다.
<br>
위 코드의 마지막 두 문장은 원자적으로 수행된다.

```cpp
volatile int vi(0);

vi = 10;
std::cout << vi;
++vi;
--vi;
```
위 코드가 실행되는 동안 vi의 값을 다른 스레드들이 읽으면 그 어떤 값(-12, 49213, ...)이든 볼 수 있다.
<br>
즉, 메모리에 write와 reader가 동시에 접근하는 상황이 발생하는 data race가 발생한다.

## 다중 스레드 환경에서 std::atomic과 volatile의 행동 차이
```cpp
++ac;
++vc;
```
2개의 스레드가 동시에 위 코드를 실행한다면 ac의 값은 반드시 2이다. 반면 vc는 2일수도 있고 아닐수도 있다.

```cpp
a = b;
x = y;
```
컴파일러는 필요에 의해 위 코드의 순서를 변경할수도 있다. 또한 하드웨어나, 코어에 의해 실행 순서가 변경될수도 있다.
<br>
하지만 std::atomic을 사용하면 코드 순서 재배치에 대한 제약이 발생한다. 즉, std::atomic 변수를 기록하는 문장 이전에 나온 그 어떤 코드도 그 문장 이후에 실행되지 않는다.

```cpp
std::atomic<bool> valAvailable(false);
auto imptValue = computeImportantValue();
valAvilable = true;
```
컴파일러와 하드웨어는 imptValue 이후에 valAvailable이 호출된다는 것을 보장한다.
<br>
반면 volatile은 그런 코드 재배치 제약이 가해지지 않는다.

즉, volatile은 동시적 프로그래밍에 유용하지 않다.
<br>
대신, volatile은 volatile이 적용된 변수가 사용하는 메모리가 보통의 방식으로 행동하지 않는다는 점을 컴파일러에게 알려준다.
<br>
여기서 보통의 방식이란 메모리의 한 장소에 어떤 값을 기록하면 다른 어떤 값을 덮어쓰지 않는 한 그 값이 유지된다는 특성을 의미한다.

```cpp
auto y = x; // x값을 읽는다
y = x;      // x값을 다시 읽는다
x = 10;     // x값을 기록한다
x = 20;     // x값을 다시 기록한다.
```
컴파일러는 둘째 줄을 제거해서 목적코드를 최적화할 수 있다. 둘째 줄은 첫째 줄과 정확히 같은 일을 한다.
<br>
또한 컴파일러는 세번 째 줄을 지울 수 있다.
<br>
위 구조는 컴파일러가 템플릿 인스턴스화와 인라인화, 다양한 순서 재배치 과정에서 드물지않게 생겨난다.

메모리 대응 입출력(Memory mapped I/O)같은 특별한 메모리는 위 코드를 함부로 제거할 수 없다.
<br>
이런 메모리들은 RAM보다는 외부 감지기나 디스플레이, 프린터, 네트워크 포트 같은 주변장치와 실제로 통신하는 메모리들이다.
<br>
즉 volatile은 해당 코드가 특별한 메모리를 다룬다는 것을 컴파일러에게 알리는 수단이다.
<br>
컴파일러는 해당 메모리에 대한 연산들에서는 어떤 최적화도 수행하지 말라는 지시로 인식한다.

그런데 특별한 메모리를 다룰 때에는 남아도는 적재들과 죽은 저장들로 보이는 연산들을 반드시 유지해야한다는 사실은 이런 종류의 작업이 std::atomic이 적합하지 않다는 점을 말해준다.

```cpp
std::atomic<int> x;

auto y = x;
y = x;
```
참고로 위 코드는 컴파일되지 않는다.
<br>
std::atomic은 복사 생성을 지원하지 않기 때문이다.

```cpp
std::atomic<int> y(x.load());
y.store(x.load());
```
만약 x의 값을 y에 넣고싶다면 load, store를 사용해야 한다.

```cpp
레지스터 = x.load();
std::atomic<int> y(레지스터);
y.store(레지스터);
```
불필요한 읽기를 막기위해 레지스터에 임시로 저장하는 방식의 최적화도 있다.
<br>
그리고 이러한 최적화는 volatile이 다루는 특수한 메모리에서는 반드시 피해야하는 최적화이다.

std::atomic은 동시적 프로그래밍에 유용하나, 특별한 메모리의 접근에는 유용하지 않다.
<br>
volatile은 특별한 메모리의 접근에 유용하나, 동시적 프로그래밍에는 유용하지 않다.

```cpp
volatile std::atomic<int> val;
```
두 구문을 함께 사용하는 것도 가능하다.

## 정리
std::atomic은 뮤텍스 보호 없이 여러 스레드가 접근하는 자료를 위한 것으로, 동시적 소프트웨어의 작성을 위한 도구이다.
<br>
volatile은 읽기와 기록을 최적화로 제거하지 말아야 하는 메모리를 위한 것으로, 특별한 메모리를 다룰 때 필요한 도구이다.
