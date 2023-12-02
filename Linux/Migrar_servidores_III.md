# Cómo migrar servidores Linux, parte 3: pasos finales

## 1. Migrar usuarios y grupos

Los administradores de paquetes de Linux son muy potentes y reproducibles, y al migrar los paquetes de su sistema en el tutorial anterior, habrá migrado la mayoría de los ajustes de configuración necesarios. Sin embargo, esto omite algunas de las configuraciones que puede haber cambiado manualmente en su servidor anterior, como los permisos de usuario y grupo. Estos también deberán migrarse o recrearse.

Afortunadamente, todas las configuraciones de usuarios y grupos de Linux están contenidas en unos pocos archivos. Estos archivos incluyen:

* /etc/passwd : este archivo define los usuarios y sus atributos. A pesar de su nombre, este archivo no contiene información de contraseña. Incluye nombre de usuario, números de usuario y de grupo principal, directorios de inicio y shells predeterminados.

* /etc/shadow : este archivo contiene la configuración de contraseña real para cada usuario. Debe contener una línea para cada uno de los usuarios definidos en el passwdarchivo, junto con un hash de su contraseña y cierta información sobre las políticas de contraseña.

* /etc/group : este archivo define cada grupo disponible en su sistema. Esto incluye el nombre del grupo y el número de grupo asociado, junto con cualquier membresía del grupo.

* /etc/gshadow : este archivo contiene una línea para cada grupo del sistema. Enumera el nombre de un grupo, una contraseña que pueden utilizar quienes no son miembros del grupo para acceder al grupo, una lista de administradores y otros miembros.

Nunca debes copiar estos archivos directamente de un sistema en vivo a otro. Los números de usuario y grupo se incrementan automáticamente cuando se crean en cada sistema y crearán conflictos si no coinciden. En su lugar, puede migrarlos de forma selectiva usando awk, como en el tutorial anterior.

### 1.1 Crear archivos de migración

Creará un nuevo archivo de migración asociado con cada uno de los archivos anteriores. Esto le permitirá migrarlos todos sistemáticamente, empezando por /etc/passwd.

Primero, deberá establecer si los ID de usuario habituales comienzan a contar desde 500 o desde 1000 en su sistema fuente. La mayoría de los entornos Linux modernos comienzan a contar desde 1000 para reservar más espacio para los usuarios del sistema, pero si está migrando desde un sistema muy antiguo, puede contar desde 500. Para verificar, puede imprimir las últimas líneas de su /etc/passwd archivo para ver cuál es su propio El número de cuenta de usuario es:

``` 
less /etc/passwd
Output
…
vault:x:997:997::/home/vault:/bin/bash
stunnel4:x:112:119::/var/run/stunnel4:/usr/sbin/nologin
sammy:x:1001:1002::/home/sammy:/bin/sh
``` 

En este caso, sería 1000, ya que sus ID de usuario habituales, en la tercera columna del resultado, parecen ser 1000 o más. No exportaremos usuarios o grupos por debajo de este límite. También excluirá la nobody cuenta a la que se le asigna automáticamente un ID de 65534.

Con awk, puede crear un archivo de sincronización para su /etc/passwd archivo. Los awk comandos de este tutorial se proporcionarán tal cual, debido a su sintaxis compleja.

```
awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534)' /etc/passwd > ~/passwd.sync
``` 

A continuación, puede utilizar la misma sintaxis y el mismo límite de ID de usuario para exportar su /etc/group archivo:

``` 
awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534)' /etc/group > ~/group.sync
``` 

Para analizar el archivo /etc/shadow, puede utilizar los datos de su /etc/passwd archivo como entrada:

``` 
awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=35534) {print $1}' /etc/passwd | tee - | egrep -f - /etc/shadow > ~/shadow.sync
```

El mismo enfoque funciona para /etc/gshadow:

``` 
awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534) {print $1}' /etc/group | tee - | egrep -f - /etc/gshadow > ~/gshadow.sync
```

Después de haber probado estos comandos y haber verificado que crean archivos de exportación a partir de datos reales, puede agregarlos al sync.sh script que mantuvo desde el último tutorial. Puede ejecutar cada uno de estos comandos de forma remota, es decir, como parte del script que se ejecuta en su máquina de destino, obteniendo la salida de la máquina de origen original, precediéndolos con y entrecomillando la del comando.ssh source_server awk

