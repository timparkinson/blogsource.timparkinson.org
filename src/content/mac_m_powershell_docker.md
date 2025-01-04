+++
date = '2025-01-04T06:01:53Z'
draft = false
title = 'Powershell Docker on ARM Macs'
description = "The image doesn't work"
tags = ["tech", "note", "mac", "powershell", "docker"]
+++

There is [an issue](https://github.com/PowerShell/PowerShell/issues/20162) with the powershell docker image {{< mark >}}mcr.microsoft.com/powershell{{< /mark >}} when run on ARM Macs. It doesn't do the architecture properly and running a container will return an error `exec /usr/bin/pwsh: exec format error`. One solution, for now, is to use the {{< mark >}}mcr.microsoft.com/powershell:preview-mariner-2.0-arm64{{< /mark >}} image. 

```bash
docker run --rm -it mcr.microsoft.com/powershell:preview-mariner-2.0-arm64
```