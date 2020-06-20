# Helix.Module.Sdk  

MSBuild extensions to support SDK-style project formats and multi-targeting in Sitecore Helix solutions.


## Extending / Contributing

To create a SDK nuget: 

1. Bump up version number in Helix.Module.Build.Sdk.nuspec
2. Run `nuget pack` from terminal. 

Copy the generated nupkg to your solution.

Ex. 

Copy nuget to .\_build\nugets

Add the folder as source in thesolution nuget.config

```xml
  <packageSources>
    ...
    <add key="Local" value="_build\nugets" />
  </packageSources>
```

## Usage  

- Update your .csproj files to SDK style format..
  
- Add the following line in your .csproj files.

```xml
<Project Sdk="....">
    <!-- Update with correct SDK version number -->
    <Sdk Name="Helix.Module.Build.Sdk" Version="0.2.13.5" />
    ...
    <PropertyGroup>
    ...
     <!-- Set role type of project, ex. platform-->
        <SitecoreRoleType>platform</SitecoreRoleType>
    </PropertyGroup>
```  

Add a file named Solution.props in the solution root folder for Solution wide properties such as publish paths.

### Publish Paths

### Support Sitecore Role Types

- platform
- ....
- rendering

More to come... 

You can make your own SitecoreRoleType specific for your solution:

Example:

In a .csproj file set the SitecoreRoleType property to the name of your new role type

```xml
<SitecoreRoleType>myrole</SitecoreRoleType>
```  

Add a file $(AdditionalSitecoreRolePropertiesDir)\myrole.props

Example of how myrole.props could look for a dotnetcore 3.1 web app

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

### MSBuild Properties load order  

This SDK injects properties before Microsoft Common Targets are loaded to ensure the Sitecore Role type has been set before it is used to dynamically import other properties.  

#### Very High level overview

Directory.Build.Props 
SDK Properties and MS Common props
Project file properties
Solution.props
Solution.props.user
Additional role type specific solution properties
    (ex. from files called Platform.props, Identity.props, XConnect.props)
Additional role type specific user properties
    (ex. from files called Platform.props.user, Identity.props.user, XConnect.props.user)
Additional role type specific properties from this SDK

Directory.Build.targets
SDK targets and MS Common targets

See ... for a detailed msbuild load order overview. Above high-level overview is just to support the use of this SDK.  