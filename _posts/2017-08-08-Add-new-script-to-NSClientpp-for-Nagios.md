---
title: "2017-08-08 Add new script to NSClient++ for Nagios"
date: 2017-08-08
tags: [PowerShell,Nagios,NSClient++]
---

# Add-NscWrappedScript
Here is a function to add a new script to NSClient++ for Nagios on a local or remote computer. 

It copies your script to Script directory and creates a new entry in nsc.ini or nsclient.ini under [Wrapped Scripts].

```powershell
Function Add-NscWrappedScript {
    param(
        [parameter(Mandatory = $true)]
        [ValidateNotNullorEmpty()]
        $CommandLine,
        [ValidateScript( {Test-Path $_ })]
        $PathToScript,
        [parameter()]
        [ValidateNotNullorEmpty()]
        $ScriptName,        
        [switch]$BackupIniFile,
        [parameter(ValueFromPipeline)]
        [ValidateNotNullorEmpty()]
        [String[]]$ComputerName,
        $NscFolder = "$env:ProgramFiles\NSClient*",
        [switch]$Force
    )
    BEGIN {
        if ($PathToScript) {
            $ScriptContent = Get-Content $PathToScript
            Write-Debug "Script content: `n$($ScriptContent | out-string)"
            if (!$ScriptName) {
                $ScriptName = Split-Path $PathToScript -Leaf
            }
            Write-Debug "Script name $ScriptName"
        }        
        $patternWS = "[\[|[\/settings\/external scripts\/][w|W]rapped [s|S]cripts\]"
        $NSCini = "nsc.ini", "nsclient.ini"
        $NSCiniBackup = "nsc_$(get-date -Format "yyyyMMdd_HHmm")`.ini"
		$VerboseSwitch = $PSCmdlet.MyInvocation.BoundParameters["Verbose"].IsPresent
        $ScriptBlock = {            
            try {
                if ($using:NscFolder) {
                    if ($using:VerboseSwitch){
                      $VerbosePreference = "continue"
                    }
                    Write-Verbose "Running remote on $env:computername"
                    $NscFolder = $using:NscFolder
                    $BackupIniFile = $using:BackupIniFile
                    $ScriptContent = $using:ScriptContent
                    $ScriptName = $using:ScriptName
                    $NSCini = $using:NSCini
                    $NSCiniBackup = $using:NSCiniBackup
                    $patternWS = $using:patternWS
                    $CommandLine = $using:CommandLine
                }
            }
            catch {
                Write-Verbose "Running local"
            }
            #find NSC folder
            $Folders = Get-ChildItem "$NSCFolder"
            Write-Verbose "Folders found $($folders.count)"
            Write-Debug "$($folders | out-string)"
            foreach ($folder in $Folders) {
                try {                    
                    $NscIniPath = "$($folder.FullName)\$($NSCini[0])"
                    if (!(Test-Path $NscIniPath)) {
                        $NscIniPath = "$($folder.FullName)\$($NSCini[1])"
                        if (!(Test-Path $NscIniPath)) {
                            Write-Error "$NscIniPath missing"
                        }
                    }                    
                    #if command is missing add it
                    $CommandLineRegexEscaped = [regex]::Escape($($CommandLine -replace "^;"))
                    $testCommand = Select-String -Path $NscIniPath -pattern ($CommandLineRegexEscaped)
                    if (!($testCommand) -or $force) {
                        if ($PathToScript) {
                            $ScriptContent | out-file  "$($folder.FullName)\scripts\$ScriptName" -Force
                            Write-Verbose "    Script $ScriptName saved in $($folder.FullName)\scripts\"
                        }                       
                        #backup switch present then backup file as NSC_yyyyMMdd_HHmm.ini
                        if ($BackupIniFile) {
                            Copy-Item $NscIniPath $($nscinipath.Replace($(Split-Path $NscIniPath -Leaf), $NSCiniBackup)) -Force 
                            Write-Verbose "    NSC ini file backed up as $($nscinipath.Replace($NSCini,$NSCiniBackup))"
                        }
                        if ((Select-String -Path $NscIniPath -pattern ($CommandLineRegexEscaped))) {
                            Write-Verbose "    Replace command in ini file"
                            (Get-Content $NscIniPath) | Foreach-Object {
                                if ($_ -match $CommandLineRegexEscaped) {
                                    Write-Verbose "    Replace `"$_`" with `"$CommandLine`""
                                    $CommandLine
                                }
                                else {
                                    $_
                                }
                            } | Set-Content $NscIniPath
                        }
                        else {
                            #get content of ini file
                            (Get-Content $NscIniPath) | Foreach-Object {
                                $_ # send the current line to output
                                if ($_ -match $patternWS) {
                                    #Add Lines after the selected pattern 
                                    $CommandLine
                                    Write-Verbose "    New command inserted $CommandLine"
                                }
                            } | Set-Content $NscIniPath
                        }   
                    }
                    else {
                        Write-warning "    Command already present in $NscIniPath.`n$($testCommand.Line | out-string)`nUse -Force switch to overwrite."                        
                    }
                }
                catch {
                    $error[0]
                    return $false
                }
            }
            return $true
        }
    }
    PROCESS {
        if ($ComputerName) {
            Invoke-Command -ScriptBlock $scriptblock -ComputerName $ComputerName
        }
        else {
            & $ScriptBlock
        }          
    }   
}
```


## EXAMPLE 1
Copy new script from C:\temp\test.ps1 to "PC1" and "PC2" and add new command "check_test=test.ps1" under Wrapped Scripts in ini file. 

Backup ini file before changes.

Show verbose output.
```powershell
    PS > $params = @{
	        ComputerName = "PC1", "PC2"
	        PathToScript = "C:\temp\test.ps1"
	        CommandLine = "check_test=test.ps1"
	        BackupIniFile = $true
	        Verbose = $true
        }
    PS > Add-NscWrappedScript @params
    VERBOSE: Running remote on PC1
    VERBOSE: Folders found 1
    VERBOSE:     Script test.ps1 saved in C:\Program Files\NSClient++\scripts\
    VERBOSE:     NSC ini file backed up as C:\Program Files\NSClient++\nsc_20170519_2220.ini
    VERBOSE:     New command inserted check_test=test.ps1
    True
    VERBOSE: Running remote on PC2
    VERBOSE: Folders found 1
    VERBOSE:     Script test.ps1 saved in C:\Program Files\NSClient++\scripts\
    VERBOSE:     NSC ini file backed up as C:\Program Files\NSClient++\nsc_20170519_2220.ini
    VERBOSE:     New command inserted check_test=test.ps1
    True
