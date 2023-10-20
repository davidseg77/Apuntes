# Instalar Servidor Proxy SQUID CACHE en Ubuntu

## 1. Escenario

Voy a instalar este servidor en un Ubuntu Server. Desde un cliente, me conecto por ssh al servidor. Por ejemplo, desde una terminal Linux alojada en local. Si no pudiera conectarme po ssh, lo instalo:

``` 
sudo apt-get update
sudo apt-get install openssh-server
```

## 2. Instalar Proxy Squid

``` 
sudo apt-get update
sudo apt-get install squid
```

Comprobamos instalacion

``` 
squid -v
squid -h
```

Comprobamos estado del servicio squid

``` 
sudo service squid status
```

## 3. Configuración por defecto de Squid

Comenzamos haciendo una copia de seguridad del archivo de configuración. Es recomendable.

``` 
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.backup
```

Vamos al original con vim para eliminar todos los comentarios.

``` 
:g/^\s*#/d (Va a descomentar las líneas comentadas)
:g/^$/d (Va a eliminar todas las líneas vacías)
:wq (Guarda los cambios y sale)
```

## 4. Comprobar funcionamiento Proxy Squid

Desde un cliente, hacemos un curl a través del servidor proxy:

``` 
curl -x http://ip del servidor proxy:3128 (puerto de squid) -i http://www.google.com/
```

Se puede comprobar también en Firefox a través de la extensión FoxyProxy. 


## 5. Configurar Proxy Squid

Creamos nuestro fichero de reglas

```
sudo nano /etc/squid/conf.d/myrules.conf
```

Dentro creamos las siguientes líneas:

``` 
#Crear ACL para nuestra red local
acl miredlocal src 192.168.1.0/24 (Subred del cliente o IP del cliente)
#Permitir navegación Web para dicha ACL
http_access allow miredlocal
```

Guardamos y recargamos la configuración

``` 
service squid reload

o

sudo squid -k reconfigure (recomendable)
```

Si comprobamos de nuevo con curl, ya tendremos acceso vía proxy a la url indicada.


## 6. Registros log en Squid

Ubicación de los logs.

``` 
ls /var/log/squid
```

Si abro una terminal secundaria del servidor, mediante el siguiente comando puedo hacer un registro detallado al momento de los logs de Squid:

``` 
sudo tail -f /var/log/squid/access.log
```

* access.log = respuestas de las peticiones de clientes
* cache.log = mensajes de error de configuración


## 7. Bloquear dominios con Squid

Añadimos dos nuevas reglas a nuestro archivo de reglas:

``` 
#Crear ACL con webs de RRSS con argumentos
acl filtro_rss dstdomain .facebook.com .instagram.com
#Denegar el acceso a dicho elementos ACL
http_access deny filtro_rrss
```

Estas reglas se insertan antes de permitir navegación, es decir, entre la primera y la segunda creadas anteriormente. 

Volvemos a recargar el servicio con reconfigure y hacemos la comprobación desde el cliente con curl -x.

Como podrá apreciarse, no podremos conectarnos a facebook ni a Instagram. También podemos comprobarlo a través de la monitorización hecha con los logs en la terminal secundaria.

También puede hacerse este bloqueo a través de una regla ACL indicada en un archivo. Para ello insertamos la regla en el archivo myrules:

``` 
#Crear ACL con webs de RRSS con ficheros
acl filtro_rrss dstdomain "/etc/squid/dominios-denegados"
```

Y en un fichero creado por mi llamado dominios-denegados, yo inserto las webs que quiero bloquear:

``` 
.facebook.com
.instagram.com
```

Si hacemos el reconfigure y comprobamos el resultado será similar al hecho con las reglas ACL con argumento.


## 8. Bloquear por expresiones regulares con Squid

url_regex busca en toda la URL la expresión regular especificada en la regla. 

Creo un fichero donde voy a introducir esas expresiones.

``` 
sudo nano block-exp
torrent 
crack
```

Y creo la regla ACL en myrules

``` 
#Crear ACL con expresiones regulares a prohibir
acl block-exp url_regex "/etc/squid/block-exp"
#Denegar navegación al elemento acl block-exp
http_access deny block-exp
```

Compruebo el funcionamiento con el cliente:

``` 
curl -x http://ipservidor:3128 -I http://www.crackstation.com/
```

Y no nos dejará acceder al incluir esa expresión crack. 


## 9. Bloquear por días y horas con Squid

time para indicar días y horas.

Creamos una regla ACL para comprobar su funcionamiento:

```
#Crear ACL indicando los días y horarios de trabajo
acl WORKING time MTWHF 08:30-17:30
#Denegar navegación fuera del horario de trabajo
http_access deny !WORKING
```

Mediante las iniciales del día lo indico. El jueves se indica con H.

Y comprobaríamos de nuevo con curl.


