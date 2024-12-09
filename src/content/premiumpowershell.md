+++
date = '2024-12-07T08:31:05Z'
draft = true
title = 'Premium Powershell'
description = 'Command line Premium Bond prize checker'
tags = ["powershell", "premium bonds"]
+++

Like many others in the UK I hold some [Premium Bonds](https://www.nsandi.com/products/premium-bonds), a sort of government savings account, where instead of interest, the bonds are put in a prize draw each month with tax-free prizes ranging from £25 to £1 million. There are arguments about [whether it's worth it](https://www.moneysavingexpert.com/news/2024/11/premium-bond-prize-rate-cut-january/) as the guaranteed interest from savings account will outperform this unless you have incredible luck. I see it as a way to have a bit of a flutter, just like playing the lottery, only you get entered every month and can take the money back out if you need it. I only have quite a small holding.

You do get emailed when you win, a few days after the draw date, or you can use an [online checker](https://www.nsandi.com/prize-checker) or an app to check on the draw date. I'm a command line kinda guy though, so I wrote a powershell function to query the API that drives the prize checker instead.

```powershell
function Get-PremiumBondWinStatus {
    <#
        .SYNOPSIS
            Gets this months premium bond win status.
        .DESCRIPTION
            Gets this months premium bond win status. Includes a breakdown of the bonds which won individual prize amounts, if any.
        .PARAMETER HolderNumber
            The Premium Bond Holder number. NOTE - not the NS&I number. This is either 8 digits followed
            by a letter, or 9 or 10 digits.
        .PARAMETER Endpoint
            The URI of the Premium Bond Checker endpoint. This does not normally need to be changed.
        .EXAMPLE
            Get-PremiumBondWinStatus - HolderNumber 1234567890

            HolderNumber IsWinner Prize Breakdown
            ------------ -------- ----- ---------
            1234567890      False     0 {@{prize=0; bond_number=0; date=}}
        .INPUTS
            System.String
        .OUTPUTS
            System.Management.Automation.PSCustomObject
    #>
    [CmdletBinding()]

    param(
        [Parameter(
            Mandatory=$true,
            ValueFromPipeline=$true,
            ValueFromPipelineByPropertyName=$true
        )]
            [ValidatePattern('^(\d{8}[A-Za-z]{1}|\d{9}|\d{10})$')]
            [String]$HolderNumber,
        [Parameter()]
            [String]$Endpoint = "https://www.nsandi.com/premium-bonds-have-i-won-ajax"
    )

    process {
        $headers = @{
            Accept = '*/*'
            'Accept-Encoding' = 'gzip, deflate, br'
            'Accept-Language' = 'en-US,en;q=0.5'
            'Content-Type' = "application/x-www-form-urlencoded"
            Connection = "keep-alive"   
            Origin = 'https://www.nsandi.com'
            Referer =  'https://www.nsandi.com/prize-checker'
        }

        $body = @{
            field_premium_bond_period='this_month'
            field_premium_bond_number=$HolderNumber
        }

        Write-Verbose "Querying for $HolderNumber"
        $result = Invoke-RestMethod -Method Post -Headers $headers -Body $body -Uri $Endpoint 

        if (-not $result) {
            Write-Error -ErrorAction Stop -Message "Could not query for $HolderNumber"
        }

        if ($result.holder_number -eq 'is invalid') {
            $result.holder_number = "INVALID: $HolderNumber"
        }

        [pscustomobject]@{
            HolderNumber = $result.holder_number
            IsWinner = $result.status -ne 'no_win'
            Prize = if ($result.tagline -match "(?<prize>\d+)") {
                [int]$($matches.Prize)
            } else {
                0
            } 
            Breakdown = $result.history
        }
    }
}
```
