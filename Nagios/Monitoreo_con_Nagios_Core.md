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


## 4. Preparando el host cliente Linux

Primero, habilitamos el puerto 5666 en el firewall:

``` 
firewall-cmd --permanent --add-port=5666/tcp
firewall-cmd --reload
```

Y reiniciamos:

```
systemctl reboot
``` 

Instalamos las dependencias necesarias en este cliente Linux:

``` 
sudo apt install -y gcc glibc-common openssl openssl-devel perl wget
``` 

Acto seguido, agregamos el usuario nagios.

``` 
useradd nagios
``` 

Y descargamos de nuevo el código fuente de los plugins y de NRPE.
Para ello volvemos a copiar la dirección de enlace de los plugins de Nagios dentro de su repo oficial en Git y lo pegamos y extraemos en el cliente.

Vamos a su directorio y configuramos:

``` 
./configure
```

Ahora hacemos la compilación e instalación en un solo paso:

```
make install
```

Con **NRPE** hacemos lo mismo. Descargamos desde su repo oficial, pegamos y extraemos con tar -xzvf. Nos vamos al directorio y configuramos:

``` 
./configure
``` 

Nos mostrará el puerto (5666) y el usuario. Y ejecutamos lo siguiente:

``` 
make all
make install
make install-config
make install-init
``` 

Ahora editamos el archivo de configuración de NRPE:

``` 
cd /usr/local/nagios/etc/
nano nrpe.cfg
```

Dentro del archivo, buscamos la siguiente línea:

``` 
allowed_hosts=Trás la ip que viene, agregamos la de nuestro servidor Nagios
```

Esta línea es fundamental y siempre ha de configurarse para cada cliente dentro de la monitorización con Nagios.

Habilitamos e iniciamos nrpe:

```
systemctl enable nrpe
systemctl start nrpe
```

Para comprobar si funciona, me voy al servidor y al siguiente directorio:

``` 
cd /usr/local/nagios/libexec
```

Y lanzo el siguiente comando:

``` 
./check_nrpe -H ipdelcliente
``` 

Si nos devuelve la versión de NRPE, entonces podemos decir que existe comunicación entre nuestro servidor Nagios y el cliente. De no existir esta comunicación, nos diría que no hay ruta establecida.

## 5. Agregar el  host cliente Linux

Dentro del servidor Nagios, vamos a /usr/local/nagios/etc/objects. Y dentro de ese directorio creamos un archivo de configuración para añadir los hosts clientes. En mi caso, voy a crear uno para los clientes Linux y otro para los de Windows. Este último lo haremos en el siguiente apartado.

``` 
nano linux.cfg
``` 

Dentro incluimos el cliente con una serie de parámetros:

``` 
### Definición de servidores ###

define host{
        use             linux-server
        host_name       cliente_linux
        alias           Linux Ubuntu
        check_interval  1
        address         Ipdelcliente
}
``` 

Con **check_interval** decimos cada cuanto tiempo, en minutos, vamos a monitorizar ese servidor.

Lo siguiente será especificar que servicio queremos monitorizar de cada servidor. Dentro del mismo archivo:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Hard Disk
        check_interval             1
        check_command              check_nrpe!check_hda1
}

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Uptime
        check_interval             1
        check_command              check_nrpe!check_uptime
}

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Current Load
        check_interval             1
        check_command              check_nrpe!check_load
}

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Swap
        check_interval             1
        check_command              check_nrpe!check_swap
}

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Ping
        check_interval             1
        check_command              check_ping!500.0,20%!800.0,60%
}

define service{
        use                        generic-service
        host_name                  cliente_linux
        service_description        Usuarios activos
        check_interval             1
        check_command              check_nrpe!check_users
}
``` 

Con check ping he creado dos intérvalos, uno moderado donde indico los milisegundos y el porcentaje de paquetes perdidos (20%), y uno crítico con un porcentaje aún mayor (60%).

Con todo ello, vamos a monitorear hasta seis servicios. 

Eso sí, para que Nagios sepa que existe este archivo de configuración para clientes Linux, hemos de hacer lo siguiente:

``` 
cd .. (Para ir a /usr/local/nagios/etc)
nano nagios.cfg
``` 

Y dentro de los archivos a tener en cuenta para Nagios añadimos el nuestro de Linux:

```
cfg_file=/usr/local/nagios/etc/objects/linux.cfg
``` 

Para corroborar que nuestros archivos de configuración esten perfectamente chequeados hacemos lo siguiente:

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
``` 

