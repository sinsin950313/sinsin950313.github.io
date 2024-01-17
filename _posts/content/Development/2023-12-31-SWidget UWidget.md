---
title: "SWidget UWidget"
date: 2023-12-31
categories: ["Development", "Unreal"]
tags: ["ue"]
---
SWidget은 내부에 OnPaint()같이 Draw관련 함수가 보이지만, UWidget의 경우 Draw관련 함수가 하나도 보이지 않는 대신, SWidget의 SharedPtr을 반환하는 TakeWidget()이나 GetCachedWidget()같은 함수를 갖고있는 것을 확인할 수 있음

TakeWidget의 내부를 살펴볼 경우 SNew를 통해 직관적으로 SWidget을 생성해서 반환하는 것을 확인할 수 있는데, 이를 통해 UWidget을 사용하는 UMG의 경우 SWidget을 사용하는 Slate의 일종의 Wrapper 객체로 해석하면 될 듯