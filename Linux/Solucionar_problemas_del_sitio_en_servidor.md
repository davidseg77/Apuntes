# Cómo solucionar problemas comunes del sitio en un servidor Linux

## 1. Introducción

Todo el mundo tiene problemas con su servidor web o sitio en un momento u otro. Aprender dónde buscar cuando se encuentra con un problema y qué componentes son los posibles culpables le ayudará a solucionar estos problemas de la forma más rápida y sólida posible.

En esta guía, comprenderá cómo solucionar estos problemas para que pueda volver a poner su sitio en funcionamiento.

Los analizaremos con más profundidad en las secciones siguientes, pero por ahora, aquí hay una lista de verificación de elementos a considerar:

* ¿Está instalado su servidor web?
* ¿Está funcionando el servidor web?
* ¿Es correcta la sintaxis de los archivos de configuración de su servidor web?
* ¿Están abiertos los puertos que configuró (no bloqueados por un firewall)?
* ¿Tu configuración de DNS te dirige al lugar correcto?
* ¿La raíz del documento apunta a la ubicación de sus archivos?
* ¿Su servidor web ofrece los archivos de índice correctos?
* ¿Son correctos los permisos y la propiedad de las estructuras de archivos y directorios?
* ¿Está restringiendo el acceso a través de sus archivos de configuración?
* Si tiene un backend de base de datos, ¿se está ejecutando?
* ¿Puede su sitio conectarse a la base de datos correctamente?

Estos son algunos de los problemas comunes con los que se encuentran los administradores cuando un sitio no funciona correctamente. Por lo general, el problema exacto se puede reducir echando un vistazo a los archivos de registro de los diferentes componentes y haciendo referencia a las páginas de error que se muestran en su navegador.

A continuación, analizaremos cada uno de estos escenarios para que pueda asegurarse de que sus servicios estén configurados correctamente.


## 2. Verifique los registros

Antes de intentar localizar un problema a ciegas, intente comprobar los registros de su servidor web y cualquier componente relacionado. Por lo general, estarán en /var/log, un subdirectorio específico del servicio.

Por ejemplo, si tiene un servidor Apache ejecutándose en un servidor Ubuntu, de forma predeterminada los registros se mantendrán en formato /var/log/apache2. Verifique los archivos en este directorio para ver qué tipo de mensajes de error se están generando /var/log. Si tiene un backend de base de datos que le está dando problemas, es probable que también mantenga sus registros.

Otras cosas que se deben verificar son si los procesos mismos dejan mensajes de error cuando se inician los servicios. Si intenta visitar una página web y obtiene un error, la página de error también puede contener pistas (aunque no tan específicas como las líneas de los archivos de registro).

Utilice un motor de búsqueda para intentar encontrar información relevante que pueda indicarle la dirección correcta. En muchos casos, puede resultar útil pegar un fragmento de sus registros directamente en un motor de búsqueda para encontrar otros ejemplos del mismo problema. Los pasos a continuación pueden ayudarlo a solucionar más problemas.

## 3. ¿Está instalado su servidor web?

Lo primero que probablemente necesitará para servir sus sitios correctamente es un servidor web. En algunos casos, sus páginas web pueden ser atendidas directamente por un contenedor Docker o alguna otra aplicación, y en realidad no necesitará instalar un servidor web dedicado, pero la mayoría de las implementaciones incluirán al menos uno.

La mayoría de las personas habrán instalado un servidor antes de llegar a este punto, pero hay algunas situaciones en las que es posible que haya desinstalado accidentalmente el servidor al realizar otras operaciones de paquetes.

Si está en un sistema Ubuntu o Debian y necesita instalar el servidor web Apache, puede escribir:

``` 
sudo apt-get update
sudo apt-get install apache2
``` 

En estos sistemas, el proceso Apache se llama apache2.

Si está ejecutando Ubuntu o Debian y desea el servidor web Nginx, puede escribir:

``` 
sudo apt-get update
sudo apt-get install nginx
``` 

En estos sistemas, el proceso Nginx se llama nginx.

Si está ejecutando RHEL, Rocky Linux o Fedora y necesita utilizar el servidor web Apache, puede escribir:

```
sudo dnf install httpd
``` 

En estos sistemas, el proceso Apache se llama httpd.

Si está ejecutando RHEL, Rocky Linux o Fedora y desea utilizar Nginx, puede escribir esto. Nuevamente, elimine el "sudo" si ha iniciado sesión como root:

``` 
sudo dnf install nginx
``` 

En estos sistemas, el proceso Nginx se llama nginx. A diferencia de Ubuntu, Nginx no se inicia automáticamente después de instalarse en estas distribuciones basadas en RPM. Continúe leyendo para saber cómo iniciarlo.


## 4. ¿Está funcionando su servidor web?

Ahora que estás seguro de que tu servidor está instalado, ¿está ejecutándose?

Hay muchas formas de saber si el servicio se está ejecutando o no. Un método que es bastante multiplataforma es utilizar el netstat comando.

Ejecutar netstat con las -plunt banderas le indicará todos los procesos que utilizan puertos en el servidor. Para obtener más información sobre la ejecución netstat, puede consultar Cómo utilizar Top, Netstat, Du y otras herramientas para monitorear los recursos del servidor. Luego puede grepobtener el resultado del netstatnombre del proceso que está buscando:

``` 
sudo netstat -plunt | grep nginx
Output
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      15686/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      15686/nginx: master
``` 

Puede cambiar nginx el nombre del proceso del servidor web en su servidor. Si ve una línea como la de arriba, significa que su proceso está en funcionamiento. Si no obtiene ningún resultado, significa que consultó el proceso incorrecto o que su servidor web no se está ejecutando.

Si su servidor web no se está ejecutando, puede iniciarlo usando el sistema de inicio de su distribución de Linux. La mayoría del software diseñado para ejecutarse en segundo plano se registrará en el sistema de inicio después de instalarlo, de modo que pueda iniciarlo y detenerlo mediante programación. La mayoría de las distribuciones ahora también usan el mismo sistema de inicio, systemd que proporciona el systemctl comando.

Por ejemplo, podrías iniciar el nginx servicio escribiendo:

``` 
sudo systemctl start nginx
``` 

Si tu servidor web se inicia, puedes consultar netstatnuevamente para verificar que todo esté correcto.


## 5. ¿Es correcta la sintaxis del archivo de configuración de su servidor web?

Si su servidor web no pudo iniciarse, esto suele ser una indicación de que sus archivos de configuración necesitan algo de atención. Tanto Apache como Nginx requieren un estricto cumplimiento de su sintaxis para que su configuración se analice correctamente.

Los archivos de configuración para los servicios del sistema generalmente se encuentran dentro de un subdirectorio del /etc/directorio que lleva el nombre del proceso en sí.

Por ejemplo, puedes acceder al directorio de configuración principal de Apache en Ubuntu escribiendo:

``` 
cd /etc/apache2
``` 

El directorio de configuración de Apache en RHEL, Rocky y Fedora también refleja el nombre de RHEL para ese proceso:

``` 
cd /etc/httpd
```

La configuración se distribuirá entre muchos archivos diferentes. Al intentar y no poder iniciar un servicio, normalmente se producirán errores que apuntan al archivo de configuración y a la línea donde se encontró el problema por primera vez. Puedes empezar a investigar ese archivo.

Cada uno de estos servidores web también proporciona una forma de validar la sintaxis de configuración de sus archivos.

Si está utilizando Apache, puede utilizar el comando apache2ctl o apachectl para comprobar sus archivos de configuración en busca de errores de sintaxis:

``` 
apache2ctl configtest
``` 

``` 
Output
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
```

