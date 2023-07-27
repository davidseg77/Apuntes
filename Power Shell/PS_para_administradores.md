# Información extraida del curso PowerShell para administradores de OW

## 1. Gestión de usuarios y grupos

### 1.1 Gestión de usuarios

Para conocer todos los comandos relacionados con los usuarios:

``` 
Get-Command *localuser*
``` 

Para obtener info de los usuarios

``` 
Get-LocalUser
``` 

Para obtener info completa de un usuario en concreto

``` 
Get-LocalUser -Name manolo|fl
```

Para crear una cuenta. Primero he de encriptar la contraseña:

``` 
$contraseña=ConverTo-SecureString "password" -AsPlainText -Force

New-LocalUser david -Password $contraseña
``` 

Para modificar una cuenta o añadir

Vamos a añadirle un full name, por ejemplo:

```
Set-LocalUser david -FullName "Sr david"
``` 

Para que la contraseña nunca expire

```
Set-LocalUser david -PasswordNeverExpires $true
``` 

Renombrar un nombre de usuario

``` 
rename-localuser david -NewName dasetris

get-localuser
```

Para desactivar un usuario

```
Disable-LocalUser david
```

Para habilitar un usuario

```
Enable-LocalUser david
``` 

Para eliminar un usuario con confirmación

``` 
Remove-LocalUser -Confirm david
``` 

### 1.2 Gestión de grupos

Para ver que comandos trabajan con grupos

``` 
Get-Command *localgroup*
``` 

Para mostrar info de los grupos

``` 
Get-LocalGroup
``` 

Para mostrar info de un grupo en concreto

``` 
Get-Localgroup administradores|fl *
``` 

Para crear un grupo

``` 
New-LocalGroup asir

Get-LocalGroup -Name asir
``` 

Para modificar info del grupo

``` 
Set-LocalGroup -Description "Miembros de Asir" asir
``` 

Para renombrar un grupo

``` 
Rename-LocalGroup asir -NewName proyecto
``` 

Para eliminar un grupo

```
Remove-LocalGroup -Confirm asir
``` 

Para introducir usuarios en un grupo

``` 
Get-LocalGroupMember asir

Add-LocalGroupMember asir -Member david
``` 

Para ver un miembro del grupo

``` 
Get-LocalGroupMember asir
``` 

Para eliminar un usuario de un grupo

```
Remove-LocalGroupMember asir -Member david,monica

Get-LocalGroupMember asir
``` 

### 1.3 Creación y eliminación masiva de usuarios

Creamos el siguiente script para crear usuarios:

``` 
#Creación de un usuario
Clear
$usuario=Read-Host "Introduce nombre de usuario"
$contra=Read-Host "Introduce contraseña" -AsSecureString
New-LocalUser $usuario -Password $contra
Add-LocalGroupMember usuarios -Member $usuario
``` 

En C: he creado un directorio llamado material donde he guardado este archivo (usuarios.csv)

Ahora creamos un segundo script con la intención de crear de forma masiva cuentas de usuario.

``` 
#Creación de usuarios de forma masiva
$usuarios= Import-Csv -Path C:\material\usuarios.csv
foreach ($i in $usuarios){

$clave= ConvertTo-SecureString $i.contra -AsPlainText -Force
New-LocalUser $i.nombre -Password $clave -AccountNeverExpires -PasswordNeverExpires
Add-LocalGroupMember -Group usuarios -Member $i.nombre
}
``` 

Comprobamos con Get-LocalUser

Para eliminar usuarios de forma masiva hacemos uso del siguiente script. Tomamos como referencia el anterior:

``` 
#Eliminación de usuarios de forma masiva
$usuarios= Import-Csv -Path C:\material\usuarios.csv
foreach ($i in $usuarios){
Remove-LocalUser $i.nombre
}
```


## 2. Gestión de carpetas compartidas

Para ver las opciones del comando smbshare:

``` 
get-command *smbshare*
``` 

Para mostrar los recursos compartidos:

