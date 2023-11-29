# Cómo abrir un puerto en Linux


## 1. Listar todos los puertos abiertos

Antes de abrir un puerto en Linux, debe verificar la lista de todos los puertos abiertos y elegir un puerto efímero para abrir que no esté en esa lista.

Utilice el netstat comando para enumerar todos los puertos abiertos, incluidos TCP y UDP, que son los protocolos más comunes para la transmisión de paquetes en la capa de red.

``` 
netstat -lntu
``` 

Esto imprimirá:

* todas las tomas de escucha ( -l)
* el número de puerto ( -n)
* Puertos TCP ( -t)
* Puertos UDP ( -u)

```
Output
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address  State
tcp        0      0 127.0.0.1:5432   0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.1:27017  0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.1:6379   0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.53:53    0.0.0.0:*        LISTEN
tcp        0      0 0.0.0.0:22       0.0.0.0:*        LISTEN
tcp6       0      0 ::1:5432         :::*             LISTEN
tcp6       0      0 ::1:6379         :::*             LISTEN
tcp6       0      0 :::22            :::*             LISTEN
udp        0      0 127.0.0.53:53    0.0.0.0:*        LISTEN
``` 

Verifique que esté recibiendo resultados consistentes usando el ss comando para enumerar los sockets de escucha con un puerto abierto:

``` 
ss -lntu
``` 

Esto imprimirá:

``` 
Output
Netid  State   Recv-Q  Send-Q    Local Address:Port   Peer Address:Port
udp    UNCONN  0       0         127.0.0.53%lo:53          0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:5432        0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:27017       0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:6379        0.0.0.0:*
tcp    LISTEN  0       128       127.0.0.53%lo:53          0.0.0.0:*
tcp    LISTEN  0       128             0.0.0.0:22          0.0.0.0:*
tcp    LISTEN  0       128               [::1]:5432        0.0.0.0:*
tcp    LISTEN  0       128               [::1]:6379        0.0.0.0:*
tcp    LISTEN  0       128                [::]:22          0.0.0.0:*
``` 

Esto proporciona más o menos los mismos puertos abiertos que netstat.


## 2. Abrir un puerto en Linux para permitir conexiones TCP

Ahora, abre un puerto cerrado y haz que escuche las conexiones TCP.

Para los propósitos de este tutorial, abrirá el puerto 4000. Sin embargo, si ese puerto no está abierto en su sistema, no dude en elegir otro puerto cerrado. Solo asegúrate de que sea mayor que 1023.

Asegúrese de que el puerto 4000 no se utilice mediante el netstat comando:

``` 
netstat -na | grep :4000
``` 

O el sscomando:

``` 
ss -na | grep :4000
``` 

La salida debe permanecer en blanco, verificando así que no esté en uso actualmente, para que pueda agregar las reglas del puerto manualmente al firewall de iptables del sistema.

### 2.1 Para usuarios y ufw sistemas basados ​​en Ubuntu

**Uso ufw:** el cliente de línea de comandos para Uncomplicated Firewall.

Tus comandos se parecerán a:

``` 
sudo ufw allow 4000
``` 

### 2.2 Para otras distribuciones de Linux

Úselo iptables para cambiar las reglas de filtrado de paquetes IPv4 del sistema.

``` 
iptables -A INPUT -p tcp --dport 4000 -j ACCEPT
``` 

Consulte Cómo configurar un firewall iptables para su distribución.

Ahora que ha abierto con éxito un nuevo puerto TCP, es hora de probarlo.

Primero, inicie netcat (nc) y escuche (-l) en el puerto (-p) 4000, mientras envía la salida ls a cualquier cliente conectado:

``` 
ls | nc -l -p 4000
``` 

Ahora, después de que un cliente haya abierto una conexión TCP en el puerto 4000, recibirá el resultado de ls. Deje esta sesión en paz por ahora.

Abra otra sesión de terminal en la misma máquina.

Dado que abrió un puerto TCP, utilíce telnet para verificar la conectividad TCP. Si el comando no existe, instálelo usando su administrador de paquetes.

Ingrese la IP de su servidor y el número de puerto (4000 en este ejemplo) y ejecute este comando:

``` 
telnet localhost 4000
``` 

Este comando intenta abrir una conexión TCP en localhostun puerto 4000.

Obtendrá un resultado similar a este, que indica que se ha establecido una conexión con el programa de escucha (nc):

``` 
Output
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
while.sh
``` 

La salida de ls (while.sh, en este ejemplo) también se envió al cliente, lo que indica una conexión TCP exitosa.

Úse nmap para verificar si el puerto (-p) está abierto:

``` 
nmap localhost -p 4000
``` 

Este comando verificará el puerto abierto:

``` 
Output
Starting Nmap 7.60 ( https://nmap.org ) at 2020-01-18 21:51 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00010s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE
4000/tcp open  remoteanything

Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds
``` 

El puerto ha sido abierto. Ha abierto con éxito un nuevo puerto en su sistema Linux. Pero esto es sólo temporal, ya que los cambios se restablecerán cada vez que reinicies el sistema.


## 3. Reglas persistentes

El enfoque presentado en este artículo solo actualizará temporalmente las reglas del firewall hasta que el sistema se apague o se reinicie. Por lo tanto, se deben repetir pasos similares para abrir el mismo puerto nuevamente después de reiniciar.

**Para ufw cortafuegos**

Las reglas no se restablecen al reiniciar. Esto se debe a que está integrado en el proceso de arranque y el kernel guarda las reglas del firewall aplicando los ufw archivos de configuración apropiados.

**Para firewalld**

Deberá aplicar la --permanent bandera.

**Para iptables**

Deberá guardar las reglas de configuración. Estos tutoriales recomiendan iptables-persistent.





