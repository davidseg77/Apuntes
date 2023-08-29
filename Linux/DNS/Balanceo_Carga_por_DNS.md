# Balanceo de carga por DNS

Una manera sencilla de balancear la carga entre dos servidores que estén ofreciendo el mismo servicio es utilizar la resolución DNS. Utilizando entradas tipo A duplicadas en un servidor DNS es posible realizar de forma muy sencilla un balanceo de carga entre varios equipos, esto se conoce como DNS round robin. Según la Wikipedia, es una técnica de distribución de carga, balanceo de carga o tolerancia a fallos de aprovisionamiento múltiple y redundante de servidores de servicios IP, por ejemplo, servidores web o servidores FTP, mediante la gestión de nombres de dominio del sistema para hacer frente a las peticiones de los equipos cliente según un modelo estadístico adecuado.

## 1. Balanceo de caga DNS usando los registros A

**Escenario de prueba**

Vamos a montar tres máquinas:

* nodo1: 10.1.1.101
* nodo2: 10.1.1.102
* dns: 10.1.1.103

Dos servidores web (nodo1 y nodo2) y un servidor DNS donde vamos a instalar un servidor bind9 donde tendremos definido la zona del dominio example.com para balancear la carga entre los dos servidores web.


**Configuración del servidor DNS**

Vamos a utilizar el nombre www.example.com para acceder al servidor web, por lo tanto vamos a configurar dos registros A para indicar la dirección ip de ese nombre, uno apuntará a la ip del nodo1 y el otro a la ip del nodo2. Veamos el fichero de configuración de la zona: /var/cache/bind/db.example.com:

``` 
...
$ORIGIN example.com.
www	    IN  A   10.1.1.101
www	    IN  A   10.1.1.102
``` 

**Prueba de funcionamiento**

Podemos realizar una consulta para preguntar la ip de www.example.com y veremos como nos ofrece las dos direcciones. El orden entre las respuestas va cambiando en las distintas peticiones:

``` 
dig @10.1.1.103 www.example.com
...
www.example.com.	86400	IN	A	10.1.1.101
www.example.com.	86400	IN	A	10.1.1.102
``` 

``` 
dig @10.1.1.103 www.example.com
...
www.example.com.	86400	IN	A	10.1.1.102
www.example.com.	86400	IN	A	10.1.1.101
``` 

Si accedemos desde un navegador web, podremos observar como se va intercalando el acceso (balanceando la carga) entre los dos nodos. Hemos configurado el ordenador desde el que estamos accediendo para que consulte a nuestro dns.

Para verlo de forma más gráfica, podemos acceder con curl al fichero info.txt del servidor que contiene le nombre del nodo:

``` 
while [ True ]; do curl http://www.example.com/info.txt && sleep 1 ; done

nodo1
nodo2
...
``` 

Con esta solución estamos balanceando la carga, pero no estamos consiguiendo una alta disponibilidad: en el momento que uno de los servidores web se apaga, las peticiones que vayan dirigido a él no obtendrán respuesta. Para ciertos servicios esto es un problema porque se quedarán esperando respuesta. Lo podemos ver haciendo un ping, después de apagar el nodo1:

```
ping www.example.com
PING www.example.com (10.1.1.102) 56(84) bytes of data.
64 bytes from nodo2.example.com (10.1.1.102): icmp_seq=1 ttl=64 time=0.275 ms
...
ping www.example.com
PING www.example.com (10.1.1.101) 56(84) bytes of data.
From ahsoka (10.1.1.1) icmp_seq=9 Destination Host Unreachable
``` 

Solucionemos este problema usando otra manera de balancear la carga por DNS

## 2. Balanceo de caga DNS usando los registros NS

Una posible solución al problema que hemos indicado, es considerar el nombre del servicio www.example.com como un nombre virtual, es decir será un alias (CNAME) de un nombre de un subdominio que estará delegado en dos servidores dns que instalaremos en los servidores que ofrecen el servicio (en nuestro caso en los servidores web).

El alias que vamos a configurar sería:

``` 
www.example.com    IN   CNAME   www.http.example.com
``` 

Es decir cuando accedamos a www.example.com estaremos accediendo a www de un dominio delegado (a un subdominio) que será http.example.com. Las zona de este subdominio delegado van a estar definidas en dos servidores DNS que instalaremos en nuestros nodos que ofrecen el servicio (nodo1 y nodo2). La delegación de del dominio se hará de la siguiente manera:

``` 
http.example.com    IN  NS      nodo1.http.example.com
http.example.com    IN  NS      nodo2.http.example.com
```

De esta forma al resolver el nombre www.example.com, estaremos resolviendo el nombre www.http.example.com, y para averiguar su ip el servidor dns tendrá que preguntar a uno de los servidores dns delegados. El servidor dns preguntará de forma alternativa a uno de los servidores dns delegados, y lo más importante, si un nodo se apaga y no puede devolver la respuesta, preguntará al otro, por lo que no tendremos cortes de servicio.

Veamos la configuración completa del servidor dns principal. El fichero /var/cache/bind/db.example.com quedaría:

``` 
$ORIGIN example.com.
www	    IN  CNAME   www.http
http	IN  NS      nodo1              
http	IN  NS      nodo2
``` 

**Configuración de los servidores DNS delegados**

Como hemos dicho en nodo1 y nodo2 tenemos que instalar un servidor DNS con autoridad para la zona http.example.com. Para que sea más fácil de configurar vamos a instalar un servidor dnsmasq, y la configuración será la siguiente:

En el fichero /etc/dnsmasq.conf de nodo1:

``` 
address=/www.http.example.com/10.1.1.101
``` 

En el fichero /etc/dnsmasq.conf de nodo2:

``` 
address=/www.http.example.com/10.1.1.102
``` 

Además en los dos ficheros de configuración hemos configurado el parámetro local-ttl=0 para que no se guarde la respuesta en la cache del dns principal.


**Prueba de funcionamiento**

En este caso, el comportamiento depende del cliente:

* Algunos elegirán aleatóriamente qué servidor obtendrá la consulta.
  
* Otros simplemente harán round robin (primero le pregunta a nodo1, la siguiente pregunta va primero a nodo2,…)
  
* Y otros consultarán en el orden que se puso los servidores (por lo que un nodo recibirá todas las consultas).
  
Independientemente de como se comporte, lo importante es que si no hay respuesta de un servidor delegado, se probará a preguntar al otro.

En esta prueba comprobamos como en un primer momento está preguntando al servidor dns delegado del nodo2, cuando apagamos este nodo, las peticiones pasan al servidor dns del nodo1 y sigue accediendo al servicio. Volvemos a iniciar el nodo2, pero se sigue consultado al dns del nodo1. Cuando, finalmente, apagamos el nodo1, las consultas pasan a hacerse de nuevo al dns del nodo2. Veámoslo de manera más gráfica:

``` 
while [ True ]; do curl http://www.example.com/info.txt && sleep 1 ; done
```