``` 
Get-SmbShare
``` 

Para crear un recurso, en este caso una carpeta compartida. Primero creamos la carpeta y luego el recurso compartido:

``` 
New-Item c:\datos -ItemType Directory

New-SmbShare -Path c:\datos

Get-SmbShare
``` 

Para ver los permisos que se le ha asignado:

``` 
Get-SmbShareAccess -Name datos
``` 

Para crear un recurso compartido indicando los permisos para cada usuario:

``` 
New-SmbShare -Path c:\datos -Name datos -FullAccess david -ReadAccess monica
``` 

Para modificar un recurso, por ejemplo añadirle una descripción:

``` 
Set-SmbShare -Name datos -Description "Info privada"
``` 

O para limitar el número máximo de usuarios que pueden conectarse al recurso compartido:

``` 
Set-SmbShare -Name datos -ConcurrentUserLimit 10 -Force

Get-SmbShare datos | fl *
``` 

Para conceder o modificar permisos a un usuario dentro de un recurso compartido:

``` 
Grant-SmbShareAccess -Name datos -AccountName david -AccessRight Full -force
``` 

O para quitar al usuario dentro de los permisos:

``` 
Revoke-SmbShareAccess -Name datos -AccountName david -Force
``` 

O para denegar o bloquear el uso del recurso compartido a un usuario:

``` 
Block-SmbShareAccess -Name datos -AccountName monica -Force
``` 

Denegándolo, no lo quitamos del listado de usuarios con permisos para el recurso compartido, solo le privamos del acceso. Para desbloquear la sintaxis es la misma solo que iniciamos con Unblock.

Para eliminar el recurso compartido:

```
Remove-SmbShare -Name datos -Force

Get-SmbShare
``` 


## 3. Gestión de discos


### 3.1 Gestión de discos

Para ver los comandos a usar relacionados con discos o particiones:

``` 
Get-Command *disk*

Get-Command *partition*
``` 

Para mostrar info de los discos que tenemos:

``` 
Get-Disk | fl
``` 

Para mostrar las particiones que tenemos:

``` 
Get-Partition
``` 

O de un disco en concreto:

``` 
Get-Partition -DiskNumber 0 (Este sería el número del disco)
``` 

Para saber si tengo activada hyper-v:

``` 
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
``` 

Para activarlo:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-hyper-V-All
```

Para crear un disco virtual, tanto en vhd como en vhdx:

``` 
New-VHD -Path c:\disco1.vhd -SizeBytes 1GB -Fixed

New-VHD -Path c:\disco2.vhdx -SizeBytes 5GB -Dynamic

ls c:\disco*
``` 

Sin embargo, esos discos hay que montarlos. Así que procedemos a ello:

```
Mount-vhd -Path c:\disco1.vhd
``` 

Para desmontarlo:

``` 
Dismount-vhd -Path c:\disco1.vhd
``` 

Para iniciar el disco:

```
Initialize-Disk -Number 1
```

Para cambiar el estilo de partición del disco a mbr para que pueda inicarse:

``` 
Set-Disk -Number 1 -PartitionStyle mbr
``` 

Ahora creamos una partición:

``` 
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter v
``` 

Para formatearlo:

``` 
Format-Volume -DriveLetter v -FileSystem ntfs
``` 

Para limpiar el disco:

```
Clear-Disk -Number 1 -RemoveData
```

### 3.2 Creación de discos de forma automática

**Passthru**

Con la opción passthru forzamos a que se nos muestre el resultado en un comando siguiente. En definitiva, nos permite juntar comandos de manera secuencial. Para ello, vamos a montar el segundo de los discos creados anteriormente añadiendole la opción passthru:

``` 
Mount-VHD -Path C:\disco2.vhdx -Passthru |
>> Initialize-Disk -Passthru |
>> New-Partition -AssignDriveLetter -UseMaximumSize |
>> Format-Volume -FileSystem NTFS -Confirm:$false
``` 

**Creación automática de un disco virtual**

Y para ello hacemos uso del siguiente script:

``` 
$Disco=Read-Host "Introduce el nombre del disco (Path)"
[double]$tamano=Read-Host "Tamaño del disco en bytes"
New-VHD -Path $disco -SizeBytes $tamano -Dynamic
Mount-vhd -Path $Disco -Passthru|
Initialize-Disk -PassThru|
New-Partition -AssignDriveLetter -UseMaximumSize|
Format-Volume -FileSystem NTFS -Confirm:$false
``` 


## 4. Gestión de la red

### 4.1 Información de la red

Para ver todos los comandos que tienen que ver con los adaptadores de red:

``` 
Get-Command -Module NetAdapter
``` 

O con la red en si:

``` 
Get-Command -Module NetTCPIP
``` 

**Adaptadores**

Para ver cuales tenemos:

``` 
Get-NetAdapter
``` 

O queremos disponer de info de uno de ellos:

``` 
Get-NetAdapter -Name "wi-fi 7"
``` 

Para desactivar el adaptador para un caso concreto:

``` 
Disable-NetAdapter -Name "wi-fi 7"
``` 

**Configuración de la red**

Para mostrar la configuración de la red hay dos formas:

``` 
Get-NetIPConfiguration

