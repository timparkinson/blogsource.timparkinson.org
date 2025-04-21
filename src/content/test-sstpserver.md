+++
date = '2025-04-21T12:46:33+01:00'
draft = false
title = 'Test-Sstpserver'
description = 'Is this a functioning VPN server?'
tags = ["powershell", "sstp", "vpn"]
+++

I've been doing some VPN related stuff at work recently, and that has given me a need for a way to quickly check [SSTP](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-sstp/a4ea5dd9-21e8-41ae-adb9-15bd0a9b991c) is responding on a server. To that end I knocked up a quick powershell function.

```powershell
function Test-SstpServer {
<#
    .SYNOPSIS

        Tests whether a server is an SSTP server or not.
    
    .DESCRIPTION
        Tests whether a server is an SSTP server or not.
    
    .PARAMETER Uri
        The Uri of the server - the base i.e. https://vpn.domain.com may be used and the SSTP endpoint will be automatically added.

    .PARAMETER SkipCertificateCheck
        Ignore TLS Certificate errors.
    
    .PARAMETER SstpEndpoint
        The the SSTP specific URI endpoint.

    .OUTPUTS
        System.Boolean
    
    .NOTES
        See https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-sstp/a4ea5dd9-21e8-41ae-adb9-15bd0a9b991c
#>
    [CmdletBinding()]

    param(
        [Parameter(
            Mandatory=$true
        )]
            [string]$Uri,
        [Parameter()]
            [switch]$SkipCertificateCheck,
        [Parameter(
            DontShow
        )]
            [string]$SstpEndpoint = '/sra_{BA195980-CD49-458b-9E23-C84EE0ADCD75}/'
    )

    process {
        if ($Uri -notmatch $SstpEndpoint) {
            $Uri = $Uri + $SstpEndpoint
        }
        
        $method = New-Object -TypeName System.Net.Http.HttpMethod -ArgumentList 'SSTP_DUPLEX_POST'
        $request_message = New-Object -TypeName System.Net.Http.HttpRequestMessage -ArgumentList $method, $Uri
        $request_message.Version = 1.1

        $handler = New-Object -TypeName System.Net.Http.HttpClientHandler
        
        if ($SkipCertificateCheck){
            $handler.ServerCertificateCustomValidationCallback = [System.Net.Http.HttpClientHandler]::DangerousAcceptAnyServerCertificateValidator
        }

        $client = New-Object -TypeName System.Net.Http.HttpClient -ArgumentList $handler

        Write-Verbose -Message "Testing $Uri"
        try {
            $response = $client.Send($request_message, [System.Net.Http.HttpCompletionOption]::ResponseHeadersRead)
        } catch {
            Write-Error -ErrorAction Stop -Message "TLS Validation error for $Uri"
        }

        $response.StatusCode -eq [System.Net.HttpStatusCode]::OK
    }

}
```

Using `[System.Net.Http.HttpCompletionOption]::ResponseHeadersRead` stops the `HttpClient` waiting for an age, as it's just the OK header that we're after. 

I also added a `-SkipCertificateCheck` to ease use where the correct certificate chain may not be installed.