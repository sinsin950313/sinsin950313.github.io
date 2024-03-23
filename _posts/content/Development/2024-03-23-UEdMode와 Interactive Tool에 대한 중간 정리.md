---
title: "UEdMode와 InteractiveTool에 대한 중간 정리"
date: 2024-03-23
categories: ["Development", "Unreal"]
tags: ["unreal", "engine extension", "edmode", "interactive tool", "package, asset, object"]
---

# UEdMode와 InteractiveTool

![](/images/316240255-324caf38-eb20-4e67-8b49-7905c1fc24f6.png)

UEdMode는 입력을 Interactive Tool을 이용하여 처리하는 것으로 보임.
<br>
이 때, Interactive Tool을 보면 기본적으로 지원하는 Behavior, Gizmo, Tool이 존재하는데 해당 정보를 어디에 저장해서 어떻게 접근해야 하는지 찾아봄.

InteractiveTool에서 사용하는 Behavior, Gizmo, Tool은 모두 해당하는 Manager에 저장되어야 하는데 해당 Manager는 모두 InteractiveToolContext에 저장되어야 함.

InteractiveTool에 대한 예제를 지원하는 Editor/Experimental/InteractiveToolsFramework에서는 Subsystem을 이용해서 GizmoBuilder를 관리하는 방식을 보여주는 것으로 보이는데 UEdMode는 그냥 InteractiveToolContext를 통해 접근하는 것으로 보여짐.

![](/images/2cf966fd-600f-4f7f-ba2a-089ab9c4b1a8.png)

InteractiveToolContext에 접근해야 Behavior나 Gizmo를 등록할텐데 어떻게 InteractiveToolContext에 접근하는지 찾아본 결과가 위에 있는 Sequence Diagram.

UEdMode는 Editor 사용자에 의해 호출되면 UEdMode::Enter()가 시작되는데, 여기서 Toolkit을 생성하면서 UEdMode(자기 자신)을 매개변수로 넘겨줌.
<br>
또한 Enter시점에서 InteractiveToolContext를 생성.
<br>
UE에서 지원하는 표준 예제를 볼 때 UEdMode의 Subclass에서 Enter 시점에 RegisterTool을 통해 Tool을 등록해야 하는데, 여기서 ToolContext를 통해 ToolManager에 접근해서 Tool을 등록하는 절차를 수행함.

즉, UEdMode를 통해 Context에 접근한 후 필요한 Builder(Tool, Gizmo 등)를 등록해야 하는데, 만약 Toolkit에서 하는 작업 중 등록이 필요하다면 Init시점에 매개변수로 넘겨준 UEdMode를 통해 Context에 접근해야 함.
<br>
이는 TWeakObjectPtr<UEdMode> OwningEditorMode로 protected 상태이므로 접근 가능함.

![](/images/4085523530406bff-45ae-4500-8868-bf4a7aa2b463.png)

참고로 UInteractiveTool::Setup()은 Enter시점에서 수행되는 것이 아니라 외부의 Tick시점에서 수행됨.

# Package, Asset, Object
Package : UAsset 파일
<br>
Asset : UE에서 Package를 읽어들인 정보. Contents Browser에서 표현되는 정보.
<br>
Object : 읽어들인 Asset을 UObject로 생성한 객체

[Reference](https://lifeisforu.tistory.com/360)
