# Información extraida del curso Introducción a Docker de OW

## 1. Instalación de Docker

Para ello, vamos a la web de Docker y descargamos Docker para nuestro equipo. Seguimos los pasos que se nos indican en la web y no habrá problema.

## 2. Contenedores e imágenes

* **Docker Registry:** Es un servicio que permite almacenar imágenes. Se hallan en Docker Hub.

### 2.1 Gestión de imágenes

* docker images: Muestra las imágenes que tenemos.
* docker history: Muestra el historial de cambios que tiene cierta imagen.
* docker inspect: Se usa para mostrar información de una imagen.
* docker save/load: Salva o carga una imagen a partir de un tag.
* docker rmi: Borra una imagen. 

### 2.2 Gestión de contenedores

* docker attach: Agregamos a un contenedor.
* docker exec: Ejecutamos ciertas acciones sobre un contenedor.
* docker inspect: Se usa para mostrar información de una imagen.
* docker kill: Matamos el contenedor.
* docker logs: Muestra los logs de un contenedor.
* docker pause/unpause: Pausa o reanuda un contenedor.
* docker port: Muestra los puertos de un contenedor.
* docker ps: Muestra los procesos de un contenedor.
* docker rename: Renombra un contenedor.
* docker start/stop/restart: Inicia, detiene o reinicia un contenedor.
* docker rm: Borra un contenedor, pero este ha de estar parado.
* docker run: Ejecuta un contenedor a partir de una imagen.
* docker stats: Ver los estados de un contenedor.
* docker top: Monitoriza las acciones de un contenedor.
* docker update: Actualiza un contenedor en base a una imagen.


## 3. Construir y subir imágenes al registro

Dos maneras:

1. Construyendola a través de docker run y después subirla a Docker Hub, mediante docker commit, docker login y docker push.

2. Creando un directorio para nuestra imagen y luego crearla a través de un **Dockerfile**. Por ejemplo, este sería un Dockerfile para la creación de una imagen de apache + php + passenger.

``` 
FROM eboraas/apache-php:latest

MAINTAINER David Segura <davidsegura@gmail.com>

RUN apt-get update && apt-get install -y libapache2-mod-passenger
```

Trás ello, se construye:

``` 
docker build -t ignoto/apache-php-passenger:v1 .
```

Este comando ha de ejecutarse desde el directorio donde hayamos creado el dockerfile.

Una vez construida nuestra imagen, la vamos a llevar a un repositorio privado para tenerla registrada.

Primero, la tageamos:

``` 
docker tag + Image ID + nombre a darle (ignoto/apache-php-passenger:latest)
```

Después nos logueamos en docker hub:

``` 
docker login
```

Y pusheamos:

``` 
docker push + nombre de la imagen (ignoto/apache-php-passenger:latest)
```


## 4. Networking

* docker network connect/disconnect: Conectarnos o desconectarnos de una red.
* docker network create: Crear una red.
* docker network inspect: Inspeccionar una red de forma detallada.
* docker network ls: Listar las redes que tenemos.
* docker network rm: Eliminar una red.

Después de crear una red, la asociamos con un contenedor de la siguiente manera al crearlo:

``` 
docker run -d --network=mired --name web1 eboraas/apache-php
```


## 5. Almacenamiento

* docker volume create: Crear un volumen.
* docker volume inspect: Inspeccionar de forma detallada un volumen.
* docker volume ls: Lista las volúmenes que tengamos.
* docker volume rm: Elimina un volumen a especificar.


 




