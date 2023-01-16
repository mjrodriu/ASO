# 2º Proyecto 
Consiste en insertar los hashes y nombres de un los .exe de un equipo del dominio en una base de datos del Server,
para luego comparar alguno de estos "al instante" y comprobar su integridad.

## Conectarse a la BBDD
```PowerShell
#Conectarse a la BBDD
[void][System.Reflection.Assembly]::LoadWithPartialName("MySql.Data")
$Connection = New-Object MySql.Data.MySqlClient.MySqlConnection
$ConnectionString = "server=" + "localhost" + ";port=3306;uid=" + "root" + ";pwd=" + ";database="+"directorioactivo"
$Connection.ConnectionString = $ConnectionString
$Connection.Open()

#Recopilar la info del remoto
$equiposdered = Get-ADComputer -Filter * | where Name -NotMatch WIN-0D0T0Q94LB4 | select -ExpandProperty Name 
$resultado = Invoke-Command -ComputerName $equiposdered -ScriptBlock {$ErrorActionPreference = "SilentlyContinue" 
foreach($proceso in Get-Process)
{
  Get-FileHash $proceso.path 
}
}
$resultado | select hash, path | Export-Csv hashesremotos.csv

#Insertar datos del equipo remoto
$remotos = Import-Csv .\hashesremotos.csv
$remotos
foreach($item in $remotos)
{
    $nombre = $item.Path.Split("\")|select -Last 1
    $insert = "INSERT INTO integridad (nombreproceso,hash) VALUES('$($nombre)','$($item.hash)')"
    $Command = New-Object MySql.Data.MySqlClient.MySqlCommand($insert, $Connection)
    $DataAdapter = New-Object MySql.Data.MySqlClient.MySqlDataAdapter($Command)
    $DataSet = New-Object System.Data.DataSet
    $RecordCount = $dataAdapter.Fill($dataSet,"data")
    $DataSet.Tables[0] 
}

```
# Select de la bbdd para comparar con los resultados del equipo remoto
```PowerShell
$equiposdered = Get-ADComputer -Filter * | where Name -NotMatch WIN-0D0T0Q94LB4 | select -ExpandProperty Name 
$resultado = Invoke-Command -ComputerName $equiposdered -ScriptBlock {(Get-FileHash -path c:\windows\system32\sihost.exe).hash}

$Query ='select hash from integridad where nombreproceso = "sihost.exe"'
$Command = New-Object MySql.Data.MySqlClient.MySqlCommand($Query, $Connection)
$DataAdapter = New-Object MySql.Data.MySqlClient.MySqlDataAdapter($Command)
$DataSet = New-Object System.Data.DataSet
$RecordCount = $dataAdapter.Fill($dataSet, "data")
if($DataSet.Tables[0].hash -eq $resultado)
{
  "es igual"
}else{

 "es distinto"
}

```



