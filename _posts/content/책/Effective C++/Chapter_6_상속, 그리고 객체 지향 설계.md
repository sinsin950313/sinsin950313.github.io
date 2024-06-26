# 항목 32 : public 상속 모형은 반드시 is-a 를 따르도록 만들자
클래스 D가 클래스 B를 public 상속 받는다면 이는 D 타입으로 만들어진 모든 객체는 B 타입의 객체이지만, B 타입으로 만들어진 모든 객체가 D타입인 것은 아니다.
<br>
즉, B타입의 객체가 쓰일 수 있는 곳에는 D 타입의 객체도 쓰일수 있다고 단정(Assert)지을 수 있음.
<br>
즉, 모든 D는 B의 일종이지만(D is a B), 모든 B가 D인 것은 아니다.
<br>
이는 public 상속에서만 통한다.

```cpp
class Bird
{
public:
    virtual void fly();
};

class Penguin : public Bird
{
public:
    virtual void fly() override;
};

// =============================

class Bird
{
    ...
};

class FlyingBird : public Bird
{
public:
    virtual void fly();
};

class Penguin : public Bird
{
    ...
};
```
is-a관계는 잘못 판단할 수 도 있음.
<br>
Punguin is a Bird는 참이지만, Penguin이 fly는 할 수 없음.
<br>
하지만 어떤 소프트웨어는 새의 나는 기능보다는 부리나 날개만 관심이 있을 수 도 있는데, 이 경우 위의 설계가 더 유용할 수도 있음.
<br>
즉, 모든 소프트웨어에는 이상적인 설계란 존재하지 않음. 최고의 설계란 제작하려는 소프트웨어 시스템이 기대하는 바에 따라 달라지는 것.

```cpp
class Penguin : public Bird
{
public:
    virtual void fry() { error("Attemp to make a penguin fly!"); }
};
```
만약 Penguin 클래스의 fly를 구현하지만 에러를 내도록 만든다면 이는 '펭귄은 날 수 없다'를 의미하는 것이 아니라, 'Penguin은 날 수 있지만 날려고 하면 에러가 발생한다'를 의미.
<br>
유효하지 않은 코드를 컴파일 단계에서 막아주는 인터페이스가 좋은 인터페이스.

직관적으로는 정사각형 클래스는 직사각형 클래스를 상속해야하지만 설계시에는 불가능한 것처럼 상속은 직관적으로 생각하는 것과는 차이가 있음.
<br>
public 상속은 Base class의 '모든' 것이 Derived class에 적용된다고 단정하는 상속.

## 정리
public 상속의 의미는 "is-a" 입니다. 기본 클래스에 적용되는 모든 것들이 파생 클래스에 그대로 적용되어야 합니다. 왜냐하면 모든 파생 클래스 객체는 기본 클래스 객체의 일종이기 때문입니다.

# 항목 33 : 상속된 이름을 숨기는 일은 피하자
```cpp
int x;

void someFunc()
{
    double x;
    std::cin >> x;
}
```
유표범위 내에 동일한 이름을 정의하면 이름의 타입이 일치하는지 여부와는 상관없이 컴파일러는 이름을 가려버린다.

```cpp
class Base
{
private:
    int x;

public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived : public Base
{
public:
    virtual void mf1();
    void mf3();
    void mf4();
    ...
};

void Derived::mf4()
{
    ...
    mf2();  // Base의 mf2 호출
    ...
}

Derived d;
int x;

d.mf1();    // Derived::mf1
d.mf1(x);   // 컴파일 에러.

d.mf2();    // Base::mf2

d.mf3();    // Derived::mf3
d.mf3(x);   // 컴파일 에러
```
이처럼 유효범위로 이름이 가려지는 것은 함수도 동일하게 적용되며, 이는 반환 타입이나 인자의 형식을 무시한다.
<br>
이는 파생 클래스를 만들 때 멀리 떠러진 기본 클래스로부터 오버로드 버전을 상속하는 경우를 막고자 하는 것.