Syntax OK Básicamente significa que no hay errores importantes que impidan que el servidor se ejecute, y cada mensaje impreso antes es un error menor o una advertencia. En este caso, Could not reliably determine the server's fully qualified domain name refleja una configuración de Apache lista para usar en un servidor que aún no se ha configurado con un nombre de dominio, pero al que aún se debe poder acceder mediante su dirección IP.

Si tiene un servidor web Nginx, puede ejecutar una prueba similar escribiendo:

``` 
sudo nginx -t
Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
``` 

Si elimina un punto y coma al final de una línea de configuración /etc/nginx/nginx.conf (un error común para las configuraciones de Nginx), recibirá un mensaje como este:

``` 
sudo nginx -t 
``` 

```
Output
nginx: [emerg] invalid number of arguments in "tcp_nopush" directive in /etc/nginx/nginx.conf:18
nginx: configuration file /etc/nginx/nginx.conf test failed
```

Hay un número no válido de argumentos porque Nginx busca un punto y coma para finalizar las declaraciones. Si no encuentra ninguno, baja a la siguiente línea y lo interpreta como argumentos adicionales para la última línea.

Puede ejecutar estas pruebas para encontrar problemas de sintaxis en sus archivos. Solucione los problemas a los que hace referencia hasta que pueda conseguir que los archivos pasen la prueba.


## 6. ¿Están abiertos los puertos que configuró?

Generalmente, los servidores web se ejecutan en el puerto 80 para el tráfico web HTTP y utilizan el puerto 443 para el tráfico HTTPS cifrado con TLS/SSL. Para que los usuarios puedan llegar a su sitio, estos puertos deben ser accesibles.

Puede probar si su servidor tiene el puerto abierto ejecutándolo netcat desde su máquina local.

Deberá usar la dirección IP de su servidor remoto y decirle qué puerto verificar, de esta manera:

``` 
nc -z 111.111.111.111 80
``` 

Esto comprobará si el puerto 80 está abierto en el servidor en 111.111.111.111. Si está abierto, el comando volverá de inmediato. Si no está abierto, el comando intentará continuamente establecer una conexión, sin éxito. Puede detener este proceso presionando CTRL+C en la ventana del terminal.

Si no se puede acceder a sus puertos web, debe revisar la configuración de su firewall. Es posible que deba abrir el puerto 80 o el puerto 443.


## 7. ¿Su configuración de DNS lo dirige al lugar correcto?

Si puede acceder a su sitio mediante su dirección IP, pero no a través de un nombre de dominio que haya configurado, es posible que deba revisar su configuración de DNS.

Para que los visitantes lleguen a su sitio a través de su nombre de dominio, debe tener un registro "A" o "AAAA" que apunte a la dirección IP de su servidor en la configuración DNS. Puede consultar el registro "A" de su dominio utilizando el hostcomando en un local o:

``` 
host -t A example.com
Output
example.com has address 93.184.216.119
``` 

La línea que se le devuelve debe coincidir con la dirección IP de su servidor. Si necesita verificar un registro “AAAA” (para conexiones IPv6), puede escribir:

``` 
host -t AAAA example.com
Output
example.com has IPv6 address 2606:2800:220:6d:26bf:1447:1097:aa7
``` 

Tenga en cuenta que cualquier cambio que realice en sus registros DNS puede tardar algún tiempo en propagarse, dependiendo de su registrador de nombres de dominio. A veces puede resultar útil utilizar un sitio como https://www.whatsmydns.net/ para comprobar cuándo los cambios de DNS han entrado en vigor a nivel mundial (normalmente aproximadamente media hora). Es posible que reciba resultados inconsistentes en estas consultas después de un cambio, ya que su solicitud a menudo llegará a diferentes servidores que aún no están actualizados.


## 8. Asegúrese de que sus archivos de configuración también manejen su dominio correctamente

Si su configuración de DNS es correcta, es posible que también desee verificar los archivos de su host virtual Apache o los archivos de bloqueo del servidor Nginx para asegurarse de que estén configurados para responder a las solicitudes de su dominio.

