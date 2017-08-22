---
title: "2017-08-22 Monitor Hostname IP change"
date: 2017-08-22
tags: [PowerShell]
---

Here's a really simple but useful script to monitor hostnames for IP addresse change.
## Usage
### Settings
First you have to adjust the settings variables to your needs
```powershell
$hosts_csv_path = "C:\Temp\hosts.csv"
$LogPath = "c:\Logs\"
$email_props = @{
    SMTP    = "192.168.1.1"
    From    = "$env:computername@domain.com"
    To      = "email.to@domain.com"   
}
```
### Init
Then run the script – it will end with an error, but will load Resolve-Host function and set variables
Then run 
```powershell
PS > Resolve-Host -HostName HostNameToMonitor | 
      Export-Csv -Path $hosts_csv_path -Encoding UTF8
```
Or to add another hostname to monitor
```powershell
PS > Resolve-Host -HostName AnotherHostname | 
      Export-Csv -Path $hosts_csv_path -Encoding UTF8 -Append
```
### Run
Now with the initial start you can schedule the script and wait for an email when an IP of a hostname changes.

## Script
The main function Resolve-Host returns all IP addresses of a hostname
```powershell
function Resolve-Host {
    param($HostName)
    BEGIN {
        Write-verbose "In function Resolve-Host"
    }
    PROCESS {
        Write-Verbose "$hostName"
        $host_ips = [System.Net.Dns]::GetHostAddresses("$hostName")
        if ($?) {
            foreach ($host_ip in $host_ips) {
                Write-Verbose "    $($host_ip.ipaddresstostring)"
                [pscustomobject][ordered]@{
                    HostName = $hostName
                    IP       = $host_ip.ipaddresstostring
                    Date     = Get-Date -Format ("yyyy-MM-dd")
                }			
            }
        }
    }
}
```
Import from a csv all previous hostnames and IP addresses.
```powershell
$hosts_csv = Import-Csv $hosts_csv_path -ErrorVariable +my_error
```
Group them by hostname and call Resolve-Host foreach hostname.
Foreach result from Resolve-Host it checks if the IP is already on the list.
If not then it is added.
```powershell
$host_list = $hosts_csv | Group-Object -Property hostname -ErrorVariable +my_error

$results = foreach ($host_single in $host_list) {
    $hostname = $host_single.Name
    $query = Resolve-Host -HostName $hostname
    if ($query) {
        foreach ($row in $query) {
            if (!($host_single.group.ip -contains $row.ip)) {
                $row
                Write-Verbose "    Add $($row.ip)"
            } else {
                Write-Verbose "    Skip $($row.ip)"
            }
        }
    }
}
```
In the end if something new was found an email is sent and results added to CSV. 
```powershell
if ($results) {
    Write-Verbose "Send mail with results"
    Send-MailMessage @email_props  -Subject "Hostname IP change" -Body $(
      ($results | ForEach-Object {"$($_.ip) $($_.hostname)"}) -join "`n")
    Write-Verbose "Export results"
    $results | Export-Csv -Append $hosts_csv_path -NoTypeInformation -Encoding utf8 -ErrorVariable +my_error
}
```
## Source code
Source code on [GitHubGist](https://gist.github.com/amnich/829c50c8ec10f09b64f80c9a1ca5b9fe)