gip
``` 

Para saber si tenemos una configuración estática o dinámica:

``` 
Get-NetIPAddress -InterfaceAlias "wi-fi 7"
```

**Tabla de enrutamiento**

Para ver las tablas que tenemos:

``` 
Get-Netroute -InterfaceAlias "wi-fi 7" | ft -Autosize
``` 

**DNS**

Para ver los DNS con los que se trabaja:

```
Get-DnsClientServerAddress -InterfaceAlias "wi-fi 7"
``` 

Para ver si hay salida DNS:

```
Resolve-DnsName www.google.es
``` 

Para ver la caché:

``` 
Get-DnsClientCache
``` 

Para borrar la caché:

``` 
Clear-DnsClientCache
``` 

**Puertos**

Para ver los puertos que tenemos:

``` 
Get-NetTCPConnection
``` 
Para ver los puertos que tenemos con conexión:

``` 
Get-NetTCPConnection -State established | ft LocalAddress,LocalPort,RemoteAddress,Remote Port,State
``` 

### 4.2 Conectividad

Vamos a probar si tenemos acceso a nuestra puerta de enlace:

```
Test-Connection 192.168.1.1
``` 

O del siguiente modo, más rápido y con un solo ping:

``` 
Test-Connection 192.168.1.1 -count 1 -quiet
``` 

O llamando, por ejemplo, a Google:

``` 
Test-Connection www.google.es -count 1 -quiet
``` 

Acto seguido, vamos a crear un script de comprobación para saber si tenemos acceso a los servidores de mi red. Para ello, hemos de meter en un archivo los datos de los servidores.

**Servidores.csv**

ip,nombre,localizacion
192.168.0.1,Tauro,Dpto
192.168.1.1,Capricornio,Aula1
192.168.2.1,Libra,Aula2

Y creamos el script de comprobación:

``` 
#Conexión con los servidores
# Cabecera
Clear-Host
Write-Host " ----- Conectividad -----"
#Importamos los datos
$datos= Import-Csv -Path C:\material\servidores.csv
#Recorremos los datos
    foreach ($i in $datos) {
    $respuesta=Test-Connection $i.ip -Count 1 -quiet
    if ($respuesta -eq "true") {
        Write-Host "$i Conexión establecida"
        }else {Write-Host "$i Error de conexión"}
    }