En Apache, su archivo de host virtual podría verse así:

``` 
/etc/apache2/sites-enabled/000-default.conf
&lt;VirtualHost *:80&gt;
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html
. . .
``` 

Este host virtual está configurado para responder a cualquier solicitud en el puerto 80 del dominio example.com.

Un fragmento similar en Nginx podría verse así:

``` 
/etc/nginx/sitios habilitados/predeterminado
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .
``` 

Estos dos bloques están configurados para responder a los mismos tipos de solicitudes predeterminados.


## 9. ¿La raíz del documento apunta a la ubicación de sus archivos?

Otra consideración es si su servidor web apunta a la ubicación correcta del archivo.

Cada servidor virtual en Apache o bloque de servidores en Nginx está configurado para apuntar a un puerto o directorio local específico. Si esto se configura incorrectamente, el servidor arrojará un mensaje de error cuando intente acceder a la página.

En Apache, la raíz del documento se configura mediante la DocumentRoot directiva:

``` 
/etc/apache2/sitios habilitados/predeterminado
&lt;VirtualHost *:80&gt;
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html
. . .
``` 

Esta línea le dice a Apache que debe buscar los archivos para este dominio en el /var/www/html directorio. Si sus archivos se guardan en otro lugar, deberá modificar esta línea para que apunte a la ubicación correcta.

En Nginx, la root directiva configura lo mismo:

``` 
/etc/nginx/sitios habilitados/predeterminado
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .
``` 

En esta configuración, Nginx busca archivos para este dominio en el /usr/share/nginx/html directorio.


## 10. ¿Su servidor web proporciona los archivos de índice correctos?

Si la raíz de su documento es correcta y sus páginas de índice no se muestran correctamente cuando visita su sitio o una ubicación de directorio en su sitio, es posible que sus índices estén configurados incorrectamente.

Dependiendo de la complejidad de sus aplicaciones web, muchos servidores web seguirán sirviendo de forma predeterminada archivos de índice. Suele ser un index.html archivo o un index.php archivo según su configuración.

En Apache, puede encontrar una línea en su archivo de host virtual que configura explícitamente el orden de índice que se utilizará para directorios específicos, como este:

``` 
/etc/apache2/sitios habilitados/predeterminado
&lt;Directory /var/www/html&gt;
    DirectoryIndex index.html index.php
&lt;/Directory&gt;
``` 

Esto significa que cuando se sirve el directorio, Apache buscará un archivo llamado index.html primero e intentará servir index.php como copia de seguridad si no se encuentra el primer archivo.

Puede establecer el orden que se utilizará para servir archivos de índice para todo el servidor editando el mods-enabled/dir.conf archivo, lo que establecerá los valores predeterminados para el servidor. Si su servidor no proporciona un archivo de índice, asegúrese de tener un archivo de índice en su directorio que coincida con una de las opciones de su archivo.

En Nginx, la directiva que hace esto se llama indexy se usa así:

``` 
/etc/nginx/sitios habilitados/predeterminado
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .
``` 


## 11. ¿Están establecidos correctamente los permisos y la propiedad?

Para que el servidor web proporcione archivos correctamente, debe poder leerlos y tener acceso a los directorios donde se guardan. Esto se puede controlar a través de los permisos y la propiedad de archivos y directorios.

Para leer archivos, los directorios que contienen el contenido deben ser legibles y ejecutables por la cuenta de usuario asociada con el servidor web. En Ubuntu y Debian, Apache y Nginx se ejecutan como el usuario www-data que es miembro del www-data grupo.

En RHEL, Rocky y Fedora, Apache se ejecuta bajo un usuario llamado apache que pertenece al apache grupo. Nginx se ejecuta bajo un usuario llamado nginx que forma parte del nginx grupo.

Con esto en mente, puedes mirar los archivos y carpetas que estás alojando:

``` 
ls -l /path/to/web/root
```

