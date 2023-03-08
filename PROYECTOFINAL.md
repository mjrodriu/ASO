# PROYECTO FINAL - ASIR2
-------------------------------------------------
Proyecto sobre instalación automática de Directorio Activo + IIS + Autenticación de usuarios de Windows

-------------------------------------------------
## Instalar servicios del Directorio Activo (AD)
```Powershell
function creardirectorioactivo{
    
    Add-WindowsFeature AD-Domain-Services -IncludeManagementTools
    Add-WindowsFeature -Name "web-server" -IncludeAllSubFeature -IncludeManagementTools 
    Import-Module webadministration
    Import-Module activedirectory

    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "andel.local"
    $DomaninNetBIOSName = "andel"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = (ConvertTo-SecureString -String "Andel1928" -AsPlainText -Force)  

    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $Password -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
}
creardirectorioactivo
```
## Cambiar nombre del server (para que sea más bonito y fácil de manejar)
``` Powershell
Rename-Computer -NewName server1 -Restart
``` 
## Cambiar IP del server para establecer un rango de dominio
``` Powershell
function cambiarIP{

    $interfaz = (Get-NetIPConfiguration).InterfaceIndex
    New-NetIPAddress -InterfaceIndex $interfaz -IPAddress 192.168.1.1 -PrefixLength 24 
    Set-DnsClientServerAddress -InterfaceIndex $interfaz -ServerAddresses 8.8.8.8
}
cambiarIP
```
## Comprobar que el equipo (usuario) está dentro del rango y "se ven" (para poder unirlo)
``` Powershell
Test-Connection -ComputerName windows1 
```
## Unir equipos AD
``` Powershell
 function unirPC{
    $equipo = Read-Host "nombre equipo nuevo"
    $dominio = Read-Host "nombre del dominio"
    Add-Computer -ComputerName $equipo -DomainName $dominio -Credential andel\Administrador -Restart -Force
}
unirPC
``` 
## Crear unidades organizativas (leyendo de fichero)
``` Powershell
function crearunidades{
   
    $unidades = Import-Csv "C:\Users\Administrador\unidades.csv"
    $DC1 = "andel"
    $DC2 = "local"
    foreach($unidad in $unidades)
    {    
     New-ADOrganizationalUnit -Name $unidad.unidades -Path "DC=$DC1,DC=$DC2" 
     New-ADGroup -Name $unidad.unidades -GroupScope Global -Path "OU=$($unidad.unidades), DC=$DC1, DC=$DC2"
    }
}
crearunidades
``` 
----------------------------------------------
### Contenido de unidades.csv:
``` csv
unidades
profesores
alumnos
```
----------------------------------------------

## Crear usuarios (leyendo de fichero)
``` Powershell
function crearusuarios{
    $usuarios = Import-Csv .\usuarios.csv
    $DC1 = "andel"
    $DC2 = "local"
        foreach($usuario in $usuarios)
        {        
            if($usuario.grupo -eq "profesores")
            {
                $password = ConvertTo-SecureString -string $usuario.pass -AsPlainText -Force 
                New-ADUser -name $usuario.nombre  -Path "OU=profesores, DC=$DC1, DC=$DC2" -AccountPassword $password -Enable $true 
                Add-AdGroupMember -Identity $usuario.grupo -Members $usuario.nombre  
            }
            elseif($usuario.grupo -eq "alumnos")
            {
                $password = ConvertTo-SecureString -string $usuario.pass -AsPlainText -Force 
                New-ADUser -name $usuario.nombre  -Path "OU=alumnos, DC=$DC1, DC=$DC2" -AccountPassword $password -Enable $true 
                Add-AdGroupMember -Identity $usuario.grupo -Members $usuario.nombre 
            }
       }
 }
 crearusuarios
``` 
----------------------------------------------
### Contenido de usuarios.csv:
``` csv
grupo,nombre,pass
profesores,jesus,Andel1928
alumnos,pablo,Andel1928
alumnos,maria,Andel1928
profesores,numilen,Andel1928
``` 
----------------------------------------------

