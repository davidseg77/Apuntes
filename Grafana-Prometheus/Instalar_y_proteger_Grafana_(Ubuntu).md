# Cómo instalar y proteger Grafana en Ubuntu 22.04

## 1. Introducción

Grafana es una herramienta de monitoreo y visualización de datos de código abierto que se integra con datos complejos de fuentes como Prometheus , InfluxDB , Graphite y ElasticSearch . Grafana le permite crear alertas, notificaciones y filtros ad hoc para sus datos y, al mismo tiempo, facilita la colaboración con sus compañeros de equipo a través de funciones integradas para compartir.

En este tutorial, instalará Grafana y lo protegerá con un certificado SSL y un proxy inverso Nginx . Una vez que haya configurado Grafana, tendrá la opción de configurar la autenticación de usuario a través de GitHub, lo que le permitirá organizar mejor los permisos de su equipo.

## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un servidor Ubuntu 22.04 configurado, incluido un usuario no root con sudo privilegios y un firewall configurado con ufw.
* Un nombre de dominio completamente registrado. Este tutorial se utiliza your_domain en todas partes. Puede comprar un nombre de dominio en Namecheap , obtener uno gratis en Freenom o utilizar el registrador de dominio de su elección.
* Los siguientes registros DNS configurados para su servidor:
  - Un registro A your_domain que apunta a la dirección IP pública de su servidor.
  - Un registro A que apunta a la dirección IP pública de su servidor.www.your_domain
* Nginx configurado, incluido un bloque de servidor para su dominio.
* Un bloque de servidor Nginx con Let's Encrypt configurado.
* Opcionalmente, para configurar la autenticación de GitHub, necesitará una cuenta de GitHub asociada con una organización.

## 3. Instalar Nginx en Ubuntu con bloque de servidor para el dominio

### 3.1. Paso 1: instalar Nginx

Debido a que Nginx está disponible en los repositorios predeterminados de Ubuntu, es posible instalarlo desde estos repositorios utilizando el apt sistema de empaquetado.

Dado que esta es nuestra primera interacción con el apt sistema de empaquetado en esta sesión, actualizaremos nuestro índice de paquetes local para que tengamos acceso a los listados de paquetes más recientes. Después podremos instalar nginx:

``` 
sudo apt update
sudo apt install nginx
``` 

Presione Y cuando se le solicite confirmar la instalación. Si se le solicita que reinicie algún servicio, presione ENTER para aceptar los valores predeterminados y continuar. apt instalará Nginx y cualquier dependencia requerida en su servidor.

### 3.2. Paso 2: ajustar el firewall

Antes de probar Nginx, es necesario configurar el software de firewall para permitir el acceso al servicio. Nginx se registra como un servicio tras ufw la instalación, lo que facilita el acceso a Nginx.

Enumere las configuraciones de la aplicación ufwcon las que sabe trabajar escribiendo:

``` 
sudo ufw app list
``` 

Debería obtener una lista de los perfiles de la aplicación:

``` 
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
``` 

Como lo demuestra el resultado, hay tres perfiles disponibles para Nginx:

* **Nginx Full:** este perfil abre tanto el puerto 80 (tráfico web normal sin cifrar) como el puerto 443 (tráfico cifrado TLS/SSL)
* **Nginx HTTP:** este perfil abre solo el puerto 80 (tráfico web normal sin cifrar)
* **Nginx HTTPS:** este perfil abre solo el puerto 443 (tráfico cifrado TLS/SSL)
  
Se recomienda que habilite el perfil más restrictivo que aún permitirá el tráfico que ha configurado. Por el momento, sólo necesitaremos permitir el tráfico en el puerto 80.

Puedes habilitar esto escribiendo:

``` 
sudo ufw allow 'Nginx HTTP'
``` 

Puede verificar el cambio escribiendo:

```
sudo ufw status
```

El resultado indicará qué tráfico HTTP está permitido:

``` 
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
``` 

## 3.3. Paso 3: comprobar su servidor web

Al final del proceso de instalación, Ubuntu 22.04 inicia Nginx. El servidor web ya debería estar funcionando.

Podemos verificar con el systemd sistema de inicio para asegurarnos de que el servicio se esté ejecutando escribiendo:

``` 
systemctl status nginx

Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-03-01 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   Memory: 3.5M
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
``` 

Según lo confirmado por esto, el servicio se inició con éxito. Sin embargo, la mejor manera de probar esto es solicitar una página de Nginx.

