---
title: "Referencing ASP.NET Core in Class Library"
seoTitle: "Referencing ASP.NET Core in Class Library"
seoDescription: "How to use ASP.NET Core classes in a Class Library project using the FrameworkReference element."
datePublished: Sat Sep 21 2024 12:09:11 GMT+0000 (Coordinated Universal Time)
cuid: cm1c3wpf7000g09mgcsgrh9e1
slug: referencing-aspnet-core-in-class-library
tags: aspnet-core, dotnet

---

Yesterday, I was working on RUS, a small API project in C# that will be the base for future publications in this website, and wanted to reference some classes that are part of the ASP.NET Core SDK in a Class Library project (`ControllerBase`, `IActionResult` and more).

## Overview on project requirements

Besides the actual project implementation, I built some Base projects that I plan to reuse in the many domain-specific projects, like the ones for Advertiser in the screenshot below. They are all a Class Library project, but the `Rus.Base.Api` has some additional requirements:

* It can NOT be an API project because it’s just some base code library and not an actual web application.
    
    * In other words, the project SDK must be `Microsoft.NET.Sdk` and not `Microsoft.NET.Sdk.Web` (by using this one, it implicitly adds all the required references in the project).
        
* But it HAS to make use of some packages/classes that are present in that ASP.NET Core SDK.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726918879461/380080df-fade-4c00-b089-42fc45228bef.png align="center")

## .NET 9 Preview solution

This project was born in .NET 9, and I was just using the references to the packages that I needed when the version was the *Preview 7*. That worked fine.

```xml
<!-- Rus.Base.Api.csproj -->
<ItemGroup>
  <Reference Include="Microsoft.AspNetCore.Mvc.Abstractions">
    <HintPath>..\..\..\..\..\Program Files\dotnet\shared\Microsoft.AspNetCore.App\9.0.0-preview.7.24406.2\Microsoft.AspNetCore.Mvc.Abstractions.dll</HintPath>
  </Reference>
  <Reference Include="Microsoft.AspNetCore.Mvc.Core">
    <HintPath>..\..\..\..\..\Program Files\dotnet\shared\Microsoft.AspNetCore.App\9.0.0-preview.7.24406.2\Microsoft.AspNetCore.Mvc.Core.dll</HintPath>
  </Reference>
</ItemGroup>
```

## Upgraded to .NET 9 RC 1

However, when upgrading to *.NET 9 RC 1*, I noticed these packages were not available anymore. What I tried was adding a reference to the `Microsoft.AspNetCore.App` package, but that one is already obsolete, so I thought, let’s try the other one targeting .NET 9: `Microsoft.AspNetCore.App.Ref`. The thing is, that also did not work, and it was throwing an error when trying to install the package:

```plaintext
Package 'Microsoft.AspNetCore.App.Ref 9.0.0-rc.1.24452.1' has a package type 'DotnetPlatform' that is not supported by project 'C:\Repos\rus\backend\src\Rus.Base.Api\Rus.Base.Api.csproj'.
NuGet.Packaging.Core.PackagingException: Package 'Microsoft.AspNetCore.App.Ref 9.0.0-rc.1.24452.1' has a package type 'DotnetPlatform' that is not supported by project 'C:\Repos\rus\backend\src\Rus.Base.Api\Rus.Base.Api.csproj'.
```

## The actual solution

At that time, I was almost thinking it was a bad idea to continue doing that in the project, but knew there is no way I could not do that in a viable way. That’s when I found this page under ASP.NET Core documentation: [Use ASP.NET Core APIs in a class library](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/target-aspnetcore?view=aspnetcore-9.0&tabs=visual-studio).

There is a simple, effortless resource that you can use in the *.csproj* file that references an entire framework: the **&lt;FrameworkReference&gt;.** Look at the example:

```xml
<!-- Rus.Base.Api.csproj -->
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFramework>net9.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>

    <ItemGroup>
        <FrameworkReference Include="Microsoft.AspNetCore.App" />
    </ItemGroup>
...
```

That way, I could have the Class Library project referencing the ASP.NET Core SDK, and use the classes that I wanted on my project.

## When is this useful

That is not something that I would use every day, but a time or two when setting up some reusable projects that need to reference SDKs’ classes. Two examples are:

1. The one in this post, by building a base library to create multiple API projects in the solution.
    
2. If you want to create a NuGet package that does something for ASP.NET Core applications, and need to reference the SDK packages in there.