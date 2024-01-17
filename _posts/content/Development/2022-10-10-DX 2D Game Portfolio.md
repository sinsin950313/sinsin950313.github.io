---
title: "DX 2D Game Portfolio"
date: 2022-10-10
categories: ["Development", "KGCA"]
tags: ["2d", "directx", "kgca", "portfolio"]
---
DirectX를 이용하여 2D Game을 구현해보았다.
![](/images/4853989b-7913-4275-a5d4-627a03215cb0-image.png)

## 기술스택
![](/images/24144daa-6db5-425a-912c-97d475673294-image.png)


## 구현기술
### MVC 패턴
![](/images/139eaf01-cc89-4055-af09-c74321e123a0-image.png)
QuadTree Collision System과 DirectX를 MVC를 이용하여 구현

### Multi Layer
![](/images/18b3b19f-2a2b-475c-9711-d1129aeaaeab-image.png)
2D TopView시점에서 전투기의 높이감을 표현하기 위해 Collision Tree를 여러개 설치하는 MultiLayer를 구현
![](/images/dd29dce5-bd57-4728-9fa5-496eefed2a66-image.png)
MultiLayer를 위한 AI
![](/images/75c9f9f0-3de4-4211-8384-6086ef4324b3-image.png)
MultiLayer에서 공격 판정을 위한 상대위치 기반 HitBox
![](/images/45cdd6ed-1ab6-446f-bc77-626c8f774967-image.png)
2D TopView에서 높이감을 표현하기위해 상대 높이 차이에 따른 크기 변경

### Memory Pool
![](/images/6b3d9139-c808-41aa-a6ff-d2cbc8968b97-image.png)
![](/images/4f85f0a1-bd2b-443a-a8de-2390ce22c6d3-image.png)

지속적으로 이동하는 전투기의 특성 상 전투기의 크기만큼 배경을 할당한다면 배경의 크기에 비례하여 연산량이 증가하는 단점이 존재
Texture의 범위를 0~1 사이 이외의 값을 할당하는 Scroll방식으로 구현할 경우 반복적인 출력으로 User에서 실감나는 경험을 제공하지 못하는 문제가 존재
이 문제를 해결하기 위해 다수의 Texture가 출력될 수 있는 Tile과 Tile들을 관리, 할당하는 Tile Memory Pool 구현

### UI Text
![](/images/a90faec8-b6d5-4a20-8396-f02795d5a8e4-image.png)
dwrite.h를 통해 문자를 출력할 경우 Device에서 많은 연산이 소요됨
제한 시간후에 공수교대가 진행되는 게임의 특성 상 매 연산마다 잔여 시간의 출력이 필요하므로 연산량을 낮추기위해 원하는 위치에 원하는 문자를 입력할 경우 지정된 Font로 제한 시간을 출력할 수 있는 UI Text를 구현

## 조작법
![](/images/8a65101c-b047-456f-8e81-da9d14bb86f6-image.png)
![](/images/4a30576c-9ec6-4e86-ba94-879901e12f57-image.png)
마우스를 당기면 상승, 내리면 하강
![](/images/96082ddc-55b3-4940-8542-fd11043867fb-image.png)
![](/images/3a2d9f5d-e662-431f-b803-24e682bb82cf-image.png)
![](/images/ed63e558-2947-4846-8cb8-76e73c18ce3b-image.png)

## 구현
![](/images/254e5982-77d7-4d22-b814-6ee8634af438-image.png)
![](/images/132f0926-3f9a-4048-b8fe-b4104147ad84-image.png)
![](/images/90fb2655-e99d-442d-a0ee-05f328e277b0-image.png)
![](/images/f034f4a4-6ae3-4db1-bce6-75268e310032-image.png)