Puede acceder a la página de inicio predeterminada de Nginx para confirmar que el software se está ejecutando correctamente navegando a la dirección IP de su servidor. Si no conoce la dirección IP de su servidor, puede encontrarla utilizando la herramienta icanhazip.com, que le brindará su dirección IP pública tal como la recibió desde otra ubicación en Internet:

``` 
curl -4 icanhazip.com
``` 

Cuando tengas la dirección IP de tu servidor, ingrésala en la barra de direcciones de tu navegador:

``` 
http://your_server_ip
``` 

Debería recibir la página de inicio predeterminada de Nginx. 

Si está en esta página, su servidor está funcionando correctamente y está listo para ser administrado.

### 3.4. Paso 4: gestionar el proceso Nginx

Ahora que tiene su servidor web en funcionamiento, revisemos algunos comandos de administración básicos.

Para detener su servidor web, escriba:

``` 
sudo systemctl stop nginx
``` 

Para iniciar el servidor web cuando esté detenido, escriba:

``` 
sudo systemctl start nginx
``` 

Para detener y luego iniciar nuevamente el servicio, escriba:

``` 
sudo systemctl restart nginx
``` 

Si solo está realizando cambios de configuración, Nginx a menudo puede recargar sin perder las conexiones. Para hacer esto, escriba:

```
sudo systemctl reload nginx
``` 

De forma predeterminada, Nginx está configurado para iniciarse automáticamente cuando se inicia el servidor. Si esto no es lo que desea, puede desactivar este comportamiento escribiendo:

``` 
sudo systemctl disable nginx
``` 

Para volver a habilitar el servicio para que se inicie en el arranque, puede escribir:

```
sudo systemctl enable nginx
``` 

Ahora ha aprendido los comandos básicos de administración y debería estar listo para configurar el sitio para alojar más de un dominio.

### 3.5. Paso 5: configurar bloques de servidor (recomendado)

Cuando se utiliza el servidor web Nginx, se pueden usar bloques de servidor (similares a los hosts virtuales en Apache) para encapsular los detalles de configuración y alojar más de un dominio desde un solo servidor. Configuraremos un dominio llamado tu_dominio, pero debes reemplazarlo con tu propio nombre de dominio.

Nginx en Ubuntu 22.04 tiene un bloque de servidor habilitado de forma predeterminada que está configurado para entregar documentos desde un directorio en /var/www/html. Si bien esto funciona bien para un solo sitio, puede resultar difícil de manejar si aloja varios sitios. En lugar de modificar /var/www/html, creemos una estructura de directorios dentro /var/www de nuestro sitio your_domain, dejándola /var/www/html como el directorio predeterminado que se servirá si la solicitud de un cliente no coincide con ningún otro sitio.

Cree el directorio para su_dominio de la siguiente manera, usando la -p bandera para crear los directorios principales necesarios:

``` 
sudo mkdir -p /var/www/your_domain/html
``` 

A continuación, asigne la propiedad del directorio con la $USER variable de entorno:

``` 
sudo chown -R $USER:$USER /var/www/your_domain/html
``` 

Los permisos de sus raíces web deben ser correctos si no ha modificado su umask valor, que establece los permisos de archivos predeterminados. Para asegurarse de que sus permisos sean correctos y permitir que el propietario lea, escriba y ejecute los archivos mientras otorga solo permisos de lectura y ejecución a grupos y otros, puede ingresar el siguiente comando:

``` 
sudo chmod -R 755 /var/www/your_domain
``` 

A continuación, cree una index.html página de muestra usando nanosu editor favorito:

``` 
nano /var/www/your_domain/html/index.html
``` 

Dentro, agregue el siguiente HTML de muestra:

``` 
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
``` 

Guarde y cierre el archivo presionando Ctrl+X para salir, luego cuando se le solicite guardar, Y y luego Enter.

Para que Nginx proporcione este contenido, es necesario crear un bloque de servidor con las directivas correctas. En lugar de modificar el archivo de configuración predeterminado directamente, creemos uno nuevo en :/etc/nginx/sites-available/your_domain

``` 
sudo nano /etc/nginx/sites-available/your_domain
``` 

Pegue el siguiente bloque de configuración, que es similar al predeterminado, pero actualizado para nuestro nuevo directorio y nombre de dominio:

``` 
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
``` 

