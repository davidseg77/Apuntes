# Cómo migrar servidores Linux, parte 1: preparación del sistema

## 1. Introducción

Hay muchos escenarios en los que es posible que deba trasladar sus datos y requisitos operativos de un servidor a otro. Es posible que necesite implementar sus soluciones en un nuevo centro de datos, actualizar a una máquina más grande o realizar la transición a un nuevo hardware o un nuevo proveedor de VPS.

Cualesquiera que sean sus motivos, hay muchas consideraciones diferentes que debe tener en cuenta al migrar de un sistema a otro. Obtener configuraciones funcionalmente equivalentes puede resultar difícil si no se trabaja con una solución de gestión de configuración como Chef, Puppet o Ansible. No solo necesita transferir datos, sino también configurar sus servicios para que funcionen de la misma manera en una máquina nueva.


## 2. Hacer copias de seguridad

El primer paso a seguir al realizar cualquier acción potencialmente destructiva es crear copias de seguridad nuevas. No querrás quedarte en una situación en la que un comando rompa algo en tu máquina de producción actual antes de que el reemplazo esté en funcionamiento.

Hay varias formas diferentes de realizar una copia de seguridad de su servidor. Su selección dependerá de qué opciones tienen sentido para su escenario y con qué se siente más cómodo.

Si tiene acceso al hardware físico y a un espacio para realizar la copia de seguridad (unidad de disco, USB, etc.), puede clonar el disco utilizando cualquiera de las muchas soluciones de copia de seguridad de imágenes disponibles. Un equivalente funcional cuando se trata de servidores en la nube es tomar una instantánea o imagen desde la interfaz del panel de control.

Una vez que haya completado las copias de seguridad, estará listo para continuar. Durante el resto de esta guía, necesitarás ejecutar muchos de los comandos como root o usando sudo.


## 3. Recopilar información sobre el sistema fuente

Antes de comenzar una migración, debe configurar su sistema de destino para que coincida con su sistema de origen.

Querrá hacer coincidir todo lo que pueda entre el servidor actual y aquel al que planea migrar. Es posible que desee actualizar su servidor actual antes de migrar a un sistema de destino más nuevo y realizar otro conjunto de copias de seguridad después. Lo importante es que coincidan lo más posible al iniciar la migración real.

La mayor parte de la información que le ayudará a decidir qué sistema de servidor crear para la nueva máquina se puede recuperar con el uname comando:

``` 
uname -r
Output
5.4.0-26-generic
``` 

Esta es la versión del kernel que ejecuta su sistema actual. Para que todo funcione sin problemas, es una buena idea intentar hacer coincidir eso en el sistema de destino.

También deberías intentar hacer coincidir la distribución y la versión de tu servidor de origen. Si no conoce la versión de la distribución que tiene instalada en la máquina de origen, puede averiguarla escribiendo:

```
cat /etc/issue
Output
Ubuntu 20.04.3 LTS \n \l
``` 

Debes crear tu nuevo servidor con estos mismos parámetros si es posible. En este caso, crearía un sistema Ubuntu 20.04. También deberías intentar hacer coincidir la versión del kernel lo más posible. Por lo general, este debería ser el kernel más actualizado disponible en los repositorios de su distribución de Linux.


## 4. Configurar el acceso a la clave SSH entre los servidores de origen y de destino

Necesitará que sus servidores puedan comunicarse para poder transferir archivos. Para hacer esto, debes intercambiar claves SSH entre ellos. 

Deberá crear una nueva clave en su servidor de destino para poder agregarla al authorized_keys archivo de su servidor existente. Esto es más limpio que al revés, porque de esta manera, el nuevo servidor no tendrá una clave perdida en su authorized_keys archivo cuando se complete la migración.

Primero, en su máquina de destino, verifique que su usuario root no tenga ya una clave SSH escribiendo:

```
ls ~/.ssh
Output
authorized_keys
``` 

Si ve archivos llamados id_rsa.pub y id_rsa, entonces ya tiene claves y solo necesitará transferirlas.

