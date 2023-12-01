# Cómo utilizar Top, Netstat, Du y otras herramientas para monitorear los recursos del servidor

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

Después de eso, el htop comando estará disponible:

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

A continuación se muestran algunos atajos de teclado que le ayudarán a utilizar htop de forma más eficaz:

* **M:** ordenar procesos por uso de memoria
* **P:** ordenar procesos por uso del procesador
* **?:** Acceder a la ayuda
* **k:** Mata el proceso actual/etiquetado
* **F2:** Configurar htop. Puede elegir opciones de visualización aquí.
* **/::** Procesos de búsqueda
  
Hay muchas otras opciones a las que puede acceder mediante ayuda o configuración. Estas deberían ser sus primeras paradas para explorar la funcionalidad de htop. En el siguiente paso, aprenderá cómo monitorear el ancho de banda de su red.


## 2. Cómo monitorear el ancho de banda de su red

Si su conexión de red parece sobreutilizada y no está seguro de qué aplicación es la culpable, un programa llamado nethogs es una buena opción para averiguarlo.

En Ubuntu, puedes instalar nethogs con el siguiente comando:

``` 
sudo apt install nethogs
``` 

Después de eso, el nethogs comando estará disponible:

``` 
nethogs
Output
NetHogs version 0.8.0

  PID USER     PROGRAM                      DEV        SENT      RECEIVED       
3379  root     /usr/sbin/sshd               eth0       0.485       0.182 KB/sec
820   root     sshd: root@pts/0             eth0       0.427       0.052 KB/sec
?     root     unknown TCP                             0.000       0.000 KB/sec

  TOTAL                                                0.912       0.233 KB/sec
``` 

nethogs asocia cada aplicación con su tráfico de red.

Sólo hay unos pocos comandos que puedes usar para controlar nethogs:

* **M:** Cambia las visualizaciones entre “kb/s”, “kb”, “b” y “mb”.
* **R:** Ordenar por tráfico recibido.
* **S:** Ordenar por tráfico enviado.
* **P:** abandonar
  
iptraf-ng es otra forma de monitorear el tráfico de la red. Proporciona varias interfaces de monitoreo interactivas diferentes.

En Ubuntu, puedes instalar iptraf-ng con el siguiente comando:

``` 
sudo apt install iptraf-ng
``` 

iptraf-ng debe ejecutarse con privilegios de root, por lo que debe precederlo con sudo:

``` 
sudo iptraf-ng
``` 

Se le presentará un menú que utiliza un popular marco de interfaz de línea de comandos llamado ncurses.

Con este menú, puede seleccionar a qué interfaz desea acceder.

Por ejemplo, para obtener una descripción general de todo el tráfico de la red, puede seleccionar el primer menú y luego "Todas las interfaces".

Aquí puede ver qué direcciones IP está comunicando en todas sus interfaces de red.

Si desea que esas direcciones IP se resuelvan en dominios, puede habilitar la búsqueda DNS inversa saliendo de la pantalla de tráfico, seleccionando Configure y luego activando Reverse DNS lookups.

También puede habilitar TCP/UDP service names la visualización de los nombres de los servicios que se ejecutan en lugar de los números de puerto.

El **netstat** comando es otra herramienta versátil para recopilar información de la red.

netstat se instala de forma predeterminada en la mayoría de los sistemas modernos, pero puede instalarlo usted mismo descargándolo desde los repositorios de paquetes predeterminados de su servidor. En la mayoría de los sistemas Linux, incluido Ubuntu, el paquete que contiene netstat es net-tools:

``` 
sudo apt install net-tools
``` 

De forma predeterminada, el netstat comando por sí solo imprime una lista de sockets abiertos:

```
netstat
Output
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 192.241.187.204:ssh     ip223.hichina.com:50324 ESTABLISHED
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  5      [ ]         DGRAM                    6559     /dev/log
unix  3      [ ]         STREAM     CONNECTED     9386     
unix  3      [ ]         STREAM     CONNECTED     9385     
. . .
``` 

Si agrega una -a opción, enumerará todos los puertos, de escucha y de no escucha:

``` 
netstat -a
Output
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 *:ssh                   *:*                     LISTEN     
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     6195     @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     7762     /var/run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     6503     /var/run/dbus/system_bus_socket
. . .
``` 

Si desea filtrar para ver solo conexiones TCP o UDP, use los indicadores -t o respectivamente:-u

``` 
netstat -at
Output
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 *:ssh                   *:*                     LISTEN     
tcp        0      0 192.241.187.204:ssh     rrcs-72-43-115-18:50615 ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
``` 

Vea las estadísticas pasando el indicador “-s”:

``` 
netstat -s
Output
Ip:
    13500 total packets received
    0 forwarded
    0 incoming packets discarded
    13500 incoming packets delivered
    3078 requests sent out
    16 dropped because of missing route
Icmp:
    41 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        echo requests: 1
        echo replies: 40
. . .
``` 

Si desea actualizar continuamente la salida, puede utilizar la -c bandera. Hay muchas otras opciones disponibles para netstat que aprenderá revisando su página de manual.

En el siguiente paso, aprenderá algunas formas útiles de monitorear el uso de su disco.


## 3. Cómo monitorear el uso de su disco

Para obtener una descripción general rápida de cuánto espacio en disco queda en las unidades conectadas, puede utilizar el df programa.

