# Construyendo un servidor web Apache a través de un Dockerfile

## 1. Cree un directorio para los archivos del servidor Apache

Al principio, utilizamos el mkdir comando para crear un directorio específico para todos los archivos relacionados con Apache.

``` 
mkdir apache_folder
``` 

## 2. Crear un archivo Docker

Habiendo creado una carpeta, ahora continuamos y creamos un Dockerfile dentro de esa carpeta con el vi editor:

``` 
vi Dockerfile
``` 

Nada más ejecutar el comando anterior vi se abre un editor. Pegue el siguiente contenido en el Dockerfile:

``` 
FROM ubuntu 
RUN apt update 
RUN apt install –y apache2 
RUN apt install –y apache2-utils 
RUN apt clean 
EXPOSE 80
CMD [“apache2ctl”, “-D”, “FOREGROUND”]
``` 

Para salir del editor, presione ESC luego wq! luego Enter.

## 3. Etiquetar y crear la imagen de Docker

Ahora, construimos el Dockerfile usando el docker build comando. Dentro del cual, etiquetamos la imagen que se creará 1.0 y le damos un nombre personalizado a nuestra imagen (es decir, apache_image).

``` 
docker build -t apache_image:1.0 .
``` 

Una vez que se ha creado la imagen, debemos verificar su presencia usando docker images el comando.

El docker images comando nos proporciona una lista de todas las imágenes que se crean o se extraen de cualquier registro público/privado.

``` 
docker images
REPOSITORY                                                      TAG                 IMAGE ID            CREATED             SIZE
apache_image                                                     1.0                 a738dbef66ef        15 seconds ago      133MB
```

## 4. Ejecute la imagen de Docker como contenedor

Una vez que se haya creado la imagen, ejecútela como un contenedor localmente:

Ejecutamos el contenedor en modo independiente para que se ejecute continuamente en segundo plano. Incluir -d en el docker run comando.
Para alojar el servidor Apache, proporcionamos el puerto 80(HTTP) para el mismo. Utilice para -p 80:80 que el servidor se ejecute en localhost.
Por lo tanto, el docker run comando también toma la imagen junto con la etiqueta asociada como entrada para ejecutarla como contenedor.

``` 
docker run --name myapache -d -p 80:80 apache_image:1.0
``` 

``` 
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
443848c30b74        apache_image:1.0     "/docker-entrypoint.…"   7 seconds ago       Up 6 seconds        0.0.0.0:80->80/tcp   myapache
``` 

## 5. Revise la presencia en línea del servidor Apache

Para probar la presencia del servidor Apache en el sistema, visite cualquier navegador local y escriba localhost.

