---
title: "2017-09-04 Access your profile functions more efficient"
date: 2017-09-04
tags: [PowerShell]
---

I have in my PowerShell profile a couple of functions that I use from time to time to update something automatically or to open a file quickly.
But I use them not daily so I forget what I got there or what was the name of that function that I am looking for.

### Prefix functions
One option would be to use the same naming concept for all of them so you can find them better.
For example prefix all with 'MY-'

```powershell
function MY-open-ImportantExcelFile {...}

function MY-update-ModuleManifest {...}
```

Then you can just type MY- and tab your way thru the functions.

```powershell
PS > MY- [TAB]
PS > MY-open-ImportantExcelFile
```

If you have PSReadLine then you could also type MY- and use CTRL+SPACE to list all of them.
```powershell
PS > MY- [CTRL+SPACE]
MY-open-ImportantExcelFile    MY-update-ModuleManifest 
```

### Create a menu
Another approach would be to create a MENU function in your profile that would list all functions and enable quick execution.

```powershell
Function menu($functionName)
```
The functionName will be used as an optional filter to narrow the list.

Using the PowerShell's Abstract Syntax Tree I can load my profile file and extract all function definitions.
{% raw %}
```powershell
# Parse profile file using Language Parser
  $AST = [System.Management.Automation.Language.Parser]::ParseFile(
	    $profile,
	    [ref]$null,
	    [ref]$Null
	)
 
 # get all function definitions using the FunctionDefinitionAst
 # extract the function names using regex
 # list all except my menu function itself
  $functions = @($AST.FindAll({
	    	$args[0] -is [System.Management.Automation.Language.FunctionDefinitionAst]}
	    	, $true) | ForEach-Object {
	    		if ($_.Extent.Text -match '^\s{0,}function ([\w|\-]*)\s{0,}{{0,1}'){
					$Matches[1].trim() }
			} | Where-Object {$_ -ne "menu" -and $_ -like "*$functionName*"} | Sort-Object)
```
Then display the list as a menu with numbers

```powershell
  For ($i=0;$i -lt $functions.Count;$i++){				
		Write-Output "	[$($i.tostring('00'))]	$($functions[$i])"
	}
  # [00]    open-ImportantExcelFile
  # [01]    update-ModuleManifest
```
{% endraw %}
In the end prompt for input which function to run and execute
```powershell
  if ($functions.count -gt 0){
		$run = "RUN"
		while ($run -as [int] -eq $null){
			$Run = Read-Host -Prompt "Function to run"
		}
		if ($functions[$run]){
			Write-Warning "Run $($functions[$run]) [N to stop]"
			$confirm = Read-Host 
			if ($confirm -notlike "N*"){
				. $functions[$run]
			}
		}
	}
```

When we put our function MENU into our profile we can test it


### Source code
Code on [GitHubGist](https://gist.github.com/amnich/5099c5e472150da1d09f5ceb1142765a)
