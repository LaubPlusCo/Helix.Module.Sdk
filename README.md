# Helix.Module.Sdk  
MSBuild extensions to support SDK-style project formats and multi-targeting in Sitecore Helix solutions.  

The Helix Module build SDK aims to simplify:

* Keeping all modules for different Sitecore role types within the same Visual Studio solution
* Publishing to different role types from a single solution using only msbuild
* Creating environment specific deployments for multiple environments that has different Sitecore topologies using only msbuild
* Remove platform module's dependency on Visual Studio's WebApplication.WebPublish target

The latter enable developers to work solely in Visual Studio code even on old aspnet mvc or webforms solutions.

> **Disclaimer**  
> This concept started as a proof-of-concept and is still very much work in progress.   
> The build SDK is already being used on "live" solutions however using it is your own responsibility. It is still in a pre-release state which means you should expect features to be missing, some manual setup, potential bugs and perhaps breaking changes in upcoming versions.  


> **Please comment and contribute!**

## Usage  
_The SDK is not yet made available on Nuget.org so it has to be referenced with full version number using a separate nuget feed for now._

- Download the latest nupkg from [Releases](https://github.com/LaubPlusCo/Helix.Module.Sdk/releases)
- Copy the nupkg to a nuget feed accessible by your solution.

> It is recommended to use a local folder as a nuget feed and keep the nupkg under source control.
> F.ex. .\_build\nugets and add the folder as source in the solution nuget.config (see [example in .\Solution-Example folder](https://github.com/LaubPlusCo/Helix.Module.Sdk/blob/master/_Solution-Example/nuget.config))
>
> ```xml
>  <packageSources>
>    ...
>    <add key="Local" value="_build\nugets" />
>  </packageSources>
> ```
>  

- _Update your .csproj files to SDK style format if not done already_
- Add the following to your .csproj files ([example](https://github.com/LaubPlusCo/Helix.Module.Sdk/blob/master/_Solution-Example/src/Feature/Example/platform/Example.csproj)).

```xml
<Project Sdk="....">
    <!-- Update with the correct SDK version number -->
    <Sdk Name="Helix.Module.Build.Sdk" Version="0.2.16" />
    ...
    <PropertyGroup>
    ...
     <!-- Set role type of project, ex. platform-->
        <SitecoreRoleType>platform</SitecoreRoleType>
    </PropertyGroup>
```  

Add a file named Solution.props in the solution root folder for Solution wide properties such as publish paths.  

For more details please see the [Solution Example](https://github.com/LaubPlusCo/Helix.Module.Sdk/tree/master/_Solution-Example)

### Publish Paths

... insert documentation here ...

### Supported Sitecore Role Types

... insert documentation here ...

- platform
- ...
- rendering

## Building | Contributing  

Ensure that you have nuget version 15.x or later added to your PATH env variable. Download [nuget.exe from here](https://www.nuget.org/downloads)

To create a nuget with the build SDK: 

0. (optional) Increment the version number in Helix.Module.Build.Sdk.nuspec
1. Run `nuget pack` from terminal.  

## Defining new _Role Types_

You can also make your own SitecoreRoleType specific for your solution:

Example:

In a .csproj file set the SitecoreRoleType property to the name of your new role type

```xml
<SitecoreRoleType>myrole</SitecoreRoleType>
```  

Add a file $(AdditionalSitecoreRolePropertiesDir)\myrole.props

Example of how myrole.props could look like for a dotnetcore 3.1 web app

```xml
<Project>
    <PropertyGroup>
        <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
        <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
        <AddRazorSupportForMvc  Condition="'$(AddRazorSupportForMvc)' == ''">true</AddRazorSupportForMvc>
        <CopyXmlTransformFiles  Condition="'$(CopyXmlTransformFiles)' == ''">false</CopyXmlTransformFiles>
        <RunXmlTransforms  Condition="'$(RunXmlTransforms)' == ''">false</RunXmlTransforms>
        <UseFileSystemPublish  Condition="'$(UseFileSystemPublish)' == ''">false</UseFileSystemPublish>
        <WebPublishMethod Condition="'$(WebPublishMethod)' == ''">FileSystem</WebPublishMethod>
        <PublishProvider  Condition="'$(PublishProvider)' == ''">FileSystem</PublishProvider>
        <LastUsedBuildConfiguration Condition="'$(LastUsedBuildConfiguration)' == ''">Release</LastUsedBuildConfiguration>
        <LastUsedPlatform Condition="'$(LastUsedPlatform)' == ''">Any CPU</LastUsedPlatform>
        <ExcludeApp_Data  Condition="'$(ExcludeApp_Data)' == ''">False</ExcludeApp_Data>
        <PublishFramework  Condition="'$(PublishFramework)' == ''">netcoreapp3.1</PublishFramework>
        <DeleteExistingFiles Condition="'$(DeleteExistingFiles)' == ''">False</DeleteExistingFiles>

        <!-- Toggles *.deps/runtimeconfig.json files -->
        <PreserveCompilationContext Condition="'$(PreserveCompilationContext)' == ''">true</PreserveCompilationContext>
        <EnableDefaultContentItems Condition="'$(EnableDefaultContentItems)' == ''">true</EnableDefaultContentItems>

        <!-- Takes app offline before publish by creating App_Offline.htm file in publish target dir -->
        <SetIISAppOfflineBeforePublish Condition="'$(DeployOnBuild)' == ''">true</SetIISAppOfflineBeforePublish>

        <!-- Bring app back online after publish by deleting App_Offline.htm file in publish target dir -->
        <SetIISAppOnlineAfterPublish Condition="'$(DeployOnBuild)' == ''">true</SetIISAppOnlineAfterPublish>
    </PropertyGroup>
</Project>  
```  

In Solution.props add a publish path for the role type:

```xml
 <ItemGroup>
    ...
    <_PublishPaths Include="myrole">
        <Path>$(PublishRootPath)myrole\</Path>
    </_PublishPaths>
    ...
```

_Please contribute missing role types that you create back to the SDK by forking this repo and create a PR_

### MSBuild Properties load order  

This SDK injects properties before Microsoft Common Targets are loaded to ensure the Sitecore Role type has been set before it is used to dynamically import other properties.  

#### High level load order

- Directory.Build.Props 
- SDK Properties and MS Common props
- Project file properties
- Solution.props
- Solution.props.user
- Additional role type specific solution properties
    (ex. from files called Platform.props, Identity.props, XConnect.props)
- Additional role type specific user properties
    (ex. from files called Platform.props.user, Identity.props.user, XConnect.props.user)
- Additional role type specific properties from this SDK
- Directory.Build.targets
- SDK targets and MS Common targets 
