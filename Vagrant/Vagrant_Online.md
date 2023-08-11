# Información extraída del curso Vagrant Online de OW

## 1. Introducción

### 1.1 Lanzando nuestra primera máquina

Creamos un directorio y accedemos a él. Será desde este directorio desde donde lanzaremos nuestra primera máquina Vagrant.

``` 
vagrant init + nombre de la imagen
```

Y la levantamos

``` 
vagrant up
```

Trás levantarla, accedemos a ella:

``` 
vagrant ssh
```

### 1.2 Lanzando una máquina configurada

Añado una imagen del tipo de máquina que deseo. Por ejemplo:

``` 
vagrant box add rasmus/php7dev
``` 

Compruebo los boxes, imágenes, que tengo:

``` 
vagrant box list
```

Puedo hacer un git clone del repositorio de esta imagen en el directorio desde el cual voy a lanzar la máquina. A posteriori, lanzo la máquina.


## 2. Uso de Vagrant

### 2.1 Reempaquetar un box

Dentro del directorio donde tengo las boxes (~/.vagrant.d/boxes) existe un directorio por cada box. Puedo acceder a cada uno de ellos y ver su contenido, para por ejemplo copiarme la imagen y reempaquetarla.

``` 
vagrant box repackage centos/7 virtualbox 1703.01
```

### 2.2 Eliminar o añadir una imagen de box

Para eliminar una box:

``` 
vagrant box remove centos/7
```

Para añadir una box:

``` 
vagrant box add package.box --name centos/7
```

### 2.3 Actualización de imágenes

Para comprobar si alguna imagen esta desactualizada:

``` 
vagrant box outdated --global
```

Si tenemos dos imágenes de Ubuntu, por ejemplo, podemos eliminar la más antigua en cuanto a su versión con el siguiente comando:

``` 
vagrant box prune
```

Para actualizar:

``` 
vagrant box update
```

