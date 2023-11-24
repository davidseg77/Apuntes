# Cómo instalar WordPress con Docker Compose

## 1. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un servidor que ejecute Ubuntu 20.04, junto con un usuario no root sudo con privilegios y un firewall activo. 

* Docker instalado en su servidor.

* Docker Compose instalado en su servidor.

* Un nombre de dominio registrado. Este tutorial utilizará su_dominio en todo momento. Puede obtener uno gratis en Freenom o utilizar el registrador de dominio de su elección.

* Los dos siguientes registros DNS configurados para su servidor. 

    - Un registro A your_domain que apunta a la dirección IP pública de su servidor.
    - Un registro A que apunta a la dirección IP pública de su servidor.www.your_domain

Una vez que tenga todo configurado, estará listo para comenzar con el primer paso.


## 2. Definir la configuración del servidor web

Antes de ejecutar cualquier contenedor, el primer paso es definir la configuración de su servidor web Nginx. Su archivo de configuración incluirá algunos bloques de ubicación específicos de WordPress, junto con un bloque de ubicación para dirigir las solicitudes de verificación de Let's Encrypt al cliente Certbot para renovaciones automáticas de certificados.

Primero, cree un directorio de proyecto para su configuración de WordPress. En este ejemplo, se llama wordpress. Puede nombrar este directorio de manera diferente si desea:

``` 
mkdir wordpress
``` 

Luego navegue hasta el directorio:

``` 
cd wordpress
``` 

A continuación, cree un directorio para el archivo de configuración:

``` 
mkdir nginx-conf
``` 

Abra el archivo con nanoo su editor favorito:

``` 
nano nginx-conf/nginx.conf
``` 

En este archivo, agregue un bloque de servidor con directivas para el nombre de su servidor y la raíz del documento, y bloques de ubicación para dirigir la solicitud de certificados, procesamiento PHP y solicitudes de activos estáticos del cliente Certbot.

Agregue el siguiente código al archivo. Asegúrese de reemplazarlo your_domain con su propio nombre de dominio:

``` 
server {
        listen 80;
        listen [::]:80;

        server_name your_domain www.your_domain;

        index index.php index.html index.htm;

        root /var/www/html;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }

        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }
        
        location = /favicon.ico { 
                log_not_found off; access_log off; 
        }
        location = /robots.txt { 
                log_not_found off; access_log off; allow all; 
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }
}
``` 

Nuestro bloque de servidores incluye la siguiente información:

**Directivas:**

* listen: Esto le indica a Nginx que escuche en el puerto 80, lo que le permitirá usar el complemento webroot de Certbot para sus solicitudes de certificados. Tenga en cuenta que aún no está incluyendo el puerto 443; actualizará su configuración para incluir SSL una vez que haya obtenido exitosamente sus certificados.
  
* server_name: Esto define el nombre de su servidor y el bloque de servidor que debe usarse para las solicitudes a su servidor. Asegúrese de reemplazar your_domain en esta línea con su propio nombre de dominio.
  
* index: Esta directiva define los archivos que se utilizarán como índices al procesar solicitudes a su servidor. Modificó el orden de prioridad predeterminado aquí, moviéndose index.php al frente index.html para que Nginx priorice los archivos llamados index.php cuando sea posible.
  
* root: esta directiva nombra el directorio raíz para las solicitudes a su servidor. Este directorio, /var/www/html se crea como un punto de montaje en el momento de la compilación mediante instrucciones en su Dockerfile de WordPress. Estas instrucciones de Dockerfile también garantizan que los archivos de la versión de WordPress estén montados en este volumen.
  
**Bloques de ubicación:**

* location ~ /.well-known/acme-challenge: Este bloque de ubicación manejará las solicitudes al .well-known directorio, donde Certbot colocará un archivo temporal para validar que el DNS de su dominio se resuelva en su servidor. Con esta configuración implementada, podrá utilizar el complemento webroot de Certbot para obtener certificados para su dominio.
  
* location /: En este bloque de ubicación, try_files se utiliza una directiva para buscar archivos que coincidan con solicitudes de URI individuales. Sin embargo, en lugar de devolver un estado 404 No encontrado de forma predeterminada, pasará el control al index.php archivo de WordPress con los argumentos de la solicitud.
  
