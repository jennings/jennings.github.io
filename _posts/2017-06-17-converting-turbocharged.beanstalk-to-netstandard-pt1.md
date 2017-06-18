---
title: Converting Turbocharged.Beanstalk to .NET Standard (Part 1)
layout: post
date: 2017-06-17T18:00:00-0700
---

.NET Standard is the future of cross-platform .NET. By targeting .NET Standard,
a library will run on any .NET implementation that supports it:

* .NET Framework (Windows only)
* .NET Core (cross-platform)
* Mono (cross-platform)

But I find the ecosystem very confusing right now. The story used to be "just
install .NET Framework version X, and target it in Visual Studio", but now it
feels like the tools and file formats are changing every few months (it isn't,
but it feels that way).

I'm going to finally figure out where the ecosystem is at by converting some of
my projects to .NET Standard.

This series documents the process I followed to convert
[Turbocharged.Beanstalk][tb] to .NET Standard.

* Part 1 (this post): Converting the project files and getting the main project to build
* Part 2: [Getting the test project to build][pt2]
* Part 3: [Fixing the AppVeyor build][pt3]

[tb]: https://github.com/jennings/Turbocharged.Beanstalk
[pt2]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt2 %}
[pt3]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt3 %}


## Installing the tooling

I'm doing this work on a Mac.

I already the tooling installed, but I _think_ the only thing that needs to be
installed is [the .NET Core SDK][netcore-install]. Here's the output of `dotnet
--info` on my system.

[netcore-install]: https://www.microsoft.com/net/download/core

```
❯ dotnet --info
.NET Command Line Tools (1.0.3)

Product Information:
 Version:            1.0.3
 Commit SHA-1 hash:  37224c9917

Runtime Environment:
 OS Name:     Mac OS X
 OS Version:  10.12
 OS Platform: Darwin
 RID:         osx.10.12-x64
 Base Path:   /usr/local/share/dotnet/sdk/1.0.3
```


## Creating the new solution and project files

First, a new branch to work in (I'm starting from [this commit][d5eb5d0]):

[d5eb5d0]: https://github.com/jennings/Turbocharged.Beanstalk/commit/d5eb5d0f5168ff67b77c50952160d5f1ad089a02

```
❯ git clone git@github.com:jennings/Turbocharged.Beanstalk

❯ cd Turbocharged.Benstalk

❯ git checkout -b netstandard
Switched to a new branch 'netstandard'

❯ git rev-parse --short HEAD
d5eb5d0
```

This project currently only works on .NET Framework and Visual Studio. Since
I'm working from my Mac, the first step seems to be to convert the solution and
project files to the newest formats.

What does this project look like?

```
❯ tree -P '*.sln|*.csproj'
.
└── src
    ├── SampleApp
    │   ├── Properties
    │   └── SampleApp.csproj
    ├── Turbocharged.Beanstalk
    │   ├── Properties
    │   ├── System
    │   └── Turbocharged.Beanstalk.csproj
    ├── Turbocharged.Beanstalk.Tests
    │   ├── FakesAndMocks
    │   ├── Properties
    │   └── Turbocharged.Beanstalk.Tests.csproj
    └── Turbocharged.Beanstalk.sln

9 directories, 4 files
```

Okay, so I've got one solution file with three csproj files.

That SampleApp is a WinForms project, so I're probably not going to be able to
convert it. I'll ignore it for now.

```
❯ cd src

src❯ dotnet new sln --name Turbocharged.Beanstalk
Content generation time: 39.1996 ms
The template "Solution File" created successfully.

src❯  git status -s
 M Turbocharged.Beanstalk.sln
```

Git tells me that the solution file has been overwritten with the new format.
Cool. But there are no projects in it yet. I need to convert the project files
and add them to the solution.

The `dotnet` command can create several types of libraries, which I can see
with `dotnet new -all`:

```
❯ dotnet new -all
Template Instantiation Commands for .NET Core CLI.

    ...