```cpp
class Derived : public Base
{
public:
    // using Base::mf1
    // using Base::mf3

    virtual void mf1();
    void mf3();
    void mf4();
    ...
};

void Derived::mf1()
{
    Base::mf1();
}
```
using을 사용하여 Derived의 유효범위에서 Base의 mf1, mf3를 확인할 수 있음.
<br>
즉, 오버로드된 함수가 클래스에 들어있고, 이 중 일부 함수만 재정의를 하고싶다면 using을 사용해야 오버라이딩 하지 않은 함수도 호출할 수 있음.
<br>
또는 명시적으로 forwarding function을 만들어서 호출할수도 있음.

## 정리
파생 클래스의 이름은 기본 클래스의 이름을 가립니다. public 상속에서는 이런 이름 가림 현상은 바람직하지 않습니다.
<br>
가려진 이름을 다시 볼 수 있게 하는 방법으로, using 선언 혹은 전달 함수를 쓸 수 있습니다.

# 항목 34 : 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하자.
public 상속은 인터페이스 상속과 함수 구현 상속 2가지로 나뉨

```cpp
class Shape
{
public:
    virtual void draw() const = 0;
    virtual void error(const std::string& msg);
    int objectID() const;
};

class Rectangle : public Shape { ... }
class Ellipse : public Shape { ... }
```
Shape는 추상 클래스로서 파생 클래스에 미치는 영향은 엄청남.
<br>
이는 멤버 함수 인터페이스는 항상 상속되기 때문으로, 기본 클래스에 해당하는 모든 것들은 파생 클래스에도 해당해야 함. 즉, 어떤 클래스에서 동작하는 함수는 그 클래스의 파생 클래스에서도 동작해야 함.

선언은 순수 가상, 가상, 비가상 함수로 각각 다른 의미를 갖음.
<br>
순수 가상 함수를 선언하는 목적은 파생 클래스에게 함수의 인터페이스만을 물려주려는 것. Shape 계통을 따르는 모든 객체는 draw가 가능해야 한다를 의미.

```cpp
class Airplane
{
public:
    virtual void fly(const Airport& destination) = 0;

protected:
    void defaultFly(const Airport& destination);
};

void Airplane::fly(const Airport& destination)
{
    ...
}

class ModelA : public Airplane
{
    virtual void fly(const Airport& destination) { Airplane::fly(desination); }
}
```
단순 가상 함수는 파생 클래스로 하여금 함수의 인터페이스뿐만 아니라 그 함수의 기본 구현도 물려주려는 것.
<br>
단, 단순 가상 함수에서 함수 인터페이스와 기본 구현을 동시에 제공하는 것은 추후 파생 클래스에서 재정의를 까먹는 위험이 있을 수 있음. 이를 막기위해 default 구현을 제공하는 방법도 있음.
<br>
또는 순수 가상 함수의 구현을 제공한 후 파생 클래스에서 명시적으로 호출하도록 하여 네임스페이스의 난립을 막는 방법도 있음.

비가상 함수는 파생 클래스가 함수 인터페이스와 더불어 그 함수의 필수적인 구현을 물려받게 하는 것.
<br>
파생 클래스에서 다른 행동이 일어날 것으로 가정하지 않는다는 의미. 즉, 클래스 파생에 상관없이 변하지 않는 동작을 지정.

## 멤버 함수를 선언할 때, 클래스 설계에서 흔히 발견되는 실수 2가지

### 모든 멤버 함수를 비가상 함수로 선언하는 것
파생 클래스를 만들더라도 기본 클래스의 동작을 수정할 여지가 없어짐.
<br>
비가상 소멸자의 문제가 발생함.

### 모든 멤버 함수를 가상 함수로 선언하는 것
클래스 파생에 상관없는 불변동작을 갖고있어야 한다면 비가상 함수를 선언해야 함.

## 정리
인터페이스 상속은 구현 상속과 다릅니다. public 항속에서, 파생 클래스는 항상 기본 클래스의 인터페이스를 모두 물려받습니다.
<br>
순수 가상 함수는 인터페이스 상속만을 허용합니다.
<br>
단순 가상 함수는 인터페이스 상속과 더불어 기본 구현의 상속도 가능하도록 지정합니다.
<br>
비가상 함수는 인터페이스 상속과 더불어 필수 구현의 상속도 가하도록 지정합니다.