``` 
ssh source_server "awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534)' /etc/passwd > ~/passwd.sync"
ssh source_server "awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534)' /etc/group > ~/group.sync"
ssh source_server "awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=35534) {print $1}' /etc/passwd | tee - | egrep -f - /etc/shadow > ~/shadow.sync"
ssh source_server "awk -v LIMIT=1000 -F: '($3>=LIMIT) && ($3!=65534) {print $1}' /etc/group | tee - | egrep -f - /etc/gshadow > ~/gshadow.sync"
rsync source_server:~/passwd.sync ~/
rsync source_server:~/group.sync ~/
rsync source_server:~/shadow.sync ~/
rsync source_server:~/gshadow.sync ~/
```

Después de exportar estos datos a su máquina de destino, puede agregar automáticamente sus usuarios y grupos a la máquina de destino. Sin embargo, a diferencia de los otros comandos, este creará duplicados si se vuelve a ejecutar en el mismo entorno, por lo que debe realizarlo manualmente en lugar de agregarlo a su secuencia de comandos de migración.

Hay un comando llamado newusers que puede agregar varios usuarios desde un archivo de entrada. Sin embargo, primero querrás usar otro awk comando para eliminar los ID numéricos de tu archivo de sincronización:

``` 
awk 'BEGIN { OFS=FS=":"; } {$3=""; $4=""; } { print; }' ~/passwd.sync > ~/passwd.sync.mod
``` 

Luego puedes pasar ese archivo a newusers:

``` 
newusers ~/passwd.sync.mod
``` 

Esto agregará todos los usuarios del archivo al /etc/passwd archivo local. También creará los grupos de usuarios asociados automáticamente. Tendrá que agregar manualmente al /etc/group archivo grupos adicionales que no estén asociados con un usuario. Utilice sus archivos de sincronización como punto de referencia para editar los archivos de destino correspondientes.

Para el /etc/shadow archivo, puede copiar la segunda columna de su shadow.sync archivo a la segunda columna de la cuenta asociada en el nuevo sistema. Esto transferirá las contraseñas de sus cuentas al nuevo sistema. También puedes programar estos cambios, dependiendo de cuántas cuentas necesites transferir.


## 2. Transferir trabajos al nuevo sistema

Ahora que sus usuarios, paquetes y otros datos se transfieren desde el sistema anterior, hay un paso más: transferir el correo y los trabajos del sistema de cada uno de sus usuarios.

Puede comenzar este proceso escribiendo otro comando rsync para el directorio de spool. El spool directorio normalmente contiene cron, mail y algunos otros registros:

``` 
ls /var/spool
Output
anacron   cron   mail   plymouth   rsyslog
``` 

Para transferir el directorio de correo a nuestro servidor de destino, puede agregar otro rsync comando a su secuencia de comandos de migración:

``` 
rsync -azvP --progress source_server:/var/spool/mail/* /var/spool/mail/
``` 

Otro directorio dentro del /var/spool directorio al que debes prestar atención es el cron directorio. Este directorio contiene trabajos cron, que se utilizan para programar tareas. El crontabs subdirectorio contiene la configuración de cada usuario individual cron.

Transfiera su crontabs uso rsync:

``` 
~/sincronización.sh
rsync -azvP --progress source_server:/var/spool/cron/crontabs/* /var/spool/cron/crontabs/*
``` 

Esto manejará cron las configuraciones de los usuarios individuales. Sin embargo, no captura ninguna cron configuración de todo el sistema. Dentro del /etc directorio, hay un crontab para todo el sistema y varios otros directorios que contienen cron configuraciones.

``` 
ls /etc/cron*
Output
cron.d
cron.daily
cron.hourly
cron.monthly
crontab
cron.weekly
``` 

El crontab archivo contiene cron detalles de todo el sistema. Los otros elementos son directorios que contienen otra información cron. Míralos y decide si contienen alguna información que necesites.

Una vez más, utilice rsync para transferir la información cron relevante al nuevo sistema:

``` 
rsync -azvP --progress source_server:/etc/crontab /etc/crontab
``` 

Mientras investiga las cron configuraciones en /etc, asegúrese de no haber pasado por alto ningún otro archivo de configuración. Por ejemplo, el servidor web Nginx almacena su configuración en /etc/nginx y debe asegurarse de que su secuencia de comandos de migración la haya capturado.

Una vez que tenga su información cron en su nuevo sistema, debe verificar que funcione. La única forma de hacer esto correctamente es iniciar sesión como cada usuario individual y ejecutar los comandos en el crontab de cada usuario manualmente. Esto garantizará que no haya problemas de permisos ni rutas de archivos faltantes que impidan que estos comandos fallen silenciosamente cuando se ejecuten automáticamente.