## Para cada grupo (en este caso: profesores y alumnos) se crea una carpeta con una página web y un sitio web.
``` Powershell
function crearwebs{
foreach($grupo in (Get-ADGroup -Filter *).name){    
    $numero = 81

    if($grupo -match "profesores")
    {
       $web = New-Item C:\inetpub\wwwroot\$grupo –ItemType directory -Force
       "<html>hola "+$grupo+"</html>" | Out-File C:\inetpub\wwwroot\$grupo\index.html
       $Website = New-Website -Name $grupo -HostHeader "" -Port $numero -PhysicalPath $web -ApplicationPool "DefaultAppPool" -Force  
       Remove-WebConfigurationProperty -Filter '/system.webServer/Security/authorization'-PSPath IIS:\Sites\$grupo -Name . -AtElement @{users='*'} 
       Add-WebConfiguration -Filter '/system.webServer/Security/authorization' -value @{accessType='Allow'; roles=$grupo} -PSPath IIS:\Sites\$grupo
         
    }
    elseif($grupo -match "alumnos")
    {
       $web = New-Item C:\inetpub\wwwroot\$grupo –ItemType directory -Force
       "<html>hola "+$grupo+"</html>" | Out-File C:\inetpub\wwwroot\$grupo\index.html
       $Website = New-Website -Name $grupo -HostHeader "" -Port $numero -PhysicalPath $web -ApplicationPool "DefaultAppPool" -Force  
       Remove-WebConfigurationProperty -Filter '/system.webServer/Security/authorization'-PSPath IIS:\Sites\$grupo -Name . -AtElement @{users='*'}
       Add-WebConfiguration -Filter '/system.webServer/Security/authorization' -value @{accessType='Allow'; roles=$grupo} -PSPath IIS:\Sites\$grupo  
       $numero = $numero++
    }    
  }
}
crearwebs
``` 
------------------------------------------------------
Para la "autenticación", es decir, para configurar que sólo los usuarios tengan los permisos para entrar (mediante credenciales) a las páginas creadas,
se ha de cambiar las siguientes líneas en el fichero: ***C:\Windows\System32\inetsrv\config\applicationHost.config***.
``` xml
<!-- Dentro de la seccion "security" en el documento -->
<sectionGroup name="security"/>
<!-- Cambiar el overrideModeDefault de DENY a ALLOW -->
    <section name="access" overrideModeDefault="Allow" />


<!-- Dentro de la seccion "authentication" -->
<sectionGroup name="authentication"/>
<!-- Cambiamos en "anonymousAuthentication" y "windowsAuthentication" de DENY a ALLOW -->
    <section name="anonymousAuthentication" overrideModeDefault="Allow" />
    <section name="windowsAuthentication" overrideModeDefault="Allow" />
```
------------------------------------
Una vez realizados estos cambios, podremos deshabilitar la autenticación **ANÓNIMA**
y con esto conseguimos que los usuarios deban introducir credenciales para entrar al sitio web.

------------------------------------
### Ver configuración
``` Powershell
Get-WebConfigurationProperty -Filter '/system.webServer/security/authentication/anonymousAuthentication' -Name enabled 
``` 
### Establecer configuración
``` Powershell
Set-WebConfigurationProperty -Filter '/system.webServer/security/authentication/anonymousAuthentication' -Name enabled -Value 'false' -PSPath 'IIS:\Sites\Default Web Site'
``` 
------------------------------------------------------
A su vez, dentro del script ***crearwebs***, en las líneas siguientes, establecemos unos permisos únicos para cada grupo.

Por tanto **TODOS** los usuarios entrarán al IIS por defecto identificandose, pero a las diferentes "webs" creadas para cada grupo, 
tan solo entraran a aquella a la que pertenezca el **usuario**.

``` Powershell       
       Remove-WebConfigurationProperty -Filter '/system.webServer/Security/authorization'-PSPath IIS:\Sites\$grupo -Name . -AtElement @{users='*'}
       Add-WebConfiguration -Filter '/system.webServer/Security/authorization' -value @{accessType='Allow'; roles=$grupo} -PSPath IIS:\Sites\$grupo  
 ```      
----------------------------------------------------