* location ~ \.php$: Este bloque de ubicación manejará el procesamiento de PHP y enviará estas solicitudes a su wordpress contenedor. Debido a que su imagen de WordPress Docker se basará en la php:fpm imagen, también incluirá opciones de configuración que son específicas del protocolo FastCGI en este bloque. Nginx requiere un procesador PHP independiente para solicitudes PHP. En este caso, estas solicitudes serán manejadas por el php-fpm procesador incluido con la php:fpm imagen. Además, este bloque de ubicación incluye directivas, variables y opciones específicas de FastCGI que enviarán solicitudes a la aplicación de WordPress que se ejecuta en su wordpresscontenedor, establecerán el índice preferido para el URI de solicitud analizado y analizarán las solicitudes de URI.
  
* location ~ /\.ht: Este bloque manejará .htaccess archivos ya que Nginx no los entregará. La deny_all directiva garantiza que .htaccess los archivos nunca se entregarán a los usuarios.
  
* location = /favicon.ico, location = /robots.txt: Estos bloques garantizan que las solicitudes recibidas /favicon.ico y /robots.txt no se registren.
  
* location ~* \.(css|gif|ico|jpeg|jpg|js|png)$: este bloque desactiva el registro de solicitudes de activos estáticos y garantiza que estos activos se puedan almacenar en caché en gran medida, ya que su mantenimiento suele ser costoso.
  
Guarde y cierre el archivo cuando haya terminado de editarlo. Si usó nano, hágalo presionando CTRL+X, Y, luego ENTER.

Con su configuración de Nginx implementada, puede pasar a crear variables de entorno para pasarlas a los contenedores de su aplicación y base de datos en tiempo de ejecución.


## 3. Definir variables de entorno

Su base de datos y los contenedores de aplicaciones de WordPress necesitarán acceso a ciertas variables de entorno en tiempo de ejecución para que los datos de su aplicación persistan y sean accesibles para su aplicación. Estas variables incluyen información confidencial y no confidencial: valores confidenciales para su contraseña raíz de MySQL y el usuario y contraseña de la base de datos de la aplicación, e información no confidencial para el nombre y el host de la base de datos de su aplicación.

En lugar de configurar todos estos valores en su archivo Docker Compose (el archivo principal que contiene información sobre cómo se ejecutarán sus contenedores), configure los valores confidenciales en un .env archivo y restrinja su circulación. Esto evitará que estos valores se copien en los repositorios de su proyecto y se expongan públicamente.

En el directorio principal de su proyecto, abra un archivo llamado :~/wordpress.env

```
nano .env
``` 

Los valores confidenciales que establezca en este archivo incluyen una contraseña para el usuario raíz de MySQL y un nombre de usuario y contraseña que WordPress utilizará para acceder a la base de datos.

Agregue los siguientes nombres y valores de variables al archivo. Recuerde proporcionar sus propios valores aquí para cada variable:

``` 
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_USER=your_wordpress_database_user
MYSQL_PASSWORD=your_wordpress_database_password
``` 

Se incluye una contraseña para la cuenta administrativa raíz, así como su nombre de usuario y contraseña preferidos para la base de datos de su aplicación.

Guarde y cierre el archivo cuando haya terminado de editarlo.

Debido a que su .env archivo contiene información confidencial, desea asegurarse de que esté incluida en los archivos .gitignore y proyectos .dockerignore. Esto le indica a Git y Docker qué archivos no copiar en sus repositorios de Git e imágenes de Docker, respectivamente.

Si planea trabajar con Git para el control de versiones, inicialice su directorio de trabajo actual como repositorio con git init:

``` 
git init
``` 

Luego crea y abre un .gitignore archivo:

``` 
nano .gitignore
``` 

Añadir .env al archivo:

``` 
.env
``` 

Guarde y cierre el archivo cuando haya terminado de editarlo.

Del mismo modo, es una buena precaución agregarlo .env a un .dockerignore archivo, para que no termine en sus contenedores cuando utilice este directorio como contexto de compilación.

Abre el archivo:

``` 
nano .dockerignore
``` 

Añadir .env al archivo:

``` 
.env
``` 