Observe que hemos actualizado la root configuración de nuestro nuevo directorio y de server_name nuestro nombre de dominio.

A continuación, habilitemos el archivo creando un enlace desde él al sites-enabled directorio, desde el cual Nginx lee durante el inicio:

``` 
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
``` 

Ahora hay dos bloques de servidores habilitados y configurados para responder a solicitudes basadas en sus directivas listen y server_name:

  * your_domain: Responderá a las solicitudes de your_domainy www.your_domain.
  * default: Responderá a cualquier solicitud en el puerto 80 que no coincida con los otros dos bloques.
  * 
Para evitar un posible problema de memoria del depósito hash que puede surgir al agregar nombres de servidores adicionales, es necesario ajustar un valor único en el /etc/nginx/nginx.conf archivo. Abre el archivo:

``` 
sudo nano /etc/nginx/nginx.conf
```

Busque la server_names_hash_bucket_size directiva y elimine el # símbolo para descomentar la línea. Si está utilizando nano, puede buscar rápidamente palabras en el archivo presionando CTRL y w.

``` 
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
``` 

Guarde y cierre el archivo cuando haya terminado.

A continuación, pruebe para asegurarse de que no haya errores de sintaxis en ninguno de sus archivos Nginx:

``` 
sudo nginx -t
``` 

Si no hay ningún problema, reinicie Nginx para habilitar sus cambios:

```
sudo systemctl restart nginx
``` 

Nginx ahora debería servir su nombre de dominio. Puedes probar esto navegando a, donde deberías ver algo como esto:http://your_domain


## 4. Proteger Nginx con Let's Encrypt en Ubuntu 22.04

**Introducción**

Let's Encrypt es una autoridad certificadora (CA) que proporciona una forma accesible de obtener e instalar certificados TLS/SSL gratuitos , habilitando así HTTPS cifrado en servidores web. Simplifica el proceso al proporcionar un cliente de software, Certbot, que intenta automatizar la mayoría (si no todos) de los pasos requeridos. Actualmente, todo el proceso de obtención e instalación de un certificado está totalmente automatizado tanto en Apache como en Nginx.

### 4.1. Paso 1: instalar Certbot

Certbot recomienda utilizar su paquete snap para la instalación. Los paquetes Snap funcionan en casi todas las distribuciones de Linux, pero requieren que haya instalado snapd primero para poder administrar los paquetes snap. Ubuntu 22.04 viene con soporte para snapd listo para usar, por lo que puedes comenzar asegurándote de que tu snapd core esté actualizado:

``` 
sudo snap install core 
sudo snap refresh core
``` 

Si está trabajando en un servidor que anteriormente tenía instalada una versión anterior de certbot, debe eliminarla antes de continuar:

``` 
sudo apt remove certbot
``` 

Después de eso, puedes instalar el certbot paquete:

``` 
sudo snap install --classic certbot
``` 

Finalmente, puede vincular el certbot comando desde el directorio de instalación instantánea a su ruta, de modo que podrá ejecutarlo simplemente escribiendo certbot. Esto no es necesario con todos los paquetes, pero los snaps tienden a ser menos intrusivos de forma predeterminada, por lo que no entran en conflicto con ningún otro paquete del sistema por accidente:

``` 
sudo ln -s /snap/bin/certbot /usr/bin/certbot
``` 

Ahora que tenemos Certbot instalado, ejecútelo para obtener nuestro certificado.

### 4.2. Paso 2: Confirmar la configuración de Nginx

Certbot necesita poder encontrar el server bloque correcto en su configuración de Nginx para poder configurar SSL automáticamente. Específicamente, lo hace buscando una server_name directiva que coincida con el dominio para el que solicita un certificado.

Si siguió el paso de configuración del bloque de servidor en el tutorial de instalación de Nginx, debería tener un bloque de servidor para su dominio con la directiva ya configurada adecuadamente./etc/nginx/sites-available/example.com server_name

Para verificar, abra el archivo de configuración de su dominio usando nano su editor de texto favorito:

``` 
sudo nano /etc/nginx/sites-available/example.com
``` 

Encuentra la server_name línea existente. Debe tener un aspecto como este:

``` 
...
server_name example.com www.example.com;
...
``` 

Si es así, salga de su editor y continúe con el siguiente paso.

Si no es así, actualícelo para que coincida. Luego guarde el archivo, salga de su editor y verifique la sintaxis de sus ediciones de configuración:

``` 
sudo nginx -t
``` 

