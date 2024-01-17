---
title: "CSharpStudy 정리"
date: 2023-07-10
categories: ["Development", "CSharp"]
tags: ["csharp"]
---
##### Reference : http://www.csharpstudy.com/

# yield
여러개의 return
내부에서 collection을 만들어서 반환하는 듯
Enumerator 없이 Enumerable 구현 가능

# Exception
catch된 execption을 ex로 바로 Throw시 Trace 초기화
Exception(msg, ex)로 Inner Exception 가능
throw;로 끝나면 catch로 잡힌 Exception을 그대로 상위로 넘길 수 있음
finally keyword는 exception여부에 상관없이 무조건 실행(해제 구문에 쓰기 좋음)

# struct
c#의 모든 기본형 데이터(int, double 등)은 struct으로 제작됨
method, property 사용 가능
상속 불가능, 단 interface 구현 가능

# event
event keyword 사용
```csharp
delegate ReturnType CustomEventHandler(...);
public event CustomEventHandler EventName;
EventName(...);
```
field처럼 선언
기본 system에서 지원하는 EventHandler도 존재
+=, -=로 구독, 취소
Property로 get,set 대신 add, remove 사용(그 안에서 +=, -=사용)

# 전처리기 지시어
'#'으로 시작, ';'없음, 해당 파일에서만 작동
Partial class에서는 각각 (#define)정의해야 함
#region-#endregion : 코드 블럭 묶기
#pragma : 컴파일러 제조사가 지원하는 전처리기 명령어

# 상속
```csharp
public class DerivedClass : BaseClass { ... }
```
다중 상속 불가능, 다중 interface 가능

# abstract
abstract class, abstract method
override keyword 사용 가능
abstract class의 경우 abstract가 없는 method는 구현해야 함(interface와의 차이점)

# as 연산자
casting
기존의 명시적 casting은 실패시 exception 발생, as는 null

# is 연산자
특정 class type이나 interface를 갖고있는지 확인
bool return

# static class
모든 class member가 static인 클래스
static 생성자 존재

# Interface
내부 member들에서 접근제한자(public, private 등)를 사용하지 않음

# delegate
System.MulticastDelegate를 상속하는 class생성(직접 생성은 불가능)
class이므로 new를 이용해서 객체 생성
Invoke(), BeginInvoke()를 통해서 수행 가능

## Event, Delegate
Event는 class 외부에서 호출 불가능
Event는 =(덮어쓰기) 연산 불가능

## 고찰
외부 class의 Method를 자신의 delegate에 어떻게 등록하는가에 대해 고민했는데, 객체지향 설계상 외부 Method를 내가 신경쓸 필요가 없음

# Extension Method
클래스 밖에서 클래스의 새 메서드 정의
```csharp
public static class ExtendedClass
{
    public static ReturnType Func(ThisTypeName, ...) { ... }
}
ThisTypeName instance = new ThisTypeName;
instance.Func();
```
만약 ThisTypeName이 string이라면 기존의 string에 존재하지 않던 Func이 추가

# Partial
Code Generator가 만든 코드와 사용자가 만든 코드를 분리할 수 있음
Compiler가 나중에 합쳐서 1개의 클래스로 생성

## Partial Method
private void일 경우 선언과 구현을 분리할 수 있음
h, cpp와 비슷한 느낌이긴 한데 private void로 지정되어있어서 큰 의미는 없어보임
정의가 안되있으면 선언도 자동으로 무시되기 때문에 파일에 따라서 다른 방식의 class제작 가능

# ExpandoObject
```csharp
dynamic instance = new ExpandoObject();
instance.variable1 = 1;
instance.variable2 = "Test";
instance.Func = (x)=>{ ... };
```
동적으로 속성, method, event 추가 가능
dynamic type으로 paramter로 전달 가능
동적으로 추가된 멤버들은 IDictionary<string, Object>로 casting하면 접근 가능

# async-await
async : 해당 method가 1개 이상의 await를 갖고 있음을 알림. 대부분 Task'<'TReturn'>'반환, return TReturn하면 자동으로 변환시켜 줌
await : 결과가 나올때까지 대기
TReturn val = await Task.Run(Lambda);
Task.Run()을 사용하지 않는다면 background에서 돌리지 않음

# DirectX-Winform
Winform의 HWND를 받아서 DirectX에 넘겨주는 방식
C++/CLI을 사용하는 방법도 있지만 손이 많이가므로 그냥 DLL_Interop이 편할 듯

# DirectX-WPF
DirectX에서 HWND을 만든 후 WPF Win32 Content Hosting에 제공