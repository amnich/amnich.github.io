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
## Source code
Source code on [GitHub](https://github.com/amnich/Check-VAT_VIES)

## Example usage
```powershell
PS >  Check-VAT_VIES -TIN DE99999999999

Date                NIP           User     Result
----                ---           ----     ------
2017-08-18 16:47:04 DE99999999999 user1    True
```
