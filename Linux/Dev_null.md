# /dev/null en Linux

/dev/null en Linux es un archivo de dispositivo nulo. Esto descartará todo lo escrito en él y devolverá EOF al leer.

Este es un truco de línea de comandos que actúa como una aspiradora, que aspira todo lo que se le arroja.

## 1. /dev/null Propiedades

Esto devolverá un carácter de fin de archivo (EOF) si intenta leerlo usando el comando cat.

``` 
cat /dev/null
``` 

Este es un archivo válido, que se puede verificar usando

``` 
stat /dev/null
``` 

Esto me da una salida de

``` 
  File: /dev/null
  Size: 0               Blocks: 0          IO Block: 4096   character special file
Device: 6h/6d   Inode: 5           Links: 1     Device type: 1,3
Access: (0666/crw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-02-04 13:00:43.112464814 +0530
Modify: 2020-02-04 13:00:43.112464814 +0530
Change: 2020-02-04 13:00:43.112464814 +0530
``` 

Esto muestra que este archivo tiene un tamaño de 0 bytes y no tiene bloques asignados. Los permisos del archivo también están configurados para que cualquiera pueda leerlo o escribirlo, pero no pueda ejecutarlo.

Dado que no es un archivo ejecutable, no podemos usar la tubería usando | el operador para redirigir al archivo /dev/null. La única forma es utilizar redirecciones de archivos (>, >>o <, <<).


## 2. Redirección a /dev/null en Linux

Podemos descartar cualquier salida de un script que usemos redirigiendo a /dev/null.

Por ejemplo, podemos intentar descartar echo mensajes usando este truco.

``` 
echo 'Hello from JournalDev' > /dev/null
``` 

¡No obtendrá ningún resultado ya que se descarta!

Intentemos ejecutar un comando incorrectamente y canalizar su salida a /dev/null.

``` 
cat --INCORRECT_OPTION > /dev/null
``` 

Todavía obtenemos un resultado como este:

``` 
cat: unrecognized option '--INCORRECT'
Try 'cat --help' for more information.
``` 

¿Por qué está pasando esto? Esto se debe a que los mensajes de error provienen de stderr, pero solo descartamos la salida de stdout.

stderr También debemos tener en cuenta.

### 2.1 Descartar mensajes de error

Redirijamos el stderr a /dev/null, junto con stdout. Podemos usar el descriptor de archivo para stderr(=2) para esto.

``` 
cat --INCORRECT_OPTION > /dev/null 2>/dev/null
``` 

¡Esto nos dará lo que necesitamos!

Hay otra forma de hacer lo mismo; redirigiendo stderr a stdout primero y luego redirigiendo stdout a /dev/null.

La sintaxis para esto será:

``` 
command > /dev/null 2>&1
``` 

Observe el 2>&1 al final. Redirigimos stderr(2) a stdout(1). Solemos &1 mencionarle al shell que el archivo de destino es un descriptor de archivo y no un nombre de archivo.

``` 
cat --INCORRECT_OPTION > dev/null 2>&1
``` 

Así que si usamos 2>1, sólo redireccionaremos stderr a un archivo llamado 1. ¡Esto no es lo que queremos!

