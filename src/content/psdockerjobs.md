+++
date = '2025-01-04T06:28:43Z'
draft = true
title = 'Start-DockerJob'
description = 'extending the powershell jobs infrastructure to support docker'
tags = ["tech", "powershell", "docker"]
+++

## Introduction

I am somewhat cursed with being an early riser. During the normal working week this isn't an issue, because I'll often have out of hours change windows booked in for the various IT systems that I look after. During extended time off, such as over Christmas, however, it's a different story. It's become a bit of a Christmas tradition that I'll do a little programming project - this keeps me from getting bored and means I can hopefully learn something. 

This year I was thinking about the powershell jobs feature and pondering about having it run code in docker containers [^whyjustdocker].

## JobSourceAdapter

It turns out that the powershell jobs infrastructure is already extensible, via the abstract class [JobSourceAdapter](https://learn.microsoft.com/en-us/dotnet/api/system.management.automation.jobsourceadapter?view=powershellsdk-7.4.0). There's either not much documentation available for this or I'm dumb enough that I couldn't find it [^google]. I did manage to find [some sample code](https://github.com/microsoft/Windows-classic-samples/blob/main/Samples/PowerShell/JobSourceAdapter/cs/JobSourceAdapterSample.cs) for a file copier job, as well as a bit of [PSSwagger code](https://www.powershellgallery.com/packages/Azs.Update.Admin/0.2.0/Content/PSSwaggerUtility%5CPSSwaggerNetUtilities.Core.Code.ps1).

So I began tinkering with some code to implement a derived {{< mark >}}DockerJobSourceAdapter{{< /mark >}} class. Most of the implementation was nicked from the [sample code](https://github.com/microsoft/Windows-classic-samples/blob/main/Samples/PowerShell/JobSourceAdapter/cs/JobSourceAdapterSample.cs) with the exception being the job creation method with the signature - `public override Job2 NewJob(JobInvocationInfo specification)`. 

## Job2

Another class {{< mark >}}DockerJob{{< /mark >}} derived from [Job2](https://github.com/PowerShell/PowerShell/blob/c066cd85aa5c0dec8bb4a7007f86431693bf0542/src/System.Management.Automation/engine/remoting/client/Job2.cs) was required to represent the job and to actually interact with docker. Again, much of this, at least for my initial poking could be copied from the sample as it was mostly getting and setting job states. The `public override void StartJob()` method was the initial focus point as that's where I could start running some code in a container. This turned out to be pretty easy - really just starting a Process with the docker executable and the required arguments, using the `List<string>` form. I had started with the simple `string` form but decided to switch in order to make it easier to implement other parameters. 

There's an event handler hooked up to the {{< mark >}}OutputDataReceived{{< /mark >}} event. This detects whether the output is CLI XML and if it is uses the `PSSerializer.Deserialize(string source)` method to turn this back to a `PSObject` and add it to the job output.

## StartDockerJobCommand

This stuff is only useful if we're able to trigger it from a powershell function, so I wrote a simple class deriving from `PSCmdlet`. I can't quite believe it, but this is actually the first time I've created a powershell function in C#. It turns out that it's not too difficult. This cmdlet wrangles the the arguments required to run the container into shape then:
* Creates a `DockerJobSourceAdapter`
* Creates a parameters `Dictionary`
* Adds the docker arguments to the parameters dictionary
* Creates a `JobInfocationInfo` specification
* Creates a `DockerJob` with the specification
* Calls the `StartJob()` method
* Writes the job object to the pipeline

## Conclusion

I've got the basic plumbing in place now to be able to run arbitrary powershell in a docker container controllable by the job functions with real powershell objects returned. So I can run [^dockerimage]:
```powershell
Start-DockerJob -ScriptBlock {Get-Process} -DockerImage mcr.microsoft.com/powershell:preview-mariner-2.0-arm64

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
3      DockerJob                       Running       True            localhost

$proc = Get-Job | Receive-Job 
$proc.WorkingSet
97173504
```

So that's definitely a promising start, but there's lots more to look at:

* Input variables
* Errors/other streams
* Supporting more docker run parameters
* Properly implementing job state transitions
* Tests

If I can get whip the code into something a little more complete and robust I may well publicly release it.

[^google]: Or perhaps it's the means of [finding stuff](https://pluralistic.net/2023/10/03/not-feeling-lucky/) these days that's the problem.

[^whyjustdocker]: Of course, it might be possible to extend this concept to other back-ends - kubernetes, HPC schedulers, cloud lambda services, etc.

[^dockerimage]: I used a default for the `DockerImage` parameter of the standard Microsoft powershell image, but it seems it [doesn't work on ARM Macs]({{< ref "mac_m_powershell_docker" >}}).