---
title: "PIE가 아닌 상태에서 Widget 붙이기"
date: 2024-03-27
categories: ["Development", "Unreal"]
tags: ["unreal", "engine extenstion", "slate"]
---

![](/images/f2e28287-70f7-4e3a-ad5a-931246ab37f1.png)

<center>PIE가 아닌 상태에서 Button Widget이 붙어있다.</center>

# Viewport, ViewportClient
화면에 그리는 Viewport는 Viewport, ViewportClient, PreviewScene으로 구성된다.
<br>
[참조](https://blog.naver.com/destiny9720/221383661821)

일반적으로 UUserWidget을 Viewport에 추가할 경우 UUserWidget::AddToViewport()함수를 사용하는데 내부 구문을 살펴보면 World가 GameWorld인 경우만 추가가 가능하다.
```cpp
// UGameViewportSubsystem::AddToScreen 중
UWorld* World = Widget->GetWorld();
if (!World || !World->IsGameWorld())
{
	FFrame::KismetExecutionMessage(*FString::Printf(TEXT("The widget '%s' does not have a World."), *Widget->GetName()), ELogVerbosity::Warning);
	return false;
}
```

결국 현재 화면에 출력되는 Viewport를 찾아서 직접 추가를 해줘야하는데, Editor화면에서 보여주는 Viewport는 LevelEditor Module에 속한 Viewport이다.

```cpp
UserWidget = MakeShareable(CreateWidget(World, WidgetBlueprint->GeneratedClass.Get(), "TestWidget"));
if(UserWidget != nullptr)
{
	FLevelEditorModule &LevelEditorModule = FModuleManager::Get().GetModuleChecked<FLevelEditorModule>(TEXT("LevelEditor"));

	TWeakPtr<ILevelEditor> LevelEditorInstance = LevelEditorModule.GetLevelEditorInstance();
	TSharedPtr<SLevelViewport> LevelViewport = LevelEditorInstance.Pin()->GetActiveViewportInterface();
	LevelViewport->AddOverlayWidget(
		SNew(SOverlay)
			+ SOverlay::Slot()
				.VAlign(EVerticalAlignment::VAlign_Top)
				.HAlign(EHorizontalAlignment::HAlign_Center)
				[
					UserWidget->TakeWidget()
				]
	);
}
```

LevelEditor에서 사용하는 Viewport인 SLevelViewport는 SOverlay를 사용하는 SCompoundWidget이므로 SOverlay에 Widget을 추가하면 PIE가 아닌 상태에서도 SWidget을 Viewport에 추가할 수 있다.

## 고려사항
![](/images/737ac95d-0ba7-4a43-bc46-8480f08fb14d.png)

UUserWidget을 바로 LevelViewport에 붙여줄 경우 Desired Size를 기준으로 붙는 문제가 발생
<br>
이를 GameViewport에 붙이듯이 Screen Size로 붙이는 방법에 대한 고민 필요