Si recibe un error, vuelva a abrir el archivo de bloqueo del servidor y verifique si hay errores tipográficos o caracteres faltantes. Una vez que la sintaxis de su archivo de configuración sea correcta, vuelva a cargar Nginx para cargar la nueva configuración:

``` 
sudo systemctl reload nginx
``` 

Certbot ahora puede encontrar el server bloque correcto y actualizarlo automáticamente.

A continuación, actualicemos el firewall para permitir el tráfico HTTPS.

### 4.3. Paso 3: permitir HTTPS a través del firewall

Si tiene el ufw firewall habilitado, como lo recomiendan las guías de requisitos previos, deberá ajustar la configuración para permitir el tráfico HTTPS. Afortunadamente, Nginx registra algunos perfiles ufw durante la instalación.

Puede ver la configuración actual escribiendo:

``` 
sudo ufw status
``` 

Probablemente se verá así, lo que significa que sólo se permite tráfico HTTP al servidor web:

``` 
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
``` 

Para permitir adicionalmente el tráfico HTTPS, permita el perfil completo de Nginx y elimine la asignación redundante del perfil HTTP de Nginx:

``` 
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
``` 

Su estado ahora debería verse así:

``` 
sudo ufw status
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
``` 

A continuación, ejecutemos Certbot y obtengamos nuestros certificados.

### 4.4. Paso 4: Obtener un certificado SSL

Certbot ofrece una variedad de formas de obtener certificados SSL a través de complementos. El complemento Nginx se encargará de reconfigurar Nginx y recargar la configuración cuando sea necesario. Para utilizar este complemento, escriba lo siguiente:

``` 
sudo certbot --nginx -d example.com -d www.example.com
``` 

Esto se ejecuta certbotcon el --nginx complemento, utilizándolo -d para especificar los nombres de dominio para los que nos gustaría que el certificado sea válido.

Al ejecutar el comando, se le pedirá que ingrese una dirección de correo electrónico y acepte los términos de servicio. Después de hacerlo, debería ver un mensaje que le indica que el proceso fue exitoso y dónde se almacenan sus certificados:

``` 
Output
IMPORTANT NOTES:
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/your_domain/fullchain.pem
Key is saved at: /etc/letsencrypt/live/your_domain/privkey.pem
This certificate expires on 2022-06-01.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
* Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
* Donating to EFF: https://eff.org/donate-le
``` 

Sus certificados se descargan, instalan y cargan, y su configuración de Nginx ahora redirigirá automáticamente todas las solicitudes web a https://. Intente recargar su sitio web y observe el indicador de seguridad de su navegador. Debería indicar que el sitio está correctamente protegido, normalmente con un icono de candado. Si prueba su servidor utilizando la prueba de servidor SSL Labs, obtendrá una calificación A.

Terminemos probando el proceso de renovación.

### 4.5. Paso 5: Verificar la renovación automática de Certbot

Los certificados de Let's Encrypt solo son válidos por noventa días. Esto es para alentar a los usuarios a automatizar su proceso de renovación de certificados. El certbot paquete que instalamos se encarga de esto agregando un temporizador systemd que se ejecutará dos veces al día y renovará automáticamente cualquier certificado que esté dentro de los treinta días posteriores a su vencimiento.

Puede consultar el estado del temporizador con systemctl:

``` 
sudo systemctl status snap.certbot.renew.service
Output
○ snap.certbot.renew.service - Service for snap application certbot.renew
     Loaded: loaded (/etc/systemd/system/snap.certbot.renew.service; static)
     Active: inactive (dead)
TriggeredBy: ● snap.certbot.renew.timer
``` 

Para probar el proceso de renovación, puede realizar un ensayo con certbot:

``` 
sudo certbot renew --dry-run
``` 

Si no ve errores, ya está todo listo. Cuando sea necesario, Certbot renovará sus certificados y recargará Nginx para recoger los cambios. Si el proceso de renovación automatizado alguna vez falla, Let's Encrypt enviará un mensaje al correo electrónico que especificó, advirtiéndole cuando su certificado esté a punto de caducar.

## 5. Caso práctico

### 5.1 Instalar Grafana

En este primer paso, instalará Grafana en su servidor Ubuntu 22.04. Puede instalar Grafana descargándolo directamente desde su sitio web oficial o accediendo a un repositorio APT. Debido a que un repositorio APT facilita la instalación y administración de las actualizaciones de Grafana, utilizará ese método en este tutorial.