## 3. sitios y servicios de prueba

En este punto, debería haber terminado de agregar comandos a su secuencia de comandos de migración y transferir datos. El siguiente paso es comenzar a reiniciar todos los servicios relevantes en el nuevo servidor. Por ejemplo, puedes reiniciar el nginx servidor web ejecutando sudo systemctl restart nginx, aunque esto se habrá hecho automáticamente cuando instalaste el paquete Nginx en el nuevo servidor. 

Para cualquier otro servicio, para el cual haya escrito sus propios archivos de unidad o implementado a través de Docker, debe intentar reiniciarlos manualmente. También debe reiniciar su servidor al menos una vez para asegurarse de que estos servicios puedan reanudarse correctamente después de cualquier tiempo de inactividad. Preste atención a los archivos de registro asociados mientras realiza la prueba para ver si surge algún problema.

También puede realizar otras comprobaciones aleatorias. Por ejemplo, si tiene un /data directorio que ha transferido rsync, puede navegar a ese directorio tanto en la computadora de origen como en la de destino y ejecutar el du comando para verificar su tamaño:

``` 
cd /data
du -hs
Output
471M	.
``` 

Si existe una disparidad entre sus dos sistemas, debe investigar.

A continuación, puede verificar los procesos que se están ejecutando en cada máquina. Puede utilizar top para obtener una descripción general de los procesos activos:

``` 
top
Output
top - 21:20:33 up 182 days, 22:04,  1 user,  load average: 0.00, 0.01, 0.00
Tasks: 124 total,   3 running, 121 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  1.0 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :    981.3 total,     82.8 free,    517.8 used,    380.7 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    182.0 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
     11 root      20   0       0      0      0 I   0.3   0.0  29:45.20 rcu_sched
  99465 root      20   0  685508  27396   5372 S   0.3   2.7 161:41.83 node /root/hell
 104207 vault     20   0  837416 236528 128012 S   0.3  23.5 134:53.49 vault
 175635 root      20   0   11000   3824   3176 R   0.3   0.4   0:00.03 top
      1 root      20   0  170636   9116   4200 S   0.0   0.9   8:50.40 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:01.04 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
    . . .
``` 

También puede replicar algunas de las comprobaciones que realizó inicialmente en la máquina de origen para ver si ha reproducido correctamente su entorno en la nueva máquina. Puedes volver a ejecutar netstat las -nlp banderas para obtener una descripción general:

``` 
netstat -nlp
Output
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8200            0.0.0.0:*               LISTEN      104207/vault
tcp        0      0 0.0.0.0:1935            0.0.0.0:*               LISTEN      3691671/nginx: mast
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      3691671/nginx: mast
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      3691671/nginx: mast
tcp        0      0 0.0.0.0:1936            0.0.0.0:*               LISTEN      197885/stunnel4
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      162540/systemd-reso
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      129518/sshd: /usr/s
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      99465/node /root/he
tcp        0      0 0.0.0.0:8088            0.0.0.0:*               LISTEN      3691671/nginx: mast
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      3691671/nginx: mast
tcp        0      0 0.0.0.0:56733           0.0.0.0:*               LISTEN      170269/docker-proxy
tcp6       0      0 :::80                   :::*                    LISTEN      3691671/nginx: mast
tcp6       0      0 :::22                   :::*                    LISTEN      129518/sshd: /usr/s
tcp6       0      0 :::443                  :::*                    LISTEN      3691671/nginx: mast
tcp6       0      0 :::56733                :::*                    LISTEN      170275/docker-proxy
udp        0      0 127.0.0.53:53           0.0.0.0:*                           162540/systemd-reso
raw6       0      0 :::58                   :::*                    7           162524/systemd-netw
raw6       0      0 :::58                   :::*                    7           162524/systemd-netw
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path
unix  2      [ ACC ]     STREAM     LISTENING     5313074  1/systemd            /run/systemd/userdb/io.systemd.DynamicUser
unix  2      [ ACC ]     SEQPACKET  LISTENING     12985    1/systemd            /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     12967    1/systemd            /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     12980    1/systemd            /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     16037236 95187/systemd        /run/user/0/systemd/private
…
``` 

También puedes volver a ejecutar lsof:

``` 
lsof
Output
COMMAND       PID            USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
node\x20/   99465            root   20u  IPv4 16046039      0t0  TCP 127.0.0.1:3000 (LISTEN)
vault      104207           vault    8u  IPv4  1134285      0t0  TCP *:8200 (LISTEN)
sshd       129518            root    3u  IPv4  1397496      0t0  TCP *:22 (LISTEN)
sshd       129518            root    4u  IPv6  1397507      0t0  TCP *:22 (LISTEN)
systemd-r  162540 systemd-resolve   12u  IPv4  5313507      0t0  UDP 127.0.0.53:53
systemd-r  162540 systemd-resolve   13u  IPv4  5313508      0t0  TCP 127.0.0.53:53 (LISTEN)
docker-pr  170269            root    4u  IPv4  1700561      0t0  TCP *:56733 (LISTEN)
docker-pr  170275            root    4u  IPv6  1700573      0t0  TCP *:56733 (LISTEN)
stunnel4   197885        stunnel4    9u  IPv4  1917328      0t0  TCP *:1936 (LISTEN)
sshd      3469804            root    4u  IPv4 22246413      0t0  TCP 159.203.102.125:22->154.5.29.188:36756 (ESTABLISHED)
nginx     3691671            root    7u  IPv4  2579911      0t0  TCP *:8080 (LISTEN)
nginx     3691671            root    8u  IPv4  1921506      0t0  TCP *:80 (LISTEN)
nginx     3691671            root    9u  IPv6  1921507      0t0  TCP *:80 (LISTEN)
nginx     3691671            root   10u  IPv6  1921508      0t0  TCP *:443 (LISTEN)
nginx     3691671            root   11u  IPv4  1921509      0t0  TCP *:443 (LISTEN)
nginx     3691671            root   12u  IPv4  2579912      0t0  TCP *:8088 (LISTEN)
nginx     3691671            root   13u  IPv4  2579913      0t0  TCP *:1935 (LISTEN)
nginx     3691674        www-data    7u  IPv4  2579911      0t0  TCP *:8080 (LISTEN)
nginx     3691674        www-data    8u  IPv4  1921506      0t0  TCP *:80 (LISTEN)
nginx     3691674        www-data    9u  IPv6  1921507      0t0  TCP *:80 (LISTEN)
nginx     3691674        www-data   10u  IPv6  1921508      0t0  TCP *:443 (LISTEN)
nginx     3691674        www-data   11u  IPv4  1921509      0t0  TCP *:443 (LISTEN)
nginx     3691674        www-data   12u  IPv4  2579912      0t0  TCP *:8088 (LISTEN)
nginx     3691674        www-data   13u  IPv4  2579913      0t0  TCP *:1935 (LISTEN)
``` 

Si transfirió un servidor web o aplicaciones web, también debe probar sus sitios en el nuevo servidor. Dependiendo de su configuración, es posible que deba migrar su nombre de dominio y volver a registrar los certificados HTTPS antes de poder hacerlo. Si su nuevo servidor está detrás de una VPN u otra capa de ingreso, es posible que pueda probarlo detrás de una URL diferente antes de realizar una transición de producción completa.

También querrás migrar las reglas de tu firewall, que normalmente están contenidas en /etc/sysconfig/iptables y /etc/sysconfig/ip6tables.

Antes de cargar las reglas en su nuevo servidor, debe revisarlas para detectar cualquier cosa que deba actualizarse, como cambios de direcciones o rangos de IP.


## 4. Cambiar la configuración de DNS

Una vez que tenga todos los datos más recientes en su servidor de destino y haya probado sus puntos finales web, puede modificar los servidores DNS de su dominio para que apunten a su nuevo servidor. Asegúrese de que cada referencia a la IP del servidor anterior se reemplace con la información del nuevo servidor.

Los cambios de DNS suelen tardar entre unos minutos y una hora en propagarse a la mayoría de los ISP de Internet domésticos. Después de que su DNS se haya actualizado para reflejar sus cambios, es posible que deba ejecutar el script de migración por última vez para asegurarse de que se transfieran todas las solicitudes perdidas que aún iban a su servidor original.


## 5. Conclusión

Su nuevo servidor ahora debería estar en funcionamiento, aceptando solicitudes y manejando todos los datos que estaban en su servidor anterior. Debe continuar monitoreando de cerca el nuevo servidor para detectar cualquier anomalía.

Las migraciones no son triviales. La mejor posibilidad de migrar exitosamente un servidor en vivo es comprender su sistema lo mejor que pueda antes de comenzar. Cada sistema es diferente y cada vez tendrás que solucionar nuevos problemas. No intente migrar si no tiene tiempo para solucionar los problemas que puedan surgir.