```
## EXAMPLE 2
Try to add command "check_test_ps1=check_test.ps1 arg1 arg2" and copy file from "C:\temp\test.ps1" and save it in NSClient++ as check_test.ps1.
```powershell
    PS > 	$params = @{	
			PathToScript = "C:\temp\test.ps1"
			CommandLine = "check_test_ps1=check_test.ps1 arg1 arg2"
			ScriptName = "check_test.ps1"
			Verbose = $true
		}
    PS > Add-NscWrappedScript @params
    VERBOSE: Running local
    VERBOSE: Folders found 1
    WARNING:     Command already present.
    ;check_test_ps1=check_test.ps1 arg1 arg2
    Use -Force switch to overwrite.
    False
Command ends with a warning because command is already present in ini file.
```

## EXAMPLE 3
Use of Force switch to overwrite and existing line in ini file. 
```powershell
Replacing ";check_test_ps1=check_test.ps1 arg1 arg2" with "check_test_ps1=check_test.ps1 arg1 arg2" - the command will now be enabled.
      PS > $params = @{	
        PathToScript = "C:\temp\test.ps1"
        CommandLine = "check_test_ps1=check_test.ps1 arg1 arg2"
        ScriptName = "check_test.ps1"
        Verbose = $true
        Force = $true
      }
      PS > Add-NscWrappedScript 
      VERBOSE: Running local
      VERBOSE: Folders found 1
      VERBOSE:     Script check_test.ps1 saved in C:\Program Files\NSClient++-0.3.9-x64-\scripts\
      VERBOSE:     Replace command in ini file
      VERBOSE:     Replace ";check_test_ps1=check_test.ps1 arg1 arg2" with "check_test_ps1=check_test.ps1 arg1 arg2"
      True
```

[GitHub source code](https://github.com/amnich/Add-NscWrappedScript)