# 항목 35 : 가상 함수 대신 쓸 것들도 생각해두는 자세를 시시때때로 길러 두자
```cpp
class GameCharacter
{
public:
    virtual int healthValue() const;
};
```
위 코드 이외에 다른 방식으로 구현할 수 있는 방법에 대한 것

## 비가상 인터페이스 관용구를 통한 템플릿 메서드 패턴
```cpp
class GameCharacter
{
public:
    int healthValue const
    {
        ...
        int retVal = doHealthValue();
        ...
        return retVal;
    }

private:
    virtual int doHealthValue() const { ... }   // 파생 클래스에서 이 함수를 재정의할 수 있음
};
```
가상 함수는 반드시 private 멤버로 두어야 한다라는 이론에 따른 방식.
<br>
public 비가상 멤버 함수를 통해 private 가상 함수를 간접적으로 호출하는 방식으로, 비가상 함수 인터페이스 관용구(non-virtual interface, NVI)라고 함.
<br>
Template Method 패턴의 구현 방식. 비가상 함수를 가상 함수의 Wrapper로 사용.
<br>
사전 동작과 사후 동작을 미리 명시할 수 있음.
<br>
가상 함수의 재정의를 파생 클래스에서 하면서 호출할 수 없다는 점에서 설계상의 모순이 있는 것 처럼 보이지만, 가상 함수를 '재정의' 하는 것은 어떤 동작을 '어떻게' 구현할 것인가를 지정하는 것이고, 가상 함수를 '호출'하는 것은 그 동작이 수행될 '시점'을 지정하는 일.

## 함수 포인터로 구현한 전략 패턴
```cpp
class GameCharacter;

int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter
{
public:
    typedef int (*HealthCalcFunc) (const GameCharacter&);

    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) { ... }

    int healthValue() const { return healthFunc(*this); }

private:
    HealthCalcFunc healthFunc;
};
```
Strategy 패턴의 응용
<br>
생성자 인자에 따라 같은 클래스 객체도 다른 계산 방식을 가질 수 있음.
<br>
객체의 계산 방식을 런타임 중 수정할 수 있음.
<br>
단, 외부 함수는 public으로 공개된 함수를 통해서만 객체에 접근할 수 있음. 만약 그 외의 정보가 필요하다면 friend를 사용하거나 다른 함수의 공개에 대한 결정이 필요함. 즉, 캡슐화 정도를 낮춰야 할 수 있다는 문제가 있음.

## std::function으로 구현한 Strategy 패턴
```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter
{
public:
    typedef std::function<int(const GameCharacter&)> HealthCalcFunc;

    explicit GameCharacter(HelathCalcFunc hcf = defaultHelathCalc) : healthFunc(hcf) { ... }

    int healthValue() const { return healthFunc(*this); }

private:
    HealthCalcFunc healthFunc;
};

short CalcHealth(const GameCharacter&);

struct HealthCalculator
{
    int operator(const GameCharacter&) const { ... }
};

class GameLevel
{
public:
    float health(const GameCharacter&) const;
};

class EvilBadGuy : public GameCharacter { ... };

class EyeCandyCharacter : public GameCharacter { ... };

EvilBadGuy edg1(calcHealth);
EyeCandyCharacter ecc1(HealthCalculator());
GameLevel currentLevel;
EvilBadGuy ebg2(std::bind(&GameLevel::health, currentLevel, _1));
```
함수 포인터 대신 함수 객체를 사용하는 방식.
<br>
시그니처와 호환되는 함수 호출성 개체 어떤 것들과도 호환이 가능함. 시그니처란 매개변수, 반환 타입을 의미.

