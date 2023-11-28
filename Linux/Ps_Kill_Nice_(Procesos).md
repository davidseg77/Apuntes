# Cómo utilizar ps, kill y nice para gestionar procesos en Linux

## 1. Cómo ver los procesos en ejecución en Linux

Puede ver todos los procesos que se ejecutan en su servidor usando el top comando:

``` 
top
Output
top - 15:14:40 up 46 min,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  56 total,   1 running,  55 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1019600k total,   316576k used,   703024k free,     7652k buffers
Swap:        0k total,        0k used,        0k free,   258976k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND           
    1 root      20   0 24188 2120 1300 S  0.0  0.2   0:00.56 init               
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd           
    3 root      20   0     0    0    0 S  0.0  0.0   0:00.07 ksoftirqd/0        
    6 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0        
    7 root      RT   0     0    0    0 S  0.0  0.0   0:00.03 watchdog/0         
    8 root       0 -20     0    0    0 S  0.0  0.0   0:00.00 cpuset             
    9 root       0 -20     0    0    0 S  0.0  0.0   0:00.00 khelper            
   10 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kdevtmpfs 
``` 

Las primeras líneas de salida proporcionan estadísticas del sistema, como la carga de CPU/memoria y el número total de tareas en ejecución.

Puede ver que hay 1 proceso en ejecución y 55 procesos que se consideran inactivos porque no utilizan activamente los ciclos de la CPU.

El resto del resultado mostrado muestra los procesos en ejecución y sus estadísticas de uso. De forma predeterminada, top los clasifica automáticamente por uso de CPU, para que pueda ver primero los procesos más ocupados. top continuará ejecutándose en su shell hasta que lo detenga usando la combinación de teclas estándar Ctrl+C para salir de un proceso en ejecución. Esto envía una kill señal, indicando al proceso que se detenga correctamente si es posible.

Una versión mejorada de top, llamada htop, está disponible en la mayoría de los repositorios de paquetes. En Ubuntu, puedes instalarlo con apt:

``` 
sudo apt install htop
``` 

Después de eso, el htopcomando estará disponible:

``` 
htop
Output
  Mem[|||||||||||           49/995MB]     Load average: 0.00 0.03 0.05 
  CPU[                          0.0%]     Tasks: 21, 3 thr; 1 running
  Swp[                         0/0MB]     Uptime: 00:58:11

  PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
 1259 root       20   0 25660  1880  1368 R  0.0  0.2  0:00.06 htop
    1 root       20   0 24188  2120  1300 S  0.0  0.2  0:00.56 /sbin/init
  311 root       20   0 17224   636   440 S  0.0  0.1  0:00.07 upstart-udev-brid
  314 root       20   0 21592  1280   760 S  0.0  0.1  0:00.06 /sbin/udevd --dae
  389 messagebu  20   0 23808   688   444 S  0.0  0.1  0:00.01 dbus-daemon --sys
  407 syslog     20   0  243M  1404  1080 S  0.0  0.1  0:00.02 rsyslogd -c5
  408 syslog     20   0  243M  1404  1080 S  0.0  0.1  0:00.00 rsyslogd -c5
  409 syslog     20   0  243M  1404  1080 S  0.0  0.1  0:00.00 rsyslogd -c5
  406 syslog     20   0  243M  1404  1080 S  0.0  0.1  0:00.04 rsyslogd -c5
  553 root       20   0 15180   400   204 S  0.0  0.0  0:00.01 upstart-socket-br
``` 

htop proporciona una mejor visualización de múltiples subprocesos de CPU, un mejor conocimiento de la compatibilidad con el color en terminales modernos y más opciones de clasificación, entre otras características. A diferencia de top, no siempre se instala de forma predeterminada, pero puede considerarse un reemplazo directo. Puede salir htop presionando Ctrl+C como con top. 

En la siguiente sección, aprenderá cómo utilizar herramientas para consultar procesos específicos.


## 2. Cómo utilizar ps para enumerar procesos

top y htop proporciona una interfaz de panel para ver los procesos en ejecución similar a un administrador de tareas gráfico. Una interfaz de panel puede proporcionar una descripción general, pero generalmente no devuelve resultados procesables directamente. Para ello, Linux proporciona otro comando estándar llamado ps para consultar los procesos en ejecución.

Ejecutar ps sin argumentos proporciona muy poca información:

``` 
ps
Output
  PID TTY          TIME CMD
 1017 pts/0    00:00:00 bash
 1262 pts/0    00:00:00 ps
``` 

Este resultado muestra todos los procesos asociados con el usuario actual y la sesión del terminal. Esto tiene sentido si actualmente solo está ejecutando el bash shell y este ps comando dentro de esta terminal.

Para obtener una imagen más completa de los procesos en este sistema, puede ejecutar ps aux:

``` 
ps aux
Output
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2  24188  2120 ?        Ss   14:28   0:00 /sbin/init
root         2  0.0  0.0      0     0 ?        S    14:28   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    14:28   0:00 [ksoftirqd/0]
root         6  0.0  0.0      0     0 ?        S    14:28   0:00 [migration/0]
root         7  0.0  0.0      0     0 ?        S    14:28   0:00 [watchdog/0]
root         8  0.0  0.0      0     0 ?        S<   14:28   0:00 [cpuset]
root         9  0.0  0.0      0     0 ?        S<   14:28   0:00 [khelper]
…
``` 

Estas opciones indican ps que se muestren los procesos propiedad de todos los usuarios (independientemente de su asociación de terminal) en un formato más legible para los humanos.

Al utilizar tuberías, puede buscar dentro de la salida de ps aux usando grep para devolver el nombre de un proceso específico. Esto es útil si cree que se ha bloqueado o si necesita detenerlo por algún motivo.