Si no ve esos archivos, cree un nuevo par de claves usando ssh-keygen:

``` 
ssh-keygen -t rsa
``` 

Presione "Entrar" a través de todas las indicaciones para aceptar los valores predeterminados.

Ahora, puede transferir la clave al servidor de origen canalizándola a través de ssh:

``` 
cat ~/.ssh/id_rsa.pub | ssh other_server_ip "cat >> ~/.ssh/authorized_keys"
``` 

Ahora debería poder realizar SSH libremente a su servidor de origen desde el sistema de destino sin proporcionar una contraseña:

``` 
ssh other_server_ip
``` 

Esto hará que cualquier paso de migración adicional sea mucho más sencillo.


## 5. Cree una lista de requisitos

Ahora harás un análisis en profundidad de tu sistema.

Durante el curso de las operaciones, sus requisitos de software pueden cambiar. A veces, los servidores antiguos tienen algunos servicios y software que fueron necesarios en algún momento, pero que han sido reemplazados.

En general, los servicios innecesarios pueden desactivarse y, si son completamente innecesarios, desinstalarse, pero evaluarlos puede llevar mucho tiempo. Necesitará descubrir qué servicios se están utilizando en su servidor de origen y luego decidir si esos servicios deberían existir en su nuevo servidor.

La forma en que descubre servicios y niveles de ejecución depende del tipo de sistema "init" que emplea su servidor. El sistema de inicio es responsable de iniciar y detener los servicios, ya sea por orden del usuario o automáticamente. 

Para enumerar los servicios que están registrados con Systemd, puede usar el systemctl comando:

``` 
systemctl list-units -t service
Output
  UNIT                                 LOAD   ACTIVE SUB     DESCRIPTION               >
  accounts-daemon.service              loaded active running Accounts Service          >
  apparmor.service                     loaded active exited  Load AppArmor profiles    >
  apport.service                       loaded active exited  LSB: automatic crash repor>
  atd.service                          loaded active running Deferred execution schedul>
  blk-availability.service             loaded active exited  Availability of block devi>
  cloud-config.service                 loaded active exited  Apply the settings specifi>
  cloud-final.service                  loaded active exited  Execute cloud user/final s>
  cloud-init-local.service             loaded active exited  Initial cloud-init job (pr>
  cloud-init.service                   loaded active exited  Initial cloud-init job (me>
  console-setup.service                loaded active exited  Set console font and keyma>
  containerd.service                   loaded active running containerd container runti>
…
``` 

Para la gestión de sus servicios, Systemd implementa un concepto de “objetivos”. Mientras que los sistemas con sistemas de inicio tradicionales sólo pueden estar en un "nivel de ejecución" a la vez, un servidor que utiliza Systemd puede alcanzar varios objetivos al mismo tiempo. Esto es más flexible en la práctica, pero determinar qué servicios están activos puede resultar más difícil.

Puede ver qué objetivos están actualmente activos escribiendo:

``` 
systemctl list-units -t target
Output
  UNIT                   LOAD   ACTIVE SUB    DESCRIPTION
  basic.target           loaded active active Basic System
  cloud-config.target    loaded active active Cloud-config availability
  cloud-init.target      loaded active active Cloud-init target
  cryptsetup.target      loaded active active Local Encrypted Volumes
  getty.target           loaded active active Login Prompts
  graphical.target       loaded active active Graphical Interface
  local-fs-pre.target    loaded active active Local File Systems (Pre)
  local-fs.target        loaded active active Local File Systems
  multi-user.target      loaded active active Multi-User System
  network-online.target  loaded active active Network is Online
…
``` 

Puede enumerar todos los objetivos disponibles escribiendo:

```
systemctl list-unit-files -t target
Output
UNIT FILE                     STATE           VENDOR PRESET
basic.target                  static          enabled
blockdev@.target              static          enabled
bluetooth.target              static          enabled
boot-complete.target          static          enabled
cloud-config.target           static          enabled
cloud-init.target             enabled-runtime enabled
cryptsetup-pre.target         static          disabled
cryptsetup.target             static          enabled
ctrl-alt-del.target           disabled        enabled
…
``` 