## 고전적인 Strategy 패턴
```cpp
class GameCharacter;

class HealthCalcFunc
{
public:
    virtual int calc(const GameCharacter& gc) const { ... }
};

HealthCalcFunc defaultHealthCalc;

class GameCharacter
{
public:
    explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc) : pHealthCalc(phcf) { ... }

    int healthValue const { return pHealthCalc->calc(*this); }

private:
    HealthCalcFunc* pHealthCalc;
};
```

## 요약
어떤 문제를 해결하기 위한 설계를 찾을 때 가상 함수를 대신하는 방법들도 고려해보자.

1. 비가상 인터페이스 관용구 : 공개되지 않은 가상 함수를 비가상 public 멤버 함수로 감싸서 호출하는, 템플릿 메서드 패턴의 형태
2. 가상 함수를 함수 포인터 데이터 멤버로 대체
3. 가상 함수를 std::function 데이터 멤버로 대체하여, 호환되는 시그니처를 가진 함수 호출성 객체를 사용.
4. 한쪽 클래스 계통에 속해있는 가상 함수를 다른 쪽 계통에 속해있는 가상 함수로 대체하는 전략 패턴.

## 정리
가상 함수 대신 쓸 수 있는 다른 방법으로 NVI 관용구 및 전략 패턴을 들 수 있습니다. 이 중 NVI 관용구는 그 자체가 템플릿 메서드 패턴의 한 예입니다.
<br>
객체에 필요한 기능을 멤버 함수로부터 클래스 외부의 비멤버 함수로 옮기면, 그 비멤버 함수는 그 클래스의 public 멤버가 아닌 것들을 접근할 수 없다는 단점이 생깁니다.
<br>
std::function 객체는 일반화된 함수 포인터처럼 동작합니다. 이 객체는 주어진 대상 시그니처와 호환되는 모든 함수호출성 개체를 지원합니다.

# 항목 36 : 상속받은 비가상 함수를 파생 클래스에서 재정의하는 것은 절대 금물
```cpp
class B
{
public:
    void mf();
};

class D : public B
{
public:
    void mf();
};

D x;

B* pB = &x;
pB->mf();   // B::mf 호출
```
가상 함수는 동적 바인딩으로 묶이지만 비가상함수는 정적 바인딩으로 묶이기 때문에 비가상 함수를 재정의할 경우 포인터에 따라 다른 행동을 보일 수 있음.

public 상속은 is-a 를 의미. 비가상 멤버 함수는 클래스 파생에 관계없이 불변동작을 정해두는 것을 의미.
<br>
즉, D 객체에 해당되는 모든 것들은 D 객체에 그대로 적용되고, B에서 파생된 클래스는 mf 함수의 인터페이스와 구현을 모두 물려받아야 함.
<br>
만약 D에서 B를 재정의한다면 이 부분에서 모순이 발생.
<br>
소멸자는 이번 항의 특수한 조건이라고 볼 수 있음.

## 정리
상속받은 비가상 함수를 재정의하는 일은 절대로 하지 맙시다.

# 항목 37 : 어떤 함수에 대해서도 상속받은 기본 매개변수 값은 절대로 재정의하지 말자
가상 함수는 동적으로 바인딩되지만, 기본 매개변수는 정적으로 바인딩된다.(비가상 함수는 이전 장에서 재정의하지 말라고 이미 명시함.)

```cpp
class Shape
{
public:
    enum ShapeColor { Red, Green, Blue };

    virtual void draw(ShapeColor color = Red) const = 0;
};

class Rectangle : public Shape
{
public:
    virtual void draw(ShapeColor color = Green) const;
};

class Circle : public Shape
{
public:
    virtual void draw(ShapeColor color) const;
};

Shape* ps;
Shape* pc = new Circle;
Shape* pr = new Rectangle;
```
ps, pc, pr은 모두 Shape에 대한 포인터로 선언되어있기 때문에 각각의 정적 타입 모두 Shape이다.
<br>
객체의 동적 타입은 현재 그 객체가 진짜로 무엇이냐에 따라 결정되는 타입. 이 객체가 어떻게 동작할 것이냐를 가리키는 타입. 동적 타입은 런타임 중 변경될 수 있음.
<br>
pc, pr의 기본 draw를 호출할 경우 수행은 파생 함수의 수행을 하지만 매개변수는 Shape의 값이 들어가는 것을 볼 수 있음.
<br>
이는 기본 매개변수를 동적으로 바인딩한다면 프로그램 실행 중 가상 함수의 기본 매개변수 값을 결정할 방법을 컴파일러에서 마련해줘야 하므로 런타임 효율성이 떨어짐.