Descargue la clave Grafana GPG con wget y luego canalice la salida a gpg. Esto convertirá la clave GPG de base64 a formato binario. Luego canalice la salida para tee almacenar la clave en el archivo./usr/share/keyrings/grafana.gpg

``` 
wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null
``` 

En este comando, la opción -q desactiva el mensaje de actualización de estado wget y -O genera el archivo que descargó en la terminal. Estas dos opciones garantizan que solo se canalice el contenido del archivo descargado. La > /dev/null opción ocultará la salida de tu terminal por motivos de seguridad.

A continuación, agregue el repositorio de Grafana a sus fuentes APT:

``` 
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
``` 

Actualice su caché APT para actualizar sus listas de paquetes:

``` 
sudo apt update
``` 

Ahora puedes continuar con la instalación:

``` 
sudo apt install grafana
``` 

Una vez que Grafana esté instalado, utilícelo systemctlpara iniciar el servidor Grafana:

``` 
sudo systemctl start grafana-server
``` 

A continuación, verifique que Grafana se esté ejecutando verificando el estado del servicio:

```
sudo systemctl status grafana-server
``` 

Recibirá un resultado similar a este:

``` 
Output
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2022-09-27 14:42:15 UTC; 6s ago
     Docs: http://docs.grafana.org
 Main PID: 4132 (grafana-server)
    Tasks: 7 (limit: 515)
...
``` 

Este resultado contiene información sobre el proceso de Grafana, incluido su estado, el identificador del proceso principal (PID) y más. active (running) muestra que el proceso se está ejecutando correctamente.

Por último, habilite el servicio para que inicie Grafana automáticamente al arrancar:

``` 
sudo systemctl enable grafana-server
``` 

Recibirá el siguiente resultado:

``` 
Output
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /usr/lib/systemd/system/grafana-server.service.
``` 

Esto confirma que systemd se han creado los enlaces simbólicos necesarios para iniciar automáticamente Grafana.

Grafana ahora está instalado y listo para usar. A continuación, protegerá su conexión a Grafana con un proxy inverso y un certificado SSL.

### 5.2 Configurar el proxy inverso

El uso de un certificado SSL garantizará que sus datos estén seguros al cifrar la conexión hacia y desde Grafana. Pero, para utilizar esta conexión, primero deberá reconfigurar Nginx como proxy inverso para Grafana.

Abra el archivo de configuración de Nginx que creó cuando configuró el bloque del servidor Nginx con Let's Encrypt en los Requisitos previos. Puedes usar cualquier editor de texto, pero para este tutorial usaremos nano:

``` 
sudo nano /etc/nginx/sites-available/your_domain
``` 

Ubica el siguiente bloque:

``` 
...
	location / {
        try_files $uri $uri/ =404;
	}
...
``` 

Debido a que ya configuró Nginx para comunicarse a través de SSL y debido a que todo el tráfico web a su servidor ya pasa a través de Nginx, solo necesita decirle a Nginx que reenvíe todas las solicitudes a Grafana, que se ejecuta en el puerto de forma predeterminada 3000.

Elimine la try_files línea existente en esto location block y reemplácela con las siguientes opciones:

``` 
/etc/nginx/sitios-disponibles/su_dominio
...
	location / {
	   proxy_set_header Host $http_host;
	   proxy_pass http://localhost:3000;
	}
...
``` 

Esto asignará el proxy al puerto apropiado y pasará un nombre de servidor en el encabezado.

Además, para que las conexiones de Grafana Live WebSocket funcionen correctamente, agregue la siguiente sección fuera de la server sección:

``` 
/etc/nginx/sitios-disponibles/su_dominio
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {
    ...
``` 

Luego agregue la siguiente locationsección al bloque del servidor:

``` 
/etc/nginx/sitios-disponibles/su_dominio
server {
...
	location /api/live {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $http_host;
        proxy_pass http://localhost:3000;
	}
...
``` 

Los encabezados Upgrade y Connection no se envían desde el cliente al servidor proxy. Por lo tanto, para que el servidor proxy sepa acerca de la intención del cliente de cambiar el protocolo a WebSocket, estos encabezados deben pasarse explícitamente.

La configuración final se verá así:

``` 
/etc/nginx/sitios-disponibles/su_dominio
map $http_upgrade $connection_upgrade {
	default upgrade;
	'' close;
}

server {
	...

	root /var/www/your_domain/html;
	index index.html index.htm index.nginx-debian.html;

	server_name your_domain www.your_domain;

	location / {
		proxy_set_header Host $http_host;
		proxy_pass http://localhost:3000;
	}

	location /api/live {
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection $connection_upgrade;
		proxy_set_header Host $http_host;
		proxy_pass http://localhost:3000;
	}

   ...
}
``` 

Una vez que haya terminado, guarde y cierre el archivo presionando CTRL+X, Y y luego, ENTER si está usando nano.

Ahora, pruebe la nueva configuración para asegurarse de que todo esté configurado correctamente:

``` 
sudo nginx -t
``` 

Recibirá el siguiente resultado:

``` 
Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
``` 

Finalmente, active los cambios recargando Nginx:

``` 
sudo systemctl reload nginx
``` 

Ahora puede acceder a la pantalla de inicio de sesión predeterminada de Grafana apuntando su navegador web a . Si no puede comunicarse con Grafana, verifique que su firewall esté configurado para permitir el tráfico en el puerto y luego vuelva a seguir las instrucciones anteriores.https://your_domain443

Con la conexión a Grafana cifrada, ahora puede implementar medidas de seguridad adicionales, comenzando por cambiar las credenciales administrativas predeterminadas de Grafana.

### 5.3 Actualización de credenciales

Debido a que cada instalación de Grafana utiliza las mismas credenciales administrativas de forma predeterminada, se recomienda cambiar su información de inicio de sesión lo antes posible. En este paso, actualizará las credenciales para mejorar la seguridad.

Comience navegando desde su navegador web. Esto abrirá la pantalla de inicio de sesión predeterminada donde verá el logotipo de Grafana, un formulario que le pedirá que ingrese un correo electrónico o nombre de usuario y contraseña, un botón Iniciar sesión y ¿Olvidó su contraseña? enlace.https://your_domain

Ingrese admin los campos Correo electrónico o nombre de usuario y Contraseña y luego haga clic en el botón Iniciar sesión.

En la siguiente pantalla, se le pedirá que haga su cuenta más segura cambiando la contraseña predeterminada.

Ingrese la contraseña que desea comenzar a usar en los campos Nueva contraseña y Confirmar nueva contraseña.

Desde aquí, puede hacer clic en Enviar para guardar la nueva información o presionar Omitir para omitir este paso. Si omite, se le pedirá que cambie la contraseña la próxima vez que inicie sesión.

Para aumentar la seguridad de su configuración de Grafana, haga clic en Enviar. Irás al panel de bienvenida a Grafana.

Ahora ha protegido su cuenta cambiando las credenciales predeterminadas. A continuación, realizará cambios en su configuración de Grafana para que nadie pueda crear una nueva cuenta de Grafana sin su permiso.

### 5.4 deshabilitar los registros de Grafana y el acceso anónimo

Grafana ofrece opciones que permiten a los visitantes crear cuentas de usuario por sí mismos y obtener una vista previa de los paneles sin registrarse. Cuando no se puede acceder a Grafana a través de Internet o cuando funciona con datos disponibles públicamente, como estados de servicio, es posible que desee permitir estas funciones. Sin embargo, cuando se utiliza Grafana en línea para trabajar con datos confidenciales, el acceso anónimo podría ser un problema de seguridad. Para solucionar este problema, realizará algunos cambios en la configuración de Grafana.

Comience abriendo el archivo de configuración principal de Grafana para editarlo:

``` 
sudo nano /etc/grafana/grafana.ini
```

Ubique la siguiente allow_sign_up directiva bajo el [users] título:

``` 
...
[users]
# disable user signup / registration
;allow_sign_up = true
...
``` 

Al habilitar esta directiva true se agrega un botón Registrarse a la pantalla de inicio de sesión, lo que permite a los usuarios registrarse y acceder a Grafana.

Al deshabilitar esta directiva falsese elimina el botón Registrarse y se fortalece la seguridad y privacidad de Grafana.

Descomente esta directiva eliminando el ; al principio de la línea y luego configurando la opción en false:

``` 
...
[users]
# disable user signup / registration
allow_sign_up = false
...
``` 

A continuación, ubique la siguiente enabled directiva bajo el [auth.anonymous] título:

```
...
[auth.anonymous]
# enable anonymous access
;enabled = false
...
``` 

