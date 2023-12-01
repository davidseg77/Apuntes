# Cómo utilizar Nmap para buscar puertos abiertos

## 1. Introducción

La creación de redes es un tema amplio y abrumador para muchos administradores de sistemas en ciernes. Existen varias capas, protocolos e interfaces, y muchas herramientas y utilidades que se deben dominar para comprenderlos.

En las redes TCP/IP y UDP, los puertos son puntos finales para las comunicaciones lógicas. Una única dirección IP puede tener muchos servicios en ejecución, como un servidor web, un servidor de aplicaciones y un servidor de archivos. Para que cada uno de estos servicios se comunique, cada uno escucha y se comunica en un puerto específico. Cuando realiza una conexión a un servidor, se conecta a la dirección IP y a un puerto.

En este tutorial explorará los puertos con más detalle. Utilizará el netstatprograma para identificar puertos abiertos y luego utilizará el nmapprograma para obtener información sobre el estado de los puertos de una máquina en una red. Cuando haya terminado, podrá identificar puertos comunes y escanear sus sistemas en busca de puertos abiertos.


## 2. Identificación de puertos comunes

Los puertos se especifican mediante un número que va de 1 a 65535.

Muchos de los puertos siguientes 1024 están asociados con servicios que Linux y los sistemas operativos tipo Unix consideran críticos para las funciones esenciales de la red, por lo que debe tener privilegios de root para asignarles servicios.

Los puertos entre 1024 y 49151 se consideran “registrados”. Esto significa que se pueden “reservar” (en un sentido muy amplio de la palabra) para ciertos servicios emitiendo una solicitud a la IANA (Autoridad de Números Asignados de Internet). No se aplican estrictamente, pero pueden dar una pista sobre los posibles servicios que se ejecutan en un determinado puerto.

Los puertos entre 49152 y 65535 no se pueden registrar y se sugieren para uso privado.

Debido a la gran cantidad de puertos disponibles, nunca tendrá que preocuparse por la mayoría de los servicios que tienden a vincularse a puertos específicos.

Sin embargo, hay algunos puertos que vale la pena conocer por su ubicuidad. La siguiente es sólo una lista muy incompleta:

* 20 : datos FTP
* 21 : puerto de control FTP
* 22 : SSH
* 23 : Telnet (Inseguro, no recomendado para la mayoría de usos)
* 25 : SMTP
* 43 : protocolo WHOIS
* 53 : servicios DNS
* 67 : puerto del servidor DHCP
* 68 : puerto del cliente DHCP
* 80 : HTTP: tráfico web no cifrado
* 110 : puerto de correo POP3
* 113 : Servicios de autenticación de identidad en redes IRC
* 143 : puerto de correo IMAP
* 161 : SNMP
* 194 : IRC
* 389 : puerto LDAP
* 443 : HTTPS - Tráfico web seguro
* 587 : SMTP - puerto de envío de mensajes
* 631 : puerto del demonio de impresión CUPS
* 666 : DOOM: este juego heredado en realidad tiene su propio puerto especial

Estos son sólo algunos de los servicios comúnmente asociados a los puertos. Debería poder encontrar los puertos apropiados para las aplicaciones que está intentando configurar en su documentación respectiva.

La mayoría de los servicios se pueden configurar para usar puertos distintos al predeterminado, pero debe asegurarse de que tanto el cliente como el servidor estén configurados para usar un puerto no estándar.

Puede obtener una lista de algunos puertos comunes mirando el /etc/services archivo:

``` 
less /etc/services
``` 

Le dará una lista de puertos comunes y sus servicios asociados:

``` 
Output
. . .
tcpmux          1/tcp                           # TCP port service multiplexer
echo            7/tcp
echo            7/udp
discard         9/tcp           sink null
discard         9/udp           sink null
systat          11/tcp          users
daytime         13/tcp
daytime         13/udp
netstat         15/tcp
qotd            17/tcp          quote
msp             18/tcp                          # message send protocol
. . .
``` 

Dependiendo de su sistema, esto mostrará varias páginas. Presione la SPACE tecla para ver la siguiente página de entradas o presione Q para regresar al mensaje.

Esta no es una lista completa; Podrás verlo en breve.


## 3. Comprobación de puertos abiertos

Hay varias herramientas que puede utilizar para buscar puertos abiertos. Uno que se instala de forma predeterminada en la mayoría de las distribuciones de Linux es netstat.

Puede descubrir rápidamente qué servicios está ejecutando emitiendo el comando con los siguientes parámetros:

``` 
sudo netstat -plunt
``` 

Verás resultados como los siguientes:

``` 
Output
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      785/sshd        
tcp6       0      0 :::22                   :::*                    LISTEN      785/sshd 
```

Esto muestra el puerto y el socket de escucha asociados con el servicio y enumera los protocolos UDP y TCP.

La nmap herramienta es otro método que puede utilizar para identificar puertos.


## 4. Usando Nmap

Parte de proteger una red implica realizar pruebas de vulnerabilidad. Esto significa intentar infiltrarse en su red y descubrir debilidades de la misma manera que lo haría un atacante.

De todas las herramientas disponibles para esto, esta nmap es quizás la más común y poderosa.

Puede instalar nmap en una máquina Ubuntu o Debian ingresando:

``` 
sudo apt-get update
sudo apt-get install nmap
``` 

Uno de los beneficios secundarios de instalar este software es un archivo de mapeo de puertos mejorado. Puede ver una asociación mucho más extensa entre puertos y servicios consultando este archivo:

``` 
less /usr/share/nmap/nmap-services
```

Verás un resultado como este:

``` 
Output
. . .
tcpmux  1/tcp   0.001995        # TCP Port Service Multiplexer [rfc-1078]
tcpmux  1/udp   0.001236        # TCP Port Service Multiplexer
compressnet     2/tcp   0.000013        # Management Utility
compressnet     2/udp   0.001845        # Management Utility
compressnet     3/tcp   0.001242        # Compression Process
compressnet     3/udp   0.001532        # Compression Process
unknown 4/tcp   0.000477
rje     5/udp   0.000593        # Remote Job Entry
unknown 6/tcp   0.000502
echo    7/tcp   0.004855
echo    7/udp   0.024679
echo    7/sctp  0.000000
. . .
``` 

Además de tener casi 20 mil líneas, este archivo también tiene campos adicionales, como la tercera columna, que enumera la frecuencia de apertura de ese puerto descubierta durante los análisis de investigación en Internet.


## 5. Escaneando puertos con nmap

Nmap puede revelar mucha información sobre un host. También puede hacer que los administradores del sistema objetivo piensen que alguien tiene intenciones maliciosas. Por este motivo, pruébelo únicamente en servidores de su propiedad o en situaciones en las que haya notificado a los propietarios.

Los nmap creadores proporcionan un servidor de prueba ubicado en scanme.nmap.org.

Éste o sus propios servidores son buenos objetivos para practicar nmap.

A continuación se muestran algunas operaciones comunes que se pueden realizar con nmap. Los ejecutaremos todos con privilegios sudo para evitar devolver resultados parciales para algunas consultas. Algunos comandos pueden tardar bastante en completarse.

Busque el sistema operativo host:

``` 
sudo nmap -O scanme.nmap.org
``` 

Omita la parte de descubrimiento de red y asuma que el host está en línea. Esto es útil si recibe una respuesta que dice "Nota: el host parece inactivo" en sus otras pruebas. Añade esto a las otras opciones:

``` 
sudo nmap -PN scanme.nmap.org
``` 

Escanee sin realizar una búsqueda DNS inversa en la dirección IP especificada. Esto debería acelerar sus resultados en la mayoría de los casos:

``` 
sudo nmap -n scanme.nmap.org
``` 

Escanee un puerto específico en lugar de todos los puertos comunes:

``` 
sudo nmap -p 80 scanme.nmap.org
``` 

Para buscar conexiones TCP, nmap puede realizar un protocolo de enlace de tres vías (que se explica a continuación) con el puerto de destino. Ejecútelo así:

``` 
sudo nmap -sT scanme.nmap.org
``` 

Para buscar conexiones UDP, escriba:

``` 
sudo nmap -sU scanme.nmap.org
``` 

Busque cada puerto abierto TCP y UDP:

``` 
sudo nmap -n -PN -sT -sU -p- scanme.nmap.org
```

Un análisis TCP “SYN” aprovecha la forma en que TCP establece una conexión.

Para iniciar una conexión TCP, el extremo solicitante envía un paquete de "solicitud de sincronización" al servidor. Luego, el servidor devuelve un paquete de "acuse de sincronización". Luego, el remitente original envía un paquete de "acuse de recibo" al servidor y se establece una conexión.

Sin embargo, un escaneo “SYN” interrumpe la conexión cuando el servidor devuelve el primer paquete. Esto se denomina escaneo “medio abierto” y solía promocionarse como una forma de escanear subrepticiamente puertos, ya que la aplicación asociada con ese puerto no recibiría el tráfico porque la conexión nunca se completa.

Esto ya no se considera sigiloso con la adopción de firewalls más avanzados y la señalización de solicitudes SYN incompletas en muchas configuraciones.

Para realizar un escaneo SYN, ejecute:

``` 
sudo nmap -sS scanme.nmap.org
``` 

Un enfoque más sigiloso es enviar encabezados TCP no válidos que, si el host cumple con las especificaciones TCP, deberían enviar un paquete de vuelta si ese puerto está cerrado. Esto funcionará en servidores que no estén basados ​​en Windows.

Puede utilizar los indicadores “-sF”, “-sX” o “-sN”. Todos ellos producirán la respuesta que buscamos:

``` 
sudo nmap -PN -p 80 -sN scanme.nmap.org
``` 

Para ver qué versión de un servicio se está ejecutando en el host, puede probar este comando. Intenta determinar el servicio y la versión probando diferentes respuestas del servidor:

``` 
sudo nmap -PN -p 80 -sV scanme.nmap.org
``` 

Finalmente, puedes usar nmap para escanear varias máquinas.

Para especificar un rango de direcciones IP con “-” o “/24” para escanear varios hosts a la vez, use un comando como el siguiente:

``` 
sudo nmap -PN xxx.xxx.xxx.xxx-yyy
``` 

O escanee un rango de red en busca de servicios disponibles con un comando como este:

``` 
sudo nmap -sP xxx.xxx.xxx.xxx-yyy
```

Hay muchas otras combinaciones de comandos que puedes usar, pero esto debería ayudarte a comenzar a explorar las vulnerabilidades de tu red.


## 6. Conclusión

Comprender la configuración del puerto y cómo descubrir cuáles son los vectores de ataque en su servidor es sólo un paso para proteger su información y su VPS. Sin embargo, es una habilidad esencial.

Descubrir qué puertos están abiertos y qué información se puede obtener de los servicios que aceptan conexiones en esos puertos le brinda la información que necesita para bloquear su servidor. Cualquier información extraña que se filtre de su máquina puede ser utilizada por un usuario malintencionado para intentar explotar vulnerabilidades conocidas o desarrollar otras nuevas. Cuanto menos puedan descubrir, mejor.