```cpp
class Shape
{
public:
    enum ShapeColor { Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
};

class Rectangle : public Shape
{
public:
    virtual void draw(ShapeColor color = Red) const;    // 매번 재정의 해줘야 하고, 부모 값이 바뀌면 모두 변경해줘야 함
};

// =============================

class Shape
{
public:
    enum ShapeColor { Red, Green, Blue };
    void draw(ShapeColor color = Red) const
    {
        doDraw(color);
    }

private:
    virtual void doDraw(ShapeColor color) const = 0;
};

class Rectangle : public Shape
{
private:
    virtual void doDraw(ShapeColor color) const;
};
```
가상 함수의 기본 매개변수를 상속받는 과정에서 고정하고 싶다면 비가상 인터페이스 관용구를 적용하는 방법이 있음.

## 정리
상속받은 기본 매개변수 값은 절대로 재정의해서는 안 됩니다. 왜냐하면 기본 매개변수 값은 정적으로 바인딩되는 반면, 가상 함수는 동적으로 바인딩되기 때문입니다.

# 항목 38 : has-a 혹은 is-implemented-in-terms-of를 모형화할 때는 객체 합성을 사용하자
합성이란 어떤 타입의 객체들이 그와 다른 타입의 객체들을 포함하고있을 경우에 성립하는 타입들 사이의 관계를 의미.
<br>
합성은 "has-a" 또는 "is-implemented-in-terms-of"의 의미를 갖음.
<br>
뜻이 2개인 이유는 소프트웨어 개발에서 프로그래머가 대하는 영역(domain)은 응용 영역과 구현 영역으로 구성되기 때문으로 응용 영역이란 일상생활에서 볼 수 있는 사물(사람, 이동수단, 비디오 프레임 등)을 본 뜬 것이라면, 구현 영역은 시스템 구현만을 위한 인공물(버퍼, 뮤텍스, 탐색 트리 등)을 의미.
<br>
객체 합성이 응용 영역의 객체들 사이에서 일어난다면 has-a 관계이고, 구현 영역에서 일어난다면 is-implemented-in-terms-of 관계.

```cpp
template<typename T>
class Set : public std::list<T> { ... }
```
만약 Set Container를 만들고 싶어서 std::list를 사용한다면 public을 사용하는 순간 is-a관계가 되므로 잘못된 상속이 됨.

```cpp
template<typename T>
class Set
{
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;

private:
    std::list<T> rep;
};
```
즉, Set 객체는 list 객체를 써서 구현되는(is implemented in terms of)의 형태의 설계가 적용되어야 함.

## 정리
객체 합성(composition)의 의미는 public 상속이 갖니 의미와 완전히 다릅니다.
<br>
응용 영역에서 객체 합성의 의미는 has-a입니다. 구현 영역에서는 is-implemented-in-terms-of의 의미를 갖습니다.

# 항목 39 : private 상속은 심사숙고해서 구사하자
```cpp
class Person { ... };
class Student : private Person { ... }

void eat(const Person& p);
void study(const Student& s);

Person p;
eat(p); // 컴파일 성공

Student s;
eat(s); // 컴파일 실패
```
private 상속은 파생 클래스 객체를 기본 클래스 객체로 변환하지 않음.
<br>
기본 클래스로부터 물려받은 멤버는 파생 클래스에서 모조리 private 멤버가 됨.
<br>
private 상속의 의미는 is-implemented-in-terms-of를 의미.
<br>
B 클래스로부터 private 상속을 통해 D 클래스를 파생시킨 것은, B 클래스에서 쓸 수 있는 기능들 몇 개를 활용할 목적으로 한 행동이지 B 타입과 D 타입의 객체 사이에 어떤 개념적 관계가 있는 것은 아님.
<br>
즉, private 상속은 구현 기법 중 하나. 기본 클래스로부터 물려받은 것들이 전부 private으로 되는 이유도 기본 클래스는 그저 구현 세부사항이라는 것을 의미.
<br>
단, 가능하다면 객체 합성을 사용하고 꼭 필요한 경우에만 private 상속을 사용할 것. 주로 비공개 멤버를 접근할 때 또는 가상 함수를 재정의할 경우.