Sin ninguna opción, su salida se ve así:

``` 
df
Output
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/vda        31383196 1228936  28581396   5% /
udev              505152       4    505148   1% /dev
tmpfs             203920     204    203716   1% /run
none                5120       0      5120   0% /run/lock
none              509800       0    509800   0% /run/shm
``` 

Esto genera el uso del disco en bytes, lo que puede ser un poco difícil de leer.

Para solucionar este problema, puede especificar la salida en un formato legible por humanos:

``` 
df -h
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda         30G  1.2G   28G   5% /
udev            494M  4.0K  494M   1% /dev
tmpfs           200M  204K  199M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            498M     0  498M   0% /run/shm
``` 

Si desea ver el espacio total en disco disponible en todos los sistemas de archivos, puede pasar la --total opción. Esto agregará una fila en la parte inferior con información resumida:

``` 
df -h --total
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda         30G  1.2G   28G   5% /
udev            494M  4.0K  494M   1% /dev
tmpfs           200M  204K  199M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            498M     0  498M   0% /run/shm
total            32G  1.2G   29G   4%
```

df puede proporcionar una visión general útil. Otro comando du proporciona un desglose por directorio.

**du** analizará el uso del directorio actual y de cualquier subdirectorio. El resultado predeterminado de duejecutar en un directorio de inicio casi vacío se ve así:

``` 
du
Output
4    ./.cache
8    ./.ssh
28    .
``` 

Una vez más, puede especificar una salida legible por humanos pasándola -h:

``` 
du -h
Output
4.0K    ./.cache
8.0K    ./.ssh
28K    .
``` 

Para ver los tamaños de archivos y los directorios, escriba lo siguiente:

``` 
du -a
Output
0    ./.cache/motd.legal-displayed
4    ./.cache
4    ./.ssh/authorized_keys
8    ./.ssh
4    ./.profile
4    ./.bashrc
4    ./.bash_history
28    .
``` 

Para obtener un total en la parte inferior, puede agregar la -c opción:

``` 
du -c
Output
4    ./.cache
8    ./.ssh
28    .
28    total
```

Si sólo te interesa el total y no los detalles, puedes emitir:

``` 
du -s
Output
28    .
``` 

También hay una ncurses interfaz para du, apropiadamente llamada ncdu, que puedes instalar:

``` 
sudo apt install ncdu
``` 

Esto representará gráficamente el uso de su disco:

``` 
ncdu
Output
--- /root ----------------------------------------------------------------------
    8.0KiB [##########] /.ssh                                                   
    4.0KiB [#####     ] /.cache
    4.0KiB [#####     ]  .bashrc
    4.0KiB [#####     ]  .profile
    4.0KiB [#####     ]  .bash_history
``` 

Puede recorrer el sistema de archivos usando las flechas hacia arriba y hacia abajo y presionando Enter en cualquier entrada del directorio.

En la última sección, aprenderá cómo monitorear el uso de su memoria.


## 4. Cómo controlar el uso de su memoria

Puede verificar el uso actual de la memoria en su sistema usando el free comando.

Cuando se usa sin opciones, el resultado se ve así:

``` 
free
Output
              total        used        free      shared  buff/cache   available
Mem:        1004896      390988      123484        3124      490424      313744
Swap:             0           0           0
``` 

Para mostrar en un formato más legible, puede pasar la -m opción para mostrar la salida en megabytes:

``` 
free -m
Output
              total        used        free      shared  buff/cache   available
Mem:            981         382         120           3         478         306
Swap:             0           0           0
``` 

La Mem línea incluye la memoria utilizada para el almacenamiento en búfer y en caché, que se libera tan pronto como se necesita para otros fines. Swap es la memoria que se ha escrito en un swapfile disco para conservar la memoria activa.

Finalmente, el vmstat comando puede generar diversa información sobre su sistema, incluida la memoria, el intercambio, el disco io y la información de la CPU.

Puede usar el comando para obtener otra vista del uso de la memoria:

``` 
vmstat
Output
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0  99340 123712 248296    0    0     0     1    9    3  0  0 100  0
``` 

Puedes ver esto en megabytes especificando unidades con la -S bandera:

``` 
vmstat -S M
Output
procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
 1  0      0     96    120    242    0    0     0     1    9    3  0  0 100  0
``` 

Para obtener algunas estadísticas generales sobre el uso de la memoria, escriba:

``` 
vmstat -s -S M
Output
          495 M total memory
          398 M used memory
          252 M active memory
          119 M inactive memory
           96 M free memory
          120 M buffer memory
          242 M swap cache
            0 M total swap
            0 M used swap
            0 M free swap
. . .
``` 

Para obtener información sobre el uso de caché de los procesos individuales del sistema, escriba:

``` 
vmstat -m -S M
Output
Cache                       Num  Total   Size  Pages
ext4_groupinfo_4k           195    195    104     39
UDPLITEv6                     0      0    768     10
UDPv6                        10     10    768     10
tw_sock_TCPv6                 0      0    256     16
TCPv6                        11     11   1408     11
kcopyd_job                    0      0   2344     13
dm_uevent                     0      0   2464     13
bsg_cmd                       0      0    288     14
. . .
``` 

Esto le dará detalles sobre qué tipo de información se almacena en el caché.

