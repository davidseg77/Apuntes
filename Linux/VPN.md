# Instalación y configuración de OpenVPN en Ubuntu


## 1. Creación del servidor

Puede crearse bien desde local o bien en la nube. Por ejemplo, desde Azure creando una VM Ubuntu. En este tutorial vamos a instalar OpenVPN en Ubuntu Server en una máquina virtual en la nube (Azure) y, luego, lo vamos a configurar para que podamos acceder desde nuestro dispositivo móvil y desde nuestro ordenador. En este caso tendremos que utilizar una máquina virtual como servidor y resto de clientes deben tener conectividad con la máquina.

• Ubuntu 20.04 como servidor
◦ Red Interna 10.XX.XX.0/24
◦ Adaptador puente

• Ubuntu como cliente
◦ Adaptador puente

• Teléfono móvil
◦ Conectado a una red wifi en la misma red local


Hecho esto, nos conectamos desde nuestra terminal por ssh a esa máquina:

``` 
ssh nombreVM@ip
``` 

Indicamos la contraseña y accedemos. Hecho esto, el primer paso será actualizar los repositorios. Para instalar OpenVPN utilizamos los siguientes comandos:

``` 
sudo apt update
sudo apt install openvpn easy-rsa -y
``` 

Como puedes comprobar, además de OpenVPN, hemos instalado la herramienta de gestión de
infraestructura de clave pública (PKI) Easy-RSA, necesaria para crear las claves.
Comprobamos la ubicación de cada herramienta:

``` 
whereis openvpn
whereis easy-rsa
```

Para ver la ayuda y la versión:

``` 
openvpn --help
openvpn --version
```

## 2. Configuración de la autoridad de certificación

Para configurar su propia Autoridad de certificación (CA) y generar certificados y claves para un servidor OpenVPN y varios clientes, primero es necesario crear la Infraestructura de Clave Pública (PKI) con la herramienta easy-rsa.

Para garantizar que los cambios que realicemos no se puedan perder cuando se actualice el paquete
easy-rsa vamos a copiar el directorio de la herramienta dentro del directorio de OpenVPN.

``` 
cp -r /usr/share/easy-rsa /etc/openvpn/
``` 

Como usuario root, cambie al directorio recién creado /etc/openvpn/easy-rsa

``` 
cd /etc/openvpn/easy-rsa
``` 

Crear la Infraestructura de Clave Pública (PKI):

``` 
sudo ./easyrsa init-pki
``` 

Una vez inicializada la PKI, debemos crear la Autoridad de Certificación (CA):

``` 
./easyrsa build-ca
``` 

Una vez ejecutado, debemos seguir el sencillo asistente de generación de CA. La contraseña que nos pide es para proteger la clave privada(.key) de la CA, algo fundamental. En este punto habremos creado la clave pública(ca.crt) y privada (ca.key) de la CA.

Una vez que hemos creado la claves de la CA, deberemos crear el certificado del servidor, y los
certificados de los clientes. A continuación, deberemos firmarlo con la clave privada de la CA.

**Nota:** Se puede crear un fichero vars para modificar la configuración de los certificados por defecto.
Existe un fichero vars.example que contiene ejemplos integrados para la configuración de Easy-RSA.
DEBES nombrar este archivo a 'vars' si desea que se use como archivo de configuración.

Cada equipo (clientes y servidor) debe tener su par de claves (pública y privada), el certificado de la CA (ca.crt) y la clave privada de tls-crypt (ta.key).

La clave privada de la CA (ca.key) solo debe estar en el ordenador que se use para firmar y solo accesible para el administrador.


## 3. Crear claves del servidor

Invoque easyrsa con la opción gen-req seguida de un nombre común (CN) para la máquina. El CN puede ser el que prefiera, pero puede resultarle útil que sea descriptivo. Durante este tutorial, el CN del servidor de OpenVPN será servidor-redesplus. Asegúrese de incluir también la opción nopass. Si no lo hace, se protegerá con contraseña el archivo de solicitud, lo que puede generar problemas de permisos más adelante.

A continuación, generaremos un par de claves para el servidor:

``` 
./easyrsa gen-req servidor-redesplus nopass
``` 

Con esto, se crearán una clave privada para el servidor (.key) y un archivo de solicitud de firma de certificado (CSR) .req para el servidor de OpenVPN.

Una vez creado el certificado, deberemos firmarlo con la CA en modo «server»:

```
./easyrsa sign-req server servidor-redesplus
```

Y ya hemos creado la clave pública (.crt) que utilizaremos posteriormente en el fichero de configuración de OpenVPN. Completados estos pasos, ha firmado la solicitud de certificado del servidor de OpenVPN usando la clave privada del servidor de CA. El archivo .crt resultante contiene la clave de cifrado pública del servidor de OpenVPN, así como una nueva firma del servidor de CA. El objetivo de la firma es indicar a todos los que confían en el servidor de CA que también pueden confiar en el servidor de OpenVPN
cuando se conecten a él.

**Copiar las claves generadas al directorio de openvpn**

Para terminar de configurar los certificados, copie los archivos servidor-redesplus.crt y ca.crt desde
el servidor de CA al servidor de OpenVPN:

``` 
sudo cp /usr/share/easy-rsa/pki/issued/servidor-redesplus.crt /etc/openvpn/server/
sudo cp /usr/share/easy-rsa/pki/ca.crt /etc/openvpn/server/
sudo cp /etc/openvpn/easy-rsa/pki/private/servidor-redesplus.key
/etc/openvpn/server/
``` 

Ahora su servidor de OpenVPN está casi listo para aceptar conexiones. En el siguiente paso, realizará
algunos pasos adicionales para aumentar la seguridad del servidor.

**Crear la clave tls-crypt (tls-auth en sistemas antiguos)**

Para obtener una capa de seguridad adicional, añadiremos una clave secreta extra compartida que el
servidor y todos los clientes usarán con la directiva tls-crypt de OpenVPN. Esta opción se usa para
confundir el certificado TLS que se usa cuando un servidor y un cliente se conectan inicialmente.
También lo usa el servidor de OpenVPN para realizar comprobaciones rápidas de los paquetes entrantes:
si se firma un paquete usando la clave previamente compartida, el servidor lo procesa; si no se firma, el
servidor sabe que es de una fuente no confiable y puede descartarlo sin tener que realizar otras tareas
de descifrado.

Esta opción lo ayudará a asegurarse de que su servidor de OpenVPN pueda hacer frente al tráfico sin
autenticación, a los escáneres de puerto y a los ataques de denegación de servicio, que pueden restringir
recursos del servidor. También hace que sea más difícil identificar el tráfico de red de OpenVPN.
Para generar la clave tls-crypt antes compartida, ejecute lo siguiente en el servidor de OpenVPN en el
directorio ~/easy-rsa:

``` 
cd /etc/openvpn/server
openvpn --genkey --secret ta.key
``` 

Y lo comprobamos:

``` 
ls -l
```

Esta clave ta.key deberemos colocarla en el servidor y en TODOS los clientes.


## 4. Crear claves del cliente

El cliente VPN también necesitará un certificado para autenticarse en el servidor. Por lo general, crea un certificado diferente para cada cliente. Esto puede hacerse en el servidor (como las claves y los certificados anteriores) y luego distribuirse de forma segura al cliente. O viceversa: el cliente puede generar y enviar una solicitud que es enviada y firmada por el servidor.

En nuestro caso vamos generar los certificados en el servidor y luego los incluiremos todos en un fichero ovpn que será el que distribuyamos al cliente. Para mantener una buena organización vamos a guardar las claves de los cliente en un directorio que llamaremos keys:

``` 
sudo mkdir /etc/openvpn/client/keys
```

Para proteger los certificados y configuración de los clientes vamos a dar solo permisos a root:

``` 
sudo chmod -R 700 /etc/openvpn/client
``` 

Para crear el certificado, ingrese lo siguiente en una terminal mientras es usuario root:

```
cd /etc/openvpn/easy-rsa
sudo ./easyrsa gen-req cliente1-redesplus nopass
```

Una vez creado el certificado, deberemos firmarlo con la CA en modo «client»:

``` 
sudo ./easyrsa sign-req client cliente1-redesplus
``` 

Ponemos la contraseña de la CA y listo.

Ahora copiamos las claves generadas al directorio de OpenVPN:

``` 
sudo cp /etc/openvpn/easy-rsa/pki/issued/cliente1-redesplus.crt /etc/openvpn/client/keys/
sudo cp /etc/openvpn/easy-rsa/pki/private/cliente1-redesplus.key /etc/openvpn/client/keys/
sudo cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/client/keys
sudo cp /etc/openvpn/server/ta.key /etc/openvpn/client/keys/
```