Debajo de esto, opcionalmente puede agregar archivos y directorios asociados con el desarrollo de su aplicación:

``` 
.env
.git
docker-compose.yml
.dockerignore
``` 

Guarde y cierre el archivo cuando haya terminado.

Con su información confidencial en su lugar, ahora puede pasar a definir sus servicios en un docker-compose.yml archivo.


## 4. Definir servicios con Docker Compose

Su docker-compose.yml archivo contendrá las definiciones de servicio para su configuración. Un servicio en Compose es un contenedor en ejecución y las definiciones de servicio especifican información sobre cómo se ejecutará cada contenedor.

Con Compose, puede definir diferentes servicios para ejecutar aplicaciones de múltiples contenedores, ya que Compose le permite vincular estos servicios con redes y volúmenes compartidos. Esto será útil para su configuración actual, ya que creará diferentes contenedores para su base de datos, aplicación de WordPress y servidor web. También creará un contenedor para ejecutar el cliente Certbot para obtener certificados para su servidor web.

Para comenzar, cree y abra el docker-compose.yml archivo:

``` 
nano docker-compose.yml
``` 

Agregue el siguiente código para definir la versión del archivo Compose y db el servicio de base de datos:

``` 
version: '3'

services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes: 
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network
``` 

La db definición del servicio contiene las siguientes opciones:

* **image:** Esto le indica a Compose qué imagen extraer para crear el contenedor. Está fijando la mysql:8.0 imagen aquí para evitar conflictos futuros a medida que la mysql:latest imagen continúa actualizándose. 
container_name: Esto especifica un nombre para el contenedor.

* **restart:** Esto define la política de reinicio del contenedor. El valor predeterminado es no, pero ha configurado el contenedor para que se reinicie a menos que se detenga manualmente.
  
* **env_file:** Esta opción le indica a Compose que desea agregar variables de entorno desde un archivo llamado .env, ubicado en su contexto de compilación. En este caso, el contexto de compilación es su directorio actual.
  
* **environment:** Esta opción le permite agregar variables de entorno adicionales, además de las definidas en su .env archivo. Establecerá la MYSQL_DATABASE variable igual a wordpress para proporcionar un nombre para la base de datos de su aplicación. Como se trata de información no confidencial, puede incluirla directamente en el docker-compose.yml archivo.
  
* **volumes:** Aquí, está montando un volumen con nombre llamado db data al /var/lib/mysql directorio en el contenedor. Este es el directorio de datos estándar para MySQL en la mayoría de las distribuciones.

* **command:** esta opción especifica un comando para anular la instrucción CMD predeterminada para la imagen. En este caso particular, agregará una opción al mysqld comando estándar de la imagen de Docker, que inicia el servidor MySQL en el contenedor. Esta opción,--default-authentication-plugin=mysql_native_password establece la --default-authentication-plugin variable del sistema en mysql_native_password, especificando qué mecanismo de autenticación debe gobernar las nuevas solicitudes de autenticación al servidor. Dado que PHP y, por lo tanto, su imagen de WordPress no admitirán el nuevo valor predeterminado de autenticación de MySQL, debe realizar este ajuste para autenticar al usuario de la base de datos de su aplicación.
  
* **networks:** Esto especifica que el servicio de su aplicación se unirá a la app-network red, que usted definirá en la parte inferior del archivo.

A continuación, debajo de db la definición de su servicio, agregue la definición del wordpress servicio de su aplicación:

``` 
...
  wordpress:
    depends_on: 
      - db
    image: wordpress:5.1.1-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network
```

En esta definición de servicio, nombrará su contenedor y definirá una política de reinicio, como lo hizo con el db servicio. También está agregando algunas opciones específicas para este contenedor:

* **depends_on:** Esta opción garantiza que sus contenedores comenzarán en orden de dependencia, con el wordpress contenedor comenzando después del db contenedor. Su aplicación de WordPress depende de la existencia de la base de datos de su aplicación y del usuario, por lo que expresar este orden de dependencia permitirá que su aplicación se inicie correctamente.
  