``` 
ps aux | grep bash
Output
sammy         41664   0.7  0.0 34162880   2528 s000  S     1:35pm   0:00.04 -bash
sammy         41748   0.0  0.0 34122844    828 s000  S+    1:35pm   0:00.00 grep bash
``` 

Esto devuelve tanto el grep proceso que acaba de ejecutar como el bash shell que se está ejecutando actualmente. También devuelve su uso total de memoria y CPU, cuánto tiempo han estado ejecutándose y, en el resultado resaltado arriba, su ID de proceso. En sistemas Linux y similares a Unix, a cada proceso se le asigna un ID de proceso o PID. Así es como el sistema operativo identifica y realiza un seguimiento de los procesos.

Una forma rápida de obtener el PID de un proceso es con el pgrep comando:

``` 
pgrep bash
Output
1017
``` 

El primer proceso generado en el arranque, llamado init, recibe el PID de "1".

``` 
pgrep init
Output
1
``` 

Este proceso es entonces responsable de generar todos los demás procesos del sistema. Los procesos posteriores reciben números PID más grandes.

El padre de un proceso es el proceso responsable de generarlo. Los procesos principales tienen un PPID , que puede ver en los encabezados de las columnas en muchas aplicaciones de gestión de procesos, top incluidas htop y ps.

Cualquier comunicación entre el usuario y el sistema operativo sobre procesos implica la traducción entre nombres de procesos y PID en algún momento durante la operación. Es por eso que estas utilidades siempre incluirán el PID en su salida. En la siguiente sección, aprenderá cómo usar PID para enviar señales de parada, reanudación u otras señales a procesos en ejecución.


## 3. Cómo enviar señales de procesos en Linux

Todos los procesos en Linux responden a señales. Las señales son una forma a nivel del sistema operativo de indicar a los programas que finalicen o modifiquen su comportamiento.

La forma más común de pasar señales a un programa es mediante el kill comando. Como es de esperar, la funcionalidad predeterminada de esta utilidad es intentar finalizar un proceso:

``` 
kill PID_of_target_process
```

Esto envía la señal TERM al proceso. La señal TERM le indica al proceso que finalice. Esto permite que el programa realice operaciones de limpieza y salga sin problemas.

Si el programa se comporta mal y no sale cuando se le da la señal TERM, puede escalar la señal pasando la KILLseñal:

``` 
kill -KILL PID_of_target_process
``` 

Esta es una señal especial que no se envía al programa.

En cambio, se entrega al kernel del sistema operativo, que cierra el proceso. Esto se utiliza para evitar programas que ignoran las señales que se les envían.

Cada señal tiene un número asociado que se puede pasar en lugar del nombre. Por ejemplo, puede pasar "-15" en lugar de "-TERM" y "-9" en lugar de "-KILL".

Las señales no sólo se utilizan para cerrar programas. También se pueden utilizar para realizar otras acciones.

Por ejemplo, muchos procesos que están diseñados para ejecutarse constantemente en segundo plano (a veces llamados "demonios") se reiniciarán automáticamente cuando se les dé la señal de colgar HUP o colgar. El servidor web Apache normalmente funciona de esta manera.

``` 
sudo kill -HUP pid_of_apache
``` 

El comando anterior hará que Apache recargue su archivo de configuración y reanude la entrega de contenido.

Puede enumerar todas las señales que se pueden enviar con kill la -l bandera:

``` 
kill -l
Output
1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
``` 

Aunque la forma convencional de enviar señales es mediante el uso de PID, también existen métodos para hacerlo con nombres de procesos regulares.

El pkill comando funciona casi exactamente de la misma manera que kill, pero en su lugar opera con un nombre de proceso:

``` 
pkill -9 ping
``` 

El comando anterior es el equivalente a:

``` 
kill -9 `pgrep ping`
``` 

Si desea enviar una señal a cada instancia de un determinado proceso, puede utilizar el kill all comando:

``` 
killall firefox
``` 

El comando anterior enviará la señal TERM a cada instancia de firefox ejecución en la computadora.


## 4. Cómo ajustar las prioridades del proceso

A menudo, querrás ajustar a qué procesos se les da prioridad en un entorno de servidor.

Algunos procesos pueden considerarse críticos para su situación, mientras que otros pueden ejecutarse siempre que queden recursos sobrantes.

Linux controla la prioridad a través de un valor llamado amabilidad.

Las tareas de alta prioridad se consideran menos agradables porque tampoco comparten recursos. Los procesos de baja prioridad, por otro lado, son buenos porque insisten en utilizar sólo recursos mínimos.

Cuando corriste top al principio del artículo, había una columna marcada "NI". Este es el gran valor del proceso:

``` 
top
[secondary_label Output] 
Tasks:  56 total,   1 running,  55 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.3%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1019600k total,   324496k used,   695104k free,     8512k buffers
Swap:        0k total,        0k used,        0k free,   264812k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND           
 1635 root      20   0 17300 1200  920 R  0.3  0.1   0:00.01 top                
    1 root      20   0 24188 2120 1300 S  0.0  0.2   0:00.56 init               
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd           
    3 root      20   0     0    0    0 S  0.0  0.0   0:00.11 ksoftirqd/0
``` 

Los valores de Nice pueden oscilar entre -19/-20 (prioridad más alta) y 19/20 (prioridad más baja) dependiendo del sistema.

Para ejecutar un programa con un cierto valor agradable, puede usar el nice comando:

``` 
nice -n 15 command_to_execute
``` 

Esto sólo funciona al comenzar un nuevo programa.

Para alterar el valor agradable de un programa que ya se está ejecutando, se utiliza una herramienta llamada renice:

``` 
renice 0 PID_to_prioritize
``` 

