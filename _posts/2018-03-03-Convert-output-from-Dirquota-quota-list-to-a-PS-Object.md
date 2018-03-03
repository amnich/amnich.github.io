---
title: "2018-03-03 Convert output from Dirquota quota list to a PS Object"
date: 2018-03-03
tags: [PowerShell]
---

On Windows Server 2008 R2 the Get-FSRMQuota is not available. This cmdlet gets a File Server Resource Manager (FSRM) quota or all FSRM quotas on the server.

You can run the "Dirquota quota list" command to list all quotas but what you get is a text output. 

```
C:\Dirquota quota list

Quota Path:             D:\Share\Folder
Share Path:             \\SERVER\Share$\Folder
Source Template:        Basic-dir (Matches template)
Quota Status:           Enabled
Limit:                  10,00 GB (Soft)
Used:                   7,62 GB (76%)
Available:              2,38 GB
Peak Usage:             9,24 GB (2017-09-06 08:25)
Thresholds:
   Warning ( 85%):      E-mail
   Warning ( 99%):      E-mail
 
```

I have taken the code from [a script from technet gallery by Ben Wilkinson](https://gallery.technet.microsoft.com/scriptcenter/Find-and-Report-on-cc49120e) as a base and created a function to convert this text output to a PowerShell object.

Function is available to download on [GitHub](https://github.com/amnich/ConvertFrom-DirquotaList/blob/master/ConvertFrom-DirquotaList.ps1)

Now You can pipe the results to it:

```Powershell
PS > Dirquota quota list | ConvertFrom-DirquotaList

QuotaPath     : D:\Share\Folder
SharePath     : \\SERVER\Share$\Folder
LimitGB       : 10
UsedGB        : 7,62
UsedPC        : 76
AvailableGB   : 2,38
AvailablePC   : 24
PeakGB        : 9,24
PeakDate      : 2017-09-06 08:25
Template      : Basic-dir
TemplateMatch : Matches template
LimitType     : Soft
Status        : Enabled
```

I didn't bother with converthing the Thresholds.

It should work with English, German and Polish output from 'Dirquota quota list'