* **image:** Para esta configuración, estás utilizando la 5.1.1-fpm-alpine imagen de WordPress. Como se analizó en el Paso 1, el uso de esta imagen garantiza que su aplicación tendrá el php-fpm procesador que Nginx requiere para manejar el procesamiento de PHP. Esta también es una alpine imagen derivada del proyecto Alpine Linux, que ayudará a mantener bajo el tamaño general de la imagen. 
  
* **env_file:** Nuevamente, especifica que desea extraer valores de su .env archivo, ya que aquí es donde definió el usuario y la contraseña de la base de datos de su aplicación.
  
* **environment:** Aquí, estás usando los valores que definiste en tu .env archivo, pero los estás asignando a los nombres de variables que espera la imagen de WordPress: WORDPRESS_DB_USER y WORDPRESS_DB_PASSWORD. También está definiendo un WORDPRESS_DB_HOST, que será el servidor MySQL que se ejecuta en el db contenedor al que se puede acceder en el puerto predeterminado de MySQL, 3306. Será WORDPRESS_DB_NAME el mismo valor que especificó en la definición del servicio MySQL para su MYSQL_DATABASE: wordpress.
  
* **volumes:** Está montando un volumen con nombre llamado wordpress al /var/www/html punto de montaje creado por la imagen de WordPress. Usar un volumen con nombre de esta manera le permitirá compartir el código de su aplicación con otros contenedores.
  
* **networks:** También está agregando el wordpress contenedor a la app-network red.

A continuación, debajo de la wordpress definición del servicio de la aplicación, agregue la siguiente definición para su webserver servicio Nginx:

``` 
...
  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - app-network
``` 

Aquí, está nombrando su contenedor y haciéndolo dependiente del wordpress contenedor en orden inicial. También estás usando una alpine imagen: la 1.15.12-alpine imagen de Nginx.

Esta definición de servicio también incluye las siguientes opciones:

* **ports:** Esto expone el puerto 80 para habilitar las opciones de configuración que definió en su nginx.confarchivo en el Paso 1.
  
* **volumes:** Aquí, está definiendo una combinación de volúmenes con nombre y montajes de enlace :
wordpress:/var/www/html: Esto montará el código de su aplicación de WordPress en el /var/www/html directorio, el directorio que configuró como root en su bloque de servidor Nginx.

* **./nginx-conf:/etc/nginx/conf.d:** Esto vinculará el montaje del directorio de configuración de Nginx en el host al directorio relevante en el contenedor, asegurando que cualquier cambio que realice en los archivos del host se reflejará en el contenedor.
  
* **certbot-etc:/etc/letsencrypt:** Esto montará los certificados y claves de Let's Encrypt relevantes para su dominio en el directorio apropiado del contenedor.
* 
También ha agregado este contenedor a la app-network red.

Finalmente, debajo de su webserver definición, agregue su última definición de servicio para el certbot servicio. Asegúrese de reemplazar la dirección de correo electrónico y los nombres de dominio que aparecen aquí con su propia información:

``` 
certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain -d www.your_domain
```

Esta definición le dice a Compose que extraiga la certbot/certbot imagen de Docker Hub. También utiliza volúmenes con nombre para compartir recursos con el contenedor Nginx, incluidos los certificados de dominio y la clave certbot-etc y el código de la aplicación en formato wordpress.

Nuevamente, solía depends_on especificar que el certbot contenedor debe iniciarse una vez que el webserver servicio se esté ejecutando.

También incluyó una command opción que especifica un subcomando para ejecutar con el certbot comando predeterminado del contenedor. El certonly subcomando obtendrá un certificado con las siguientes opciones:

* **--webroot:** Esto le indica a Certbot que use el complemento webroot para colocar archivos en la carpeta webroot para la autenticación. Este complemento depende del método de validación HTTP-01, que utiliza una solicitud HTTP para demostrar que Certbot puede acceder a recursos desde un servidor que responde a un nombre de dominio determinado.
  
* **--webroot-path:** Esto especifica la ruta del directorio webroot.
  
* **--email:** Su correo electrónico preferido para registro y recuperación.
  
* **--agree-tos:** Esto especifica que usted acepta el Acuerdo de Suscriptor de ACME.
  
* **--no-eff-email:** Esto le indica a Certbot que no desea compartir su correo electrónico con Electronic Frontier Foundation (EFF). Siéntase libre de omitir esto si lo prefiere.
  
