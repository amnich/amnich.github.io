---
title: "2017-08-07 DASAN Fibre Channel Switch MAC listing"
date: 2017-08-07
tags: [PowerShell,MAC,DASAN]
---

# How to get a list of MAC addresses with port number of all connected hosts?

## SNMPWALK
Using SNMPWALK query the OID 1.3.6.1.4.1.6296.101.3.13.1.1.3

What we get back is something like this:

    SNMPv2-SMI::enterprises.6296.101.3.13.1.1.3.102.1.6.0.81.129.66.40.31 = Hex-STRING: 00 51 81 42 28 1F 

or if you have the MIB installed

    DASAN-SMI::sleMgmt.3.13.1.1.3.102.1.6.0.81.129.66.40.31 = Hex-STRING: 00 51 81 42 28 1F 

DASAN-SMI::sleMgmt.3.13.1.1.3.**102**.**1**.6.0.81.129.66.40.31 = Hex-STRING: **00 51 81 42 28 1F** 

Where **102** is the VLAN number and the following **1** is the port number on switch. MAC is the Hex string on the end **00 51 81 42 28 1F**.
## REGEX
We need a regex for that to gather the parts we need:

```
3\.13\.1\.1\.3\.(?<VLAN>\d{1,3})\.(?<Port>\d{1,3}).*= Hex-STRING: (?<MAC>.*)
```
## FUNCTION
In the end we wrap it up into a function
```powershell
function Get-MACDasan {
    [CmdletBinding()]
    param(	
        [Parameter(Mandatory = $True)] 
        [ValidateScript({$_ -match [IPAddress]$_ })] 
          [string]$Ip,
        [Parameter(Mandatory = $True)]
        [ValidateNotNull()]
          [string]$Community,
          [string]$Snmpwalk = "C:\usr\bin\snmpwalk.exe" #path to snmpwalk.exe
	
    )
    $Oid = '1.3.6.1.4.1.6296.101.3.13.1.1.3'
    $cmd = "$snmpwalk -v1 -c$community -Ox $Ip $Oid"
    $snmpResults = Invoke-Expression $cmd  #execute snmpwalk command
    Write-Verbose "$($snmpResults | Out-String)"
    # regex for VLAN, PORT and MAC
    $regex = '3\.13\.1\.1\.3\.(?<VLAN>\d{1,3})\.(?<Port>\d{1,3}).*= Hex-STRING: (?<MAC>.*)'
    foreach ($single in $snmpResults) {
        if ($single -match $regex) {
            New-Object PSCustomObject -Property @{
                Source = $ip
                VLAN   = $Matches.VLAN
                MAC    = $Matches.MAC.replace(' ', '')
                Port   = $Matches.Port				
            }			
        }
    }
}
```

[GitHub source code](https://github.com/amnich/Get-MACDasan.ps1)