Cuando lo ejecutamos nos muestra, en caso de que los hubiera, los errores presentes en nuestros archivos de configuración.

Si hemos de corregir alguno, reiniciamos Nagios:

``` 
systemctl restart nagios
``` 

Como detalle, al ser muy usado este comando de verificación, podemos crearle un alias:

``` 
nano .bashrc
``` 

Y ahí creamos el alias de este modo:

``` 
nagioscheck='/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg'
```

Trás ello, cargamos de nuevo el archivo bashrc para que los cambios se hagan efectivos:

``` 
source .bashrc
```

Si vamos al navegador, a nuestra interfaz de Nagios, puede que algunos de nuestros servicios no estén funcionando. Para ello, nos vamos al cliente, al archivo de configuración de nrpe. 

``` 
nano /usr/local/nagios/etc/nrpe.cfg
```

Y agregamos los comandos, las líneas, oportunas:

``` 
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
command[check_uptime]=/usr/local/nagios/libexec/check_uptime
``` 

Y volvemos a reiniciar nrpe (systemctl restart nrpe).

**Apunte** 

Dentro del directorio objects, dentro de /nagios/etc, tenemos el archivo template.cfg. Ahí podemos ver una plantilla de como poder definir los servicios a monitorizar.


## 6. Agregar el host cliente Windows

Vamos dentro del cliente Windows a la página de nsclient, que viene a ser como NRPE para Linux.

<www.nsclient.org>

Y descargamos la última versión. Dentro del proceso de instalación escogemos el modo Complete y en Allowed hosts la ip del servidor Nagios. También activamos todos los apartados Enable que aparecen en esa pestaña.

Hecho esto vamos al firewall. Dentro de las reglas de entrada vemos que ya están creadas las de nsclient, a modo de verificación.

Después volvemos a escribir en Inicio **services.msc**. Buscamos el servicio local NSClient, hacemos doble clic en él y de ahí a la pestaña Iniciar sesión. Vamos a habilitar la opción *Permitir que el servicio interactúe con el escritorio* Y reiniciamos sesión refrescando arriba. 

Ahora pasamos al servidor Nagios para agregar el host Windows. 

``` 
cd /usr/local/nagios/etc/objects
``` 

Y de ahi vamos al archivo windows.cfg. Dentro definimos nuestro cliente Windows.

``` 
### Definición de servidores ###

define host{
        use             windows-server
        host_name       cliente_windows
        alias           Windows 2016
        address         Ipdelcliente
}
``` 

Lo siguiente será especificar que servicio queremos monitorizar de cada servidor. Dentro del mismo archivo:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        host_name                  cliente_windows
        service_description        NSClient++ Version
        check_command              check_nt!CLIENTVERSION
}

define service{
        use                        generic-service
        host_name                  cliente_windows
        service_description        Uptime
        check_command              check_nt!UPTIME
}

define service{
        use                        generic-service
        host_name                  cliente_windows
        service_description        CPU Load
        check_command              check_nt!CPULOAD!-1 5,80,90
}

define service{
        use                        generic-service
        host_name                  cliente_windows
        service_description        Memory Usage
        check_command              check_nt!MEMUSE!-w 80 -c 90
}

define service{
        use                        generic-service
        host_name                  cliente_windows
        service_description        C:\ Drive Space
        check_command              check_nt!USEDDISKSPACE!-l c -w 80 -c 90
}
``` 

Eso sí, para que Nagios sepa que existe este archivo de configuración para clientes Windows, hemos de hacer lo siguiente:

``` 
cd .. (Para ir a /usr/local/nagios/etc)
nano nagios.cfg
``` 

Y dentro de los archivos a tener en cuenta para Nagios añadimos el nuestro de Linux:

```
cfg_file=/usr/local/nagios/etc/objects/windows.cfg
``` 

Para corroborar que nuestros archivos de configuración esten perfectamente chequeados hacemos lo siguiente:

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
``` 

Cuando lo ejecutamos nos muestra, en caso de que los hubiera, los errores presentes en nuestros archivos de configuración.

Si hemos de corregir alguno, reiniciamos Nagios:

``` 
systemctl restart nagios
``` 

En mi caso, una vez accedido a la interfaz de Nagios, veo que tengo caido el cliente Windows. Si compruebo desde el servidor con Ping, veo que no hay comunicación entre servidor Nagios y cliente Windows. Para solucionarlo, voy al cliente Windows, a cmr (Windows+R) y hacemos ping a nuestro servidor Nagios. Hay comunicación. 

Vamos al firewall y en reglas de entrada nueva regla personalizada, todos los programas, protocolo ICMPv4, y le ponemos nombre (Permitir ICMP, por ejemplo). Y volvemos a comprobar con ping desde el servidor Nagios. 

Para comprobar que hay comunicación entre el NRPE del servidor Nagios y NSClient, vamos dentro del servidor al directorio libexec:

``` 
./check_nrpe -H ipclientewindows
``` 

Ahora vamos dentro del cliente Windows al directorio de NSClient, en Archivos de programa, y accedemos al archivo de configuración de NSClient. Ahí revisamos el allowed hosts y algunas reglas de monitoreo ya especificadas anteriormente. 

Si hacemos algún cambio ahí, vamos a inicio, services.msc y en servicios locales, en Administrar credenciales accedemos y refrescamos las de NSClient.


## 7. Notificaciones por correo

Dentro del servidor Nagios vamos a nuestro directorio habitual de trabajo:

``` 
cd /usr/local/nagios/etc/objects
```

Y pasamos al archivo de contactos:

``` 
nano contacts.cfg
``` 

Aquí podemos definir los contactos a los cuales queremos enviar las notificaciones vía email.

``` 
define contact {

    contact_name        davidseg
    use                 generic-contact

    alias               Nagios Admin
    email               correo del usuario
}
```

Y podemos definir un grupo de contactos justo debajo en el archivo:

``` 
define contactgroup {

    contactgroup_name   admins
    alias               Nagios Administrators
    members             davidseg,monica
}
```

Una vez guardados los cambios, vamos al archivo templates.cfg para definir otros aspectos de las notificaciones, como por ejemplo el periodo. 

Visto esto, vamos al directorio home del servidor Nagios para descargar un repositorio REMI que nos va a permitir trabajar con PHP para enviar notificaciones por email vía SMTP.

Trás el REMI, descargamos los siguientes paquetes PHP:

``` 
sudo apt install php php-cli php-gd php-curl php-zip php-intl php-mbstring php-xml -y
``` 

Vamos a hacerlo con PHP porque es una manera sencilla de lidiar con los SMTP que usan TLS. Sin embargo, si usamos cuenta de Google habrá que activar el poder trabajar con aplicaciones menos seguras, al no estar Nagios creado por Google. 

Ahora instalamos un aplicativo de PHP llamado composer. 

``` 
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
``` 

Hacemos un ls para comprobar que lo tenemos y hacemos uso del siguiente comando:

``` 
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
``` 

Para comprobar que esta instalado, ponemos composer y ejecutamos. 

Acto seguido, vamos a tirar del proyecto smtp cli. Este es el programa basado en PHP que nos va a realizar el envío de correos.

```
wget https://github.com/boolean-world/smtp-cli/archive/master.zip
``` 

Y lo pasamos al directorio opt:

``` 
unzip -d /opt master.zip
```

Pasamos al proyecto dentro de ese directorio:

``` 
cd /opt/smtp-cli-master/
``` 

Y dentro escribimos **composer install**. Y creamos el archivo config.json y dentro de este archivo vamos a especificar los datos de SMTP de nuestra cuenta con la contraseña, puerto que vamos a querer que use para el envío de correos. 

``` 
{
 "host": "smtp.gmail.com",
 "username": "usuario@gmail.com",
 "password": "password",
 "secure": "tls",
 "port": 587       
}
``` 

Ahora hay que adaptar Nagios para que use los correos en base al archivo smtp-cli.php

Para ello vamos a nuestro directorio habitual de nagios (/usr/local/nagios/etc/objects), y al archivo command.cfg. Aquí solo hay que quitar dentro del command notify-host-by-email, justo después del parámetro LONGDATETIME la ruta /usr/bin/mail -s. Y ponemos /opt/smtp-cli-master/smtp-cli.php