* **--staging:** Esto le indica a Certbot que le gustaría utilizar el entorno de prueba de Let's Encrypt para obtener certificados de prueba. El uso de esta opción le permite probar sus opciones de configuración y evitar posibles límites de solicitud de dominio. 
  
* **-d:** Esto le permite especificar los nombres de dominio que le gustaría aplicar a su solicitud. En este caso, has incluido your_domain y . Asegúrese de reemplazarlos con su propio dominio .www.your_domain
Debajo de la certbot definición del servicio, agregue las definiciones de su red y volumen:

``` 
...
volumes:
  certbot-etc:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge  
``` 

Su clave de nivel superior volumes define los volúmenes certbot-etc, wordpress y dbdata. Cuando Docker crea volúmenes, el contenido del volumen se almacena en un directorio en el sistema de archivos del host, /var/lib/docker/volumes/que es administrado por Docker. Luego, el contenido de cada volumen se monta desde este directorio en cualquier contenedor que utilice el volumen. De esta forma, es posible compartir código y datos entre contenedores.

La red puente definida por el usuario app-network permite la comunicación entre sus contenedores ya que están en el mismo host demonio Docker. Esto agiliza el tráfico y la comunicación dentro de la aplicación, ya que abre todos los puertos entre contenedores en la misma red puente sin exponer ningún puerto al mundo exterior. Por lo tanto, sus contenedores db, wordpress y webserver pueden comunicarse entre sí y solo necesita exponer el puerto 80 para el acceso frontal a la aplicación.

El siguiente es docker-compose.yml el expediente en su totalidad:

``` 
version: '3'

services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes: 
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on: 
      - db
    image: wordpress:5.1.1-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network

  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - app-network

  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain -d www.your_domain

volumes:
  certbot-etc:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge  
``` 

Guarde y cierre el archivo cuando haya terminado de editarlo.

Con las definiciones de servicio implementadas, está listo para iniciar los contenedores y probar sus solicitudes de certificados.


## 5. Obtención de certificados y credenciales SSL

Inicie sus contenedores con el docker-compose up comando, que creará y ejecutará sus contenedores en el orden que haya especificado. Al agregar la -d bandera, el comando ejecutará los contenedores db, wordpress y webserver en segundo plano:

``` 
docker-compose up -d
``` 

El siguiente resultado confirma que se han creado sus servicios:

``` 
Output
Creating db ... done
Creating wordpress ... done
Creating webserver ... done
Creating certbot   ... done
``` 

Usando docker-compose ps, verifique el estado de sus servicios:

``` 
docker-compose ps
``` 

Una vez completado, sus servicios db, wordpress y webserver estarán Up y el certbot contenedor habrá salido con un 0 mensaje de estado:

``` 
Output
  Name                 Command               State           Ports       
-------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0                      
db          docker-entrypoint.sh --def ...   Up       3306/tcp, 33060/tcp
webserver   nginx -g daemon off;             Up       0.0.0.0:80->80/tcp 
wordpress   docker-entrypoint.sh php-fpm     Up       9000/tcp  
``` 

Cualquier valor que no esté Up en la State columna de db, wordpress o webserver servicios, o un estado de salida que no sea el 0 del certbot contenedor significa que es posible que deba verificar los registros del servicio con el docker-compose logs comando:

```
docker-compose logs service_name
``` 

Ahora puede verificar que sus certificados se hayan montado en el webserver contenedor con docker-compose exec:

``` 
docker-compose exec webserver ls -la /etc/letsencrypt/live
``` 

Una vez que sus solicitudes de certificado se realicen correctamente, el siguiente es el resultado:

``` 
Output
total 16
drwx------    3 root     root          4096 May 10 15:45 .
drwxr-xr-x    9 root     root          4096 May 10 15:45 ..
-rw-r--r--    1 root     root           740 May 10 15:45 README
drwxr-xr-x    2 root     root          4096 May 10 15:45 your_domain
``` 

Ahora que sabe que su solicitud será exitosa, puede editar la certbot definición del servicio para eliminar la --staging marca.

Abierto docker-compose.yml:

``` 
nano docker-compose.yml
``` 

