---
title: "Unreal Detail Panel Customization과 Unity Inspector View Customization"
date: 2023-12-23
categories: ["Development", "Unreal"]
tags: ["ue"]
---
Unreal의 Detail Panel Customization을 보면 IDetailPanelCustomization과 IPropertyTypeCustomization 2개의 Interface를 제공

이는 각각 Unity의 CustomEditor, CustomPropertyDrawer Attribute에 대응하는데, 특이한점이 있다면 IPropertyTypeCustomization은 Class의 Member Variable은 인식하지 못하는 것으로 보임

재미있는점은 IPropertyTypeCustomization으로 Detail을 Customize한 Class는 어찌됐든 IPropertyTypeCustomization을 사용한다는 점

USTRUCT이 지정된 경우에만 Property Parsing을 수행하는걸지도?