---
title: Converting Turbocharged.Beanstalk to .NET Standard (Part 2)
layout: post
date: 2017-06-17T18:30:00-0700
---

This is part 2 of a series about converting [Turbocharged.Beanstalk][tb] to
.NET Standard so it can run on .NET Framework, .NET Core, and Mono.

* Part 1: [Converting the project files and getting the main project to build][pt1]
* Part 2 (this post): Getting the test project to build
* Part 3: [Fixing the AppVeyor build][pt3]

[tb]: https://github.com/jennings/Turbocharged.Beanstalk
[pt1]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt1 %}
[pt3]: {% post_url 2017-06-17-converting-turbocharged.beanstalk-to-netstandard-pt3 %}

At the end of [part 1][pt1], I had just created [this commit][1d5009c].
Time to try to get the unit tests working.

[1d5009c]: https://github.com/jennings/Turbocharged.Beanstalk/commit/1d5009ca5a2edd32ab4770c038ac46772c754efe


## Getting the test project to build

First I'll just try building and see what happens.

```
src/Turbocharged.Beanstalk❯ cd ../Turbocharged.Beanstalk.Tests

...urbocharged.Beanstalk.Tests❯ dotnet restore; dotnet build

	...a whole lot of output...

...
FakesAndMocks/FakeConsumer.cs(15,41): error CS0246: The type or namespace name 'Job' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
SerializationFacts.cs(21,9): error CS0246: The type or namespace name 'ConnectionConfiguration' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
SerializationFacts.cs(22,9): error CS0246: The type or namespace name 'WorkerOptions' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(477,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(478,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(479,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(480,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(481,81): error CS0246: The type or namespace name 'WorkerFailureBehavior' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(544,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(545,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(546,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(547,21): error CS0103: The name 'WorkerFailureBehavior' does not exist in the current context [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(548,86): error CS0246: The type or namespace name 'WorkerFailureBehavior' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(16,9): error CS0246: The type or namespace name 'IConsumer' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(17,9): error CS0246: The type or namespace name 'IProducer' could not be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
    0 Warning(s)
    38 Error(s)

Time Elapsed 00:00:02.27
```

38 errors? Ugh.

Wait... most of these errors are from not being able to resolve anything from
Turbocharged.Beanstalk. Oops. That's because I didn't add it as a project
reference.

```
...urbocharged.Beanstalk.Tests❯ dotnet add reference ..\Turbocharged.Beanstalk\Turbocharged.Beanstalk.csproj
Reference `..\Turbocharged.Beanstalk\Turbocharged.Beanstalk.csproj` added to the project.
```

And I'll try building again.

```
...urbocharged.Beanstalk.Tests❯ dotnet build
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

    ...output...

/usr/local/share/dotnet/sdk/1.0.3/Microsoft.Common.CurrentVersion.targets(1964,5): warning MSB3277: Found conflicts between different versions of the same dependent assembly that could not be resolved.  These reference conflicts are listed in the build log when log verbosity is set to detailed. [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(6,12): error CS0579: Duplicate 'System.Reflection.AssemblyCompanyAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(7,12): error CS0579: Duplicate 'System.Reflection.AssemblyConfigurationAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(8,12): error CS0579: Duplicate 'System.Reflection.AssemblyDescriptionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(9,12): error CS0579: Duplicate 'System.Reflection.AssemblyFileVersionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(11,12): error CS0579: Duplicate 'System.Reflection.AssemblyProductAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(12,12): error CS0579: Duplicate 'System.Reflection.AssemblyTitleAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
obj/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.AssemblyInfo.cs(13,12): error CS0579: Duplicate 'System.Reflection.AssemblyVersionAttribute' attribute [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
ConnectionFacts.cs(3,14): error CS0234: The type or namespace name 'Configuration' does not exist in the namespace 'System' (are you missing an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
SerializationFacts.cs(3,14): error CS0234: The type or namespace name 'Configuration' does not exist in the namespace 'System' (are you missing an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
    1 Warning(s)
    9 Error(s)

Time Elapsed 00:00:02.35
```

Okay, I've got the same `AssemblyInfo.cs` problem from last time. Delete that
file and I've fixed seven errors.

The only two errors left are because `System.Configuration` isn't available.
Instead, I'll use [`Microsoft.Extensions.Configuration`][config] to read
configuration out of a JSON file.

[config]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration

```
...urbocharged.Beanstalk.Tests❯ dotnet add package Microsoft.Extensions.Configuration
...urbocharged.Beanstalk.Tests❯ dotnet add package Microsoft.Extensions.Configuration.Json
...urbocharged.Beanstalk.Tests❯ dotnet add package Microsoft.Extensions.Configuration.FileExtensions
...urbocharged.Beanstalk.Tests❯ dotnet restore
```

Now I'll replace all references to `ConfigurationManager` with
`IConfigurationRoot`.

And... go!