La configuración enabled para true otorga a los usuarios no registrados acceso a sus paneles; configurar esta opción para falselimitar el acceso al panel solo a usuarios registrados.

Descomente esta directiva eliminando el ;al principio de la línea y luego configurando la opción en false.

``` 
...
[auth.anonymous]
# enable anonymous access
enabled = false
...
``` 

Guarde el archivo y salga de su editor de texto.

Para activar los cambios, reinicie Grafana:

``` 
sudo systemctl restart grafana-server
``` 

Verifique que todo esté funcionando verificando el estado del servicio de Grafana:

```
sudo systemctl status grafana-server
``` 

Como antes, el resultado informará que Grafana es active (running).

Ahora, apunte su navegador web a . Para volver a la pantalla de registro , lleve el cursor a su avatar en la parte inferior izquierda de la pantalla y haga clic en la opción Cerrar sesión que aparece.https://your_domain

Una vez que haya cerrado sesión, verifique que no haya ningún botón Registrarse y que no pueda iniciar sesión sin ingresar las credenciales de inicio de sesión.

En este punto, Grafana está completamente configurado y listo para usar. A continuación, puede simplificar el proceso de inicio de sesión para su organización autenticándose a través de GitHub.

### 5.5 Configurar una aplicación GitHub OAuth

Como método alternativo para iniciar sesión, puede configurar Grafana para que se autentique a través de GitHub, que proporciona acceso de inicio de sesión a todos los miembros de organizaciones autorizadas de GitHub. Esto puede resultar particularmente útil cuando desea permitir que varios desarrolladores colaboren y accedan a métricas sin tener que crear credenciales específicas de Grafana.

Comience iniciando sesión en una cuenta de GitHub asociada con su organización y luego navegue hasta su página de perfil de GitHub.

Cambie el contexto de configuración haciendo clic en el enlace Cambiar a otra cuenta en la parte superior de la pantalla y luego seleccionando su organización en el menú desplegable. Esto cambiará el contexto de Configuración personal a Configuración de organización.

En la siguiente pantalla, verá el perfil de su organización, donde puede cambiar configuraciones como el nombre para mostrar de su organización, el correo electrónico de su organización y la URL de su organización.

Debido a que Grafana utiliza OAuth, un estándar abierto para otorgar acceso remoto a terceros a recursos locales, para autenticar usuarios a través de GitHub, deberá crear una nueva aplicación OAuth dentro de GitHub.

Haga clic en el enlace Aplicaciones OAuth en Configuración de desarrollador en la parte inferior izquierda de la pantalla.

Si aún no tiene ninguna aplicación OAuth asociada con su organización en GitHub, se le informará que no hay ninguna aplicación propiedad de la organización. De lo contrario, verá una lista de las aplicaciones OAuth que ya están conectadas a su cuenta.

Haga clic en el botón Registrar una aplicación para continuar.

En la siguiente pantalla, complete los siguientes detalles sobre su instalación de Grafana:

* **Nombre de la aplicación:** esto le ayuda a distinguir las diferentes aplicaciones OAuth entre sí.
* **URL de la página de inicio:** esto le indica a GitHub dónde encontrar Grafana. Escriba en este campo y reemplácelo con su dominio.https://your_domainyour_domain
* **Descripción de la aplicación:** proporciona una descripción del propósito de su aplicación OAuth.
* **URL de devolución de llamada de la aplicación:** esta es la dirección a la que se enviará a los usuarios una vez autenticados correctamente. Para Grafana, este campo debe establecerse en .https://your_domain/login/github

Tenga en cuenta que los usuarios de Grafana que inicien sesión a través de GitHub verán los valores que ingresó en los primeros tres campos anteriores, así que asegúrese de ingresar algo significativo y apropiado.

Cuando esté completo, el formulario se verá así.

Haga clic en el botón Registrar aplicación.

Ahora será redirigido a una página que contiene información general sobre su nueva aplicación OAuth, incluido el ID del cliente. Luego haga clic en el botón Generar un nuevo secreto de cliente para obtener un nuevo secreto de cliente. Tome nota de ambos valores, porque deberá agregarlos al archivo de configuración principal de Grafana para completar la configuración.

Con su aplicación GitHub OAuth creada, ahora está listo para reconfigurar Grafana para usar GitHub para la autenticación.

## 5.6 Configurar Grafana como una aplicación GitHub OAuth

Para completar la autenticación de GitHub para su configuración de Grafana, ahora deberá realizar algunos cambios en sus archivos de configuración de Grafana.

