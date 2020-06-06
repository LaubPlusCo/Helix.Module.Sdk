# Helix.Module.Sdk  

MSBuild extensions to support SDK-style project formats and multi-targeting in Sitecore Helix solutions.

## Usage  

Add the following line in your .csproj files.

```xml
<Project Sdk="....">
    <Sdk Name="Helix.Module.Build.Sdk" />
    ...
    <PropertyGroup>
    ...
        <SitecoreRoleType>platform</SitecoreRoleType>
    </PropertyGroup>
```  

Add a file named Solution.props in the solution root folder.

Important; this file control the publish output directories for the individual Sitecore role types if not passed as command-line property to msbuild.



### Load order  

This SDK injects properties before Microsoft Common Targets are loaded to ensure the Sitecore Role type set in the project has been set before used to dynamically load properties.  

#### High level overview

Directory.Build.Props 
SDK Properties and MS Common props

Project Propertires
Solution Properties
Role type SDK Properties
Role type solution properties
Directory.Build.targets
SDK targets and MS Common targets

See ... for a detailed msbuild load order overview. Above order only intent to support customization of this SDK.   