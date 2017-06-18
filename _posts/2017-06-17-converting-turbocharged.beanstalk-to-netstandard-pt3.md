---
title: Converting Turbocharged.Beanstalk to .NET Standard (Part 3)
layout: post
date: 2017-06-17T18:50:00-0700
---

This is part 3 of a series about converting [Turbocharged.Beanstalk][tb] to
.NET Standard so it can run on .NET Framework, .NET Core, and Mono.

* Part 1: [Converting the project files and getting the main project to build][pt1]
* Part 2: [Getting the test project to build][pt2]
* Part 3 (this post): Fixing the AppVeyor build

[tb]: https://github.com/jennings/Turbocharged.Beanstalk
[pt1]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt1 %}
[pt2]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt2 %}

In [part 2][pt2], I had just created [this commit][3e6d8b6].

The tests are passing, but the [AppVeyor][appveyor] build is
[failing][failed-build].

[3e6d8b6]: https://github.com/jennings/Turbocharged.Beanstalk/commit/3e6d8b6a1e355aa46cc1b91734970dd3e637c1c7 
[appveyor]: https://www.appveyor.com/
[failed-build]: https://ci.appveyor.com/project/jennings/turbocharged-beanstalk/build/0.1.53


## Fixing the AppVeyor build

The output from the build ends with this:

```
msbuild "C:\projects\turbocharged-beanstalk\src\Turbocharged.Beanstalk.sln" /verbosity:minimal /logger:"C:\Program Files\AppVeyor\BuildAgent\Appveyor.MSBuildLogger.dll"
Microsoft (R) Build Engine version 14.0.25420.1
Copyright (C) Microsoft Corporation. All rights reserved.
C:\projects\turbocharged-beanstalk\src\Turbocharged.Beanstalk\Turbocharged.Beanstalk.csproj(1,1): error MSB4041: The default XML namespace of the project must be the MSBuild XML namespace. If the project is authored in the MSBuild 2003 format, please add xmlns="http://schemas.microsoft.com/developer/msbuild/2003" to the <Project> element. If the project has been authored in the old 1.0 or 1.2 format, please convert it to MSBuild 2003 format.
C:\projects\turbocharged-beanstalk\src\Turbocharged.Beanstalk.Tests\Turbocharged.Beanstalk.Tests.csproj(1,1): error MSB4041: The default XML namespace of the project must be the MSBuild XML namespace. If the project is authored in the MSBuild 2003 format, please add xmlns="http://schemas.microsoft.com/developer/msbuild/2003" to the <Project> element. If the project has been authored in the old 1.0 or 1.2 format, please convert it to MSBuild 2003 format.
Command exited with code 1
```

This AppVeyor image is using MSBuild 14, which cannot read the new project file
format. I need to use an image supporting the new format.

I could do this in the AppVeyor settings, but then these settings wouldn't
follow any forks of the repository. Instead, I'll create an `appveyor.yml` in
the root of the repository.

```yaml
version: 1.0.{build}
image: Visual Studio 2017
configuration: Release

before_build:
  - cmd: dotnet restore

build:
  - verbosity: minimal

test_script:
  - cmd: dotnet test src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj

after_build:
  - cmd: dotnet pack src/Turbocharged.Beanstalk/Turbocharged.Beanstalk.csproj --configuration Release

artifacts:
  - path: '**/*.nupkg'
```

And now commit and push.

```
Build started
git clone -q --branch=netstandard https://github.com/jennings/Turbocharged.Beanstalk.git C:\projects\turbocharged-beanstalk
git checkout -qf 66f9673efad5dbdd31453382787570fc697e43d8
dotnet restore
MSBUILD : error MSB1003: Specify a project or solution file. The current working directory does not contain a project or solution file.
Command exited with code 1
```

Oh poop. The build script expected the solution file to be in the root of the
repository.

I could add `cd` to all the build scripts, or I could just move the solution
file.

Guess which one is easier.

```
❯ git commit -m 'move the solution file to the root of the repository'
[netstandard c45da68] move the solution file to the root of the repository
 1 file changed, 3 insertions(+), 3 deletions(-)
 rename src/Turbocharged.Beanstalk.sln => Turbocharged.Beanstalk.sln (91%)

❯ git push
```

And now [AppVeyor says][successful-build]:

[successful-build]: https://ci.appveyor.com/project/jennings/turbocharged-beanstalk/build/1.0.57

```
Build success
```

Yay!