```cpp
class Timer
{
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;
};

class Widget : private Timer
{
private:
    virtual void onTick() const;
};
```
Widget은 Timer의 일부가 아니고, Timer의 기능을 재정의할 용도이므로 private 상속을 선택해야 함.

```cpp
class Widget
{
public:
    class WidgetTimer : public Timer
    {
    public:
        virtual void onTick() const;
    };

private:
    WidgetTimer timer;
};
```
public 상속에 객체 합성 조합이 더 자주 쓰이기는 함.
<br>
1. Widget 클래스를 설계하는데 있어서 파생은 가능하게 하되, 파생 클래스에서 onTick을 재정의할 수 없도록 할 수 있음.
2. Widget의 컴파일 의존성을 최소화할 수 있음. Widget이 WidgetTimer에 대한 포인터만 갖는다면 WidgetTimer를 외부에 선언함으로써 컴파일 의존성을 줄일 수 있음.

```cpp
class Empty { };

class HoldsAnInt
{
private:
    int x;
    Empty e;
};

class HoldsAnInt : private Empty
{
private:
    int x;
};
```
데이터가 전혀 없는 클래스(비정적 데이터, 가상 함수, 가상 기본 클래스 등)인 공백 클래스는 크기가 0보다 크지만 private 상속을 사용할 경우에는 공백 기본 클래스 최적화(Empty Base Optimization, EBO)가 적용된다. 단, 이 역시도 다중 상속에서는 적용할 수 없음.
<br>
즉, typedef, enum, static 데이터 멤버, 비가상 함수만을 갖는 객체의 구현을 사용하고자 할 경우에서 공간 최적화가 필요하다면 private 상속을 고려할 수 있음.
<br>
또는 is-a관계가 아닌 두 클래스 중 한 클래스가 다른쪽 클래스의 protected 멤버에 접근해야 하거나 가상 함수를 재정의해야 한다면 private 상속을 사용할 수 있음.
<br>
보통은 public 상속과 객체 합성을 섞는 방식으로 구현.

## 정리
private 상속의 의미는 is-implemented-in-terms-of입니다. 대개 객체 합성과 비교해서 쓰이는 분야가 많지는 않지만, 파생 클래스 쪽에서 기본 클래스의 protected 멤버에 접근해야 할 경우 혹은 상속받은 가상 함수를 재정의해햐 할 경우에는 private 상속이 나름대로 의미가 있습니다.
<br>
객체 합성과 달리, private 상속은 공백 기본 클래스 최적화(EBO)를 활성화시킬 수 있습니다. 이 점은 객체 크기를 가지고 고민하는 라이브러리 개발자에게 꽤 매력적인 특징이 되기도 합니다.

# 항목 40 : 다중 상속은 심사숙고해서 사용하자
```cpp
class BorrowableItem
{
public:
    void checkout();
};

class ElectronicGadget
{
private:
    bool checkout() const;
};

class MP3Player : public BorrowableItem, public ElectronicGadget { ... }

MP3Player mp;
mp.checkout();
mp.BorrowableItem::checkout();
```
다중 상속의 가장 큰 위험은 둘 이상의 기본 클래스로부터 똑같은 이름을 상속받으면서 모호성 문제가 발생할 수 있음.
<br>
컴파일러는 해당 함수의 접근 가능 여부를 따지기 전에 주어진 호출에 대해 최적으로 일치하는 함수인지를 먼저 확인한다.

