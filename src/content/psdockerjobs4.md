+++
date = '2025-02-08T07:02:48Z'
draft = false
title = 'Start-DockerJob Part 4'
description = 'Error handling'
tags = ["tech", "powershell", "docker"]
+++

* [Part 1]({{< ref "psdockerjobs" >}}) - sketching out the idea.
* [Part 2]({{< ref "psdockerjobs2" >}}) - fixing up job properties and adding powershell parameters.
* [Part 3]({{< ref "psdockerjobs3" >}}) - Adding context support

## Oops

I realised I hadn't published this post for some weeks. I think because I've not had time to fiddle with this recently. I am hoping to use it at work, which will hopefully let me polish it a little more.

## Error Handling

It turns out the reason I wasn't getting error output in my jobs was that I'd not called `process.BeginErrorReadLine();` - insert Homer Simpson D'oh GIF here. It took me a bit of poking to work that out.

So with my `HandleJobErrorReceived` delegate now being triggered and the data being sent in CLI XML it's just a case of running `PSSerializer.Deserialize()` followed by `Error.Add()` right? Unfortunately not because an `ErrorRecord` is deserialized to a `PSObject`. I dug through the powershell engine code, assisted by a helpful comment and [link to some code](https://github.com/Jaykul/Information/blob/master/Source/Classes/DateTimeOffsetConverter.ps1) from Joel Bennet. I'd hacked together something that worked by building an ErrorRecord from the properties of the deserialized ErrorRecord, but wondered if there was something more elegant. Time to go spelunking in the powershell code. It turns out that the powershell engine does the [same thing](https://github.com/PowerShell/PowerShell/blob/172d0b4d8a7d3c69147b1e6ac149fdaf406d9be3/src/System.Management.Automation/engine/ErrorPackage.cs#L1285) but unfortunately all of the relevant methods are marked `internal` or `private` so I can't use them.

## Refactor

Happy that I'd gotten errors rippling through to the Job object, I turned to a little refactor. I created a simple `IDockerCommand` interface with a single method signature:

```csharp
using System.Diagnostics;

namespace PwshDockerJobs;

public interface IDockerCommand
{
    public void Command (List<string> arguments, DataReceivedEventHandler? outputHandler = null, DataReceivedEventHandler? errorHandler = null);
}
```

Next up a concrete implementation of the interface. I moved the mostly duplicated code from the `StartJob`, `StopJob`, etc methods into the `Command` method, adding a few ifs to handle hooking up the output and error handler callbacks if they're passed in. All that was left was to add a nullable `IDockerCommand` parameter to the constructor of the `DockerJob` class and to initialize the concrete implementation when the parameter is null. This will allow the passing of a mock object for testing that doesn't really run any docker commands. 

## Location property

When starting to add basic unit tests I realised that I'd left the `Location` property of the `DockerJob` object just returning "localhost". I figured it made sense to map this onto the docker context, if one was being passed in, or to default to "localhost".

```csharp
public override string Location 
    {
        get {
            return _context ?? "localhost";
        }
    }
```

## Unit tests

I started to add unit tests for the `DockerJob` class and this started to expose a couple of places where I needed tweaks. I added a couple of test helper methods to the class in order to expose getting and setting the job state, so I could arrange the tests for different scenarios. 

I also switched `IDockerCommand` and it's concrete implementation to using two signatures for the `Command` method, with parameters for the output and event handler callbacks, since these are only needed on the `StartJob()` methods. There was some expression tree oddities trying to use default nullable parameters within the mocking frameworks.

## WSL2 Contexts

I'd done some basic tests across local and remote machines running MacOS, Linux and Windows. I also ran it on my development VM at work which is setup for docker on the base Windows OS and on WSL2, with contexts defined on both to be able to run containers on the other. In this scenario I discovered an oddity - no output was returned to the Windows side when running on the Linux context. Very strange. 

After a bit of investigation it looks like I'm [hitting a bug](https://github.com/docker/cli/issues/3586) in the docker CLI. There's [a pull request](https://github.com/docker/cli/pull/4548) which addresses this but it appears to be sat in limbo. I'll keep an eye on it being fixed. 

