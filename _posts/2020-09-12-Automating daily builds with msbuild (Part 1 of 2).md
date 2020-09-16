---
layout: post
title: Automating daily builds with msbuild and VS 2019 (Part 1/2) - Setting up automatic versioning
date: 2020-09-12 14:10
category: Visual Studio 2019
author: Zeb Rasco
tags: [msbuild]
summary: 
---

For my first blog post, we'll review the strategy I used to automate daily builds for an UPS daemon I'm working on, HIDUPSResponder (will cover this in a future blog post). We will use this approach and start from scratch with an example project called ExampleDaily.

If you aren't interested in setting up daily builds and just want to add automatic versioning to your projects, you can skip to the [section on auto-versioning](#set-up-auto-versioning) and not concern yourself with WiX or Serilog.

## Rationale

Creating an automated build process will make your life easier in the long run. We will use both an msbuild script and the WiX toolset to create a daily build which can be executed via a batch file. Doing this work upfront mitigates the risk of human error in future builds and provides a consistent product.

I have fashioned the various files involved in such a way that they resemble a template. Once you get the hang of it, you shouldn't have too much of a problem migrating the build setup to other VS solutions. 

## Our example

In my case, I decided to create four different builds per deployment:

1. A framework-dependent portable ZIP
1. A self-contained portable ZIP
1. A framework-dependent MSI package
1. A self-contained MSI package 

In addition, such deployment packages can be set up for any configuration (Debug vs Release) and platform you choose (x86 and x64). Here is an example of the files produced for the x86 Release version:

![x86 Release files screenshot](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-12-14-27-42.png)'

And the x64 Debug version. Note the addition of "Debug" in the filenames:

![x64 Debug files screenshot](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-12-14-39-17.png)

## Requirements

To get started, you'll need the following installed:
- [Visual Studio 2019 Community Edition (or greater)](https://visualstudio.microsoft.com/downloads/)
- [WiX toolset v3.11 (or greater)](https://wixtoolset.org/releases/)
- [Wix Toolset Visual Studio 2019 Extension](https://marketplace.visualstudio.com/items?itemName=WixToolset.WixToolsetVisualStudio2019Extension)

## Procedure

For this example, we'll set up a basic Hello World project and accompanying WiX project. We'll assume the reader is familiar with Visual Studio already, so not every single step will have an accompanying screenshot.

First, make sure you install the WiX toolset and Visual Studio Extension. You'll have to restart Visual Studio for the changes to take effect.

Now, create an empty console project and name it ExampleDaily. Make sure the project and solution are NOT in the same directory.

[![](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-15-10-27-37.png)](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-15-10-27-37.png)

#### Set up auto-versioning
You'll now end up with a typical empty project. Our first task is to enable auto-versioning by making our builds non-deterministic and generating the assembly info per build.

1. Edit the ExampleDaily.csproj file and add the following into your `<PropertyGroup>` tag:
```xml
	<GenerateAssemblyInfo>False</GenerateAssemblyInfo>
	<Deterministic>False</Deterministic>
```

2. Back in your Program.cs file, add the following between your using block and your namespace declaration:
```csharp
[assembly: AssemblyVersion("1.0.*")]
```

Setting `<GenerateAssemblyInfo>` to false ensures that we don't have any future conflicts with our versioning, and setting `<Deterministic>` to false allows us to use wildcards when generating the assembly info in the Program.cs file.

#### Add some dependencies

Now, let's add a dependency. We need at least one dependency to demonstrate that the build process will copy dependencies along with the executable. In this example, we'll install the Serilog.Sinks.Console NuGet package, using the package manager console (Tools->NuGet Package Manager->Package Manager Console):

```
Install-Package Serilog.Sinks.Console -ProjectName ExampleDaily
```

#### Auto-versioning in action

Let's now demonstrate that auto-versioning works. We'll invoke an instance of Serilog and output the version number to the console. Add the following Using directives:

```csharp
using Serilog;
using System.Reflection;
```

And replace your Main() function with this:
```csharp
            // Get our assembly version in the form of (Major.Minor.Daily.Revision)
            string version = Assembly.GetEntryAssembly().GetName().Version.ToString();

            // Set up Serilog to log to console
            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .WriteTo.Console()
                .CreateLogger();

            // Log.<LogLevel> will now print to the console
            Log.Information($"This is our example daily build program! Current version: {version}");
```

When all is said and done, you should see something similar to the following:

[![](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-15-11-12-20.png)](/assets/2020-09-12-Automating%20daily%20builds%20with%20msbuild/2020-09-15-11-12-20.png)

Now, run the program and verify everything is working correctly. You should see the correct version number. Here is mine:

```
[11:17:04 INF] This is our example daily build program! Current version: 1.0.7563.20311
```

Then, after a rebuild all command. Note that the revision number has increased after rebuilding:

```
[11:17:37 INF] This is our example daily build program! Current version: 1.0.7563.20327
```

## Conclusion

This is the approach I use to set up auto-versioning in my .NET Core 3.1 applications. In the next post, we'll use this example project as a foundation for automated daily builds. If you have any questions or feedback, please leave a comment. See you soon!