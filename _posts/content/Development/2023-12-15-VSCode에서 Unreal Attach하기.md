---
title: "VSCode에서 Unreal Attach하기"
date: 2023-12-15
categories: ["Development", "Unreal"]
tags: ["ue", "vscode"]
---
.uproject에서 build하면 생성되는 code-workspace에 미리 적혀있는 launch-configurations에 추가
```cpp
{
	"name": "Attach UEProject (DebugGame)",
	"request": "attach",
	"processId": "${command:pickProcess}",
	"preLaunchTask": "UEProject Win64 DebugGame Build",
	"type": "cppvsdbg",
	"visualizerFile": ...
}
```
그냥 복사할 경우 request, processId 수정에 주의