```
...urbocharged.Beanstalk.Tests❯ dotnet build

	...output...

Build FAILED.

MiscellaneousFacts.cs(16,48): error CS1061: 'Type' does not contain a definition for 'Assembly' and no extension method 'Assembly' accepting a first argument of type 'Type' could be found (are you missing a using directive or an assembly reference?) [/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/Turbocharged.Beanstalk.Tests.csproj]
    0 Warning(s)
    1 Error(s)

Time Elapsed 00:00:02.62
```

What? Oh, I'm using [Phil Haack's trickery][async-void-tests] to avoid
accidentally creating `async void` tests. It uses reflection, and apparently
`Type` no longer has a property named `Assembly` on it. I just need to change
`type.Assembly` to `type.GetTypeInfo().Assembly`, and I'm good.

[async-void-tests]: http://haacked.com/archive/2014/11/11/async-void-methods/

Okay, now to see if it builds:

```
...urbocharged.Beanstalk.Tests❯ dotnet build
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

  Turbocharged.Beanstalk -> /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk/bin/Debug/netstandard1.4/Turbocharged.Beanstalk.dll
  Turbocharged.Beanstalk.Tests -> /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/bin/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:02.91
```

Hooray!


## Verifying the tests pass

These tests require a running instance of Beanstalk to test against. I'll run
it locally using Docker.

```
...urbocharged.Beanstalk.Tests❯ docker create --name beanstalkd -p 11300:11300 schickling/beanstalkd
56d4329fc273977df85cb37a2837247a360b18dca3b1147d687eb7bcf7f8e908

...urbocharged.Beanstalk.Tests❯ docker start beanstalkd
beanstalkd
```

Okay, now I have Beanstalk running on port 11300. I'll run the tests.

```
...urbocharged.Beanstalk.Tests❯ dotnet test
Build started, please wait...
Build completed.

Test run for /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/bin/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.dll(.NETCoreApp,Version=v1.1)
Microsoft (R) Test Execution Command Line Tool Version 15.0.0.0
Copyright (c) Microsoft Corporation.  All rights reserved.

Starting test execution, please wait...
[xUnit.net 00:00:00.5299848]   Discovering: Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:00.6858187]   Discovered:  Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:00.7478363]   Starting:    Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:00.8914508]     Turbocharged.Beanstalk.Tests.SerializationFacts.CanSerializeAndDeserializeAJob [FAIL]
[xUnit.net 00:00:00.8925155]       System.TypeInitializationException : The type initializer for 'Turbocharged.Beanstalk.Tests.Settings' threw an exception.
[xUnit.net 00:00:00.8925976]       ---- System.IO.FileNotFoundException : The configuration file 'settings.json' was not found and is not optional. The physical path is '/Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/bin/Debug/netcoreapp1.1/settings.json'.

	...several dozen copies of this error...
```

Oh, the tests can't find `settings.json` because it isn't getting copied to the
output directory. I need to add the following to
`Turbocharged.Beanstalk.Tests.csproj`:

```xml
<ItemGroup>
    <None Include="settings.json" CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

And now:

```
...urbocharged.Beanstalk.Tests❯ dotnet test
Build started, please wait...
Build completed.

Test run for /Users/stephen/src/github.com/jennings/Turbocharged.Beanstalk/src/Turbocharged.Beanstalk.Tests/bin/Debug/netcoreapp1.1/Turbocharged.Beanstalk.Tests.dll(.NETCoreApp,Version=v1.1)
Microsoft (R) Test Execution Command Line Tool Version 15.0.0.0
Copyright (c) Microsoft Corporation.  All rights reserved.

Starting test execution, please wait...
[xUnit.net 00:00:00.5141871]   Discovering: Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:00.6738833]   Discovered:  Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:00.7304119]   Starting:    Turbocharged.Beanstalk.Tests
[xUnit.net 00:00:06.1207816]     Turbocharged.Beanstalk.Tests.ConnectionFacts.ConnectWorker_FollowsFailureOptionsIfTheDeserializerThrows [SKIP]
[xUnit.net 00:00:06.1209331]       This test occasionally fails, especially in AppVeyor. I think it's due to timing of the worker getting shut down.
Skipped  Turbocharged.Beanstalk.Tests.ConnectionFacts.ConnectWorker_FollowsFailureOptionsIfTheDeserializerThrows
[xUnit.net 00:00:06.3554792]   Finished:    Turbocharged.Beanstalk.Tests

Total tests: 45. Passed: 44. Failed: 0. Skipped: 1.
Test Run Successful.
Test execution time: 6.9826 Seconds
```

Yay, it worked! I'd better [commit][3e6d8b6] before something breaks it.

[3e6d8b6]: https://github.com/jennings/Turbocharged.Beanstalk/commit/3e6d8b6a1e355aa46cc1b91734970dd3e637c1c7

```
...urbocharged.Beanstalk.Tests❯ git add -A

...urbocharged.Beanstalk.Tests❯ git commit -m 'make unit tests buildable and passing on .NET Core'
[netstandard 3e6d8b6] make unit tests buildable and passing on .NET Core
 7 files changed, 54 insertions(+), 44 deletions(-)
```
