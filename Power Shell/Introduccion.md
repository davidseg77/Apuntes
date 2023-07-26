# Información del curso Powershell para principiantes de OW

## 1. Ayuda de Power Shell

### 1.1 CMDLET y módulos

En primer lugar, para conocer la version de PS.

```
get-host
``` 

Para actualizar la ayuda de PS:

``` 
update-help
``` 

Para conocer la fecha del sistema

``` 
get-date
``` 

Para conocer la ruta donde estoy

``` 
get-location
``` 

Para ver el contenido del directorio actual

``` 
get-childitem
``` 

Para obtener los usuarios del sistema

``` 
get-localuser
``` 

Para obtener los grupos

``` 
get-localgroup
``` 

Para obtener los adaptadores de red

``` 
get-netadapter
``` 

Para que me muestre todos los comandos, por ejemplo, que contengan la palabra localgroup

``` 
get-command *localgroup*
``` 

Para conocer los módulos de mi sistema

``` 
get-module
``` 

Para conocer los comandos que tiene un módulo

``` 
get-command -module (nombre del módulo en cuestión)
``` 

Para ver los módulos que estan disponibles:

``` 
get-module -ListAvailable
``` 

Para importar un módulo 

``` 
import-module + nombre del modulo + -verbose
``` 

Para quitar un módulo

``` 
remove-module + nombre del modulo
``` 

### 1.2 Ayuda de CMDLET, atajos y alias

Para obtener ayuda de un comando

``` 
get-help + nombre del comando
``` 

Para eliminar un usuario

``` 
remove-localuser + nombre del usuario
``` 

Para ver el historial

``` 
get-history
``` 

Dentro del historial si marcamos r más el número del comando en el historial nos lo ejecutará.

Si hacemos Control + R nos saldrá un buscador que nos dirá los comandos ya usados en base a la palabra escrita en dicho buscador.

Para saber si un comando tiene alias

``` 
get-alias -Definition + nombre del comando
``` 


## 2. Gestión de archivos y carpetas

Para ver los archivos ocultos:

``` 
get-ChildItem c:\ -Attributes hidden
``` 

Para crear un directorio:

``` 
new-item prueba1 -ItemType Directory
```

Para crear un archivo dentro de ese directorio:

``` 
new-item .\prueba1\texto1.txt
``` 

También puede hacerse con los alias md y ni

Para eliminar un directorio de manera recursiva:

``` 
remove-item .\prueba1\ -recurse
``` 

Para mover un archivo:

``` 
Move-Item *.jpg fotos
``` 

Para renombrar un archivo

```
Rename-Item + nombre actual + nuevo nombre

Rename-Item .\fotos\ recuerdos
```

Para copiar un archivo:

``` 
copy-item .\recuerdos\ + directorio nuevo -recurse
``` 

El recurse se añade por si este directorio tuviera otros subdirectorios, para no generar conflictos.

Para ver el contenido de un archivo:

```
get-content .\prueba.txt
``` 

Todos estos comandos tienen alias, que vienen a ser similares a los comandos empleados para todas estas acciones en Linux. Por ejemplo, para ver el contenido de un archivo también podemos hacer uso del comando cat.


## 3. Tuberías y redireccionamiento

Por ejemplo, para ver dentro de los archivos cuales tienen más de 100 MB y me los ordene en forma descendente por propiedad de tamaño.

```
get-childitem -Recurse | Where-Object {$_.Length -gt 10Mb} | Sort-Object -Descending -Property Length
``` 

Para ver las conexiones cuyo estado esté establecido:

``` 
Get-NetTCPConnection | Where-Object{$_.State -eq 'Established'}
``` 

## 4. Iniciación a los scripts

### 4.1 Introducción

Accedemos a PS en modo administrador. Vemos la política establecida por defecto:

``` 
Get-ExecutionPolicy
``` 

Y añadimos la nuestra a desear:

``` 
Set-ExecutionPolicy RemoteSigned
``` 

Con esto decimos que nuestros scripts se pueden ejecutar, pero los que vengan de forma externa han de estar autenticados.

Abrimos el notepad para iniciar nuestro script:

```
notepad
``` 

Añadimos lo siguiente al script:

``` 
Clear
Write-Host "Bienvenidos a la PowerShell"
Get-Date
``` 

El archivo ha de tener extensión ps1. Para ejecutarlo solo tenemos que poner el nombre.

Los scripts también pueden desarrollarse en **PowerShell ISE**, que nos permite escribir, depurar y ejecutar script.

### 4.2 Práctica: PowerShell ISE

Si creamos el mismo script, podemos con ISE ejecutarlo desde su interfaz e incluso ejecutar una parte del script con la opción del menu superior Ejecutar selección.


## 5. Fundamentos de scripts

### 5.1 Comentarios, variables y constantes

Los comentarios se usan mediante almohadillas. Para un bloque de comentarios se sigue el siguiente procedimiento:

```
<# Este es un ejemplo de 
bloque de comentarios
#>
``` 

Las variables se definen mediante el simbolo $. Para conocer el tipo de una variable:

```
$edad.GetType()
``` 

### 5.2 Variables y operadores

Para leer una variable por pantalla dentro de un script:

``` 
$nombre=Read-host "Como te llamas?"
[int]$edad= Read-host "Cual es tu edad?"

Write-host "Tu nombre es $nombre y tu edad es $edad"
``` 

Con el int entre corchetes declaramos la variable de tipo entera.

**Concatenación**

Ejemplo dentro de un script:

``` 
$nombre="David"
$apellido="Segura"
$nombrecompleto= $nombre + " " + $apellido
Write-Host "Tu nombre completo es: $nombrecompleto"
``` 

### 5.3 Estructuras y funciones

Ejemplo de script que nos permite saber si una ip o servidor tiene conectividad a la red:

``` 
#Conexión con el servidor
Clear
Write-Host "Conectividad"
$ip=Read-host "Introduce la ip del servidor"
if (Test-connection $ip -count 1 -Quiet) {
    Write-Host "$ip Conexión establecida"}
else {
    Write-Host "$ip Error de conexión"}
``` 

* Con **-count** indicamos que solo queremos hacer un ping y con el **-Quiet** que nos muestre la información de manera simplificada.

### 5.4 Estructura repetitiva

Vamos a crear un archivo de tipo .txt donde vamos a insertar dos ips de dos servidores más la de google (8.8.8.8). Hecho esto, creamos un script que recorra las ips del archivo .txt y mediante un foreach nos devuelva si la ip recorrida tiene conexión o no.

``` 
#Conexión con el servidor
Clear
Write-Host "Conectividad"
$datos=Get-Content C:\Users\david\servidores.txt
foreach ($i in $datos)
$respuesta=Test-Connection $i -count 1 -Quiet
if ($respuesta -eq "true") {
    Write-Host "$i Conexión establecida"}
else {
    Write-Host "$i Error de conexión"}
```




















