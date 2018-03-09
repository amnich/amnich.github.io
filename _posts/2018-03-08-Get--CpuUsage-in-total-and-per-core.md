---
title: "2018-03-08 Get-CpuUsage in total and per core"
date: 2018-03-08
tags: [PowerShell]
---

## INTRO

A simple function Get-CpuUsage that allows to get CPU usage over a period of time.

```powershell
PS C:\> Get-CpuUsage -SampleInterval 2 -MaxSamples 5 -PerCore

ComputerName Name   Average
------------ ----   -------
COMPUTER1    _Total 13     
COMPUTER1    0      9,6    
COMPUTER1    1      8,6    
COMPUTER1    2      12,2   
COMPUTER1    3      21    
```

Full code on [GitHub](https://github.com/amnich/Get-CpuUsage)

## Functionality

Function can get the CPU usage from a local or remote computer(s). 

With the parameters SampleInterval and MaxSamples you can specify the time between samples in seconds and the number of samples to get from each counter.

Returned are single readings and the average value for each processor core or/and usage in total.

## Details

### Get-Counter -Counter '238(_Total)\6'

To get the CPU total usage only the Get-Counter cmdlet is used to query the '\Processor(_Total)\% Processor Time' counter.

But if you try to run `Get-Counter -Counter '\Processor(_Total)\% Processor Time'` on a Non-US system you get an error
```
Get-counter : The specified object was not found on the computer.
```

Performance counter names are localized. Yay Microsoft!

But there is a solution - you can use language-neutral ID numbers.

So '238(_Total)\6' translates to '\Processor(_Total)\% Processor Time'

```powershell
$data = Get-Counter -Counter "\238(_Total)\6" -SampleInterval $SampleInterval -MaxSamples $MaxSamples 
$counter = (($data.countersamples).cookedvalue | Measure-Object -Average).average
```

How to get the codes? Great details about this and ready functions you can find in an [article by Tobias Weltner on PowerShellMagazine.](http://www.powershellmagazine.com/2013/07/19/querying-performance-counters-from-powershell/)

### Format output using XML

Returned from the above is a PSCustomObject

```powershell
$object =  New-Object PSCustomObject -Property @{
  ComputerName = $Env:COMPUTERNAME
  Name = "_Total"
  Average = [Math]::Round($counter,2)
  Data = $data
  SingleReadings = Foreach ($single in $data){
    New-Object PSCustomObject -Property @{
      TimeStamp = $single.CounterSamples[0].TimeStamp
      Cookedvalue = $single.CounterSamples[0].Cookedvalue
      Name = "_Total"
    }
  }		
}
```
This isn't looking to good
```
SingleReadings : @{TimeStamp=2018-03-07 13:22:26; Cookedvalue=20,3181892812853; Name=_Total}
ComputerName   : COMPUTER1
Name           : _Total
Data           : Microsoft.PowerShell.Commands.GetCounter.PerformanceCounterSampleSet
Average        : 20,32
```

I use a XML formatting file to define the output format.

First change the type name of our PSCustomObject
```
$object.PSObject.TypeNames.Insert(0,'My.CpuUsage')
```

Then I create a new format.ps1xml (My.CpuUsage.format.ps1xml) file and define the output format to show by default only ComputerName, Name and Average.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration>
	<ViewDefinitions>
		<View>
			<Name>My.CpuUsage</Name>
			<ViewSelectedBy>
				<TypeName>Deserialized.My.CpuUsage</TypeName>
				<TypeName>My.CpuUsage</TypeName>				
			</ViewSelectedBy>
			<TableControl>
				<TableHeaders>
					<TableColumnHeader/>												
					<TableColumnHeader/>
					<TableColumnHeader/>
				</TableHeaders>
				<TableRowEntries>
					<TableRowEntry>
						<TableColumnItems>
							<TableColumnItem>
								<PropertyName>ComputerName</PropertyName>
							</TableColumnItem>							
							<TableColumnItem>
								<PropertyName>Name</PropertyName>
							</TableColumnItem>
							<TableColumnItem>
								<PropertyName>Average</PropertyName>
							</TableColumnItem>							
						</TableColumnItems>
					</TableRowEntry>
				</TableRowEntries>
			</TableControl>
		</View>
	</ViewDefinitions>
</Configuration>
```

To import the format use `Update-FormatData` like:
```powershell
PS > Update-FormatData -AppendPath .\My.CpuUsage.format.ps1xml`
```

Then the output will look like this:

```
ComputerName Name   Average
------------ ----   -------
COMPUTER1    _Total 20,32
```

I have most of my functions inside a module and I import the custom formats in PSD1 'FunctionsToExport'.

Read more [about Format.ps1xml](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_format.ps1xml?view=powershell-6)

### Per core

To get the CPU usage per core I used a different approach and used the WMI class Win32_PerfFormattedData_PerfOS_Processor.
```powershell
$res = Get-WmiObject -Query "select Name, PercentProcessorTime from Win32_PerfFormattedData_PerfOS_Processor" 
$now = Get-Date
foreach ($single in $res){
  New-Object pscustomobject -Property @{
    TimeStamp = $now
    Cookedvalue = $single.PercentProcessorTime
    Name = $single.Name
  }
}
```
It's possible to get the core usage with counters like for the total. There's more than one way to skin a cat.

## Usage examples

```powershell
PS C:\> Get-CpuUsage -ComputerName COMPUTER2 -SampleInterval 1 -MaxSamples 5

ComputerName Name   Average
------------ ----   -------
COMPUTER2    _Total 23,23
```

```powershell
PS C:\> $results = Get-CpuUsage -ComputerName COMPUTER2 -SampleInterval 1 -MaxSamples 5
PS C:\> $results.SingleReadings

TimeStamp                Cookedvalue Name  
---------                ----------- ----  
2018-03-07 13:12:03 34,8307210757895 _Total
2018-03-07 13:12:04 39,4556991872146 _Total
2018-03-07 13:12:05 42,6071994534768 _Total
2018-03-07 13:12:06 33,2035942544808 _Total
2018-03-07 13:12:07 33,6261072727046 _Total
```

```powershell
PS C:\> Get-CpuUsage -SampleInterval 2 -MaxSamples 5 -PerCore

ComputerName Name   Average
------------ ----   -------
COMPUTER1    _Total 13     
COMPUTER1    0      9,6    
COMPUTER1    1      8,6    
COMPUTER1    2      12,2   
COMPUTER1    3      21    
 ```
