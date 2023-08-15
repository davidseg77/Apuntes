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







