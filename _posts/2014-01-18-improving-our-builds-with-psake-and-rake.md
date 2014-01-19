---
title: Improving our builds with Psake and Rake
layout: post
---

I believe these are good things:

* Having a build server that builds and deploys automatically
* The ability to produce a build in one step

For these reasons, our team has been using [Continua CI][continua] to build our
Windows projects and [TeamCity][teamcity] for our iOS projects. I have more
experience with Continua so that's what I'll discuss here, but the same ideas
apply to TeamCity.

Continua CI lets you design a build process with a web-based tool that has
first-class support for running MSBuild scripts, creating NuGet packages,
running various testing tools, or just executing programs and scripts.

Until now, our build process has been written in this tool. This gave us a
graphical way of editing the build process without mucking around in a batch
file or other scripting language. Continua was reponsible for remembering how
to build and deploy our projects, and we could deploy anything by signing into
Continua and starting a build.

Unfortunately, we've realized this comes with several problems:

* The build server was the only machine that could produce a build. If the
  server went down, we couldn't deploy (our downtime was our infrastructure's
  fault; we've never had a reliability problem with Continua itself).

* The build script is unversioned. As our build process changed and we edited
  the build script in Continua, we found we could no longer build old versions
  automatically. Hotfixing old versions became painful since we had to produce
  the build and deployment manually.

To fix this, we are putting the build logic in a [Psake][psake] build script.
Psake is a build automatation tool written in PowerShell. You define the steps
of your build as "tasks", then choose which task you want to execute.

    ## default.ps1 (the build script)

    Task Build {
        echo "Building the solution"
        & msbuild $solutionFile
    }

    Task Deploy {
        echo "Deploying to the devepment environment"
    }

Then invoke the build script with the `Invoke-psake` function:

    PS C:\> Invoke-psake Build

    ---------------[Build]---------------
    Building the solution
    ... msbuild output ...

    ---------------[Deploy]---------------
    Deploying to the development environment
    ... output ...

    Build Succeeded!

    --------------------------------------
    Build Time Report
    --------------------------------------
    Name            Duration
    ----            --------
    Build           00:00:09.4624557
    Deploy          00:00:12.2191711
    Total:          00:00:21.6816268

You can do more than just execute commands from Psake. The build script is just
PowerShell, so anything you can do from PowerShell can be done during a build.
This is great for us because we make heavy use of Microsoft's development
ecosystem, and all of Microsoft's System Center products and Windows Azure have
PowerShell modules to do almost anything we could want, including creating VMs
and deploying software do them.

Our build script in Continua has been reduced from this:

* Run "msbuild build"
* Run "msbuild package"
* Copy the deployment packages somewhere
* Run MSTest
* Run MSDeploy several times
* Run deployment and integration tests

to this:

* Run "psake deploy"

Continua is still invaluable, but as the tool to automatically kick off our
builds whenever we push changes and notify us when something bad happens. It no
longer knows how to build the projects, only how to call Psake.

For our iOS projects, we're using [Rake][rake], which is equally awesome, but
build scripts are written in Ruby instead of PowerShell. Rakefiles have the
same basic structure as Psake build files:

    task :build do
        # Build with Xcode
        sh "xcodebuild ..."
    end

    task :testflight => [:build] do
        # Upload to TestFlight
    end

and invoked with `rake testflight`.

[continua]: http://www.finalbuilder.com
[teamcity]: http://www.jetbrains.com/teamcity/
[rake]: http://rake.rubyforge.org
[psake]: https://github.com/psake/psake