Busque la sección del archivo con la certbot definición del servicio y reemplace la --staging bandera en la command opción con la --force-renewal bandera, que le indicará a Certbot que desea solicitar un nuevo certificado con los mismos dominios que un certificado existente. La siguiente es la certbot definición del servicio con el indicador actualizado:

``` 
...
  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain -d www.your_domain
...
``` 

Ahora puede ejecutar docker-compose up para recrear el certbot contenedor. También incluirá la --no-deps opción para indicarle a Compose que puede omitir el inicio del webserver servicio, ya que ya se está ejecutando:

``` 
docker-compose up --force-recreate --no-deps certbot
``` 

El siguiente resultado indica que su solicitud de certificado fue exitosa:

``` 
Output
Recreating certbot ... done
Attaching to certbot
certbot      | Saving debug log to /var/log/letsencrypt/letsencrypt.log
certbot      | Plugins selected: Authenticator webroot, Installer None
certbot      | Renewing an existing certificate
certbot      | Performing the following challenges:
certbot      | http-01 challenge for your_domain
certbot      | http-01 challenge for www.your_domain
certbot      | Using the webroot path /var/www/html for all unmatched domains.
certbot      | Waiting for verification...
certbot      | Cleaning up challenges
certbot      | IMPORTANT NOTES:
certbot      |  - Congratulations! Your certificate and chain have been saved at:
certbot      |    /etc/letsencrypt/live/your_domain/fullchain.pem
certbot      |    Your key file has been saved at:
certbot      |    /etc/letsencrypt/live/your_domain/privkey.pem
certbot      |    Your cert will expire on 2019-08-08. To obtain a new or tweaked
certbot      |    version of this certificate in the future, simply run certbot
certbot      |    again. To non-interactively renew *all* of your certificates, run
certbot      |    "certbot renew"
certbot      |  - Your account credentials have been saved in your Certbot
certbot      |    configuration directory at /etc/letsencrypt. You should make a
certbot      |    secure backup of this folder now. This configuration directory will
certbot      |    also contain certificates and private keys obtained by Certbot so
certbot      |    making regular backups of this folder is ideal.
certbot      |  - If you like Certbot, please consider supporting our work by:
certbot      | 
certbot      |    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
certbot      |    Donating to EFF:                    https://eff.org/donate-le
certbot      | 
certbot exited with code 0
``` 

Con sus certificados implementados, puede continuar y modificar su configuración de Nginx para incluir SSL.


## 6. Modificar la configuración del servidor web y la definición del servicio

Habilitar SSL en su configuración de Nginx implicará agregar una redirección HTTP a HTTPS, especificar su certificado SSL y ubicaciones de claves, y agregar parámetros y encabezados de seguridad.

Como vas a recrear el webserver servicio para incluir estas adiciones, puedes detenerlo ahora:

``` 
docker-compose stop webserver
```

Antes de modificar el archivo de configuración, obtenga el parámetro de seguridad Nginx recomendado de Certbot usando curl:

``` 
curl -sSLo nginx-conf/options-ssl-nginx.conf https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf
```

Este comando guardará estos parámetros en un archivo llamado options-ssl-nginx.conf, ubicado en el nginx-conf directorio.

A continuación, elimine el archivo de configuración de Nginx que creó anteriormente:

``` 
rm nginx-conf/nginx.conf
``` 

Cree y abra otra versión del archivo:

``` 
nano nginx-conf/nginx.conf
``` 

Agregue el siguiente código al archivo para redirigir HTTP a HTTPS y agregar credenciales, protocolos y encabezados de seguridad SSL. Recuerde reemplazar your_domain con su propio dominio:

``` 
server {
        listen 80;
        listen [::]:80;

        server_name your_domain www.your_domain;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }

        location / {
                rewrite ^ https://$host$request_uri? permanent;
        }
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name your_domain www.your_domain;

        index index.php index.html index.htm;

        root /var/www/html;

        server_tokens off;

        ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

        include /etc/nginx/conf.d/options-ssl-nginx.conf;

        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;
        # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
        # enable strict transport security only if you understand the implications

        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }
        
        location = /favicon.ico { 
                log_not_found off; access_log off; 
        }
        location = /robots.txt { 
                log_not_found off; access_log off; allow all; 
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }
}
``` 

