+++
date = '2025-01-12T12:57:08Z'
draft = true
title = 'Start-DockerJob Part 3'
description = 'Contexts, Job operations'
tags = ["tech", "powershell", "docker"]
+++

## ... Continuing from

* [Part 1]({{< ref "psdockerjobs" >}}) - sketching out the idea.
* [Part 2]({{< ref "psdockerjobs2" >}}) - fixing up job properties and adding powershell parameters.

## Docker contexts

One of the ideas bubbling around in my head with this hacky little project is being able to push jobs to remote machines running docker rather than just the local machine. For a while docker has had the concept of contexts - basically an in-command registry of entries to other hosts on which you can run commands. I use this on Windows machines to be able to run both Windows and Linux containers (via WSL2) easily on the same machine. 

Adding a `DockerContext` parameter to the `StartDockerJobCommand` class was pretty simple and it was just a case of adding this to the arguments for the docker run command as well as passing it into the `DockerJobSourceAdapter` and the `DockerJob` classes so that the other Job commands could use the correct context (see section below).

I tested by creating a docker context on my laptop pointing at a [Raspberry Pi](https://raspberrypi.org/) (over SSH). I also installed docker and OpenSSH on a Windows machine and created a context pointing at it. I'm happy to say both of these things just worked!

## Job operations

During another hour or so I had to spend I ran through the list of job operation methods `StopJob`, `SuspendJob`, `ResumeJob` and made these issue the appropriate docker commands. I haven't really done much testing of these at the moment.

## Next steps

* Error handling - I haven't quite figured this out yet. At the moment I've cloned the stdout stream handling but that doesn't work.
* Additional options handling - I'd quite like to be able to pass credential specs for use with GMSAs.
* Testing - I need to implement unit and integration tests. And that leads to;
* Refactor - I suspect I ought to refactor the code to have something like an `IDockerCommand` interface to make testing easier and the code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).




