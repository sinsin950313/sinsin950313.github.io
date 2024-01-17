---
title: "Unreal 'dotnet'() 에러"
date: 2023-05-13
categories: ["Development", "Unreal"]
tags: ["ue5", "dotnet"]
---
글을 작성하는 것을 목적으로 문제를 해결한 것이 아니라 기억이 정확하지는 않지만 혹시라도 길을 헤멜 누군가를 위해 대략적으로 작성해둔다

UE5를 c++로 project를 만들려고 할 때 

> Running C:/Program Files/Epic Games/UE_5.1/Engine/Build/BatchFiles/Build.bat Development Win64 -Project="C:/Private_Map/ATeamProject/ATeamProject.uproject" -TargetType=Editor -Progress -NoEngineChanges -NoHotReloadFromIDE
Running UnrealBuildTool: dotnet "..\..\Engine\Binaries\DotNET\UnrealBuildTool\UnrealBuildTool.dll" Development Win64 -Project="C:/Private_Map/ATeamProject/ATeamProject.uproject" -TargetType=Editor -Progress -NoEngineChanges -NoHotReloadFromIDE
'dotnet'?(?? ?????Ǵ??ܺ???, ???????????α׷?, ?Ǵ?
??? ??? ?ƴմϴ?

이런 형태의 에러가 발생했다.

해석도 불가능한 상태라 뭐가 문제인지 알 수 없었는데 비슷한 키워드로 검색하던 중 

> unreal 5.1'dotnet' is not recognized as an internal or external command,  operable program or batch file.

이런 종류의 에러에 대한 글이 올라온것을 볼 수 있었다.

일단 해결책은 visual installer의 .net sdk를 다운받는것으로 기억한다
![](/images/9b282c68-db71-4a7a-85b7-7171dc31783a-image.PNG)

이 것을 다운받고 UE를 다시 켠 후 project를 생성하면 

> unreal 5.1 The framework 'Microsoft.NETCore.App', version '6.0.0' was not found.

이런 종류의 문제가 발생하는데 일단 전에 비해서 문자가 확실하게 보이므로 그나마 나은 상황이다
https://bloodstrawberry.tistory.com/1138

아무래도 이전에 문자열이 이상하게 나온건 정말로 파일이 존재하지 않아서였거나 한글-영문 문제로 인해 깨진것으로 짐작된다

아무튼 이렇게 하고도 아직 문제는 해결되지 않았는데 이번 문제는 쉽게 해결된다

https://dotnet.microsoft.com/en-us/download
여기를 들어가서 dotnet6.0을 다운받고 설치하면 깔끔하게 프로젝트가 생성된다.

UE의 Regacy로 인해 발생한 문제가 아닐까 짐작한다.
