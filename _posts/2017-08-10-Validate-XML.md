---
title: "2017-08-10 Validate XML with XSD"
date: 2017-08-10
tags: [PowerShell,XML,XSD]
---
# Valide XML file with XSD and show errors
I had a need to give a user a quick tool to validate a generated XML file based on a schema XSD file and show where the errors are located.
## Test-Xml
For the first part I needed to validate a XML file against XSD schema and get back error messages with at least error message, line and position/column in file.

That is covered with [Test-Xml](https://github.com/amnich/Search-XmlError/blob/master/Test-Xml.ps1) which returns error messages from validation.

 It uses a System.Xml.XmlReader and ValidationEventHandler to capture the errors.
 {% highlight powershell %}
 PS > Test-Xml -XmlFile C:\my.xml -XsdFile C:\schema.xsd
 {% endhighlight %}
```
PS > Test-Xml -XmlFile C:\my.xml -XsdFile C:\schema.xsd    
    SourceObject       :
    SourceUri          : file:///C:/my.xml
    LineNumber         : 8
    LinePosition       : 414720
    SourceSchemaObject :
    Message            : The element 'Look' in namespace 'http://xml.foo.bar/' has incomplete content. Lis
                        t of possible elements expected: 'A_1' in namespace 'http://xml.foo.bar/'.
    Data               : System.Collections.ListDictionaryInternal
    InnerException     :
    TargetSite         :
    StackTrace         :
    HelpLink           :
    Source             :
    HResult            : -2146231999
    
```
## Show-XmlError
Second part is to show the XML fragment before and after the error. 

For that is the [Show-XmlError](https://github.com/amnich/Search-XmlError/blob/master/Show-XmlError.ps1) which accepts input from Test-Xml. 

```
PS > Test-Xml -XmlFile C:\my.xml -XsdFile C:\schema.xsd | Show-XmlError
    ======= Error in Line 8 Column 414720  ============= 
    <Look typ="G">
    <LookOne>1453</LookOne>
    <Number>.</Number>
    <Name>Carl Johann GmbH Bonn</Name>
    <Address>Vor den Siebenburgen 123 , Bonn</Address>
    <LookFoo>DDD/11/03/2</LookFoo>
    <DataOne>2015-03-17</DateOne>
    <DateTwo>2015-03-17</DateTwo>
    <A_0>3627.18</A_0>
    --> The element 'Look' in namespace 'http://xml.foo.bar/' has incomplete content. List
        of possible elements expected: 'A_1' in namespace 'http://xml.foo.bar/'..  <--
    </Look>
    <Look typ="G">
    <LookOne>1454</LookOne>
    <Number>.</Number...
```

It also accepts manual XML file input with specified line and column number of error.

```powershell
PS > Show-XmlError -XmlFile C:\my.xml -Line 8 -Column 414720

```

I added a [open file dialog](https://pastebin.com/CvbjVKsH), hardcoded the XSD file path, converted the script to an EXE file and done :)

```powershell
try{
	$filename = Get-Filename -filter "XML | *.xml" -title "Select XML file to validate"
	if ($filename){	
		Test-Xml -XmlFile $filename -XsdFile "D:\Foo\schema.xsd" | Show-XmlError -Pause
		Write-Output "END....."
		Pause
	}
	else{
		throw "XML file missing"
	}
}
catch{
	throw "Error: $($Error[0])"
}
```
This is how it looks like for the user. 
![gif]({{ site.url }}/assets/images/20170810_XML.gif)