El bloque del servidor HTTP especifica la raíz web para las solicitudes de renovación de Certbot al .well-known/acme-challenge directorio. También incluye una directiva de reescritura que dirige las solicitudes HTTP al directorio raíz a HTTPS.

El bloque del servidor HTTPS habilita ssl y http2. Para leer más sobre cómo HTTP/2 itera en los protocolos HTTP y los beneficios que puede tener para el rendimiento del sitio web, lea la introducción a Cómo configurar Nginx con soporte HTTP/2 en Ubuntu 18.04.

Este bloque también incluye su certificado SSL y ubicaciones de claves, junto con los parámetros de seguridad de Certbot recomendados que guardó en nginx-conf/options-ssl-nginx.conf.

Además, se incluyen algunos encabezados de seguridad que le permitirán obtener calificaciones A en cosas como los sitios de prueba del servidor SSL Labs y Security Headers. Estos encabezados incluyen X-Frame-Options, X-Content-Type-Options, Referrer Policy, Content-Security-Policyy X-XSS-Protection. El encabezado HTTP Strict Transport Security (HSTS) está comentado; habilítelo solo si comprende las implicaciones y ha evaluado su funcionalidad de "precarga".

Sus directivas root y index también se encuentran en este bloque, al igual que el resto de los bloques de ubicación específicos de WordPress que se analizan en el Paso 1.

Una vez que haya terminado de editar, guarde y cierre el archivo.

Antes de recrear el webserver servicio, deberá agregar una 443 asignación de puerto a webserver la definición de su servicio.

Abra su docker-compose.yml archivo:

``` 
nano docker-compose.yml
``` 

En la webserver definición del servicio, agregue la siguiente asignación de puertos:

``` 
...
  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - app-network
Aquí está el docker-compose.ymlarchivo completo después de las ediciones:

~/wordpress/docker-compose.yml
version: '3'

services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes: 
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on: 
      - db
    image: wordpress:5.1.1-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network

  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - app-network

  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain -d www.your_domain

volumes:
  certbot-etc:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge  
``` 

Guarde y cierre el archivo cuando haya terminado de editarlo.

Recrea el webserver servicio:

``` 
docker-compose up -d --force-recreate --no-deps webserver
``` 

Consulta tus servicios con docker-compose ps:

``` 
docker-compose ps
``` 

El resultado debe indicar que sus servicios db, wordpress y webserver se están ejecutando:

``` 
Output
  Name                 Command               State                     Ports                  
----------------------------------------------------------------------------------------------
certbot     certbot certonly --webroot ...   Exit 0                                           
db          docker-entrypoint.sh --def ...   Up       3306/tcp, 33060/tcp                     
webserver   nginx -g daemon off;             Up       0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
wordpress   docker-entrypoint.sh php-fpm     Up       9000/tcp    
``` 

Con sus contenedores en ejecución, puede completar su instalación de WordPress a través de la interfaz web.


## 7. completar la instalación a través de la interfaz web

Con sus contenedores en ejecución, finalice la instalación a través de la interfaz web de WordPress.

En su navegador web, navegue hasta el dominio de su servidor. Recuerde sustituirlo your_domain por su propio nombre de dominio:

https://your_domain

Seleccione el idioma que desea utilizar. 

Después de hacer clic en Continuar, accederá a la página de configuración principal, donde deberá elegir un nombre para su sitio y un nombre de usuario. Es una buena idea elegir aquí un nombre de usuario fácil de recordar (en lugar de “admin”) y una contraseña segura. Puedes utilizar la contraseña que WordPress genera automáticamente o crear la tuya propia.

Finalmente, deberá ingresar su dirección de correo electrónico y decidir si desea disuadir a los motores de búsqueda de indexar su sitio.

Al hacer clic en Instalar WordPress en la parte inferior de la página, accederá a un mensaje de inicio de sesión.

Una vez que haya iniciado sesión, tendrá acceso al panel de administración de WordPress.

Una vez completada la instalación de WordPress, puede tomar medidas para asegurarse de que sus certificados SSL se renueven automáticamente.


## 8. Renovación de certificados

