# CreateZipOfOutputs.MSBuildTask

[![NuGet Package](https://img.shields.io/nuget/v/CreateZipOfOutputs.MSBuildTask.svg)](https://www.nuget.org/packages/CreateZipOfOutputs.MSBuildTask/)

## Summary

This is a NuGet package for Visual Studio MSBuild task.

Just by installing this package, a zip archive file (.zip) that contains the output files will be created in the output folder when building the project (at default configuration, it works only "Release" build).

The zip archive file name contains assembly version text.

## Install

    PM> Install-Package CreateZipOfOutputs.MSBuildTask

## How to determine the zip archive file name?

This MSBuild task determine the zip arichive file name by follow step.

1. Read all text of `Properties\AssemblyInfo.cs` C# source code file if it exists.
2. Search version number text written in `[assembly: AssemblyVersion]` attribute by Regular Expressions pattern.
3. Combine the output name and version number text.

For example, if you install this NuGet package into "FooBar.csproj" C# project, and the project has `Properties\AssemblyInfo.cs` C# source code like bellow:

```csharp
// You can specify all the values or you can default the Build and Revision Numbers
// by using the '*' as shown below:
// [assembly: AssemblyVersion("1.0.*")]
[assembly: AssemblyVersion("2.34.0.*")]
[assembly: AssemblyFileVersion("2.34.0.0")]
```

Then, after build the project, you will get a zip file named `"FooBar - v.2.34.0.zip"` in the output folder.

If you want to customize the zip file name, you can do it.  
Please see "How to configure?" section, especially `ZipOfOutputsPath` property.

## What types of files are excluded in the zip archive file?

At the default settings, this MSBuild task exclude the files which has these file extension.

- *.nupkg
- *.iso
- *.zip
- *.pdb
- *.xml
- *.application
- *.vshost.exe.manifest
- *.vshost.exe
- *.vshost.exe.config

If you want to customize this list, you can do it.  
Please see "How to configure?" section, `ZipOfOutputsExcludeFilesPattern` property.

## How to configure?

Add "CreateZipOfOutputs.props" file to the project like bellow.

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <AssemblyInfoCSPath>$(ProjectDir)Properties\AssemblyInfo.cs</AssemblyInfoCSPath>
    <AssemblyInfoCSContent>$([System.IO.File]::ReadAllText("$(AssemblyInfoCSPath)"))</AssemblyInfoCSContent>
    <AssemblyInfoCSVersionTextRegexPattern>(?m)^[ \t]*\[assembly:[ \t]AssemblyVersion\("([\d\.]+)[\.\*]*"[^)]*\)[^\]]*\][ \t\r\n]*$</AssemblyInfoCSVersionTextRegexPattern>
    <AssemblyInfoCSVersionText>$([System.Text.RegularExpressions.Regex]::Match($(AssemblyInfoCSContent), $(AssemblyInfoCSVersionTextRegexPattern)).Groups[1].Value.TrimEnd('.'))</AssemblyInfoCSVersionText>
    <ZipOfOutputsPath>$(ProjectDir)$(OutputPath)$(TargetName) - v.$(AssemblyInfoCSVersionText).zip</ZipOfOutputsPath>
    <DisableCreateZipOfOutputs Condition="'$(Configuration)' != 'Release'">true</DisableCreateZipOfOutputs>
    <ZipOfOutputsExcludeFilesPattern >**\*.nupkg;**\*.iso;**\*.zip;**\*.pdb;**\*.xml;**\*.application;**\*.vshost.exe.manifest;**\*.vshost.exe;**\*.vshost.exe.config</ZipOfOutputsExcludeFilesPattern>
  </PropertyGroup>
```

"CreateZipOfOutputs" Build task read "CreateZipOfOutputs.props" file in the project folder if it exists, and use those properties high priority.

## license

[Mozilla Public License Version 2.0](lICENSE)