``` 


### 4.3 Configuración estática y dinámica

**Configuración estática**

Para realizar una configuración estática vamos a realizar los siguientes pasos:

* Vemos la configuración de la interfaz de red

``` 
gip -InterfaceAlias "wi-fi 7"
``` 

* Borramos la ip de la interfaz de red

``` 
Remove-NetIPAddress -InterfaceAlias "wi-fi 7" -Confirm:$false
``` 

* Borramos la puerta de enlace:

``` 
Remove-NetRoute -InterfaceAlias "wi-fi 7" -Confirm:$false
``` 

* Establecemos la nueva IP para la interfaz de red:

``` 
New-NetIPAddress -InterfaceAlias "wi-fi 7" -IPAddress 192.168.0.5 -PrefixLength 24 -DefaultGateway 192.168.0.1
``` 

* Establecemos las DNS

``` 
Set-DnsClientServerAddress -InterfaceAlias "wi-fi 7" -ServerAddresses 8.8.8.8, 8.8.8.4
``` 

* Y volvemos a comprobar

``` 
gip -InterfaceAlias "wi-fi 7"
``` 

**Configuración dinámica**

Para realizar una configuración estática vamos a realizar los siguientes pasos:

* Vemos la configuración de la interfaz de red

``` 
gip -InterfaceAlias "wi-fi 7"
``` 

* Borramos la ip de la interfaz de red

``` 
Remove-NetIPAddress -InterfaceAlias "wi-fi 7" -Confirm:$false
``` 

* Borramos la puerta de enlace:

``` 
Remove-NetRoute -InterfaceAlias "wi-fi 7" -Confirm:$false
``` 

* Habilitamos el DHCP para la interfaz de red:

``` 
Set-NetIPInterface -InterfaceAlias "wi-fi 7" -Dhcp Enabled
``` 

* Establecemos las DNS

```
Set-DnsClientServerAddress -InterfaceAlias "wi-fi 7" -ResetServerAddresses
``` 

* Restauramos el adaptador

``` 
Restart-NetAdapter -Name "wi-fi 7"
``` 

* Y volvemos a comprobar

``` 
gip -InterfaceAlias "wi-fi 7"
``` 

### 4.4 Configurar la red a través de un script

Este script nos facilitará configurar la IP de forma estática o dinámica.

``` 
# Configuración de IP
# Definición de funciones
Function Get-Menu{
Clear-Host
Write-Host "Configuración IP"
Write-host "1.- IP-Fija"
Write-Host "2.- IP-DHCP"
Write-Host "3.- Salir"
}
Function Get-Adaptador {
Write-Host "Configuración de la IP"
Get-NetAdapter|ft -AutoSize
$script:interfaz = Read-Host "Introduzca la interfaz (IfIndex)"
$script:nombre = Read-Host "Introduzca el nombre (name)"
#Borramos datos
Remove-NetIPAddress -InterfaceIndex $interfaz -Confirm:$false
Remove-NetRoute -InterfaceIndex $interfaz -Confirm:$false
}
Function Ip-Fija {
Get-Adaptador
#Creamos la nueva IP
$ip = Read-host "Introduzca IP"
$mascara = Read-Host "Introduca la máscar (nºs de unos)"
$gateway = Read-Host "Introduzca el gateway"
$dns1 = Read-host "Introduzca el primer DNS"
$dns2 = Read-host "Introduzca el segundo DNS"
New-NetIPAddress -InterfaceIndex $interfaz $ip -PrefixLength $mascara -DefaultGateway $gateway
Set-DnsClientServerAddress -InterfaceIndex $interfaz -ServerAddresses ("$dns1","$dns2")
Restart-NetAdapter -Name $nombre
}
Function IP-Dhcp {
Get-Adaptador
#Establecemos IP por Dhcp
Set-NetIPInterface -InterfaceIndex $interfaz -Dhcp enabled
Set-DnsClientServerAddress -InterfaceIndex $interfaz -ResetServerAddresses
#Restablecer el interfaz
Restart-NetAdapter -Name $nombre
}
#Inicio
do{
Get-Menu
$opcion = Read-Host "Elija una opción"
switch ($opcion){
'1'{Ip-Fija}
'2'{IP-Dhcp}
'3'{exit}
Default {Write-Host "Opción incorrecta"}
}
$intro = Read-Host "Pulse intro para continuar"
}while ($true)
``` 

## 5. Gestión de procesos y servicios

### 5.1 Gestión de procesos

Para ver que comandos tienen que ver con los procesos:

``` 
Get-Command *process*
```

**Informacióm**

Para buscar info de los procesos

```
Get-Process
``` 

Para ver que procesos consumen más CPU:

``` 
Get-Process | sort cpu -Descending | Select-Object -First 10
```

Para saber donde está guardado un proceso:

```
(Get-Process -Name notepad).Path
```

Para eliminar un proceso:

``` 
Stop-Process -Name notepad
```

O por su id:

``` 
Stop-Process 15952
```

Para comenzar un proceso:

``` 
Start-Process -FilePath c:\Windows\notepad.exe
```

Para iniciar una aplicación de Windows:

``` 
Start-Process Microsoft-edge://
```

### 5.2 Gestión de servicios

Para ver que comandos tienen que ver con servicios:

``` 
Get-Command *service*
``` 

**Información**

Para ver que servicios tenemos:

``` 
Get-Service
```

Podemos filtrarlos por nombres, estado o nombre de display. Por ejemplo, para ver los activos:

```
Get-Service  | ?{$_.Status -eq "Running"}
```

Para iniciar un servicio:

``` 
Start-Service -Name Spooler
```

**Modificar**

Para modificar por ejemplo el display name, el StartType, Status...

Por ejemplo, vamos a modificar el estado del servicio Spooler

``` 
Get-Service -Name Spooler | fl *