## 5. Configuración del servidor (server.conf)

El fichero de configuración del servidor se encuentra en la siguiente ruta:

``` 
ls /usr/share/doc/openvpn/examples/sample-config-files/
```

Copiamos fichero de muestra a nuestra carpeta:

``` 
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/server/
sudo gunzip /etc/openvpn/server/server.conf.gz
```

Es recomendable descargarse el archivo en local:

``` 
scp -p maquinaservidor@ipservidor:/etc/openvpn/server/server.conf .
```

Hecho esto a modo de prueba, elimino el archivo server.conf del servidor y lo creo de nuevo con estos parámetros:

``` 
# 1. INTERFAZ DE ESCUCHA
;local a.b.c.d

# 2. PUERTO A UTILIZAR (TCP O UDP). POR DEFECTO ES 1194.
port 1194

# 3. PROTOCOLO A UTILIZAR: TCP O UDP
;proto tcp
proto udp

# 4. TIPO DE TUNEL: dev tun (enrutamiento IP) o dev tap (puente Ethernet)
;dev tap
dev tun
;dev-node MyTap

# 5. MODIFICAR EL NOMBRE DE LAS CLAVES POR EL DE LAS QUE HEMOS UTILIZADO
ca ca.crt
cert servidor-redesplus.crt
key servidor-redesplus.key  # Mantener secreto este fichero

# 6. DESACTIVAR la directiva  Diffie-Hellman
;dh dh2048.pem
dh none

# 7. TOPOLOGIA DE LA RED (SE RECOMIENDA SUBNET)
topology subnet

# 8. DIRECCIONES DE SUBRED DE LA VPN (ip_servidor=10.8.0.1) SOLO PARA dev tun
server 10.8.0.0 255.255.255.0

# 9. CONFIGURAMOS PARA QUE LOS CLIENTES TENGAN LA MISMA IP SIEMPRE
ifconfig-pool-persist /var/log/openvpn/ipp.txt

# SOLO PARA dev tap
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100
;server-bridge

# PARA PERMITIR QUE LOS CLIENTES ACCEDAN A OTRAS REDES PRIVADAS DETRAS DEL SERVIDOR
;push "route 192.168.10.0 255.255.255.0"
;push "route 192.168.20.0 255.255.255.0"

# Para asignar direcciones IP específicas a clientes
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252

# (AVANZADO)PARA CREAR UN SCRIPT QUE MODIFIQUE DINÁMICAMENTE EL FIREWALL SI QUEREMOS REGLAS DIFERENTES PARA USARIOS O GRUPOS
;learn-address ./script

#10. Si está habilitada, esta directiva configurará todos los clientes para redirigir su puerta de enlace predeterminado a través de la VPN (Salida a Internet por la VPN)
push "redirect-gateway def1 bypass-dhcp"

#11. Ciertas configuraciones de red específicas de Windows se puede enviar a los clientes, como DNS
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

#12. HABILITAMOS COMUNICACION ENTRE LOS CLIENTES#
;client-to-client

#13. PARA UTILIZAR LAS MISMAS CLAVES CON TODOS LOS CLIENTES
;duplicate-cn

#14. HABILITAMOS KEEPALIVE PARA SABER SI EL TUNEL SE HA CAIDO
keepalive 10 120

#15. ACTIVAMOS UNA CLAVE SECRETA EXTRA 
;tls-auth ta.key 0 # This file is secret
tls-crypt ta.key

#16. TIPO DE CIFRADO 
;cipher AES-256-CBC
cipher AES-256-GCM
auth SHA512

#17. COMPRESIÓN
;compress lz4-v2
;push "compress lz4-v2"

;comp-lzo

#18. Nº MÁXIMO DE CLIENTES SIMULTÁNEOS
max-clients 100

#19. SIN PERMISOS DE USUARIO Y GRUPO
user nobody
group nogroup

#20. CLAVE Y TUNEL PERSISTENTE
persist-key
persist-tun

#21. CONEXIONES ACTUALES 
status /var/log/openvpn/openvpn-status.log

#22. LOGS (syslog por defecto)
;log         /var/log/openvpn/openvpn.log
;log-append  /var/log/openvpn/openvpn.log

#23. VERBOSITY
verb 3

#24. SILENCIAR REGISTROS log REPETIDOS
;mute 20

#25. NOTIFICACIÓN DE REINICIO
explicit-exit-notify 1
```

