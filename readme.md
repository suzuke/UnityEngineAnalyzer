UnityEngineAnalyzer
===================

UnityEngineAnalyzer is a set of Roslyn analyzers that aim to detect common problems in Unity3D C# code. Unity3D makes it easy for us to make cross platform games, but there are hidden rules about performance and AOT, which might only come with experience, testing or reading the forums. It is hoped that such problems can be caught before compilation.

Analyzers
---------------------

| Analyzer                               | Name                       | Gategory       | Severity    |
| -------------------------------------- | -------------------------- | ---------------| ------------|
| [UEA0001](Documents/analyzers/UEA0001) |DoNotUseOnGUI               | GC             |Info         |
| [UEA0002](Documents/analyzers/UEA0002) |DoNotUseStringMethods       | GC             |Info         |
| [UEA0003](Documents/analyzers/UEA0003) |EmptyMonoBehaviourMethod    | Miscellaneous  |Warning      |
| [UEA0004](Documents/analyzers/UEA0004) |UseCompareTag               | GC             |Warning      |
| [UEA0005](Documents/analyzers/UEA0005) |DoNotUseFindMethodsInUpdate | Performance    |Warning      |
| [UEA0006](Documents/analyzers/UEA0006) |DoNotUseCoroutines          | GC             |Info         |
| [UEA0007](Documents/analyzers/UEA0007) |DoNotUseForEachInUpdate     | Performance    |Warning      |
| [UEA0008](Documents/analyzers/UEA0008) |UnsealedDerivedClass        | Performance    |Warning      |
| [UEA0009](Documents/analyzers/UEA0009) |InvokeFunctionMissing       | Performance    |Warning      |
| [UEA0010](Documents/analyzers/UEA0010) |DoNotUseStateNameInAnimator | Performance    |Warning      |
| [UEA0011](Documents/analyzers/UEA0011) |DoNotUseStringPropertyNames | Performance    |Warning      |
| [UEA0012](Documents/analyzers/UEA0012) |CameraMainIsSlow            | GC             |Warning      |
| [UEA0013](Documents/analyzers/UEA0013) |UseNonAllocMethods          | GC             |Warning      |
| [UEA0014](Documents/analyzers/UEA0014) |AudioSourceMuteUsesCPU      | Performance    |Info         |
| [UEA0015](Documents/analyzers/UEA0015) |InstantiateTakeParent       | Performance    |Warning      |
| -------------------------------------- |----------------------------|--------------- | ----------- |
| [AOT0001](Documents/analyzers/AOT0001) |DoNotUseRemoting            | AOT            |Info         |
| [AOT0002](Documents/analyzers/AOT0002) |DoNotUseReflectionEmit      | AOT            |Info         |
| [AOT0003](Documents/analyzers/AOT0003) |TypeGetType                 | AOT            |Info         |

Building CLI executable
---------------------

CLI requires .NET Core 2.1

```
dotnet publish -c Release -r win10-x64
```
or
```
dotnet publish -c Release -r ubuntu.16.10-x64
```

Command Line Interface
---------------------

In order to use the Command Line Interface (CLI), download the latest release of UnityEngineAnalyzer then unzip the archive (https://github.com/vad710/UnityEngineAnalyzer/releases).

1. Open a Command Prompt or Powershell Window
1. Run `Linty.CLI.exe <project path>`
1. Observe the analysis results
1. (Optional) In the same location as the project file are `report.json` and `UnityReport.html` files containing the results of the analysis
    * Use command `-e customexporter exporter2 ...` to load custom exporters
1. (Optional) configuration file path.
    * Use command `-c configureFilePath.json` to load custom configurations
	* Configuration json, allows to enable / disable analyzers
1. (Optional) minimal severity for reports
	* Use command `-s Info/Warning/Error` to defined used minimal severity for reporting
	* Default is Warning
1.	(Optional) Unity version for check
	* Use command `-v UNITY_2017_1/UNITY_5_5/UNITY_4_0/...` to Unity version
	* For default analyzer will try to find ProjectVersion.txt file and parse version automatically.

Example:

`> Linty.CLI.exe C:\Code\MyGame.CSharp.csproj` 


Visual Studio Integration
-------------------------

In Visual Studio 2017, go to `Tools > Nuget Package Manager > Manage Nuget Packages for Solution...`. Search for and install `UnityEngineAnalyzer`

Configuration
-------------

Under projects right-click `Analyzers` to modify the severity or to disable the rule completely.

![](https://raw.githubusercontent.com/meng-hui/UnityEngineAnalyzer/master/Documents/configuration.png)

Limitations
-----------

- HTML Report requires FireFox or XOR (Corss Origin Request) enabled in other browsers
- It doesn't have rules for all of [Mono's AOT Limitations](https://developer.xamarin.com/guides/ios/advanced_topics/limitations/)
- IL2CPP might change the limitations of AOT compilation


```C#
// AOT0001: System.Runtime.Remoting is not suppported
using System.Runtime.Remoting;
// AOT0002: System.Reflection.Emit is not supported
using System.Reflection.Emit;
using UnityEngine;

class FooBehaviour : MonoBehaviour 
{
	void Start()
	{
		// AOT0003: Reflection only works for looking up existing types
		Type.GetType("");

		// UEA0002: Using string methods can lead to code that is hard to maintain
		SendMessage("");

		// UEA0006: Use of coroutines cause some allocations
		StartCoroutine("");
	}

	// UEA0001: Using OnGUI causes allocations and GC spikes
	void OnGUI()
	{

	}

	// UEA0003: Empty MonoBehaviour methods are executed and incur a small overhead
	void FixedUpdate()
	{

	}

	void OnTriggerEnter(Collider other)
	{
		// UEA0004: Using CompareTag for tag comparison does not cause allocations
		if (other.tag == "")
		{

		}
	}

	void Update()
	{
		// UEA0005: Warning to cache the result of find in Start or Awake
		GameObject.Find("");
	}
}
```

License
-------

See [LICENSE](https://raw.githubusercontent.com/meng-hui/UnityEngineAnalyzer/master/LICENSE)
