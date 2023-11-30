# Cómo utilizar el comando de fuser de Linux

## 1. Introducción

El fuser comando es una utilidad de Linux diseñada para encontrar qué proceso está utilizando un archivo, directorio o socket determinado. También proporciona información sobre el usuario propietario de ese proceso y el tipo de acceso.

## 2. Cómo utilizar la utilidad del fuser

Puede revisar la fuser página del manual para obtener una descripción general de todas las opciones que puede utilizar fuser. También puede ejecutarlo fuser solo sin opciones para obtener una descripción general de fuser la sintaxis:

``` 
fuser
Output
No process specification given
Usage: fuser [-fMuv] [-a|-s] [-4|-6] [-c|-m|-n SPACE] [-k [-i] [-SIGNAL]] NAME...
       fuser -l
       fuser -V
Show which processes use the named files, sockets, or filesystems.

  -a,--all              display unused files too
  -i,--interactive      ask before killing (ignored without -k)
  -k,--kill             kill processes accessing the named file
  -l,--list-signals     list available signal names
  -m,--mount            show all processes using the named filesystems or block device
  -M,--ismountpoint     fulfill request only if NAME is a mount point
  -n,--namespace SPACE  search in this name space (file, udp, or tcp)
  -s,--silent           silent operation
  -SIGNAL               send this signal instead of SIGKILL
  -u,--user             display user IDs
  -v,--verbose          verbose output
  -w,--writeonly        kill only processes with write access
  -V,--version          display version information
  -4,--ipv4             search IPv4 sockets only
  -6,--ipv6             search IPv6 sockets only
  -                     reset options

  udp/tcp names: [local_port][,[rmt_host][,[rmt_port]]]
``` 

## 3. Cómo ver los procesos que se ejecutan en un directorio

fuser También se puede utilizar con la opción -v, que ejecuta la herramienta en modo detallado. La opción detallada se utiliza para producir más resultados para que el usuario pueda observar lo que fuser se está haciendo. Ejecutar fuser en el directorio actual, incluyendo la -v opción:

``` 
fuser -v .
Output
                     USER        PID ACCESS COMMAND
/home/sammy:         sammy     17604 ..c.. bash
``` 

En este caso, el único proceso que se ejecuta en este directorio es el shell interactivo bash desde el que está ejecutando comandos en este momento.

Cuando se ejecuta en modo detallado, la fuser utilidad brinda información sobre USER y PID  de un proceso. El carácter debajo muestra el tipo de acceso, en este caso significa el directorio actual. Hay otros tipos de acceso, como ejecutable en ejecución, directorio raíz, archivo abierto y archivo asignado o biblioteca compartida. ACCESS COMMAND c ACCESS


## 4. Cómo encontrar procesos utilizando sockets de red

Es posible que también necesite buscar procesos que utilicen sockets TCP y UDP. Para demostrar este ejemplo, primero creará ncun escucha TCP en el puerto 8002, de modo que haya un proceso en ejecución que pueda observar:

``` 
nc -l -p 8002
``` 

Esto bloqueará el terminal mientras esté funcionando. En otra ventana de terminal, use fuser para buscar el proceso que se ejecuta en el puerto TCP 8002 con la -n opción:

``` 
fuser -v -n tcp 8002
Output
                     USER        PID ACCESS COMMAND
8002/tcp:            sammy     17985 F.... nc
``` 

**Nota:** De forma predeterminada, la fuser herramienta verificará los sockets IPv4 e IPv6, pero puede cambiar esto con las opciones -4 y -6 para verificar solo conexiones IPv4 o solo IPv6, respectivamente.

Este resultado muestra que la identificación del proceso (PID) del proceso que usa netcat es 17985 y el comando que se usó para iniciarlo es 'nc'. La identificación del proceso (PID) se puede utilizar de muchas maneras, incluso para detener o finalizar un proceso en ejecución. También puedes usar fuser para finalizar procesos que se ejecutan en puertos específicos usando la -k bandera:

``` 
fuser -k 8002/tcp
Output
8002/tcp:            18056
``` 

Si regresa a la primera ventana de su terminal, notará que el nc programa se eliminó y se devolvió al shell.

La utilidad del fusor también se puede utilizar para enviar señales específicas a un proceso. Cuando se usa con la opción -k, el comando fusor envía la señal KILL a un proceso. Hay muchas otras señales que se pueden enviar a un proceso en ejecución específico. Puedes enumerarlos con fuser -l:

``` 
fuser -l
Output
HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM TERM STKFLT
CHLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR SYS
``` 

