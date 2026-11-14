# Windows baseline build inputs

This inventory describes public Windows fork commit `2ad50e06f9bcef053e9fa773a8939a9484059081`, based on Safe Exam Browser `v3.10.3` (`ebe28a2553894a5f67d8c69175679f4ca832f73b`). See [toolchain.json](toolchain.json) for the source pins, commands and observed tool versions.

## Solution and dependencies

- [SafeExamBrowser.sln](../SafeExamBrowser.sln) defines Debug and Release for x64 and x86. The managed projects target .NET Framework 4.8 and include the runtime, client, service, configuration tool, reset utility and unit tests. The solution selects `Setup` in both Release configurations and `SetupBundle` only in Release x86. The ARM64 CEF package is a restore input. The solution has no ARM64 build configuration.
- Use native Windows x64, PowerShell, NuGet CLI on PATH, Visual Studio 2022 with x64 MSBuild and .NET desktop build tools, and the .NET Framework 4.8 targeting pack/reference assemblies. Install the .NET Framework 4.8 runtime for execution. The solution’s Visual Studio metadata describes its saved version and the workflow selects an installed MSBuild in `[17.0,18.0)`. The [upstream README](../README.md#requirements) lists Windows 10 version 1803 as the minimum and the Visual C++ 2015–2022 Redistributable as a runtime prerequisite.
- The authoritative package pins live in each project’s `packages.config` and in the [SystemComponents project’s PackageReference declarations](../SafeExamBrowser.SystemComponents/SafeExamBrowser.SystemComponents.csproj). See the [browser manifest](../SafeExamBrowser.Browser/packages.config) for CefSharp 152.0.100 and CEF 152.0.10, the [test manifest](../SafeExamBrowser.Applications.UnitTests/packages.config) for MSTest 4.4.1 and Moq 4.21.0, and the [runtime manifest](../SafeExamBrowser.Runtime/packages.config) for OpenCover 4.7.1221. The package restore uses runner NuGet configuration. The historical job listed NuGet.org and Visual Studio’s local offline source. There is no checked-in NuGet source configuration or PackageReference lock file.
- The SystemComponents project restores Microsoft.Windows.SDK.Contracts 10.0.28000.2705 from NuGet. The managed projects declare no standalone Windows SDK version. Record the SDK used for packaging and SignTool separately. The precise NuGet CLI version and Windows SDK used by the hosted job were unverified.

## Repeat the Debug x64 build

From the root of a clean checkout at the inventoried fork commit on native Windows x64, run the restore and compile step from the [manual GitHub Actions probe](../.github/workflows/windows-x64-build-probe.yml). The workflow fetches its own fixed source commit. Verify the checkout SHA before a local build.

```powershell
$vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
$msbuild = & $vswhere -latest -version '[17.0,18.0)' -products '*' -requires Microsoft.Component.MSBuild -find 'MSBuild\Current\Bin\amd64\MSBuild.exe' | Select-Object -First 1
if (-not $msbuild) {
  throw 'Visual Studio 2022 x64 MSBuild was not found.'
}
nuget restore SafeExamBrowser.sln -NonInteractive
if ($LASTEXITCODE -ne 0) {
  exit $LASTEXITCODE
}
& $msbuild SafeExamBrowser.sln /verbosity:minimal /property:LangVersion=latest /property:Configuration=Debug /property:Platform=x64
```

The hosted probe then runs each unit-test assembly from `bin\x64\Debug` with the Visual Studio VSTest console and the restored MSTest 4.4.1 `net462` adapter. The upstream [AppVeyor test configuration](../appveyor-test.yml) declares Debug x64 and x86 on Visual Studio 2022 with a separate OpenCover coverage step.

[Historical run 43458510798](https://github.com/Iconic-University/ibrowser-windows/actions/runs/43458510798) used workflow commit `35fa4312dfd2ae035c26c55749b62731a3fca855` and compiled source `780ae837043a7368b19112ee5e84e6df62258347`. Restore, Debug x64 compilation and all 720 tests across 11 suites passed on `windows-2022` image `20261112.366.7` with MSBuild `17.14.60+43b635718` and x64 VSTest `17.14.0`. Only the workflow’s source-pin line differs between that compiled commit and the inventoried fork commit. This inventory used static inspection and the historical log. A fresh native run of the inventoried commit and a Debug x86 run remain unverified.

## Installer inputs and remaining Windows checks

- The [AppVeyor release configuration](../appveyor-release.yml) builds Release x64 first and stages its MSI for Release x86. [Setup](../Setup/Setup.wixproj) produces `Setup\bin\<platform>\Release\Setup.msi`. The x86 Release job also builds [SetupBundle](../SetupBundle/SetupBundle.wixproj), whose [chain](../SetupBundle/Bundle.wxs) requires both Release MSIs and emits `SetupBundle\bin\x86\Release\SetupBundle.exe`.
- Both installer projects import WiX v3 targets and declare WiX Toolset 3.11 or newer. Use a compatible WiX v3 installation with MSBuild integration, Heat, WiX UI/NetFx extensions for the MSI and NetFx/Util/Bal extensions for the bundle. Setup uses the `WIX` environment variable to find Heat and Git to restore harvested source fragments. The bundle pre-build step uses curl and writable `C:\Temp` to fetch the [.NET Framework 4.8 web installer](../SetupBundle/DotNetFramework.wxs) and a WebView2 installer. The checked-in chain references .NET, [VC++ 14.44.35211.0](../SetupBundle/VisualCppRuntime.wxs) remote payloads with architecture-specific metadata, and the MSIs. The WebView2 download is a build step with no entry in that chain.
- The installer projects expect Windows SDK SignTool, the configured publisher signing certificate and authorized access to its private key in the machine store, plus DigiCert timestamp service access. Their targets sign application binaries, MSI/cabinets and the bundle engine and final bundle. The public Debug probe does not exercise these targets. Exact WiX and SignTool versions and an authorized signing environment remain unverified.
- The [runtime](../SafeExamBrowser.Runtime/SafeExamBrowser.Runtime.csproj) and [configuration tool](../SebWindowsConfig/SebWindowsConfig.csproj) copy an architecture-specific integrity DLL from `C:\SEB` when available. The hosted run reported the missing module for both projects. An authorized source for that module and the applicable distribution rights remain to be established for release parity.
- Installer creation, signature verification, install and smoke checks, and physical Windows lockdown and integrity qualification remain outstanding. Use a native Windows packaging environment for the release inputs and a physical Windows device for the security checks.