Get-Service -Name Spooler -StartupType disable

Get-Service -Name Spooler | fl *
```


## 6. Gestión de tareas programadas

### 6.1 Creación y modificación de una tarea

Para ver los comandos relacionados con las tareas programadas:

``` 
Get-Command *scheduledtask*
```

Para ver información de las tareas que tenemos:

``` 
Get-ScheduledTask
```

Para ver cuando se va a ejecutar una tarea:

``` 
(Get-ScheduledTask -TaskName reboot_battery).Triggers
```

**Crear una tarea**

El primer paso es crear la acción, después indicar el trigger y por último registrar la tarea.

- Vamos a hacerlo probando con una tarea que apague el pc todos los días a las cuatro.

Primero, creamos la acción:

``` 
$accion=New-ScheduledTaskAction -Execute "powershell.exe" -Argument "c:\material\ApagarOrdenador.ps1
```

Acto seguido, he creado el siguiente script:

```
#Apagar el ordenador
C:\Windows\System32\shutdown.exe /s /t 600
``` 

Creo el trigger:

``` 
$disparador=New-SchedulesTaskTrigger -Daily -At 16:00
``` 

Y registramos la tarea:

``` 
Register-ScheduledTask -Action $action -Trigger $disparador -TaskName "apagar pc" -Description "Apagar el PC a las 16:00" 
```

Hecho todo esto, solo queda lanzar la tarea para probar:

``` 
Start-ScheduledTask -Taskname "apagar ordenador"
```

Para modificar la tarea, por ejemplo si queremos que esta tarea se ejecute a otra hora:

``` 
$disparador=New-ScheduledTaskTrigger -Daily -At 23:55

Set-ScheduledTask -TaskName "apagar ordenador" -Trigger $disparador
```

### 6.2 Activar, desactivar, exportar, importar y eliminar una tarea

**Activar/desactivar una tarea**

Para desactivar:

``` 
Get-ScheduledTask -Taskname "apagar ordenador"

