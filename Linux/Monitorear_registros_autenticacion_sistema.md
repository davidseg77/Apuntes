# Cómo monitorear los registros de autenticación del sistema en Ubuntu

## 1. Cómo monitorear los inicios de sesión del sistema

Un componente fundamental de la gestión de la autenticación es monitorear el sistema después de haber configurado a sus usuarios.


## 2. Revisar los intentos de autenticación

Los sistemas Linux modernos registran todos los intentos de autenticación en un archivo discreto. Este se encuentra en /var/log/auth.log. Puede ver este archivo usando less:

``` 
sudo less /var/log/auth.log
Output
May  3 18:20:45 localhost sshd[585]: Server listening on 0.0.0.0 port 22.
May  3 18:20:45 localhost sshd[585]: Server listening on :: port 22.
May  3 18:23:56 localhost login[673]: pam_unix(login:session): session opened fo
r user root by LOGIN(uid=0)
May  3 18:23:56 localhost login[714]: ROOT LOGIN  on '/dev/tty1'
Sep  5 13:49:07 localhost sshd[358]: Received signal 15; terminating.
Sep  5 13:49:07 localhost sshd[565]: Server listening on 0.0.0.0 port 22.
Sep  5 13:49:07 localhost sshd[565]: Server listening on :: port 22.
. . .
``` 

Cuando haya terminado de ver el archivo, puede utilizar q para salir less.


## 3. Cómo utilizar el comando "last"

Por lo general, sólo le interesarán los intentos de inicio de sesión más recientes. Puedes verlos con la last herramienta:

``` 
last
Output
demoer   pts/1        rrcs-72-43-115-1 Thu Sep  5 19:37   still logged in   
root     pts/1        rrcs-72-43-115-1 Thu Sep  5 19:37 - 19:37  (00:00)    
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 19:15   still logged in   
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 18:35 - 18:44  (00:08)    
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 18:20 - 18:20  (00:00)    
demoer   pts/0        rrcs-72-43-115-1 Thu Sep  5 18:19 - 18:19  (00:00)
``` 

Esto proporciona una versión formateada de la información guardada en otro archivo, /etc/log/wtmp.

A juzgar por la primera y tercera línea, los usuarios actualmente están conectados al sistema. El tiempo total dedicado a iniciar sesión en el sistema durante otras sesiones ya cerradas se proporciona mediante un conjunto de valores separados por guiones.


## 4. Cómo utilizar el comando "lastlog"

También puede ver la última vez que cada usuario del sistema inició sesión utilizando el lastlog comando.

Esta información se proporciona accediendo al /etc/log/lastlog fichero. Luego se ordena según las entradas del /etc/passwd archivo:

``` 
lastlog
Output
Username         Port     From             Latest
root             pts/1    rrcs-72-43-115-1 Thu Sep  5 19:37:02 +0000 2013
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
games                                      **Never logged in**
. . .
``` 

Puede ver la última hora de inicio de sesión de cada usuario en el sistema.

Observe cómo casi todos los usuarios del sistema tendrán **Never logged in**. Muchas de estas cuentas del sistema no se utilizarán para iniciar sesión directamente, por lo que esto es normal.


## 5. Conclusión

La autenticación de usuarios en Linux es un área relativamente flexible de la gestión del sistema. Hay muchas maneras de lograr el mismo objetivo con herramientas ampliamente disponibles.

Es importante comprender dónde guarda el sistema la información sobre los inicios de sesión para que pueda monitorear su servidor en busca de cambios que no reflejen su uso.

