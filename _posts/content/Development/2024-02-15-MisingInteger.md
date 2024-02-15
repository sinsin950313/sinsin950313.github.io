---
title: "Missing Integer"
date: 2024-02-15
categories: ["Development", "CodingTest"]
tags: ["cpp", "codingtest"]
math: true
---
```cpp
#include <algorithm>
#include <set>

int solution(vector<int> &A) {
    std::set<int> AlreadyChecked;

    std::sort(A.begin(), A.end());

    int MinValue = 1;
    for(int Var : A)
    {
        if(Var > 0)
        {
            if(AlreadyChecked.find(Var) == AlreadyChecked.end() && Var != MinValue)
            {
                return MinValue;
            }
            else
            {
                AlreadyChecked.insert(Var);
                MinValue = Var + 1;
            }
        }
    }

    return MinValue;
}
```
# 풀이 방법
자연수 중 Vector를 통해 들어온 값을 제외한 가장 작은 수를 찾는 문제

Vector를 통해 들어올 수 있는 값은 중복될 수 있고 음수나 0이 들어올 수 있음

일단 작은수부터 큰수로 정렬
<br>
가장 작은 자연수인 1을 MinValue로 지정한 후 비교를 시작
<br>
Vector를 통해 비교하는 값이 MinValue와 같을 경우 이는 현재 생각하고있는 가장 작은수는 Vector를 통해 들어왔다는 의미
<br>
또는 AlreadyChecked에 이미 있는 값이라면 중복해서 들어온 값이라는 뜻

AlreadyChecked에 존재하지 않고, MinValue와 Var가 같지 않다면 MinValue는 이전까지 들어온 값과 현재 비교중인 값 사이에 있다는 의미이므로 MinValue 반환

반복문을 벗어난다면 Vector는 자연수를 순서대로 넣었다는 의미이지만 MinValue가 매번 Var + 1로 갱신했으므로 Vector를 통해 들어온 값 중 가장 큰 값 + 1의 값을 반환
