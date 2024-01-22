---
title: "Easy Coding Test with VSCode"
date: 2024-01-22
categories: ["Development", "portfolio"]
tags: ["cpp", "portfolio", "vscode", "bat", "cmake"]
---
![](/images/298609727-7986e450-a9b3-49b1-bf9a-ef64373c94fc.PNG)

# VSCode에서 쉽게 Coding Test 연습하기
백준이나 HackerRank의 경우 std::cin으로 예제를 입력받는데 Test를 위해 Build이후 직접 값을 넣어주기 번거로워서 자동화시키는 bat파일입니다.

# 사용법
cmd에서 폴더에 위치한 후
```
main -h
```
를 누르면 도움말이 보입니다

```
main -n { New Solution Name }
```
새로운 Test를 풀기위해 자동으로 cpp파일을 생성합니다

```
main -b { Solution Name } { Argument File Name } { Argument Keyword } { Answer Keyword }
```
확장자명 없이 cpp파일이랑 그외 조건들을 입력하면 자동으로 Build 이후 실행합니다.

# Argument File
ini를 확장자로 사용하며 { Argument Keyword } { Answer Keyword } Argument Answer 만으로 구성됩니다.

{ Argument Keyword }로 시작하여 그 아래로 Argument를 작성합니다.
<br>
{ Answer Keyword } 이후 답을 작성합니다

다시 { Argument Keyword }가 존재한다면 다음 예제를 실행합니다.

[Source](https://github.com/sinsin950313/EasyCodingTestWithVSCode)
