# Commandlet
https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Commandlets/UCommandlet

## 주요 함수
```cpp
class UCommandlet : public UObject
{
	GENERATED_UCLASS_BODY()

    // Helper 뭐시기 적힌 구문들은 다 큰 의미가 없음?
	/** Description of the commandlet's purpose */
	UPROPERTY()
	FString HelpDescription;

	/** Usage template to show for "ucc help" */
	UPROPERTY()
	FString HelpUsage;

	/** Hyperlink for more info */
	UPROPERTY()
	FString HelpWebLink;

	/** The name of the parameter the commandlet takes */
	UPROPERTY()
	TArray<FString> HelpParamNames;

	/** The description of the parameter */
	UPROPERTY()
	TArray<FString> HelpParamDescriptions;

    // 여기서 설정을 어떻게 하느냐에 따라 Commandlet을 Editor에서 돌리는지 Client - Server에서 돌리는지를 결정하는 듯
	/**
	 * Whether to load objects required in server, client, and editor context.  If IsEditor is set to false, then a
	 * UGameEngine (or whatever the value of /Script/Engine.Engine.GameEngine is) will be created for the commandlet instead
	 * of a UEditorEngine (or /Script/Engine.Engine.EditorEngine), unless the commandlet overrides the CreateCustomEngine method.
	 */
	UPROPERTY()
	uint32 IsServer:1;

	UPROPERTY()
	uint32 IsClient:1;

	UPROPERTY()
	uint32 IsEditor:1;
	/**
	 * Entry point for your commandlet
	 *
	 * @param Params the string containing the parameters for the commandlet
	 */
	virtual int32 Main(const FString& Params) { return 0; }

    // - 또는 / 로 시작하는 단어들을 Switch라고 명명하는 것으로 보임
	/**
	 * Parses a string into tokens, separating switches (beginning with - or /) from
	 * other parameters
	 *
	 * @param	CmdLine		the string to parse
	 * @param	Tokens		[out] filled with all parameters found in the string
	 * @param	Switches	[out] filled with all switches found in the string
	 *
	 * @return	@todo document
	 */
	static void ParseCommandLine( const TCHAR* CmdLine, TArray<FString>& Tokens, TArray<FString>& Switches );

	/**
	 * Parses a string into tokens, separating switches (beginning with - or /) from
	 * other parameters
	 *
	 * @param	CmdLine		the string to parse
	 * @param	Tokens		[out] filled with all parameters found in the string
	 * @param	Switches	[out] filled with all switches found in the string
	 * @param	Params		[out] filled with all switches found in the string with the format key=value
	 *
	 * @return	@todo document
	 */
	static void ParseCommandLine( const TCHAR* CmdLine, TArray<FString>& Tokens, TArray<FString>& Switches, TMap<FString, FString>& Params );
};
```

## 특이 사항
    1. Helper 뭐시기로 시작하는 FString들은 큰 의미가 없음
    2. IsEditor, IsServer, IsClient를 적절하게 설정해줘야 실행 환경에서 Commandlet이 적절하게 수행됨
    3. -, / 로 시작하는 매개변수를 Switch라고 하는 것으로 보이며 ParseCommandLine 함수를 통해 간단하게 매개변수로 들어온 값들을 분리할 수 있음

## 예제
```cs
// Build.cs
using UnrealBuildTool;

public class Test : ModuleRules
{
    public Test(ReadOnlyTargetRules Target) : base(Target)
    {
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
	
        PublicDependencyModuleNames.AddRange(new string[] {
            "Core",
            "CoreUObject",
            "Engine",
            "InputCore"
        });

        PrivateDependencyModuleNames.AddRange(new string[] {
            "UnrealEd", // Required for Commandlet functionality
        });
    }
}
```
Commandlet은 UnrealEd 모듈에 포함되어있으므로 해당 모듈이 필수적으로 추가되어야 함

```cpp
// Test.h
#pragma once

#include "Commandlets/Commandlet.h"
#include "SquadMemberTestCommandlet.generated.h"

UCLASS()
// 클래스 이름은 Commandlet으로 끝나야 함
class UTestCommandlet : public UCommandlet
{
    GENERATED_BODY()

public:
    UTestCommandlet();

    virtual int32 Main(const FString& Params) override;
};
```
Commandlet header include 후 Main override

```cpp
UTestCommandlet::UTestCommandlet()
{
    IsEditor = true;
    IsClient = false;
    IsServer = false;
}

int32 UTestCommandlet::Main(const FString &Params)
{
    TArray<FString> Tokens;
    TArray<FString> Switches;
    ParseCommandLine(*Params, Tokens, Switches);

    // Logic
    UE_LOG(LogTemp, Display, TEXT("Hello World!"));	// Display로 설정해야 cmd에서 보임

    // 정상 종료
    return 0;
}
```
Editor, Client, Server 어디에서 Commandlet을 수행할 것인지를 Constructor에서 미리 정의

