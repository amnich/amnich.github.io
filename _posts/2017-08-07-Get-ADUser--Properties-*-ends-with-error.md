---
title: "Get-ADUser -Properties * ends with error"
date: 2017-08-07
tags: [PowerShell,Get-AdUser,CryptographicException]
---

Get-ADUser -Filter * -Properties * breaks with an error
```powershell
PS > Get-ADUser -Filter * -Properties *
Get-ADUser : Cannot find the requested object.
```    
 If we look into the last error CategoryInfo we will get more details.
```powershell
PS > $Error[0].CategoryInfo
Category   : NotSpecified
Activity   : Get-ADUser
Reason     : CryptographicException
TargetName : User1
TargetType : ADUser
```

It's an CryptographicException on AD user User1.

So let's grab all properties from a user in AD where it works.
```powershell
$user = Get-ADUser TestUser -Properties *
$props = $user.psobject.properties.name
```
Then we can test all AD user accounts or this particular one.

Then in a catch block test on that user each property, one by one, until we find the one that is corrupted.
```powershell
$users = Get-ADUser User1
# or test all AD Users
# $users = Get-ADUser -Filter *
foreach ($user in $users){	
	try {
		$User | Get-ADUser -Properties * | Out-Null
	}
	catch{
		Write-Output $user.SamAccountName
		foreach ($prop in $props){			
			try {
				$user | Get-ADUser -Properties $prop | Out-Null
			}
			catch{
				Write-Output "    $prop"				
			}
		}
	}
}
```

OUTPUT:
```
User1
    Certificates
```

Now we know we have an issue with the property **certificates** that is mapped to the **userCertificate** attribute in AD.

Correct, delete or whatever you have to do with that corrupted certificate.

[Source Code](https://gist.github.com/amnich/5e2ae590f3a7a65d0ac8899273557f9d#file-test-aduserproperties-ps1)

