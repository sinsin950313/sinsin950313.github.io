---
title: "Perforce-UE5 build error"
date: 2023-05-13
categories: ["Development", "Unreal"]
tags: ["perforce", "ue5", "build"]
---
혹시라도 Perforce를 통해 받은 .uproject를 통해 프로젝트를 빌드하려 할때 만약 compile이 되지 않는다면서 수동으로 빌드를 하라고 한다던가
> ATeamProject could not be compiled. Try rebuilding from source manually.

또는 로그파일에

> Building 9 actions with 4 processes...
[1/9] Resource Default.rc2
[2/9] Compile SharedPCH.Engine.ShadowErrors.cpp
[3/9] Compile ATeamProject.cpp
[4/9] Compile ATeamProject.init.gen.cpp
[5/9] Compile ATeamProjectGameModeBase.cpp
[6/9] Compile ATeamProjectGameModeBase.gen.cpp
[7/9] Link UnrealEditor-ATeamProject.lib
   C:\Private_Map\ATeamProject\Intermediate\Build\Win64\UnrealEditor\Development\ATeamProject\UnrealEditor-ATeamProject.lib ??̺귯?? ??C:\Private_Map\ATeamProject\Intermediate\Build\Win64\UnrealEditor\Development\ATeamProject\UnrealEditor-ATeamProject.exp ??ü?? ??ϰ????ϴ?
[8/9] Link UnrealEditor-ATeamProject.dll
   C:\Private_Map\ATeamProject\Intermediate\Build\Win64\UnrealEditor\Development\ATeamProject\UnrealEditor-ATeamProject.suppressed.lib ??̺귯?? ??C:\Private_Map\ATeamProject\Intermediate\Build\Win64\UnrealEditor\Development\ATeamProject\UnrealEditor-ATeamProject.suppressed.exp ??ü?? ??ϰ????ϴ?
[9/9] WriteMetadata ATeamProjectEditor.target
Unhandled exception: System.UnauthorizedAccessException: Access to the path 'C:\Private_Map\ATeamProject\Plugins\RLPlugin\Binaries\Win64\UnrealEditor.modules' is denied.
   at Microsoft.Win32.SafeHandles.SafeFileHandle.CreateFile(String fullPath, FileMode mode, FileAccess access, FileShare share, FileOptions options)
   at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String fullPath, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize)
   at System.IO.Strategies.OSFileStreamStrategy..ctor(String path, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize)
   at System.IO.Strategies.FileStreamHelpers.ChooseStrategyCore(String path, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize)
   at System.IO.Strategies.FileStreamHelpers.ChooseStrategy(FileStream fileStream, String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options, Int64 preallocationSize)
   at System.IO.StreamWriter.ValidateArgsAndOpenPath(String path, Boolean append, Encoding encoding, Int32 bufferSize)
   at System.IO.File.WriteAllText(String path, String contents)
   at UnrealBuildTool.WriteMetadataMode.ExecuteInternal(CommandLineArguments Arguments, ILogger Logger) in D:\build\++UE5\Sync\Engine\Saved\CsTools\Engine\Source\Programs\UnrealBuildTool\Modes\WriteMetadataMode.cs:line 194
   at UnrealBuildTool.WriteMetadataMode.Execute(CommandLineArguments Arguments, ILogger Logger) in D:\build\++UE5\Sync\Engine\Saved\CsTools\Engine\Source\Programs\UnrealBuildTool\Modes\WriteMetadataMode.cs:line 105
   at UnrealBuildTool.UnrealBuildTool.Main(String[] ArgumentsArray) in D:\build\++UE5\Sync\Engine\Saved\CsTools\Engine\Source\Programs\UnrealBuildTool\UnrealBuildTool.cs:line 648
LogInit: Warning: Still incompatible or missing module: ATeamProject

이런 종류의 로그가 남는다면 Project에서 사용하는 모든 파일을 Perforce에서 checkout하고 다시 빌드하는 것을 추천한다

이는 빌드에 사용하는 파일들 중 Perforce에서 checkout하지 않은 파일들이 Read only로 되어있어서 발생하는 문제로 checkout을 통해 RW가 가능하게 된다면 정상적으로 빌드된다.