Para comenzar, abra el archivo de configuración principal de Grafana:

```
sudo nano /etc/grafana/grafana.ini
``` 

Localice el [auth.github] encabezado y descomente esta sección eliminando el ; al principio de cada línea excepto la siguiente, que no se cambiará en este tutorial:

``` 
;allowed_domains =
;team_ids =
;role_attribute_path =
;role_attribute_strict = false
;allow_assign_grafana_admin = false
``` 

A continuación, realice los siguientes cambios:

* Establecer enabledy allow_sign_up a true. Esto habilitará la autenticación de GitHub y permitirá a los miembros de la organización permitida crear cuentas ellos mismos. Tenga en cuenta que esta configuración es diferente de la allow_sign_uppropiedad [users]que cambió en el Paso 4.
  
* Establezca client_id y client_secret en los valores que obtuvo al crear su aplicación GitHub OAuth.

* Establezca allowed_organizations el nombre de su organización para asegurarse de que solo los miembros de su organización puedan registrarse e iniciar sesión en Grafana.


La configuración completa se verá así:

``` 
...
[auth.github]
enabled = true
allow_sign_up = true
client_id = your_client_id_from_github
client_secret = your_client_secret_from_github
scopes = user:email,read:org
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
;allowed_domains =
;team_ids =
allowed_organizations = your_organization_name
;role_attribute_path =
;role_attribute_strict = false
;allow_assign_grafana_admin = false
...
``` 

Ahora le ha contado a Grafana todo lo que necesita saber sobre GitHub. Para completar la configuración, deberá habilitar las redirecciones detrás de un proxy inverso. Esto se hace estableciendo un root_url valor debajo del [server] encabezado.

``` 
...
[server]
root_url = https://your_domain
...
``` 

Guarde su configuración y cierre el archivo.

Luego, reinicie Grafana para activar los cambios:

``` 
sudo systemctl restart grafana-server
```

Por último, verifique que el servicio esté funcionando:

``` 
sudo systemctl status grafana-server
```

El resultado indicará que el servicio es active (running).

Ahora, pruebe su nuevo sistema de autenticación navegando a . Si ya ha iniciado sesión en Grafana, coloque el mouse sobre el registro de avatar en la esquina inferior izquierda de la pantalla y haga clic en Cerrar sesión en el menú secundario que aparece junto a su nombre.https://your_domain

En la página de inicio de sesión, verá una nueva sección debajo del botón Iniciar sesión original que incluye un botón Iniciar sesión con GitHub con el logotipo de GitHub.

Haga clic en el botón Iniciar sesión con GitHub para ser redirigido a GitHub, donde iniciará sesión en su cuenta de GitHub y confirmará su intención de autorizar Grafana .

Haga clic en el botón verde Autorizartu_organización_github botón.

Ahora iniciará sesión con su cuenta Grafana existente. Si aún no existe una cuenta de Grafana para el usuario con el que inició sesión, Grafana creará una nueva cuenta de usuario con permisos de Visualizador, lo que garantiza que los nuevos usuarios solo puedan usar los paneles existentes.

Para cambiar los permisos predeterminados para nuevos usuarios, abra el archivo de configuración principal de Grafana para editarlo.

``` 
sudo nano /etc/grafana/grafana.ini
``` 

Ubique la auto_assign_org_role directiva debajo del [users] encabezado y descomente la configuración eliminando el ; al principio de la línea.

Establezca la directiva en uno de los siguientes valores:

* **Viewer:** solo puede utilizar paneles existentes.
* **Editor:** puede usar, modificar y agregar paneles.
* **Admin:** tiene permiso para hacer todo.

Este tutorial establecerá la asignación automática en Viewer:

``` 
/etc/grafana/grafana.ini
...
[users]
...
auto_assign_org_role = Viewer
...
``` 

Una vez que haya guardado los cambios, cierre el archivo y reinicie Grafana:

``` 
sudo systemctl restart grafana-server
``` 

Consulta el estado del servicio:

``` 
sudo systemctl status grafana-server
``` 

Como antes, el estado será active (running).

En este punto, ha configurado Grafana completamente para permitir que los miembros de su organización GitHub se registren y utilicen su instalación de Grafana.

**Conclusión**

En este tutorial, instaló, configuró y aseguró Grafana, y también aprendió cómo permitir que los miembros de su organización se autentiquen a través de GitHub.