```cpp
class File { ... };
class InputFile : public File { ... };
class OutputFile : public File { ... };
class IOFile : public InputFile, public OutputFile { ... };
```
또는 둘 이상의 클래스로부터 동일한 기본 클래스를 상속받는 다이아몬드 상속이 발생할 수 있음.
<br>
이 경우 기본 클래스의 데이터 멤버가 경로 개수만큼 중복 생성된다.

```cpp
class File { ... };
class InputFile : virtual public File { ... };
class OutputFile : virtual public File { ... };
class IOFile : public InputFile, public OutputFile { ... };
```
가상 기본 클래스 상속을 한다면 중복은 제거할 수 있음.
<br>
가상 기본 클래스 상속을 사용할 경우 가상 상속을 쓰지 않는 클래스보다 일반적으로 크기가 더 크고, 데이터 멤버에 접근하는 속도도 느리다. 즉, 가상 상속은 비싸다.

또한, 가상 상속은 초기화 규칙도 복잡하다.
1. 초기화가 필요한 가상 기본 클래스로부터 클래스가 파생된 경우, 이 파생 클래스는 가상 기본 클래스와의 거리에 상관없이 가상 기본 클래스의 존재를 염두에 두고 있어야 한다.
2. 기존의 클래스 계통에서 파생 클래스를 새로 추가할 때도 그 파생 클래스는 가상 기본 클래스의 초기화를 떠맡아야 한다.

즉, 꼭 써야할 필요가 없으면 굳이 가상 상속을 쓰지 않는 것이 좋음.
<br>
만약 가상 상속이 정말 필요하다면 가상 기본 클래스에는 최대한 데이터를 넣지 않는것이 좋음.

```cpp
class IPerson
{
public:
    virtual ~IPerson();

    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
};

std::shared_ptr<IPerson> makePerson(DatabaseID personIdentifier);

DatebaseID askUserForDatasID();

DatabaseID id(askUserForDatabaseID());
std::shared_ptr<IPerson> pp(makePerson(id));

class PersonInfo
{
public:
    explicit PersonInfo(DatabaseID pid);
    virtual ~PersonInfo();

    virtual const char* theName() const;
    virtual const char* theBirthDate() const;

private:
    virtual const char* valueDelimOpen() const;
    virtual const char* valueDelimClose() const;
};

class CPerson : public IPerson, private PersonInfo
{
public:
    explicit CPerson(DatabaseID pid) : PersonInfo(pid) { ... }

    virtual std::string name() const { return PersonInfo::theName(); }
    virtual std::string birthDate() constr { return PersonInfo::theBirthDate(); }

private:
    const char* valueDelimOpen() const { return ""; }
    const char* valueDelimClose() const { return ""; }
};
```
is-implemented-in-terms-of 구조의 경우 다중 상속을 이용하기 좋음.
<br>
객체 합성과 private 상속인 다중 상속을 이용하여 CPerson을 효과적으로 정의. 즉, 인터페이스의 public 상속과 구현의 private 상속을 조합하는 방식.

다중 상속은 객체 지향 기법으로 소프트웨어를 개발하는데 쓰이는 도구 중 하나이지만, 단일 상속에 비해 사용이 복잡하고 이해하기 어렵다.
<br>
또한, 대부분의 경우 단일 상속으로 다중 상속과 동일한 구현을 할 수 있음.
<br>
즉, 다중 상속을 사용하고자 한다면 여러번 고민해봐야 한다.

## 정리
다중 상속은 단일 상속보다 확실히 복잡합니다. 새로운 모호성 문제를 일으킬 뿐만 아니라 가상 상속이 필요해질 수도 있습니다.
<br>
가상 상속을 쓰면 크기 비용, 속도 비용이 늘어나며, 초기화 및 대입 연산의 복잡도가 커집니다. 따라서 가상 기본 클래스에는 데이터를 두지 않는 것이 현실적으로 가장 실용적입니다.
<br>
다중 상속을 적법하게 쓸 수 있는 경우가 있습니다. 여러 시나리오 중 하나는, 인터페이스 클래스로부터 public 상속을 시킴과 동시에 구현을 돕는 클래스로부터 private 상속을 시키는 것입니다.
