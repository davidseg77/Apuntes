# Cómo utilizar SFTP para transferir archivos de forma segura con un servidor remoto

## 1. Introducción

FTP, el Protocolo de transferencia de archivos, era un método popular y no cifrado para transferir archivos entre dos sistemas remotos. A partir de 2022, la mayoría del software moderno lo ha dejado obsoleto debido a la falta de seguridad y, en su mayoría, solo se puede utilizar en aplicaciones heredadas.

SFTP, que significa Protocolo seguro de transferencia de archivos, es un protocolo separado integrado en SSH que puede implementar comandos FTP a través de una conexión segura. Normalmente, puede actuar como un reemplazo directo en cualquier contexto donde todavía se necesita un servidor FTP.

En casi todos los casos, SFTP es preferible a FTP debido a sus características de seguridad subyacentes y su capacidad de aprovechar una conexión SSH. FTP es un protocolo inseguro que sólo debe usarse en casos limitados o en redes de confianza.

Aunque SFTP está integrado en muchas herramientas gráficas, esta guía demostrará cómo usarlo a través de su interfaz de línea de comandos interactiva.


## 2. Cómo conectarse con SFTP

De forma predeterminada, SFTP utiliza el protocolo SSH para autenticar y establecer una conexión segura. Debido a esto, están disponibles los mismos métodos de autenticación que están presentes en SSH.

Aunque puede autenticarse con contraseñas de forma predeterminada, le recomendamos crear claves SSH y transferir su clave pública a cualquier sistema al que necesite acceder. Esto es mucho más seguro y puede ahorrarle tiempo a largo plazo.

Si puede conectarse a la máquina mediante SSH, entonces habrá completado todos los requisitos necesarios para utilizar SFTP para administrar archivos. Pruebe el acceso SSH con el siguiente comando:

``` 
ssh sammy@your_server_ip_or_remote_hostname
``` 

Si eso funciona, vuelva a salir escribiendo:

``` 
exit
``` 

Ahora podemos establecer una sesión SFTP emitiendo el siguiente comando:

```
sftp sammy@your_server_ip_or_remote_hostname
``` 

Conectará el sistema remoto y su mensaje cambiará a un mensaje SFTP.

Si está trabajando en un puerto SSH personalizado (no en el puerto predeterminado 22), puede abrir una sesión SFTP de la siguiente manera:

``` 
sftp -oPort=custom_port sammy@your_server_ip_or_remote_hostname
``` 

Esto lo conectará al sistema remoto a través del puerto especificado.


## 3. Obtener ayuda en SFTP

El comando más útil para aprender primero es el comando de ayuda. Esto le da acceso a un resumen de los otros comandos SFTP. Puede llamarlo escribiendo cualquiera de estos en el mensaje:

``` 
help
``` 

o

``` 
?
``` 

Esto mostrará una lista de los comandos disponibles:

``` 
Output
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp grp path                     Change group of file 'path' to 'grp'
chmod mode path                    Change permissions of file 'path' to 'mode'
chown own path                     Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-Ppr] remote [local]          Download file
help                               Display this help text
lcd path                           Change local directory to 'path'
. . .
``` 

Exploraremos algunos de los comandos que ve en las siguientes secciones.


## 4. Navegando con SFTP

Podemos navegar a través de la jerarquía de archivos del sistema remoto usando una serie de comandos que funcionan de manera similar a sus contrapartes de shell.

Primero, orientémonos averiguando en qué directorio nos encontramos actualmente en el sistema remoto. Al igual que en una sesión de shell típica, podemos escribir lo siguiente para obtener el directorio actual:

``` 
pwd
Output
Remote working directory: /home/demouser
``` 

Podemos ver el contenido del directorio actual del sistema remoto con otro comando familiar:

``` 
ls
Output
Summary.txt     info.html       temp.txt        testDirectory
``` 

Tenga en cuenta que los comandos disponibles dentro de la interfaz SFTP no coinciden 1:1 con la sintaxis típica del shell y no tienen tantas funciones. Sin embargo, implementan algunas de las opciones opcionales más importantes, como agregar -la para lsver más metadatos y permisos de archivos:

``` 
ls -la
Output
drwxr-xr-x    5 demouser   demouser       4096 Aug 13 15:11 .
drwxr-xr-x    3 root     root         4096 Aug 13 15:02 ..
-rw-------    1 demouser   demouser          5 Aug 13 15:04 .bash_history
-rw-r--r--    1 demouser   demouser        220 Aug 13 15:02 .bash_logout
-rw-r--r--    1 demouser   demouser       3486 Aug 13 15:02 .bashrc
drwx------    2 demouser   demouser       4096 Aug 13 15:04 .cache
-rw-r--r--    1 demouser   demouser        675 Aug 13 15:02 .profile
. . .
``` 

Para llegar a otro directorio, podemos emitir este comando:

``` 
cd testDirectory
``` 

Ahora podemos atravesar el sistema de archivos remoto, pero ¿qué pasa si necesitamos acceder a nuestro sistema de archivos local? Podemos dirigir comandos hacia el sistema de archivos local precediéndolos con un lfor local.

Todos los comandos analizados hasta ahora tienen equivalentes locales. Podemos imprimir el directorio de trabajo local:

``` 
lpwd
Output
Local working directory: /Users/demouser
``` 

Podemos enumerar el contenido del directorio actual en la máquina local:

``` 
lls
Output
Desktop			local.txt		test.html
Documents		analysis.rtf		zebra.html
``` 

También podemos cambiar el directorio con el que queremos interactuar en el sistema local:

``` 
lcd Desktop
``` 


## 5. Transferir archivos con SFTP

