---
title: "Easy Coding Test with VSCode"
date: 2024-01-22
categories: ["Development", "portfolio"]
tags: ["cpp", "portfolio", "vscode", "bat", "cmake"]
---
![](/images/300192303-da22c38d-a804-47cf-8c23-a903cae4f983.PNG)

# VSCode에서 쉽게 Coding Test 연습하기
백준이나 HackerRank의 경우 std::cin으로 예제를 입력받는데 Test를 위해 Build이후 직접 값을 넣어주기 번거로워서 자동화시키는 bat파일입니다.

# 사전 설정
cmake가 환경 변수에 등록되어 있어야 합니다.
<br>
source 이름과 ini파일 이름이 동일해야합니다.
<br>
확장자명 없이 파일 이름만 입력해야 합니다.
<br>
ini파일 내부에는 Arg, Ans을 통해 구분되며 Arg, Ans 순서가 유지되어야 합니다.
<br>
Debug Mode로 변경 시 { Debug Sample Order }는 정수입니다.
<br>
Debug Mode는 VSCode에서 cmake: Debug를 통해 호출해야 합니다.

# 사용법

## Help

![](/images/300191910-c6555b00-8d19-42c3-bfeb-5f9df246a572.PNG)

cmd에서 폴더에 위치한 후
```
main -h
```
를 누르면 도움말이 보입니다

## New Coding Test

![](/images/300192055-fb9d4fe1-689a-42d4-b178-7660e9a6ba56.PNG)

```
main -n { New Solution Name }
```
새로운 Test를 풀기위해 자동으로 cpp파일과 Sample을 저장하는 ini파일을 생성합니다

## Build

![](/images/300192169-39a97878-829f-42a5-a98d-1d6009da1093.PNG)

```
main -b { Solution Name }
```
확장자명 없이 cpp파일을 입력하면 자동으로 Build 합니다.

## Run

![](/images/300192303-da22c38d-a804-47cf-8c23-a903cae4f983.PNG)

```cpp
main -r { Solution Name }
```
ini파일이 미리 작성되어 있어야 합니다.

# ini File

![](/images/300192609-96ea59b0-1992-42b2-aeb4-8fcfaa7b9627.PNG)

Test를 수행할 Sample들을 저장하는 파일입니다.

ini를 확장자로 사용하고, cpp와 이름이 동일해야합니다.

"-Arg-"로 시작하여 그 아래로 Argument를 작성합니다.
<br>
"-Ans-" 이후 답을 작성합니다

Run을 통해 다수의 예제를 수행하고싶다면 Answer 아래에 다시 -Arg-부터 반복적으로 작성하면 됩니다.
<br>
자세한 예제는 Sample Directory를 확인하면 됩니다.

[Source](https://github.com/sinsin950313/EasyCodingTestWithVSCode)
