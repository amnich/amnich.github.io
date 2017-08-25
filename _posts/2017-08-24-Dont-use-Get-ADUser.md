---
title: "2017-08-24 Don't use Get-ADUser every time! Or maybe use?"
date: 2017-08-24
tags: [PowerShell, AD]
---

When it comes to do some queries in the AD I often see that others advise 
> "Don't call Get-ADUser every time. Better get everything local and query against it."

Putting the obvious greater AD load aside, how much slower is it?

Let's test and compare.
## Prepare
About 2000 user accounts in AD.

Test scenarios:
* Get-ADUser each time
* Get-ADUsers to a variable and then work with that object
* Convert local results to a hashtable, arraylist, deserialize it with to and from csv, to and from json  

```powershell
$ADUsers = Get-ADUser -Filter *               # AD Results 
$ADUsersHT = @{}                              # HashTable
[collections.arraylist]$ADUsersArray = @{}    # ArrayList
#create ht and array
foreach ($aduser in $ADUsers){
	$ADUsersHT.Add($aduser.Samaccountname,$aduser)
	$null = $ADUsersArray.Add($aduser)
}
$ADusersCSV = $ADUsers | ConvertTo-CSV | ConvertFrom-Csv   # CSV
$ADUsersJSON = $AdusersCSV | ConvertTo-Json | ConvertFrom-Json # JSON
```

Get 100 random users from AD to be used in the loop
```powershell
$users = @()
for ($i=0;$i -lt 100;$i++){
	$users += $adusers[$(Get-Random -Minimum 0 -Maximum ($ADUsers.Count - 1))].Samaccountname
}
```
## Tests
### Get user from AD by sAMAccountName
  * Normal Get-ADUser 
  ```powershell
    Get-AdUser $user
  ```
  * Where-Object on AD results
  ```powershell
    $ADUsers | Where-Object samaccountname -EQ $user
  ```
* Where method on AD results
```powershell
  $ADUsers.Where({$_.samaccountname -EQ $user})
```
* Where method with first 1 on AD results
```powershell
  $ADUsersCSV.Where({$_.samaccountname -EQ $user},'First',1)
```
* Foreach on AD results
  ```powershell 
  foreach ($aduser in $ADUsers){
    if ($aduser.samaccountname -eq $user){
      $aduser										
    }
  }
```
* Foreach with break on AD results
  ```powershell 
  foreach ($aduser in $ADUsers){
    if ($aduser.samaccountname -eq $user){
      $aduser					
      break
    }
  }
  ```
* Where-Object on CSV results
```powershell
  $ADUsersCSV | Where-Object samaccountname -EQ $user
```

* Where method on CSV results
```powershell
  $ADUsersCSV.Where({$_.samaccountname -EQ $user})
```
* Where-Object on JSON results
```powershell
  $ADUsersJSON | Where samaccountname -EQ $user}
```
* Foreach on JSON
```powershell
  foreach ($aduser in $ADUsersJSON){
    if ($aduser.samaccountname -eq $user){
      $aduser					      
    }
  }
```
* Foreach with break on JSON
```powershell
  foreach ($aduser in $ADUsersJSON){
    if ($aduser.samaccountname -eq $user){
      $aduser					
      break
    }
  }
```
* Where-Object on ArrayList
```powershell
  $ADusersArray | Where-Object samaccountname -EQ $user}
```
* HashTable
```powershell
  $ADUsersHT[$user]
```

### Results

Average results from 10 runs each 100 users in a loop. (Given time for 100 user queries)
* **Normal Get-ADUser  - 800 ms**   
* Where-Object on AD results - **9843 ms** 
* Where method on AD results - 4895 ms
* Where method with first 1 on AD results - 2526 ms
* Foreach on AD results - 2066 ms
* Foreach with break on AD results - 1094 ms 
* Where-Object on CSV results - **12571 ms**
* Where method on CSV results - 2877 ms
* Where-Object on JSON results - **12513 ms**
* Foreach on JSON - 860 ms
* **Foreach with break on JSON - 463 ms**
* Where-Object on ArrayList - **12381 ms**
* **HashTable - 7 ms**

## Summary

Worst times on Where-Object. It was very slow. ~ 11827 ms
```powershell
  $ADUsers | Where samaccountname -EQ $user
```
The Where method was faster. ~ 3886 ms
```powershell
  $ADUsersCSV.Where({$_.samaccountname -EQ $user})
```
It got better with First option. ~ 1948 ms
```powershell
  $ADUsersCSV.Where({$_.samaccountname -EQ $user},'First',1)
```
Foreach, with break if applicable, is the right way to deal with objects if you already burned them in memory. **1094 ms**
```powershell
  foreach ($aduser in $ADUsers){
    if ($aduser.samaccountname -eq $user){
      $aduser					
      break
    }
  }
```
 
If a deserialized object is something you can live with then it gets a tick faster.
```powershell
  $ADusersCSV = $ADUsers | ConvertTo-CSV | ConvertFrom-Csv
  foreach ($aduser in $ADUsersCSV){
    if ($aduser.samaccountname -eq $user){
      $aduser					
      break
    }
  } 
```
Still 860 ms and 463 ms with break on deserialized objects. 

**Why not stick to get the user direct with Get-Aduser if the AD guys are not knocking at your door? It is fast ~ 800 ms.**
```powershell
  Get-AdUser $user
```

**But nothing beats a HashTable. It is super-fast - 7 ms. **

It was 66 times faster than the best from the rest and 1795 times faster than the worst result.

```powershell
  $ADUsersHT["$user"]
```

The HashTable is not something for every use case but is worth to consider when you want to speed up things.

### Source code
Code on [pastebin](https://pastebin.com/amAFSG6j)

## Join the Discussion
[Reddit r/PowerShell](https://www.reddit.com/r/PowerShell/comments/6vtenn/dont_use_getaduser_every_time_or_maybe_use/)
