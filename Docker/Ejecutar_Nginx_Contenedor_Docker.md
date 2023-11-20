# Cómo ejecutar Nginx en un contenedor Docker en Ubuntu 22.04

# 1. Introducción

Nginx es un servidor web de código abierto que se utiliza para servir sitios web estáticos o dinámicos, proxy inverso, equilibrio de carga y otras capacidades de servidor proxy y HTTP. Fue creado para manejar grandes cantidades de conexiones simultáneas y es un servidor web popular que se utiliza para alojar algunos de los sitios más grandes y de mayor tráfico de Internet.

Docker es una popular herramienta de creación de contenedores de código abierto que se utiliza para proporcionar un entorno de ejecución portátil y consistente para aplicaciones de software, al tiempo que consume menos recursos que un servidor o una máquina virtual tradicional. Docker utiliza contenedores, entornos de espacio de usuario aislados que se ejecutan a nivel del sistema operativo y comparten recursos del sistema, como el kernel y el sistema de archivos.

Al incluir Nginx en contenedores, es posible reducir algunos gastos generales de administración del sistema. Por ejemplo, no tendrá que administrar Nginx a través de un administrador de paquetes ni compilarlo desde el código fuente. El contenedor Docker le permite reemplazar todo el contenedor cuando se lanza una nueva versión de Nginx. De esta manera, sólo necesitas mantener el archivo de configuración de Nginx y tu contenido.


## 2. Requisitos previos

Para seguir este tutorial, necesitará lo siguiente:

* Un servidor Ubuntu, incluido un usuario sudo no root y un firewall.
* Docker instalado en su servidor. 


## 3. Paso 1: Descargar Nginx desde Docker Hub

Docker mantiene un sitio llamado Dockerhub , un repositorio público de archivos Docker que incluye imágenes oficiales y enviadas por los usuarios. Las imágenes oficiales de Docker se pueden utilizar para desarrollar rápidamente una aplicación, evitando tener que crear su propia imagen. La comunidad Docker mantiene estas imágenes y, a menudo, están diseñadas para los casos de uso más comunes.

Puede descargar Nginx desde una imagen de Docker prediseñada, con una configuración de Nginx predeterminada, ejecutando el siguiente comando:

``` 
docker pull nginx
```

Esto descarga todos los componentes necesarios para el contenedor. Docker los almacenará en caché, por lo que cuando ejecute el contenedor no necesitará descargar la imagen del contenedor cada vez.

Ahora que tiene Nginx instalado, puede configurar el contenedor para que sea accesible públicamente como servidor web. Para iniciar su contenedor Nginx Docker, ejecute este comando:

``` 
docker run --name docker-nginx -p 80:80 nginx
``` 

Aquí hay un resumen rápido de lo que sucede con este comando:

* **run** es el comando para crear un nuevo contenedor
* La **--name** bandera es la forma en que especifica el nombre del contenedor. Si se deja en blanco, se generará un nombre como nostálgico_hopper Será asignado.
* **-p** especifica el puerto que está exponiendo en el formato de -p local-machine-port:internal-container-port. En este caso, está asignando el puerto :80 del contenedor al puerto :80 del servidor.
* **nginx** es el nombre de la imagen en Docker Hub.

En un navegador web, ingrese la dirección IP de su servidor para revelar la página de inicio predeterminada de Nginx:

Tenga en cuenta también que en su terminal, se actualiza un registro para Nginx cuando realiza solicitudes a su servidor. Esto se debe a que está ejecutando su contenedor de forma interactiva.

En su terminal, ingrese CTRL+C para detener la ejecución del contenedor.

Debido a que cerró el contenedor, ya no podrá ver la página de destino. Puede verificar el estado del contenedor con este comando:

```
docker ps -a
``` 

El resultado revela que el contenedor Docker ha salido.

Elimine el docker-nginx contenedor existente con este comando:

``` 
docker rm docker-nginx
``` 

En el siguiente paso, separará el contenedor para permitir que funcione de forma independiente.

## 4. Paso 2: Ejecutar en modo independiente

Cree un contenedor Nginx nuevo y separado con este comando:

``` 
docker run --name docker-nginx -p 80:80 -d nginx
``` 

Al adjuntar la -d bandera, está ejecutando este contenedor en segundo plano.

La salida es el ID del contenedor:

``` 
Output
b91f3ce26553f3ffc8115d2a8a3ad2706142e73d56dc279095f673580986257
``` 

Al ejecutar el docker pscomando, encontrará información nueva sobre su contenedor:

``` 
docker ps
``` 

En lugar de Exited (0) X minutes ago, ahora tienes Up About a minuteen la STATUS columna. Observe también que la asignación de puertos también forma parte del resultado.

Ingrese la dirección IP de su servidor en un navegador para acceder nuevamente a la página de inicio predeterminada de Nginx. Esta vez se ejecuta en segundo plano porque usted especificó la -d bandera, que le indica a Docker que ejecute este contenedor en modo independiente.

Ahora tiene una instancia en ejecución de Nginx en un contenedor independiente. Actualmente, el contenedor no tiene acceso a ninguno de los archivos de su sitio web.

Detenga el contenedor ejecutando el siguiente comando:

``` 
docker stop docker-nginx
``` 

Ahora que el contenedor está detenido, puedes eliminarlo ejecutando el siguiente comando:

``` 
docker rm docker-nginx
``` 

Ahora que comprende cómo ejecutar Nginx independientemente de su contenedor, en el siguiente paso creará y configurará Nginx para crear una página de destino.

