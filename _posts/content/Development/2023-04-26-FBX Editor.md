---
title: "FBX Editor"
date: 2023-04-26
categories: ["Development", "portfolio"]
tags: ["cpp", "mfc", "directx", "portfolio"]
---

# Character Tool
![](/images/234487473-02631f39-ed76-4d9e-88b1-e64328b5cda5.png)

FBX를 읽고 수정할 수 있는 툴 입니다.

## 개요
- 제작 기간 : 1달
- 개발 인원 : 1인
- 개발 환경
  - C++
  - DirectX
  - MFC
- 주요 기술
  - FBX SDK
  - Bone Hierarchy - Socket
  - Skinning Mesh
  - Multi Texture
  - 3D Camera

## 주요 기능
![](/images/234487650-7102ac4f-6e39-436a-9f21-5b6991a86887.png)

Bone구조 확인 및 Socket 추가 가능

![](/images/234487689-d9f3f809-fe35-4f49-817a-aa6a97412def.png)

Animation 추가 및 삭제 가능

![](/images/234487875-0533e8c1-0e2e-4961-9fa5-22e94075063b.PNG)

Text파일 IO 가능

## 사용설명서

![](/images/234503814-1ed8e4d1-d1e0-4024-b414-afbcf7682726.PNG)

Character Editor\Debug\CharacterToolWindow에 있는 CharacterToolWindow.exe 실행

![](/images/234488872-1ac2f2f4-27f7-4987-84fb-6dbca71bd305.png)

FBX 및 Text파일인 Script 파일 IO

![](/images/234504209-582b8dfe-9efc-4ca9-9a07-dc8adb172b6a.PNG)

상대 경로를 유지하면서 FBX또는 Script 디렉토리에 파일 배치 및 실행

![](/images/234488617-1f22864f-3cb9-485d-b712-9b619b588f13.png)

1번 : Animation 이름 및 길이, Animation 추가 및 삭제

2번 : Animation Name 수정, Animation 길이 수정 및 Cutting

![](/images/234488660-431aa8ce-5995-4f94-a71e-3a970e2b7c9d.png)

1번 : Bone 계층 구조 확인

2번 : 1번에서 선택된 위치에 상대좌표로 Socket 추가

[다운로드](https://naver.me/5oizaMNG)

[Source](https://github.com/sinsin950313/KGCA/tree/main/Project/KGCA41/CharacterToolWindow)