## 10. Personalizar el mensaje de error

Ubicación de los mensajes de error

``` 
ls /usr/share/squid/errors/
```

Para crear el mensaje de error:

``` 
sudo nano /usr/share/squid/errors/Spanish/ERR_BLOCKED_RRSS
```

Y dentro del archivo puedo crear una especie de mensaje de error.

Hecho esto, en nuestro fichero de configuración definimos el directorio.

``` 
#Definir el directorio
error_directory /usr/share/squid/errors/Spanish
```

Esto lo ponemos al principio del archivo de configuración. Y al final la siguiente regla:

``` 
#Aplicar un mensaje a un elemento ACL
deny_info ERR_BLOCKED_RRSS filtro_rrss
```

Guardamos, recargamos con reconfigure y hacemos un curl para probar. Si tratamos de acceder a facebook, nos dirá que no podemos acceder con el mensaje editado.

## 11. Autenticación de usuarios en Squid

Vamos a instalar la herramienta de Apache2 Utils

```
sudo apt-get install apache2-utils
```

Creamos el fichero donde vamos a almacenar los usuarios con sus contraseñas. 

``` 
sudo htpasswd -c /etc/squid/squid_password hugo 
```

Y creo otro:

```
sudo htpasswd /etc/squid/squid_password marta 
```

Comprobamos con cat. Y buscamos el fichero basic_ncsa_auth.

```
sudo find / basic_ncsa_auth | grep basic_ncsa_auth
```

Para probarlo:

``` 
/usr/lib/squid/basic_ncsa_auth /etc/squid/squid_password
```

Al ejecutar, se nos quedará un caracter parpadeando. Hay habrá que poner el usuario y su contraseña. Nos dirá OK si es un usuario autenticado.

Una vez creado el fichero, ahora toca configurar squid.conf para verificar a esos usuarios cuando se conectan.

Vamos a squid.conf y ponemos estas líneas arriba:

``` 
#(1) Indicar la ubicación de basic_ncsa_auth y del fichero con los usuarios
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_password
#(2) Indicamos el mensaje para el cliente
auth_param basic realm Autenticación de usuarios de Squid
#(3)Nº máx de login
auth_param basic children 5
#(4)Duración más del login
auth_param basic credentialsttl 2 hours 
```

Y de ahí vamos a myrules, donde tenemos que crear una ACL.

```
#(5) elemento ACL donde se requiere autenticación de usuario
acl usuarios proxy_auth REQUIRED
#(6)Permitir solo el acceso http para usuarios autenticados
http_access allow usuarios
```

Para probarlos, primero recargamos con reconfigure (sudo squid -k reconfigure). Y hacemos el curl con el usuario y su contraseña de la siguiente manera:

``` 
curl -x http://hugo:1234@ipservidor:3128 -I http://www.google.com/
```

## 12. Comprobar si Squid funciona correctamente

Para ello instalamos squidclient en el propio servidor

``` 
sudo apt install squidclient
```

Y mostramos estadísticas.

``` 
squidclient agr:info
```

## 13. Configurar la caché de Squid

En squid.conf añado esta línea en la parte de arriba.

``` 
#Cache del disco
cache_dir ufs /var/spool/squid 100 16 256
```

* ufs es el sistema de almacenamiento que usa.
* /var/spool/squid es la unidad y carpeta donde se guardarán los ficheros de la caché.
* 100 = tamaño máximo que tendrá la caché en MB.
* 16 = número de carpetas de primer nivel.
* 256 = número de subcarpetas de segundo nivel.

Justo debajo podemos añadir también la caché de buffer.

```
#Cache de Buffer
cache_mem 512 MB
```

Guardamos y reconfigure. 


## 14. Listas públicas de bloqueo de dominios

Probamos descargando la blackweb de maravento, un repo que cuenta con infinidad de listas públicas bastante actualizadas.

``` 
wget -q -N https://raw.githubusercontent.com/maravento/blackweb/master/blackweb.tar.gz && cat blackweb.tar.gz* | tar xzf -

``` 

Si diera fallo, primero descargamos el fichero y luego lo extraemos por separado.

```
wget -q -N https://raw.githubusercontent.com/maravento/blackweb/master/blackweb.tar.gz
``` 

```
tar -xzf blackweb.tar.gz
``` 

Y ahora creamos en myrules una ACL indicando la dirección del archivo.

``` 
#Crear ACL indicando la dirección del fichero
acl blackweb dstdomain "ruta donde esté el archivo descargado blackweb.txt"
#Denegar el acceso a los dominios blackweb
http_access deny blackweb
```

Guardamos y reconfiguramos.

Con esto decimos que deniegue el acceso a todas aquellas URL contenidas dentro de la lista negra. Esto viene bien para evitar el acceso a ciertas webs en el trabajo.