Desde aquí podrás conocer qué servicios están asociados a cada target. Los objetivos pueden tener servicios u otros objetivos como dependencias, por lo que puede ver qué políticas implementa cada objetivo escribiendo:

``` 
systemctl list-dependencies target_name.target
``` 

multi-user.target es un objetivo comúnmente utilizado en los servidores Systemd que se alcanza en el punto del proceso de inicio cuando los usuarios pueden iniciar sesión. Por ejemplo, puede escribir algo como esto:

``` 
systemctl list-dependencies multi-user.target
Output
multi-user.target
● ├─apport.service
● ├─atd.service
● ├─console-setup.service
● ├─containerd.service
● ├─cron.service
● ├─dbus.service
● ├─dmesg.service
● ├─docker.service
● ├─grub-common.service
● ├─grub-initrd-fallback.service
…
``` 

Esto enumerará el árbol de dependencia de ese objetivo, brindándole una lista de servicios y otros objetivos que se inician cuando se alcanza ese objetivo.

### 5.1 Verificación de servicios a través de otros métodos

Si bien la mayoría de los servicios configurados por su administrador de paquetes se registrarán en el sistema de inicio, es posible que otros programas, como las implementaciones de Docker, no lo estén.

Puede intentar encontrar estos otros servicios y procesos observando los puertos de red y los sockets Unix que utilizan estos servicios. En la mayoría de los casos, los servicios se comunican entre sí o con entidades externas de alguna manera. Sólo hay una cierta cantidad de interfaces de servidor en las que los servicios pueden comunicarse, y verificar esas interfaces es una buena manera de detectar otros servicios.

Una herramienta que puede utilizar para descubrir puertos de red y sockets Unix en uso es netstat. Puede ejecutar netstat las -nlp banderas para obtener una descripción general:

``` 
netstat -nlp
``` 

* **-n** especifica que las direcciones IP numéricas deben mostrarse en la salida, en lugar de nombres de host o nombres de usuario. Al comprobar un servidor local, esto suele ser más informativo.
* **-l** especifica que netstat solo debe mostrar sockets de escucha activa.
* **-p** muestra el ID del proceso ( PID) y el nombre de cada proceso que utiliza el puerto o socket.

``` 
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

netstat La salida contiene dos bloques separados: uno para puertos de red y otro para sockets. Si ve servicios aquí sobre los que no tiene información a través del sistema de inicio, tendrá que averiguar por qué es así y si tiene la intención de migrar esos servicios también.

Puede obtener información similar sobre los puertos que los servicios ponen a disposición mediante el lsof comando:

``` 
lsof
``` 

``` 
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

Ambos netstat y lsof son herramientas centrales de gestión de procesos de Linux que son útiles en una variedad de otros contextos.


## 6. Recopilar versiones de paquetes

En este punto, debería tener una buena idea sobre qué servicios se están ejecutando en su máquina de origen y debería implementar en su servidor de destino.

Debe tener una lista de servicios que sabe que necesitará implementar. Para que la transición se realice sin problemas, es importante intentar hacer coincidir las versiones siempre que sea posible.

No necesariamente debe intentar revisar cada paquete instalado en el sistema fuente e intentar replicarlo en el nuevo sistema, pero debe verificar los componentes de software que son importantes para sus necesidades e intentar encontrar su número de versión.

Puede intentar obtener números de versión del propio software, a veces pasando marcas -v o --version indicadores a cada comando, pero esto es más sencillo de hacer a través del administrador de paquetes. Si tiene un sistema basado en Ubuntu/Debian, puede ver qué versión de un paquete está instalada usando el dpkg comando:

``` 
dpkg -l | grep package_name
``` 

Si, en cambio, utiliza un sistema basado en Rocky Linux, RHEL o Fedora, puede utilizarlo rpm para el mismo propósito:

``` 
rpm -qa | grep package_name
``` 

Esto le dará una buena idea de la versión del paquete que desea hacer coincidir. Asegúrese de conservar los números de versión de cualquier software relevante.