Los certificados Let's Encrypt son válidos por 90 días. Puede configurar un proceso de renovación automatizado para asegurarse de que no caduquen. Una forma de hacerlo es crear un trabajo con la cron utilidad de programación. En el siguiente ejemplo, creará un cron trabajo para ejecutar periódicamente un script que renovará sus certificados y recargará su configuración de Nginx.

Primero, abra un script llamado ssl_renew.sh:

``` 
nano ssl_renew.sh
``` 

Agregue el siguiente código al script para renovar sus certificados y recargar la configuración de su servidor web. Recuerde reemplazar el nombre de usuario de ejemplo aquí con su propio nombre de usuario no root:

``` 
#!/bin/bash

COMPOSE="/usr/local/bin/docker-compose --no-ansi"
DOCKER="/usr/bin/docker"

cd /home/sammy/wordpress/
$COMPOSE run certbot renew --dry-run && $COMPOSE kill -s SIGHUP webserver
$DOCKER system prune -af
``` 

Este script primero asigna el docker-compose binario a una variable llamada COMPOSE y especifica la --no-ansi opción que ejecutará docker-compose comandos sin caracteres de control ANSI. Luego hace lo mismo con el docker binario. Finalmente, cambia al ~/wordpress directorio del proyecto y ejecuta los siguientes docker-compose comandos:

* **docker-compose run:** Esto iniciará un certbot contenedor y anulará el command proporcionado en su certbot definición de servicio. En lugar de utilizar el certonly subcomando, se utiliza el renew subcomando, que renovará los certificados que están a punto de caducar. También se incluye la --dry-run opción de probar su script.
  
* **docker-compose kill:** Esto enviará una SIGHUP señal al webserver contenedor para recargar la configuración de Nginx.
  
Luego se ejecuta docker system prune para eliminar todos los contenedores e imágenes no utilizados.

Cierre el archivo cuando haya terminado de editarlo. Hazlo ejecutable con el siguiente comando:

``` 
chmod +x ssl_renew.sh
```

A continuación, abra su archivo raíz crontab para ejecutar el script de renovación en un intervalo específico:

``` 
sudo crontab -e 
``` 

Si es la primera vez que edita este archivo, se le pedirá que elija un editor:

``` 
Output
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]:
...
```

Al final de este archivo, agregue la siguiente línea:

``` 
crontab
...
*/5 * * * * /home/sammy/wordpress/ssl_renew.sh >> /var/log/cron.log 2>&1
``` 

Esto establecerá el intervalo de trabajo en cada cinco minutos, para que pueda comprobar si su solicitud de renovación ha funcionado según lo previsto. Se crea un archivo de registro, cron.log para registrar los resultados relevantes del trabajo.

Después de cinco minutos, verifique cron.log para confirmar si la solicitud de renovación ha tenido éxito o no:

``` 
tail -f /var/log/cron.log
``` 

El siguiente resultado confirma una renovación exitosa:

``` 
Output
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/your_domain/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

Salga ingresando CTRL+C en su terminal.

Puede modificar el crontab archivo para establecer un intervalo diario. Para ejecutar el script todos los días al mediodía, por ejemplo, modificaría la última línea del archivo de la siguiente manera:

``` 
crontab
...
0 12 * * * /home/sammy/wordpress/ssl_renew.sh >> /var/log/cron.log 2>&1
``` 

También querrás eliminar la --dry-run opción de tu ssl_renew.sh script:

``` 
#!/bin/bash

COMPOSE="/usr/local/bin/docker-compose --no-ansi"
DOCKER="/usr/bin/docker"

cd /home/sammy/wordpress/
$COMPOSE run certbot renew && $COMPOSE kill -s SIGHUP webserver
$DOCKER system prune -af
``` 

Su cron trabajo garantizará que sus certificados Let's Encrypt no caduquen renovándolos cuando sean elegibles. También puede configurar la rotación de registros con la utilidad Log rotate para rotar y comprimir sus archivos de registro.


## 9. Conclusión

En este tutorial, utilizó Docker Compose para crear una instalación de WordPress con un servidor web Nginx. Como parte de este flujo de trabajo, obtuvo certificados TLS/SSL para el dominio que desea asociar con su sitio de WordPress. Además, creó un crontrabajo para renovar estos certificados cuando sea necesario.

