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

## 5. Configuración del servidor