Disable-ScheduledTask -Taskname "apagar ordenador
```

Para activar sería igual pero Enable en vez de Disable.

**Exportar e importar una tarea**

Lo hacemos en formato .xml:

``` 
Get-ScheduledTask -Taskname "apagar ordenador" | Export-ScheduledTask | Out-File -Filepath c:\practicas\apagarordenador.xml
```

Para importarlo:

``` 
Register-ScheduledTask -xml (Get-Content c:\practicas\apagarordenador.xml | Out-String) -Taskname "apagar ordenador 2"
```

Lo he hecho es importarlo en formato xml dandole un nuevo nombre para poder realizar modificaciones.

**Eliminar una tarea**

``` 
Unregister-ScheduledTask -Taskname "apagar ordenador"
```


## 7. Gestión de la impresora

Para ver los comandos relacionados con las impresoras:

``` 
Get-Command -Module PrintManagement
```

Para obtener info de las impresoras instaladas:

``` 
Get-Printer
```

Para ver los controladores de las impresoras del equipo:

``` 
Get-PrinterDriver
```

**Instalar una impresora**

En primer lugar, descargamos el controlador. Trás ello, tenemos que añadirlo al almacen de Windows, instalamos el controlador y agregamos el puerto que permitira la conexión con la impresora. Para finalizar, instalamos la impresora.

Para ello, descargamos el controlador. Lo hacemos a traves de un archivo que podemos encontrar en Internet. Haciendo clic derecho sobre él, podemos instalarlo en el almacen de Windows.

Hacemos un cat al archivo para dar con el controlador y copiamos el nombre para instalarlo.

``` 
Add-PrinterDriver -Name "Nombre del controlador"

Get-PrinterDriver
```

Ahora definimos el puerto de comunicación:

``` 
Add-PrinterPort -Name "LocalPort:"
```

E instalamos la impresora:

```
Add-Printer -Name "Mi impresora" -DriverName "Nombre del controlador" -PortName "LocalPort:"
```

Y comprobamos con Get-Printer.

**Modificar aspectos de la impresora**

Como por ejemplo, permitir que sea de uso compartido:

``` 
Set-Printer -Name "Mi impresora" -Shared $true
```

**Eliminar impresora**

``` 
Remove-Printer -Name "Mi impresora"
```

Y tanto el puerto creado como el controlador:

```
Remove-PrinterPort -Name "LocalPort:"

Remove-PrinterDriver -Name "Nombre del controlador"

Get-PrinterPort

Get-PrinterDriver
``` 


## 8. Gestión de eventos


### 8.1 Con Get-EventLog

Para ver los comandos relacionados con Eventlog:

``` 
Get-Command *EventLog*
```

Para ver que registros podemos sacar con el comando EventLog:

``` 
Get-EventLog -list
```

Para sacar la info de los registros:

* Eventos del sistema

``` 
Get-EventLog -LogName system
```

* Últimos eventos del sistema

``` 
Get-EventLog -Logname System -Newest 5
```

* O por el id del evento

``` 
Get-EventLog -Logname System -Index 4144
```

* O por tipo de entrada error

``` 
Get-EventLog -Logname System -Newest 5 -EntryType error
```

* O por tipo de fecha

``` 
Get-EventLog -Logname System - Newest 5 -EntryType error -After 14/09/2022 -Before 16/09/2022
```

### 8.2 Con Get-WinEvent

Para ver los comandos relacionados con WinEvent:

``` 
Get-Command *winevent*
```

Para ver que registros podemos sacar con el comando EventLog:

``` 
Get-WinEvent -Listlog *
```

Para sacar la info de los registros:

* Eventos del sistema

``` 
Get-WinEvent -LogName system
```

* Últimos eventos del sistema

``` 
Get-WinEvent -Logname System -MaxEvents 5
```

## 9. Gestión de WMI/CIM

**¿Qué es CIM?**

CIM (Common Information Model). Es un estándar abierto que establece (define)
como intercambiar información entre sistemas, redes, aplicaciones y servicios.

**¿Qué es WMI?**

WMI (Windows Management Instrumentation). Es la implementación de Microsoft del CIM.

Podemos decir que WMI/CIM funciona como una base de datos ofreciendo una gran variedad de información.

WMI/CIM está compuesta por clases, que representa los distintos elementos.

Utilizaremos PowerShell para comunicarnos con WMI/CIM.

**Comandos**

Para ver los comandos relacionados con WMI y CIM:

``` 
Get-Command "*wmi*"
```

``` 
Get-Command *cim*
```





