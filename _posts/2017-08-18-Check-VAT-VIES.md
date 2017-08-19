---
title: "2017-08-18 Check VAT number on VIES"
date: 2017-08-18
tags: [PowerShell,VIES]
---

# VIES
[VIES](http://ec.europa.eu/taxation_customs/vies/) provides an SOAP API to automate the VAT number check.

In my case however it was needed to automate the check, open the web site and print it.

What I did was:
* query results using the Invoke-WebRequest
* store the results in a temp file
* replace the href and src references to http://ec.europa.eu to show the web page later on
* open page in IE
* create an object with results
* print on default printer

The Invoke-WebRequest uses a POST method to http://ec.europa.eu/taxation_customs/vies/vatResponse.html
```powershell
$POST = "memberStateCode=$country&number=$vatnumber&action=check"
```
Results are stored in a temporary file
```powershell
Invoke-WebRequest -Method Post -Body $POST -Uri 'http://ec.europa.eu/taxation_customs/vies/vatResponse.html' -OutFile $tempFile
```
Replace src and href code to display page correctly with pictures and links
```powershell
$file = $file.replace('href="','href="http://ec.europa.eu') 
$file = $file.replace('src="','src="http://ec.europa.eu')
```
Create new IE com object and navigate to page
```powershell
$ie = New-Object -com InternetExplorer.Application 
$ie.AddressBar = $false
$ie.MenuBar = $false
$ie.ToolBar = $false
$ie.visible=$true
$ie.navigate("$env:temp\vat.html")
```
Create output object with results
```powershell
$obj = New-Object Pscustomobject -Property @{
			Date = Get-Date
			NIP = $TIN
			Result = $null
			User = $env:Username
}
```
Check if VAT number was valid or not
```powershell
if ($file -match ("Yes, valid VAT number")) { 
  $obj.Result = $true			
}
elseif ($file -match ("No, invalid VAT number")) { 
  $obj.Result = $false			
}
else{
  throw "Not expected results." 
}
```
Finally automatic print
```powershell
if (-not $NoPrint){
  try {
    $ie.execWB(6,2)
  }
  catch{
    $Error[0]
  }
}
```
## Source code
Source code on [GitHub](https://github.com/amnich/Check-VAT_VIES)

## Example usage
```powershell
PS >  Check-VAT_VIES -TIN DE99999999999

Date                NIP           User     Result
----                ---           ----     ------
2017-08-18 16:47:04 DE99999999999 user1    True
```
