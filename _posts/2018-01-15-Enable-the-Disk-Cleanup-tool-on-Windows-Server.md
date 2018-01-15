---
title: "2018-01-15 Enable the Disk Cleanup tool on Windows Server."
date: 2018-01-15
tags: [PowerShell]
---

On <b>Windows Server 2008, 2008 R2 and 2012</b> you can enabled the Disk Cleanup tool without installing the Desktop Experience feature.

All necessary files are in the WinSxS folder. You just have to copy them over into proper system folders.

| OS        | Bit version           | Path  |
| ------------- |:-------------:| -----|
| Windows Server 2008 | 32 | C:\Windows\winsxs\x86_microsoft-windows-cleanmgr_31bf3856ad364e35_6.0.6001.18000_none_6d4436615d8bd133\cleanmgr.exe<br>C:\Windows\winsxs\x86_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.0.6001.18000_en-us_5dd66fed98a6c5bc\cleanmgr.exe.mui|
| Windows Server 2008 | 64 | C:\Windows\winsxs\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.0.6001.18000_none_c962d1e515e94269\cleanmgr.exe<br>C:\Windows\winsxs\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.0.6001.18000_en-us_b9f50b71510436f2\cleanmgr.exe.mui|
| Windows Server 2008 R2 | 64 |	C:\Windows\winsxs\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.1.7600.16385_none_c9392808773cd7da\cleanmgr.exe<br>C:\Windows\winsxs\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.1.7600.16385_en-us_b9cb6194b257cc63\cleanmgr.exe.mui|
| Windows Server 2012 | 64 | C:\Windows\WinSxS\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.2.9200.16384_none_c60dddc5e750072a\cleanmgr.exe<br>C:\Windows\WinSxS\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.2.9200.16384_en-us_b6a01752226afbb3\cleanmgr.exe.mui|

cleanmgr.exe goes to $env:SystemRoot\System32 and cleanmgr.exe.mui to $env:SystemRoot\System32\en-US.

I wrote a function <b>Enable-DiskCleanup</b> to check the OS version and copy the files to system directories.

First check the OS version and type and bit version.

```powershell
	$wmiOS = Get-WmiObject -Class Win32_OperatingSystem
	Write-Verbose $wmiOS.caption	
	Switch ($wmiOS){
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 1 -and $_ }) { 
	        $Version = "Windows Server 2008 R2"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 1}) { 
	        $Version = "Windows 7"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 0}) { 
	        $Version = "Windows Server 2008"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 0}) { 
	        $Version = "Windows Vista"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 2}) { 
	        $Version = "Windows Server 2012"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 2}) { 
	        $Version = "Windows 8"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 3}) { 
	        $Version = "Windows 8.1"
	        break
	    }
	    ({([version]$_.version).Major -eq 6 -and ([version]$_.version).Minor -eq 3}) { 
	        $Version = "Windows Server 2012 R2"
	        break
	    }
	    ({([version]$_.version).Major -eq 10 -and ([version]$_.version).Minor -eq 0}) { 
	        $Version = "Windows Server 2016"
	        break
	    }
	    ({([version]$_.version).Major -eq 10 -and ([version]$_.version).Minor -eq 0}) { 
	        $Version = "Windows 10"
	        break
	    }
	    ({([version]$_.version).Major -eq 5 -and ([version]$_.version).Minor -eq 2}) { 
	        $Version = "Windows Server 2003 R2"
	        break
	    }
	    ({([version]$_.version).Major -eq 5 -and ([version]$_.version).Minor -eq 2}) { 
	        $Version = "Windows XP 64-Bit"
	        break
	    }
	    ({([version]$_.version).Major -eq 5 -and ([version]$_.version).Minor -eq 1}) { 
	        $Version = "Windows XP"
	        break
	    }
	}
  $bitVersion = $wmios.OSArchitecture
	$server = $wmios.Caption -like "*Windows*Server*"
```

If the OS is a server version then set proper paths for cleanmgr.exe and cleanmgr.exe.mui

```powershell
switch ($version){
	        "Windows Server 2008 R2" {
	            $path_exe = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.1.7600.16385_none_c9392808773cd7da\cleanmgr.exe"
	            $path_mui = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.1.7600.16385_en-us_b9cb6194b257cc63\cleanmgr.exe.mui"
	            break
	        }
	        "Windows Server 2008" {
	            if ($bitVersion = '64-bit'){
	                $path_exe = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.0.6001.18000_none_c962d1e515e94269\cleanmgr.exe"
	                $path_mui = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.0.6001.18000_en-us_b9f50b71510436f2\cleanmgr.exe.mui"
	            }
	            else{            
	                $path_exe = "$env:SystemRoot\winsxs\x86_microsoft-windows-cleanmgr_31bf3856ad364e35_6.0.6001.18000_none_6d4436615d8bd133\cleanmgr.exe"
	                $path_mui = "$env:SystemRoot\winsxs\x86_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.0.6001.18000_en-us_5dd66fed98a6c5bc\cleanmgr.exe.mui"
	            }
	            break
	        }    
	        "Windows Server 2012" {
	            $path_exe = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr_31bf3856ad364e35_6.2.9200.16384_none_c60dddc5e750072a\cleanmgr.exe"
	            $path_mui = "$env:SystemRoot\winsxs\amd64_microsoft-windows-cleanmgr.resources_31bf3856ad364e35_6.2.9200.16384_en-us_b6a01752226afbb3\cleanmgr.exe.mui"
	            break
	        }
	        "Windows Server 2012 R2" {
	            Write-Warning "Must install Desktop Experience. Use Powershell command: Install-WindowsFeature Desktop-Experience"
	        }
	    }
 ```
 
Then just copy the files. <b>You have to run the function as administrator to be able to copy to System32 directory.</b>

Full code on [GitHub](https://github.com/amnich/Enable-DiskCleanup/blob/master/Enable-DiskCleanup.ps1).

Once the files are there just run the disk cleanup tool with Cleanmgr.exe

Original info how to enable it found on [this site.](https://support.appliedi.net/kb/a110/how-to-enable-the-disk-cleanup-tool-on-windows-server-2008-r2.aspx)