Si queremos descargar archivos desde nuestro host remoto, podemos hacerlo usando el get comando:

``` 
get remoteFile
Output
Fetching /home/demouser/remoteFile to remoteFile
/home/demouser/remoteFile                       100%   37KB  36.8KB/s   00:01
``` 

Como puede ver, de forma predeterminada, el get comando descarga un archivo remoto a un archivo con el mismo nombre en el sistema de archivos local.

Podemos copiar el archivo remoto a un nombre diferente especificando el nombre después:

``` 
get remoteFile localFile
``` 

El get comando también acepta algunas opciones. Por ejemplo, podemos copiar un directorio y todo su contenido especificando la opción recursiva:

``` 
get -r someDirectory
``` 

Podemos decirle a SFTP que mantenga los permisos y tiempos de acceso adecuados usando la bandera -Po :-p

``` 
get -Pr someDirectory
``` 


## 6. Transferir archivos locales al sistema remoto

La transferencia de archivos al sistema remoto funciona de la misma manera, pero con un put comando:

``` 
put localFile
Output
Uploading localFile to /home/demouser/localFile
localFile                                     100% 7607     7.4KB/s   00:00
``` 

Las mismas banderas que funcionan con getse aplican a put. Entonces, para copiar un directorio local completo, puede ejecutar put -r:

``` 
put -r localDirectory
``` 

Una herramienta familiar que resulta útil al descargar y cargar archivos es el df comando, que funciona de manera similar a la versión de línea de comandos. Con esto podrás comprobar que tienes espacio suficiente para completar las transferencias que te interesan:

``` 
df -h
Output
    Size     Used    Avail   (root)    %Capacity
  19.9GB   1016MB   17.9GB   18.9GB           4%
``` 

Tenga en cuenta que no existe una variación local de este comando, pero podemos solucionarlo emitiendo el ! comando.

El ! comando nos lleva a un shell local, donde podemos ejecutar cualquier comando disponible en nuestro sistema local. Podemos comprobar el uso del disco escribiendo:

``` 
!
``` 

y luego

``` 
df -h
Output
Filesystem      Size   Used  Avail Capacity  Mounted on
/dev/disk0s2   595Gi   52Gi  544Gi     9%    /
devfs          181Ki  181Ki    0Bi   100%    /dev
map -hosts       0Bi    0Bi    0Bi   100%    /net
map auto_home    0Bi    0Bi    0Bi   100%    /home
``` 

Cualquier otro comando local funcionará como se esperaba. Para regresar a su sesión SFTP, escriba:

``` 
exit
``` 

Ahora debería ver el retorno del mensaje SFTP.


## 7. Manipulaciones de archivos simples con SFTP

SFTP le permite realizar algunos tipos de limpieza del sistema de archivos. Por ejemplo, puedes cambiar el propietario de un archivo en el sistema remoto con:

``` 
chown userID file
``` 

Observe cómo, a diferencia del chmod comando del sistema, el comando SFTP no acepta nombres de usuario, sino que utiliza UID. Desafortunadamente, no existe una forma integrada de conocer el UID apropiado desde la interfaz SFTP.

Como solución alternativa, puede leer el /etc/passwd archivo, que asocia nombres de usuario con UID en la mayoría de los entornos Linux:

``` 
get /etc/passwd
!less passwd
Output
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
. . .
``` 

Observe cómo, en lugar de dar el ! comando por sí solo, lo hemos usado como prefijo para un comando de shell local. Esto funciona para ejecutar cualquier comando disponible en nuestra máquina local y podría haberse usado df anteriormente con el comando local.

El UID estará en la tercera columna del archivo, como lo delimitan los dos puntos.

De manera similar, podemos cambiar el grupo propietario de un archivo con:

``` 
chgrp groupID file
``` 

Nuevamente, no existe una forma integrada de obtener una lista de los grupos del sistema remoto. Podemos solucionarlo con el siguiente comando:

``` 
get /etc/group
!less group
Output
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
. . .
``` 

La tercera columna contiene el ID del grupo asociado con el nombre en la primera columna. Esto es lo que estamos buscando.

El chmod comando SFTP funciona normalmente en el sistema de archivos remoto:

``` 
chmod 777 publicFile
Output
Changing mode on /home/demouser/publicFile
``` 

No existe un comando equivalente para manipular los permisos de archivos locales, pero puede configurar la umask local, de modo que cualquier archivo copiado al sistema local tenga sus permisos correspondientes.

Eso se puede hacer con el lumask comando:

``` 
lumask 022
Output
Local umask: 022
``` 

Ahora todos los archivos normales descargados (siempre que -p no se utilice la bandera) tendrán 644 permisos.

SFTP también le permite crear directorios en sistemas locales y remotos con lmkdir y mkdir respectivamente.

El resto de los comandos de archivos apuntan solo al sistema de archivos remoto:

``` 
ln
rm
rmdir
``` 

Estos comandos replican el comportamiento principal de sus equivalentes de shell. Si necesita realizar estas acciones en el sistema de archivos local, recuerde que puede acceder a un shell emitiendo este comando:

``` 
!
``` 

O ejecute un solo comando en el sistema local precediendo el comando así !:

``` 
!chmod 644 somefile
``` 

Cuando haya terminado con su sesión SFTP, use exito bye para cerrar la conexión.

``` 
bye
``` 


## 8. Conclusión

Aunque la sintaxis SFTP es mucho menos completa que las herramientas de shell modernas, puede resultar útil para proporcionar compatibilidad con la sintaxis FTP heredada o para limitar cuidadosamente la funcionalidad disponible para los usuarios remotos de algunos entornos.