Los directorios deben ser legibles y ejecutables por el usuario o grupo web, y los archivos deben ser legibles para poder leer el contenido. Para cargar, escribir o modificar contenido, los directorios también deben poder escribirse y los archivos también deben poder escribirse.

Para modificar la propiedad de un archivo, puede utilizar chown:

``` 
sudo chown user_owner:group_owner /path/to/file
``` 

Esto también se puede hacer en un directorio. Puede cambiar la propiedad de un directorio y de todos los archivos que contiene pasando la -R bandera:

``` 
sudo chown -R user_owner:group_owner /path/to/file
``` 


## 12. ¿Está restringiendo el acceso a través de sus archivos de configuración?

La configuración de su servidor web también puede estar configurada para denegar el acceso a los archivos que está intentando publicar.

En Apache, esto se configuraría en el archivo del host virtual de ese sitio, o mediante un .htaccess archivo ubicado en el propio directorio.

Dentro de estos archivos, es posible restringir el acceso de diferentes maneras. Los directorios se pueden restringir así en Apache 2.4+:

``` 
/etc/apache2/sitios habilitados/predeterminado
&lt;Directory /usr/share&gt;
    AllowOverride None
    Require all denied
&lt;/Directory&gt;
``` 

En Nginx, estas restricciones tomarán la forma de deny directivas y estarán ubicadas en los bloques de su servidor o en los archivos de configuración principales:

``` 
/etc/nginx/sitios habilitados/predeterminado
location /usr/share {
    deny all;
}
``` 


## 13. Si tiene un backend de base de datos, ¿está ejecutándose?

Si su sitio depende de un backend de base de datos como MySQL, PostgresSQL, MongoDB, etc., debe asegurarse de que esté activo y disponible.

Puede hacerlo de la misma manera que comprobó que el servidor web se estaba ejecutando. Nuevamente, puede buscar entre procesos en ejecución con netstat y grep:

``` 
sudo netstat -plunt | grep mysql
Output
tcp        0      0 127.0.0.1:3306        0.0.0.0:*         LISTEN      3356/mysqld
``` 

Como puede ver, el servicio se está ejecutando en esta máquina. Asegúrese de conocer el nombre con el que se ejecuta su servicio cuando lo busque.

Una alternativa es buscar el puerto en el que se ejecuta su servicio. Mire la documentación de su base de datos para encontrar el puerto predeterminado en el que se ejecuta (el valor predeterminado de MySQL es 3356), o verifique sus archivos de configuración.


## 14. Si tiene un backend de base de datos, ¿puede su sitio conectarse correctamente?

El siguiente paso a seguir si está solucionando un problema con el backend de una base de datos es ver si puede conectarse correctamente. Por lo general, esto significa verificar los archivos que lee su sitio para conocer la información de la base de datos.

Por ejemplo, para un sitio de WordPress, la configuración de conexión a la base de datos se almacena en un archivo llamado wp-config.php. Debe verificar que DB_NAME, DB_USER y DB_PASSWORD sean correctos para que su sitio se conecte a la base de datos.

Puede probar si el archivo tiene la información correcta intentando conectarse a la base de datos manualmente en la línea de comando. La mayoría de las bases de datos admitirán una sintaxis similar a MySQL:

``` 
mysql -u DB_USER_value -pDB_PASSWORD_value DB_NAME_value
```

Si no puede conectarse utilizando los valores que encontró en el archivo, es posible que deba modificar los permisos de acceso de sus bases de datos.

Si todo lo demás falla, verifique los registros nuevamente.

En realidad, comprobar los registros debería ser el primer paso, pero también es un buen último paso antes de pedir más ayuda.

Si ha llegado al final de su capacidad para solucionar problemas por su cuenta y necesita ayuda, obtendrá ayuda más relevante más rápidamente al proporcionar archivos de registro y mensajes de error. Los administradores experimentados probablemente tendrán una buena idea de lo que está sucediendo si les proporciona la información que necesitan.