Templates                 Short Name       Language      Tags
-----------------------------------------------------------------------
Console Application       console          [C#], F#      Common/Console
Class library             classlib         [C#], F#      Common/Library
Unit Test Project         mstest           [C#], F#      Test/MSTest
xUnit Test Project        xunit            [C#], F#      Test/xUnit
ASP.NET Core Empty        web              [C#]          Web/Empty
ASP.NET Core Web App      mvc              [C#], F#      Web/MVC
ASP.NET Core Web API      webapi           [C#]          Web/WebAPI
Nuget Config              nugetconfig                    Config
Web Config                webconfig                      Config
Solution File             sln                            Solution

Examples:
    dotnet new mvc --auth None --framework netcoreapp1.1
    dotnet new nugetconfig
    dotnet new --help

```

So I need to create a class library and an xUnit project. I'll also delete the
skeleton source files that were generated:

```
src❯  dotnet new classlib --name Turbocharged.Beanstalk
Content generation time: 29.2166 ms
The template "Class library" created successfully.

src❯  dotnet new xunit --name Turbocharged.Beanstalk.Tests
Content generation time: 25.1647 ms
The template "xUnit Test Project" created successfully.

src❯ rm Turbocharged.Beanstalk.Tests/UnitTest1.cs Turbocharged.Beanstalk/Class1.cs

src❯  git status -s
 M Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj
 M Turbocharged.Beanstalk.sln
 M Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj
```

And now add those projects to my solution.

```
src❯ dotnet sln add Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj
Project `Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj` added to the solution.
Project `Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj` added to the solution.
```

Cool. [Time to commit!][61d7c86]

[61d7c86]: https://github.com/jennings/Turbocharged.Beanstalk/commit/61d7c8607811f6d93d74ccff996579a5f0fa591f

```
src❯ git commit -am 'convert csproj files to new project format'
[netstandard 61d7c86] convert csproj files to new project format
 3 files changed, 66 insertions(+), 225 deletions(-)
```



## Making the class library buildable

Could it possibly...just work?

```
src❯ cd Turbocharged.Beanstalk
src/Turbocharged.Beanstalk❯ dotnet restore; dotnet build

   ...a lot of output...

obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(6,12): error CS0579: Duplicate 'System.Reflection.AssemblyCompanyAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(7,12): error CS0579: Duplicate 'System.Reflection.AssemblyConfigurationAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(8,12): error CS0579: Duplicate 'System.Reflection.AssemblyDescriptionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(9,12): error CS0579: Duplicate 'System.Reflection.AssemblyFileVersionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(11,12): error CS0579: Duplicate 'System.Reflection.AssemblyProductAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(12,12): error CS0579: Duplicate 'System.Reflection.AssemblyTitleAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
obj/Debug/netstandard1.4/Turbocharged.Beanstalk.AssemblyInfo.cs(13,12): error CS0579: Duplicate 'System.Reflection.AssemblyVersionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
NewtonsoftJsonJobSerializer.cs(6,7): error CS0246: The type or namespace name 'Newtonsoft' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
Trace.cs(12,25): error CS0246: The type or namespace name 'TraceSource' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
    0 Warning(s)
    9 Error(s)

Time Elapsed 00:00:01.74
```

No, of course not.

Most of these errors are from `Properties/AssemblyInfo.cs`. The new build
system automatically generates properties that used to be specified in that
file, so I don't even need it anymore.

```
src/Turbocharged.Beanstalk❯ rm Properties/AssemblyInfo.cs
```

What's next? Oh, I haven't imported Newtonsoft.Json.

```
src/Turbocharged.Beanstalk❯ dotnet add package Newtonsoft.Json
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

  Writing /var/folders/kr/1g94089s3xv6vw34b9mgrq740000gn/T/tmpM2QiPo.tmp
info : Adding PackageReference for package 'Newtonsoft.Json' into project '/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj'.
log  : Restoring packages for /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj...
info :   GET https://api.nuget.org/v3-flatcontainer/newtonsoft.json/index.json
info :   OK https://api.nuget.org/v3-flatcontainer/newtonsoft.json/index.json 410ms
info :   GET https://api.nuget.org/v3-flatcontainer/newtonsoft.json/10.0.2/newtonsoft.json.10.0.2.nupkg
info :   OK https://api.nuget.org/v3-flatcontainer/newtonsoft.json/10.0.2/newtonsoft.json.10.0.2.nupkg 385ms
info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project '/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj'.
info : PackageReference for package 'Newtonsoft.Json' version '10.0.2' added to file '/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj'.
```

Finally, the last error is complaining about `TraceSource`. I'm sure there's
some replacement that works on .NET Standard, but I don't know what it is. For
now, I'll just comment it all out.

Okay! Time to try again. I need to run `dotnet restore` first because I added a
new package reference.

```
src/Turbocharged.Beanstalk❯ dotnet restore; dotnet build
  Restoring packages for /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj...
  Writing lock file to disk. Path: /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/obj/project.assets.json
  Restore completed in 266.09 ms for /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj.

  NuGet Config files used:
      /Users/stephen/.nuget/NuGet/NuGet.Config

  Feeds used:
      https://api.nuget.org/v3/index.json
      /Users/stephen/.dotnet/NuGetFallbackFolder
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

PhysicalConnection.cs(56,25): error CS1061: 'TcpClient' does not contain a definition for 'Close' and no extension method 'Close' accepting a first argument of type 'TcpClient' could be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]

Build FAILED.

PhysicalConnection.cs(56,25): error CS1061: 'TcpClient' does not contain a definition for 'Close' and no extension method 'Close' accepting a first argument of type 'TcpClient' could be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj]
    0 Warning(s)
    1 Error(s)

Time Elapsed 00:00:01.75
```

Hmm, `TcpClient` is missing the `Close()` method. They removed it in favor of
`Dispose()`, so I just need to replace `_client.Close()` with
`_client.Dispose()`.

Does it build now?

```
src/Turbocharged.Beanstalk❯ dotnet build
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

  Turbocharged.Beanstalk -> /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/bin/Debug/netstandard1.4/Turbocharged.Beanstalk.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:01.78
```

Yes! [Commit before anything else goes wrong][1d5009c].

[1d5009c]: https://github.com/jennings/Turbocharged.Beanstalk/commit/1d5009ca5a2edd32ab4770c038ac46772c754efe

```
src/Turbocharged.Beanstalk❯ git commit -am "make Turbocharged.Beanstalk buildable"
[netstandard 1d5009c] make Turbocharged.Beanstalk buildable
 4 files changed, 15 insertions(+), 49 deletions(-)
 delete mode 100644 src/Turbocharged.Beanstalk/Properties/AssemblyInfo.cs
```

That's enough for now.