## 5. Paso 3: crear una página web para publicarla en Nginx

En este paso, creará una página personalizada para su sitio web. Esta configuración le permite tener contenido de sitio web persistente alojado fuera del contenedor.

Cree un nuevo directorio para el contenido de su sitio web dentro del directorio de inicio:

``` 
mkdir -p ~/docker-nginx/html
```

Navegue hasta él ejecutando este comando:

``` 
cd ~/docker-nginx/html
```

Cree un archivo HTML para publicarlo en su servidor. El siguiente ejemplo usa nano, pero puedes usar tu editor de texto preferido:

``` 
nano index.html
``` 

Inserte el siguiente contenido HTML:

``` 
<html>
  <head>
    <title>Docker nginx Tutorial</title>
  </head>

  <body>
    <div class="container">
      <h1>Hello DigitalOcean</h1>
      <p>This Nginx page is brought to you by Docker and DigitalOcean</p>
    </div>
  </body>
</html>
``` 

Si está utilizando el nano editor de texto, salga y guarde este archivo presionando CTRL+X, luego Y y luego ENTER.

Ahora tiene una página de índice que reemplaza la página de inicio predeterminada de Nginx.

## 6. Paso 4: Vincular el contenedor al sistema de archivos local

En este paso, vinculará Nginx a su contenedor para que sea accesible públicamente a través del puerto: 80 y lo conectará al contenido de su sitio web en el servidor.

Docker le permite vincular directorios desde el sistema de archivos local de su máquina virtual a su contenedor. Dado que desea publicar la nueva página web, deberá proporcionarle a su contenedor los archivos para procesar.

Puede copiar los archivos en el contenedor como parte de un Dockerfile, o copiarlos en el contenedor después del hecho, pero ambos métodos dejan su sitio web en un estado estático dentro del contenedor. Al utilizar la función de volúmenes de datos de Docker, puede crear un enlace simbólico entre el sistema de archivos de su servidor y el sistema de archivos del contenedor. Esto le permite editar los archivos de su página web existente y agregar otros nuevos al directorio. Con un enlace simbólico, tu contenedor tendrá acceso a estos archivos. 

El contenedor Nginx está configurado de forma predeterminada para buscar una página de índice en /usr/share/nginx/html. En su nuevo contenedor Docker, deberá darle acceso a sus archivos en esa ubicación.

Para hacer esto, use la -v bandera para asignar la ~/docker-nginx/html carpeta desde su servidor a una ruta relativa en el contenedor /usr/share/nginx/html con este comando:

``` 
docker run --name docker-nginx -p 80:80 -d -v ~/docker-nginx/html:/usr/share/nginx/html nginx
``` 

Aquí hay una breve explicación del comando:

* -v El indicador especifica que está vinculando un volumen.
* A la izquierda de : está la ubicación de su directorio en su servidor ~/docker-nginx/html.
* A la derecha de : está la ubicación que está vinculando simbólicamente a su contenedor /usr/share/nginx/html.

Después de ejecutar ese comando, ingrese la dirección IP de su servidor en su navegador para ver su nueva página de destino.

Si está satisfecho con la configuración predeterminada de Nginx, no hay nada más que configurar.

Puede cargar más contenido al ~/docker-nginx/html/ directorio y se agregará a su sitio web en vivo.

Por ejemplo, si modifica su archivo HTML y actualiza su navegador, se actualizará en consecuencia. También puedes crear un sitio completo a partir de archivos HTML de esta manera. Por ejemplo, si agrega una about.html página, puede acceder a ella sin necesidad de interactuar con el contenedor.http://your_server_ip/about.html


## 7. Paso 5: usar su propio archivo de configuración de Nginx (opcional)

Si desea tener más control sobre cómo funciona Nginx, puede usar un archivo de configuración de Nginx personalizado con el contenedor Docker.

Primero, asegúrese de estar nuevamente en el directorio del proyecto de nivel superior:

``` 
cd ~/docker-nginx
``` 

Copie el directorio de configuración de Nginx en la carpeta de su proyecto usando el comando de copia de Docker:

``` 
docker cp docker-nginx:/etc/nginx/conf.d/default.conf default.conf
``` 

Dado que utilizará un .conf archivo personalizado para Nginx, deberá reconstruir el contenedor.

Primero detenga el contenedor:

``` 
docker stop docker-nginx
``` 

Luego retírelo:

``` 
docker rm docker-nginx
``` 

Ahora puede editar el archivo de configuración predeterminado de Nginx localmente para servir un nuevo directorio o usar un proxy_pass para reenviar el tráfico a otra aplicación o contenedor como lo haría con una instalación normal de Nginx. 

Una vez que haya guardado su archivo de configuración, es hora de crear el contenedor Nginx. Agregue una -v bandera con las rutas apropiadas para darle a un contenedor Nginx nuevo los enlaces apropiados para ejecutar desde su propio archivo de configuración. Por ejemplo:

``` 
docker run --name docker-nginx -p 80:80 -v ~/docker-nginx/html:/usr/share/nginx/html -v ~/docker-nginx/default.conf:/etc/nginx/conf.d/default.conf -d nginx
```

Este comando vincula las páginas del sitio web personalizado al contenedor.

Tenga en cuenta que deberá reiniciar el contenedor usando el docker restart comando cuando realice cambios en su archivo de configuración después de iniciar el contenedor. Esto se debe a que Nginx no se recarga en caliente si se cambia su archivo de configuración:

``` 
docker restart docker-nginx
``` 

Esto reiniciará su contenedor y sus cambios deberían reflejarse en las páginas asociadas.

