+++
date = '2025-01-05T08:25:57Z'
draft = false
title = 'Start-DockerJob Part 2'
description = 'Parameters & Properties'
tags = ["tech", "powershell", "docker"]
+++

## Arguments/Parameters

Being able to run arbitrary powershell will only get you so far if you can't provide any parameters. I modified the `StartDockerJobCommand` class to add a {{< mark >}}ArgumentList{{< /mark >}} parameter as per `Start-Job`.

```csharp
[Parameter(Position = 1)]
public object[]? ArgumentList { get; set; }
```

Then within the `ProcessRecord` method I added a little bit of code to serialize this to CLI XML and Base64 encode it before adding it to the arguments I pass to the docker process [^pwshnotdocker].

```csharp
if (ArgumentList != null && ArgumentList.Length != 0)
{
    dockerRunArguments.Add("-EncodedArguments");
    var serializedArguments = PSSerializer.Serialize(ArgumentList);
    dockerRunArguments.Add(Convert.ToBase64String(Encoding.Unicode.GetBytes(serializedArguments)));
}
```

I've also made sure that I'm specifying that the `-inputFormat` is XML, and whilst I'm on, switched to using `-EncodedCommand` rather than `-Command`.

Parameters may be accessed using the `$args` array or by specifying a param block as part of the ScriptBlock. I'm not sure how one would go about using the [using scope modifier](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scopes?view=powershell-7.4#the-using-scope-modifier) for this, so we'll park that for now. 

```powershell
$api = 'https://catfact.ninja/fact'                                        
Start-DockerJob -ScriptBlock {Invoke-Restmethod -Uri $args[0]} -ArgumentList $api  -DockerImage mcr.microsoft.com/powershell:preview-mariner-2.0-arm64

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
6      DockerJob                       Running       False           localhost

Get-Job | Receive-Job
fact                                                                        length
----                                                                        ------
In multi-cat households, cats of the opposite sex usually get along better.     75

Get-Job | Remove-Job

Start-DockerJob -ScriptBlock {param($api) invoke-restmethod -uri $api} -ArgumentList $api  -DockerImage mcr.microsoft.com/powershell:preview-mariner-2.0-arm64

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
7      DockerJob                       Running       False           localhost

Get-Job | Receive-Job
fact
----
There is a species of cat smaller than the average housecat. It is native to Africa and it is the Black-footed cat (Felis nigripes). Its top weight is 5.5 p…
```

## Get-Job Properties 

The `Get-Job` output was not showing the command used, even though I was generating it by concatenating the `dockerRunArguments` list with a space character and passing into the `JobInvocationInfo` specification. This was because I wasn't calling the appropriate base constructor on my `StartDockerJobCommand` constructor. Easily remedied. 

```csharp
public DockerJob(string jobName, List<string> arguments, string command) : base(command, jobName)
```

The `PSJobTypeName` property was even easier and just needed setting in the constructor, referencing a private const variable already defined.

```csharp
PSJobTypeName = DockerJobName;
```

```powershell
Start-DockerJob -ScriptBlock {'test'} -DockerImage mcr.microsoft.com/powershell:preview-mariner-2.0-arm64

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      DockerJob       DockerJob       Running       False           localhost            run --name c4fe0526-9298…
```

Similarly, I'd missed setting the `Name` property of the `JobInvocationInfo` soecification object. This was set in the [sample](https://github.com/microsoft/Windows-classic-samples/blob/main/Samples/PowerShell/JobSourceAdapter/cs/JobSourceAdapterSample.cs), so was an omission on my part in my first stab.


[^pwshnotdocker]: Actually, it's the arguments to the powershell interpreter within the container that are being passed as part of the arguments passed to the docker process but let's not worry about that.