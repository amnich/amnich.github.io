---
title: "2017-08-14 Find command usage in scripts"
date: 2017-08-14
tags: [PowerShell]
---

## What and where?
I have over 200 scripts in my repository and in many of them I use functions from my custom module.
The module grew with my Powershell experience.
Many functions are not used anywhere, but which one?
Some evolve and keeping a backwards compatibility is sometimes hard. What will break? (I know - Pester)

So I wanted to find in all my scripts the usage of all commands from my module.
To keep it more universal I created a function that finds a command usage in a script file.

   It gets all PS1 files (or a single file) from Path and searches for a single command, array of commands, all commands from a module or all commands used.

   With the AliasExpand switch it will find all command aliases and search also for them.

## Usage

#### Find usage of Get-ChildItem in all PS1 scripts in C:\Scripts and subdirectories
```powershell
		PS > Get-CommandUsage -Command Get-ChildItem -Path C:\Scripts -Recurse
		
		Command      : Get-ChildItem
		Script       : copy-items.ps1
		Path         : C:\Scripts\copy-items.ps1
		CommandLine  : Get-ChildItem $path_out -Filter *.pdf -ErrorVariable +my_error
		LineNumber   : 40
```
#### Find usage of Get-ChildItem and its aliases in C:\Scripts\copy-items.ps1 script. Expands alias in returned results in Command property
```powershell
		PS > Get-CommandUsage -Command Get-ChildItem -AliasExpand -Path C:\Scripts\copy-items.ps1 
		
		Command      : Get-ChildItem
		Script       : copy-items.ps1
		Path         : C:\Scripts\copy-items.ps1
		CommandLine  : Get-ChildItem $path_out -Filter *.pdf -ErrorVariable +my_error
		LineNumber   : 40
		
		Command      : Get-ChildItem
		Script       : copy-items.ps1
		Path         : C:\Scripts\copy-items.ps1
		CommandLine  : gci $path_in 
		LineNumber   : 41	
```
#### Find all used commands from module SomeModule in scripts in current directory
```powershell
		PS > Get-CommandUsage -Module SomeModule
		
		Command      : Get-SQLDataTable
		Script       : get-ServerInfo.ps1
		Path         : C:\Scripts\get-ServerInfo.ps1
		CommandLine  : Get-SQLDataTable -Query "SELECT ComputerName, Max(TimeCreated) as MaxDate from ServerLogs group by ComputerName"
		LineNumber   : 50		
```
#### Find all commands in scripts in C:\Scripts
```powershell

		PS > Get-CommandUsage -All -Path C:\Scripts
		
		Command      : Get-ChildItem
		Script       : copy-items.ps1
		Path         : C:\Scripts\copy-items.ps1
		CommandLine  : Get-ChildItem $path_in 
		LineNumber   : 4		
```
## GitHub
Full script on [GitHub](https://github.com/amnich/Get-CommandUsage)

## Note
Thanks to Seemingly Science who provided a solution how to take advantage of the Parser and Abstract Syntax Tree from System.Management.Automation.Language.