Main에서 필요 시 Token과 Switch 파싱

이후 Commandlet에서 수행하고자 하는 Logic 구현

## 사용 방법
```
[UnrealEditor-cmd.exe 경로] [.uproject 경로] -run=[Commandlet 클래스 이름] ParamsAndSwitches...
```
Commandlet 클래스 이름에는 Commandlet을 꼭 붙일 필요는 없음

Build된 exe에서 실행 시 uproject는 필요 없음(GPCC 참고)

## 출력창
```
[2025.04.20-07.39.10:195][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatASTC
[2025.04.20-07.39.10:196][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatDXT
[2025.04.20-07.39.10:196][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatETC2
[2025.04.20-07.39.10:196][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatIntelISPCTexComp
[2025.04.20-07.39.10:196][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatUncompressed
[2025.04.20-07.39.10:196][  0]LogTextureFormatOodle: Display: Oodle Texture TFO init; latest sdk version = 2.9.12
[2025.04.20-07.39.10:196][  0]LogTextureFormatOodle: Display: Oodle Texture loading DLL: oo2tex_win64_2.9.12.dll
[2025.04.20-07.39.10:196][  0]LogTextureFormatOodle: Display: Oodle Texture loading DLL: oo2tex_win64_2.9.5.dll
[2025.04.20-07.39.10:197][  0]LogTextureFormatManager: Display: Loaded Base TextureFormat: TextureFormatOodle
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_ASTC'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_DXT'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_ETC2'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'AndroidClient'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_ASTCClient'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_DXTClient'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_ETC2Client'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_Multi'
[2025.04.20-07.39.10:242][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Android_MultiClient'
[2025.04.20-07.39.10:259][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Linux'
[2025.04.20-07.39.10:259][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxEditor'
[2025.04.20-07.39.10:259][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxServer'
[2025.04.20-07.39.10:260][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxClient'
[2025.04.20-07.39.10:277][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxArm64'
[2025.04.20-07.39.10:277][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxArm64Server'
[2025.04.20-07.39.10:278][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'LinuxArm64Client'
[2025.04.20-07.39.10:297][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'Windows'
[2025.04.20-07.39.10:297][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'WindowsEditor'
[2025.04.20-07.39.10:298][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'WindowsServer'
[2025.04.20-07.39.10:298][  0]LogTargetPlatformManager: Display: Loaded TargetPlatform 'WindowsClient'
[2025.04.20-07.39.10:298][  0]LogTargetPlatformManager: Display: Building Assets For WindowsEditor
[2025.04.20-07.39.10:343][  0]LogAudioDebug: Display: Lib vorbis DLL was dynamically loaded.
[2025.04.20-07.39.10:484][  0]LogDerivedDataCache: Display: Memory: Max Cache Size: -1 MB
[2025.04.20-07.39.10:485][  0]LogZenServiceInstance: Display: Launching executable 'C:/Users/sinsi/AppData/Local/UnrealEngine/Common/Zen/Install/zenserver.exe', working dir 'C:/Users/sinsi/AppData/Local/UnrealEngine/Common/Zen/Install', data dir 'C:/Users/sinsi/AppData/Local/UnrealEngine/Common/Zen/Data', args '--port 8558 --data-dir "C:\Users\sinsi\AppData\Local\UnrealEngine\Common\Zen\Data" --http asio --gc-cache-duration-seconds 1209600 --gc-interval-seconds 21600 --gc-low-diskspace-threshold 2147483648 --http-forceloopback --quiet --owner-pid 6896'
[2025.04.20-07.39.10:595][  0]LogZenServiceInstance: Display: Unreal Zen Storage Server HTTP service at [::1]:8558 status: OK!.
[2025.04.20-07.39.10:596][  0]LogDerivedDataCache: Display: ZenLocal: Using ZenServer HTTP service at http://[::1]:8558/ with namespace ue.ddc status: OK!.
[2025.04.20-07.39.10:606][  0]LogDerivedDataCache: Display: ../../../Engine/DerivedDataCache: Performance: Latency=0.01ms. RandomReadSpeed=773.58MBs, RandomWriteSpeed=110.93MBs. Assigned SpeedClass 'Local'
[2025.04.20-07.39.10:608][  0]LogShaderCompilers: Display: Using Local Shader Compiler with 9 workers.
[2025.04.20-07.39.10:609][  0]LogShaderCompilers: Display: Compiling shader autogen file: ../../../../Test/Intermediate/ShaderAutogen/PCD3D_SM5/AutogenShaderHeaders.ush
[2025.04.20-07.39.10:609][  0]LogShaderCompilers: Display: Autogen file is unchanged, skipping write.
[2025.04.20-07.39.11:625][  0]LogEditorDomain: Display: EditorDomain is Disabled
[2025.04.20-07.39.11:740][  0]LogDeviceProfileManager: Display: Deviceprofile LinuxArm64Editor not found.
[2025.04.20-07.39.11:740][  0]LogDeviceProfileManager: Display: Deviceprofile LinuxArm64 not found.
[2025.04.20-07.39.11:757][  0]LogCsvProfiler: Display: Metadata set : deviceprofile="WindowsEditor"
[2025.04.20-07.39.11:757][  0]LogShaderCompilers: Display: Compiling shader autogen file: ../../../../Test/Intermediate/ShaderAutogen/PCD3D_SM6/AutogenShaderHeaders.ush
[2025.04.20-07.39.11:757][  0]LogShaderCompilers: Display: Autogen file is unchanged, skipping write.
[2025.04.20-07.39.11:773][  0]LogTextureEncodingSettings: Display: Texture Encode Speed: Final (cook).
[2025.04.20-07.39.11:773][  0]LogTextureEncodingSettings: Display: Oodle Texture Encode Speed settings: Fast: RDO Off Lambda=0, Effort=Normal Final: RDO Off Lambda=0, Effort=Normal
[2025.04.20-07.39.11:773][  0]LogTextureEncodingSettings: Display: Shared linear texture encoding: Disabled
[2025.04.20-07.39.11:898][  0]LogMeshReduction: Display: Using QuadricMeshReduction for automatic static mesh reduction
[2025.04.20-07.39.11:898][  0]LogMeshReduction: Display: Using SkeletalMeshReduction for automatic skeletal mesh reduction
[2025.04.20-07.39.11:898][  0]LogMeshReduction: Display: Using ProxyLODMeshReduction for automatic mesh merging
[2025.04.20-07.39.11:898][  0]LogMeshReduction: Display: No distributed automatic mesh merging module available
[2025.04.20-07.39.12:062][  0]LogVirtualization: Display: VirtualizationSystem name found in ini file: None
[2025.04.20-07.39.12:063][  0]LogVirtualization: Display: FNullVirtualizationSystem mounted, virtualization will be disabled
[2025.04.20-07.39.12:082][  0]LogLiveCoding: Display: Starting LiveCoding
[2025.04.20-07.39.12:082][  0]LogLiveCoding: Display: LiveCodingConsole Arguments: UnrealEditor Win64 Development
[2025.04.20-07.39.12:083][  0]LogLiveCoding: Display: First instance in process group "UE_Test_0x65a7c2f3", spawning console
[2025.04.20-07.39.12:094][  0]LogLiveCoding: Display: Waiting for server
[2025.04.20-07.39.12:777][  0]LogLiveCoding: Display: Successfully initialized, removing startup thread
[2025.04.20-07.39.13:243][  0]LogWorldPartition: Display: FWorldPartitionClassDescRegistry::Initialize started...
[2025.04.20-07.39.13:244][  0]LogWorldPartition: Display: FWorldPartitionClassDescRegistry::Initialize took 1.296 ms
[2025.04.20-07.39.14:123][  0]LogPython: Display: No enabled plugins with python dependencies found, skipping
[2025.04.20-07.39.15:152][  0]LogAudio: Display: Registering Engine Module Parameter Interfaces...
[2025.04.20-07.39.15:581][  0]LogLiveCoding: Display: LiveCodingConsole Arguments: TestEditor Win64 Development
[2025.04.20-07.39.17:105][  0]LogAudioCaptureCore: Display: No Audio Capture implementations found. Audio input will be silent.
[2025.04.20-07.39.17:105][  0]LogAudioCaptureCore: Display: No Audio Capture implementations found. Audio input will be silent.
[2025.04.20-07.39.18:034][  0]LogCsvProfiler: Display: Metadata set : largeworldcoordinates="1"
[2025.04.20-07.39.18:411][  0]LogTemp: Display: Hello World!	// <<<< 정상 수행
[2025.04.20-07.39.18:411][  0]LogInit: Display: 
[2025.04.20-07.39.18:411][  0]LogInit: Display: Success - 0 error(s), 0 warning(s)
[2025.04.20-07.39.18:411][  0]LogInit: Display:
Execution of commandlet took:  0.00 seconds
[2025.04.20-07.39.18:568][  0]LogShaderCompilers: Display: Shaders left to compile 0
[2025.04.20-07.39.18:635][  0]LogStudioTelemetry: Display: Shutdown StudioTelemetry Module
```
앞에 Commandlet과 무관해보이는 항목에 관해 출력하는 부분이 조금 있으니 로그를 자세히 읽어볼 것