Es importante que en la configuración del cliente estos parámetros coincidan para que haya compatibilidad. 


Si el archivo lo hubieramos editado en vez de en el servidor en el cliente, para recuperarlo en el servidor haríamos esto:

``` 
sudo scp server.conf VMcliente@ipcliente:/home/redesplus
```

Y volvemos al servidor y movemos el server.conf:

``` 
sudo cp server.conf /etc/openvpn/server
```

## 6. Configuración del cliente 

Copiamos fichero de muestra a nuestra carpeta:

``` 
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/
```

Y editamos el fichero:

``` 
sudo nano /etc/openvpn/client/client.conf
```

Quedaría del siguiente modo:

``` 
#C1.ESPECIFICAMOS QUE SOMOS UN CLIENTE
client
#C2. NOMBRE o IP DEL SERVIDOR + PUERTO
remote ipservidor 1194		#S2
;remote my-server-2 1194

proto udp					#S3

dev tun						#S4

#CONEXIÓN ALEATORIA A LOS SERVIDORES INDICADOS 
;remote-random

#C3. RESOLUCIÓN DE NOMBRES INFINITA
resolv-retry infinite	

#C4. SIN ASOCIAR PUERTO O SERVICIO
nobind

user nobody
group nogroup
persist-key					#S19
persist-tun

#CONEXIÓN CON EL SERVIDOR A TRAVES DE UN PROXY
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

#C5. SILENCIAR LOS AVISOS DUPLICADOS
;mute-replay-warnings

##CLAVES
;ca ca.crt
;cert client.crt
;key client.key
;tls-crypt ta.key			#S14

#C6.COMPROBAR LA IDENTIDAD DEL SERVIDOR
remote-cert-tls server

#CIFRADO
cipher AES-256-GCM			#S15
auth SHA512
#COMPRESIÓN
;comp-lzo					#S16

verb 3
;mute 20
```

## 7. Abrir cortafuegos y reiniciar OpenVPN


Si la VM del servidor la hemos creado con Azure, en el apartado Redes agregamos regla de seguridad de entrada. En intervalos de puertos de destino ponemos 1194 y protocolo Any. Hecho esto, lo siguiente será permitir el reenvío de paquetes entre interfaces:

``` 
sudo nano /etc/sysctl.conf (Habilitamos la línea net.ipv4.ip_forward=1)
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

Añadimos las siguientes reglas al cortafuegos:

``` 
sudo iptables -t nat -I POSTROUTING 1 -s 10.8.0.0/24 -o eth0 -j MASQUERADE
sudo iptables -I INPUT 1 -i tun0 -j ACCEPT
sudo iptables -I FORWARD 1 -i eth0 -o tun0 -j ACCEPT
sudo iptables -I FORWARD 1 -i tun0 -o eth0 -j ACCEPT
sudo iptables -I INPUT 1 -i eth0 -p udp --dport 1194 -j ACCEPT
```

Para ver las reglas del cortafuegos:

``` 
sudo iptables -L -nv
```

Para ver las reglas NAT:

``` 
sudo iptables -t nat -L -nv
```

Guardamos las reglas persistentes:

``` 
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

Configuramos VPN para que se inicie en el arranque:

``` 
sudo systemctl -f enable openvpn-server@server.service
```

E iniciamos openvpn:

``` 
sudo service openvpn-server@server start
sudo service openvpn-server@server status
```

## 8. Crear ficheros ovpn

Un fichero de extensión .ovpn es un archivo de configuración de cliente OpenVPN con un formato unificado. Este fichero incluye 5 secciones que se encuentran en el siguiente orden:

* Client.conf (Fichero de configuración del cliente)
* ca.crt (Certificado de la CA)
* cliente-redesplus.crt (Clave pública cliente)
* cliente-redesplus.key (Clave privada cliente)
* ta.key (Clave privada tls-crypt)

Lo primero que vamos a hacer es copiar el archivo cliente.conf en el mismo directorio bajo el nombre de plantilla.conf

``` 
sudo cp /etc/openvpn/client/client.conf /etc/openvpn/client/plantilla.conf
```

Y editamos el fichero

```
sudo nano /etc/openvpn/client/plantilla.conf
```

Importante que estén comentados el apartado de las keys. Trás ello, creamos un script que nos va a automatizar esa plantilla. 

``` 
sudo nano /etc/openvpn/client/make_config.sh
```