Hacemos lo mismo en el command notify-service-by-email. Y comprobamos con nagioscheck, como ya vimos en apartados anteriores. Y recargamos con nagios reload. 


## 8. Hostgroups y servicegroups

Vamos a ver como agrupar hosts y servicios de manera que podamos tener en una visión global en la interfaz de Nagios un grupo de servidores o servicios. 

Dentro del directorio habitual de trabajo (/usr/local/nagios/etc/objects), vamos por ejemplo al archivo de configuración de los clientes Linux (linux.cfg).

Entre la definición de host y de los servicios añado el define hostgroup:

``` 
### Definición de hostgroups ###

define hosstgroup{
        hostgroup_name  Servidores_Linux
        alias           Servidores_Linux
        members         cliente_linux
}
``` 

Siempre chequeamos cuando hagamos un cambio y recargamos. 

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
``` 

Cuando lo ejecutamos nos muestra, en caso de que los hubiera, los errores presentes en nuestros archivos de configuración.

Si hemos de corregir alguno, reiniciamos Nagios:

``` 
systemctl restart nagios
``` 

Y si no:

``` 
nagios reload
``` 

Para añadir un hostgroup dentro de un servicio lo hacemos del siguiente modo:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        #host_name                 cliente_linux
        hostgroup_name             Servidores_Linux                   
        service_description        Hard Disk
        contacts                   davidseg
        check_command              check_nrpe!check_hda1
}
``` 

El define servicegroup lo añadimos al final del archivo, debajo de la definición de los servicios.

``` 
### Definición de grupos de servicios ###

define servicegroup{
        servicegroup_name          Todos_los_discos
        alias                      Todos_los_discos
        members                    cliente_linux,Hard Disk
}
``` 

En **members** ponemos servidor y servicio que queremos añadir al grupo. 

Siempre chequeamos cuando hagamos un cambio y recargamos. 

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
``` 

Cuando lo ejecutamos nos muestra, en caso de que los hubiera, los errores presentes en nuestros archivos de configuración.

Si hemos de corregir alguno, reiniciamos Nagios:

``` 
systemctl restart nagios
``` 

Y si no:

``` 
nagios reload
``` 

Otra manera de definir un servicegroup sería así:

``` 
### Definición de grupos de servicios ###

define servicegroup{
        servicegroup_name          Todos_los_discos
        alias                      Todos_los_discos
}
``` 

Y en la definición del servicio hacemos lo siguiente:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        #host_name                 cliente_linux
        hostgroup_name             Servidores_Linux                   
        service_description        Hard Disk
        contacts                   davidseg
        servicegroups              Todos_los_discos
        check_command              check_nrpe!check_hda1
}
```

Y volvemos a comprobar que todo va bien y recargamos nagios. 


## 9. Manejo de contactos

Dentro del directorio habitual de trabajo (/usr/local/nagios/etc/objects), vamos al template.cfg

Y si observamos bajando en el archivo, en define host nos aparece un apartado llamado contact_groups. Por defecto, el grupo de contacto al cual se van a enviar las notificaciones va a ser admins. 

Bien, si ahora vamos al archivo contact.cfg, en define contactgroup tenemos lo siguiente:

``` 
define contactgroup {
    
    contactgroup name           admins
    alias                       Nagios Administrators
    members                     davidseg, monica
}
``` 

¿Y si yo quisiera enviar notificaciones de un servicio en concreto a solo un usuario específico? En este caso, a modo de ejemplo, iriamos al archivo de configuración de los servidores cliente Linux (linux.cfg) y dentro del servicio en cuestión añado el campo contacts:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        #host_name                 cliente_linux
        hostgroup_name             Servidores_Linux                   
        service_description        Hard Disk
        contacts                   davidseg
        servicegroups              Todos_los_discos
        check_command              check_nrpe!check_hda1
}
```

También podemos hacerlo en la definición del host. 

Ahora yo puedo definir otro contactgroup dentro de contact.cfg

``` 
define contactgroup {
    
    contactgroup name           devops
    alias                       Nagios Administrators
    members                     davidseg, monica
}
``` 

Y lo añado del siguiente modo en el servicio:

``` 
### Definición de servicios ###

define service{
        use                        generic-service
        host_name                  cliente_linux               
        service_description        Hard Disk
        contact_groups             devops
        check_command              check_nrpe!check_hda1
}
```









































































