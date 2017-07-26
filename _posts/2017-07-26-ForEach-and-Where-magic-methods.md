 [http://www.powershellmagazine.com/2014/10/22/foreach-and-where-magic-methods/](http://www.powershellmagazine.com/2014/10/22/foreach-and-where-magic-methods/)

 A great article about foeach and where methods which where introduced in PowerShell 4.0 and in my opinion are not well known.
 
 I encourage you to read it and take advantage of the improvements in performance, functionality and syntax!
 
 Just a quick peek.
 
>ForEach is a method that allows you to rapidly iterate through a collection of objects and take some action on each object in that collection.  This method provides faster performance than its older counterparts (the foreach statement and the ForEach-Object cmdlet), and it also simplifies some of the most common actions that you may want to take on the objects in the collection.  Any objects that are output by this method are returned in a generic collection of type System.Collections.ObjectModel.Collection1[psobject].

```powershell
# Get all commands that have a ComputerName parameter
$cmds = Get-Command -ParameterName ComputerName
# Now show a table making sure the parameter names and aliases are consistent
$cmds.foreach('ResolveParameter','ComputerName') | Format-Table Name,Aliases
```
>Where is a method that allows you to filter a collection of objects.  This is very much like the Where-Object cmdlet, but the Where method is also like Select-Object and Group-Object as well, includes several additional features that the Where-Object cmdlet does not natively support by itself. This method provides faster performance than Where-Object in a simple, elegant command.  Like the ForEach method, any objects that are output by this method are returned in a generic collection of type System.Collections.ObjectModel.Collection1[psobject].
```powershell
# Get the first service in our collection that is running
$services.where({$_.Status -eq 'Running'},'First')
# Get the first service in our collection that is running
$services.where({$_.Status -eq 'Running'},'First',1)
# Get the first 10 services in our collection that are running
$services.where({$_.Status -eq 'Running'},'First',10)
```
