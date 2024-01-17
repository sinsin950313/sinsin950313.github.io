---
title: "WidgetBlueprint에서 Custom Widget을 Palette에 붙이기"
date: 2023-12-27
categories: ["Development", "Unreal"]
tags: ["ue"]
---
![](/images/ec159523-a9d0-4302-bc2b-508fc42c220a-image.PNG)

```cpp
// .h
UCLASS(BlueprintType)
class UCustomButtonAnimatorComponent : public UWidget
{
    GENERATED_BODY()

#if WITH_EDITOR
	DOOZYIMITATION_API virtual const FText GetPaletteCategory() override;
#endif
}

// .cpp
#define LOCTEXT_NAMESPACE "UMG"

#if WITH_EDITOR

const FText UCustomButtonAnimatorComponent::GetPaletteCategory()
{
	return LOCTEXT("AAA", "BBB");
}

#endif

#undef LOCTEXT_NAMESPACE
```

Widget Blueprint에서 Palette에 Customize한 Widget을 추가하고자 할 경우 간단하게 UWidget을 상속하면 됨

카테고리 분류를 원할 경우 GetPaletteCategory함수 재정의

외부에서 등록하거나 하는 절차가 없는게 신기하네