El script se diseñará del siguiente modo:

``` 
#!/bin/bash

# First argument: Client identifier

KEY_DIR=/etc/openvpn/client/keys
OUTPUT_DIR=/etc/openvpn/client/files
BASE_CONFIG=/etc/openvpn/client/plantilla.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-crypt>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-crypt>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

Creamos el archivo files que es donde se va a almacenar el fichero de salida, el ovpn. 

Y damos permiso de ejecución al archivo make_config.sh, solo para el usuario root:

``` 
chmod 700 /etc/openvpn/client/make_config.sh
```

Trás esto, solo queda generar el fichero ovpn poniendole un identificador, que va a ser el nombre exacto de los certificados que le hemos puesto al cliente. 

```
sudo ./make_config.sh cliente1-redesplus
```

si hacemos un ls al directorio files vemos como se ha creado el ovpn. Con este fichero, cuando se lo damos a un cliente, ya sea en Windows, Linux, Android... simplemente con él al ejecutarlo dentro de OpenVPN se conectará con nuestro servidor y ya estaremos dentro de nuestra red virtual.


## 9. Instalar cliente VPN en Windows

Tendremos que pasar nuestro archivo ovpn, pero para ello hacemos una copia del archivo, que será el que pasemos al cliente, y después lo eliminamos del servidor. Esto se debe a que por el asunto de los privilegios dados antes, no podemos enviarlo por scp al cliente. 

``` 
sudo cp /etc/openvpn/client/files/cliente1-redesplus.ovpn /home/redesplus/

sudo chmod 444 cliente1-redesplus.ovpn

chown redesplus:redesplus cliente1-redesplus.ovpn
```

Vamos al cliente Windows, a la Powershell. Y nos traemos el archivo ovpn:

``` 
scp -p nombreservidor@ipservidor:/home/redesplus/cliente1-redesplus.ovpn .
```

Para confirmar que lo tenemos:

``` 
explorer.exe .
```

Este archivo lo cortamos y pegamos en C:\Program Files\OpenVPN\config. Ya tenemos el archivo ejecutable, pero queda instalar OpenVPN en nuestro cliente. Para ello vamos aquí:

``` 
https://openvpn.net/community-downloads/
```

En cuanto se instala, nos aparece un icono en la barra inferior. Ahí podemos dar a conectar. Para comprobar que todo marcha, ponemos en el navegador **ifconfig.me** y la IP ha de ser la del servidor VPN. O en la terminal poniendo **ipconfig**.

Para desconectar, clic derecho en el panel inferior y desconectar.


## 10. Instalar cliente VPN en Ubuntu

En primer lugar, actualizamos los repositorios (sudo apt-update). 

Comprobamos si se utiliza el systemd resolved, porque en caso de no hacerlo la configuración habría de ser diferente:

``` 
cat /etc/resolv.conf
```

Debe aparecer una ip en nameserver. Después, tenemos que instalar el paquete openvpn systemd resolved.

``` 
sudo apt install openvpn-systemd-resolved
```

Descargamos el fichero ovpn:

```
scp -p nombreservidor@ipservidor:/home/redesplus/cliente1-redesplus.ovpn .
```

Para este caso, en el archivo debemos añadir estas líneas justo antes de la etiqueta <ca>

```
script-security 2
up /etc/openvpn/update-systemd-resolved
down /etc/openvpn/update-systemd-resolved
down-pre
dhcp-option DOMAIN-ROUTE .
```

Iniciamos la conexión:

``` 
sudo openvpn --config cliente1-redesplus.ovpn
```

Comprobamos el estado de la conexión:

``` 
ip ad
ping -c2 10.8.0.1
ip route
systemd-resolve --status tun0
```

Si vemos que no ha pillado dominio DNS, nos vamos al archivo server.conf y descomentamos el punto 11, las dos líneas push que tienen que ver con las opciones para DNS del servidor.

Paramos e iniciamos el servicio de nuevo:

``` 
sudo service openvpn-server@server stop
sudo service openvpn-server@server start
```

Nos desconectamos y volvemos a conectar

```
sudo killall openvpn

sudo openvpn --config cliente1-redesplus.ovpn
```

Y volvemos a comprobar si los DNS me aparecen ya:

``` 
systemd-resolve --status tun0
```

Lo compruebo haciendo curl a una web:

```
curl ipinfo.io
```

O un ping:

``` 
ping google.es
```













