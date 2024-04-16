# Curso de monitoreo con Nagios Core

## 1. Instalación 

Contamos con un servidor para Nagios y otra VM para el cliente. Hecho esto, accedemos al servidor y configuramos el firewall para que permita el acceso para el puerto 80 y el 443 por HTTPS y nos pida un certificado para mayor seguridad. Es totalmente recomendable.

``` 
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
``` 

Y recargamos:

``` 
firewall-cmd --reload
``` 

Para verificar todas las reglas que tenemos hasta el momento:

```
firewall-cmd --list-all
``` 

Y reiniciamos el servidor:

``` 
systemctl reboot
``` 

Ahora habrá que instalar las dependencias necesarias. 

```
sudo apt install -y gettext wget net-snmp-utils openssl-devel glibc-common unzip perl epel-release gcc php gd automake autoconf httpd make glibc gd-devel net-snmp
``` 

```
sudo apt install perl-Net-SNMP
``` 

Agregamos el usuario Nagios y le damos como grupo secundario Apache:

``` 
useradd nagios
usermod -aG apache nagios
``` 

Trás este paso, accedemos al repo oficial de Nagios en Github para descargarlo. Con wget lo descargamos en el servidor y después lo descomprimimos con tar -xzvf + el archivo.

E iniciamos la configuración:

``` 
./configure
``` 

Y compilamos Nagios:

``` 
make all
``` 

Terminada la compilación, lo siguiente ya si es la instalación. 

``` 
make install
``` 

``` 
make install-init
``` 

``` 
make install-commandmode
``` 

``` 
make install-config
``` 

``` 
make install-webconf
``` 

Y con todas estas diferentes partes, habremos instalado Nagios. Ahora habilitamos el servicio Nagios para que se ejecute automáticamente con el inicio del sistema.

``` 
systemctl enable nagios
``` 

Y habilitamos Apache para la web:

``` 
systemctl enable httpd
``` 

A modo de seguridad, conviene proteger la página de Nagios para que no pueda entrar cualquiera. Por ello:

``` 
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
``` 

Con el parámetro -c decimos que es la primera vez que vamos a insertar el usuario, si quisieramos insertar nuevos usuarios ya no sería necesario -c pues de hacerlo sobreescribiríamos a dicho usuario.

A continuación, le damos una clave al usuario administrador de Nagios.

E iniciamos el servicio Nagios y el de Apache:

``` 
systemctl start nagios
systemctl start httpd
``` 

Probamos el acceso a nuestro servicio:

``` 
http://ipdelservidor/nagios
``` 

Nos pedirá las credenciales de usuario dadas con htpasswd. Y accederemos al motor de Nagios, aunque no hemos configurado nada aún. 


## 2. Instalar plugins

Vamos al repo oficial de plugins de Nagios en Github. Mediante wget más el enlace de Nagios plugins lo pegamos en nuestro servidor para después descomprimir con tar -xzvf y archivo. Dentro del directorio de plugins, configuramos:

``` 
./configure
``` 

E instalamos:

``` 
make install
``` 

Y reiniciamos:

``` 
systemctl restart nagios
``` 

Para ver todos los plugins instalados:

``` 
cd /usr/local/nagios/libexec/
ls
```

## 3. Instalar NRPE

**¿Qué es NRPE?** Nagios Remote Plugin Executor (NRPE) es un agregado que permite la ejecución de plugins de Nagios en máquinas Linux/Unix, aunque también funciona en máquinas Windows con los programas necesarios para ello.

**NRPE** nos permite el acceso a los clientes y el uso de los plugins para mostrar los resultados de los servicios que queremos monitorear.

Vamos al repo oficial de NRPE para Nagios en Github. Dentro de github.com en */NagiosEnterprises/nrpe/releases/*

Con wget pegamos y con tar -xzvf más archivo extraemos. Vamos al directorio y configuramos:

``` 
./configure
``` 

Una vez configurado, nos indicará que el puerto de NRPE es el 5666. En cada cliente con el que vayamos a efectuar la monitorización, habremos de activar en el firewall este puerto. 

Ahora activamos el ejecutador de nrpe para cualquier servicio a monitorear:

``` 
make check_nrpe
``` 

Y:

``` 
make install-plugin
``` 

Con esto, ya tendríamos instalado NRPE. 

Ahora tenemos que crear un comando para que Nagios pueda usar este comando en NRPE.

``` 
cd /usr/local/nagios/etc/objects/
``` 

En su interior, vemos una serie de archivos de configuracion (.cfg). Vamos al de comandos:

``` 
nano commands.cfg
```

Al final del archivo tenemos una serie de comandos preconfigurados. Nosotros vamos a añadir el check_nrpe:

``` 
define command {
    command_name check_nrpe
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
``` 

* **USER1** es una variable definida ya por Nagios que alude a la ruta completa donde se halla check_nrpe.
* **HOSTADDRESS** alude a la IP de nuestro servidor Nagios
* **ARG1** alude al uso de los plugins de Nagios

Y ya tendríamos configurado nuestro comando nrpe para futuras configuraciones.






























