# Cómo compartir datos entre el contenedor Docker y el host

## 1. Introducción

En general, los contenedores Docker son efímeros y se ejecutan durante el tiempo necesario para que se complete el comando emitido en el contenedor. De forma predeterminada, cualquier dato creado dentro del contenedor solo está disponible desde dentro del contenedor y solo mientras el contenedor se está ejecutando.

Los volúmenes de Docker se pueden utilizar para compartir archivos entre un sistema host y el contenedor de Docker. Por ejemplo, digamos que desea utilizar la imagen oficial de Docker Nginx y conservar una copia permanente de los archivos de registro de Nginx para analizarlos más adelante. De forma predeterminada, la nginx imagen de Docker se registrará en el /var/log/nginx directorio dentro del contenedor Docker Nginx. Normalmente no es accesible desde el sistema de archivos del host.

En este tutorial, expliquaremos cómo hacer que los datos desde el interior del contenedor sean accesibles en la máquina host.

## 2. Requisitos previos

Para seguir esta práctica, necesitará un servidor Ubuntu con lo siguiente:

* Un usuario no root sudo con privilegios.
* Docker instalado en dicho servidor.

### 3. Paso 1: Vincular un volumen

El siguiente comando creará un directorio llamado nginx logs en el directorio de inicio de su usuario actual y lo vinculará /var/log/nginx en el contenedor:

``` 
docker run --name=nginx -d -v ~/nginxlogs:/var/log/nginx -p 5000:80 nginx
``` 

Tomémonos un momento para examinar este comando en detalle:

* --name=nginxnombra el contenedor para que podamos referirnos a él más fácilmente.
* -d desconecta el proceso y lo ejecuta en segundo plano. De lo contrario, simplemente estaríamos viendo un mensaje de Nginx vacío y no podríamos usar esta terminal hasta que matáramos a Nginx.
* -v ~/nginxlogs:/var/log/nginx configura un volumen bindmount que vincula el /var/log/nginx directorio desde el interior del contenedor Nginx al ~/nginx logs directorio en la máquina host. Docker usa a : para dividir la ruta del host de la ruta del contenedor, y la ruta del host siempre es lo primero.
* -p 5000:80 establece un puerto hacia adelante. El contenedor Nginx escucha en el puerto 80 de forma predeterminada. Esta bandera asigna el puerto del contenedor 80 al puerto 5000 en el sistema host.
* nginx especifica que el contenedor debe construirse a partir de la imagen de Nginx, que emite el comando nginx -g "daemon off"para iniciar Nginx.

### 4. Paso 2: acceder a los datos en el host

Ahora tenemos una copia de Nginx ejecutándose dentro de un contenedor Docker en nuestra máquina, y el puerto de nuestra máquina host 5000 se asigna directamente a esa copia del puerto de Nginx 80.

Cargue la dirección en un navegador web, utilizando la dirección IP o nombre de host de su servidor y el número de puerto. Debería ver:http://your_server_ip:5000

Más interesante aún, si miramos en el ~/nginx logs directorio del host, veremos lo access.log creado por el contenedor nginxque mostrará nuestra solicitud:

``` 
cat ~/nginxlogs/access.log
``` 

Esto debería mostrar algo como:

``` 
Output
203.0.113.0 - - [11/Jul/2018:00:59:11 +0000] "GET / HTTP/1.1" 200 612 "-"
"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36
(KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36" "-"
``` 

Si realiza algún cambio en la ~/nginx logs carpeta, también podrá verlo desde el interior del contenedor Docker en tiempo real.

### 5. Conclusión

En este tutorial demostramos cómo crear un volumen de datos Docker para compartir información entre un contenedor y el sistema de archivos host. Esto resulta útil en entornos de desarrollo, donde es necesario tener acceso a registros para la depuración. 



