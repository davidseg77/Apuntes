# Información extraída del curso Docker para desarrolladores de OW


## 1. Comandos básicos para gestionar Docker

Aqui se incluyen aquellos que no aparecen en Introducción:

* docker info: Muestra información de Docker y el SO sobre el que se halla.
* docker --help: Muestra los comandos que gestiona Docker y qué hace cada uno de ellos.


## 2. Construcción de imágenes

Aparte de lo visto en introducción, una buena práctica cuando creamos un dockerfile es usar argumentos. Un detalle es que para cuestiones de confidencialidad, como indicar un usuario o password, estas pueden indicarse en la instrucción del docker build. Yo puedo crear este dockerfile:

``` 
FROM ubuntu
MAINTAINER davidsegura
ARG user=root
ARG password
RUN echo $user $password
```

Pero quiza es más aconsejable crear el dockerfile sin tales argumentos e indicarlos posteriormente cuando ejecutamos el build de la siguiente manera:

``` 
docker build -t imagen --build-arg user=davidseg --build-arg password=contraseña .
```

**Dockerfile multistage**

También podemos crear un dockerfile para desplegar dos o más aplicaciones o servicios de una vez. Por ejemplo, vamos a crear una que incorpora go y alpine:

``` 
FROM golang:1.7.3 
WORKDIR /go/src/github.com/pchico/docker-for-devs/go-multi-stage
RUN go get github.com/sirupsen/logrus
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest
WORKDIR /root/
COPY --from=0 /go/src/github.com/pchico/docker-for-devs/go-multi-stage/app .
CMD ["./app"]
```

Cuando llega al FROM de alpine, se resetea, por ello copiamos de 0 y así nos llevamos el binario de go a nuestra app alpine, construyendo de tal forma una aplicación multistage con el peso mínimo posible.


## 3. Desarrollo con contenedores

### 3.1 Docker compose

Es un fichero .yaml que vendrían a ser varias instrucciones docker run. Veamos, por ejemplo, como configurar una app wordpress con una base de datos:

```
db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
    wordpress:
      image: wordpress:latest
      depends_on:
        - db
      ports:
        - "8000:80"
```

En la página de Docker, podemos hallar una info más detallada sobre como poder usar Docker compose en función de lo que se busca. 

Parta levantar estos servicios, hacemos uso del siguiente comando:

``` 
docker compose up -d
```

### 3.2 Comandos más comunes para Compose


* docker-compose --help: Muestra los comandos que pueden usarse con docker-compose y que hace cada uno de ellos
  
* docker-compose up: Crea e inicia contenedores en base al compose.yaml
  
* docker-compose pull: Me extrae las imágenes que tenga indicadas en el compose.yaml. Con la opción --parallel tirará de todas las imágenes simultaneamente
  
* docker-compose build: Construirá aquellas imágenes o sevicios que tenga definidas en el compose.yaml
  
* docker-compose push: Va a hacer un push de todas las imagenes que tengan el campo build . en el compose.yaml
  
* docker-compose run: Parecido al docker-compose up, salvo que cuenta con parámetros más completos. Estos parámetros pueden verse con la ayuda, y entre tales podemos nombrar los contenedores, asignar volúmenes, variables de entorno...
  
* docker-compose rm: Borra los contenedores lanzados por el compose.yaml. Con el parámetro -s los parará primero y luego los borrará


### 3.3 Volúmenes

Estos pueden incluirse dentro de un Compose de la siguiente manera:

``` 
services:
  mysql:
    image: mysql
    volumes:
      - data:/var/lib/mysql
      - logs:/var/log/mysql
      - /etc:/etc
  analyzer:
    image: log-analyzer
    volumes:
      - logs:/var/log:ro
volumes:
  data:
  logs:   
```












