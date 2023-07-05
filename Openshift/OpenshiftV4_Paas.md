# Info del curso de Openshift V4 como Paas de OW

## 1. Despliegue de aplicaciones en OpenShift v4

### Despliegue de aplicaciones desde imágenes con oc

Para crear un despliegue desde la imagen josedom24/test_web:v1 que se llame test-web ejecutamos el comando:

```
oc new-app josedom24/test_web:v1 --name test-web
``` 

Como vemos se han creado varios recursos:

* Ha encontrado una imagen llamada josedom24/test_web:v1 en Docker Hub.
* Ha creado un recurso ImageStream que ha llamado igual que la aplicación y la ha etiquetado con la misma etiqueta que tiene la imagen y que referencia a la imagen original.
* Ha creado un recurso Deployment responsable de desplegar los recursos necesarios para ejecutar los Pods.
* Ha creado un recurso Service que nos posibilita el acceso a la aplicación.
* No ha creado un recurso Route para el acceso por medio de una URL, pero nos ha indicado el comando necesario para crearlo.

Creamos el recurso Route:

```
oc expose service/test-web
```

Y comprobamos los recursos que se han creado:

```
oc status
```

También podemos ver los recursos que hemos creado, ejecutando:

```
oc get all
```

Veamos algunos aspectos con detalle:

- Vemos que se han creado dos despliegues, dos recursos ReplicaSet: En realidad, en el proceso interno de creación del despliegue se crea un ReplicaSet pero no tiene indicada la imagen, por eso falla y a continuación se vuelve a crear otro que ya si funciona y crea el pod.

- Vemos como se ha creado un recurso ImageStream que apunta a la imagen que hemos indicado. Podemos ver los detalles de este recurso ejecutando:

```
 oc describe is test-web
```  

El recurso Deployment hace referencia en su definición al recurso ImageStream:

```
 oc describe deploy test-web
```

Podemos comprobar que si accedemos a la URL generada por el recurso Route, la aplicación está funcionando.

Para eliminar la aplicación, tenemos que borrar los recursos que hemos creado:

```
 oc delete deploy test-web
 oc delete service test-web
 oc delete route test-web
 oc delete is test-web
```

### Despliegue de aplicaciones desde código fuente con oc

**Despliegue de una página estática con un servidor apache2**

Queremos construir una imagen con un servidor web a partir de un repositorio donde tenemos una página web estática, para ello ejecutaremos:

```
oc new-app https://github.com/josedom24/osv4_html.git --name=app1
error: No language matched the source repository
```

Vemos que no es posible averiguar el lenguaje con el que está escrito, por lo que tendremos que indicar la Builder Image que vamos a utilizar.

Las Builder Image están referenciadas por una Image Stream, por ejemplo, podemos buscar las Builder Image con el servidor web apache:

```
oc new-app -S httpd
```

Obtenemos además las distintas etiquetas que podemos usar, que corresponden a distintas versiones del servidor web. Por lo tanto si queremos generar una imagen con el código de nuestro repositorio usando de base una imagen basada en httpd (al no indicar la etiqueta se escogerá la latest que apunta a la última versión de la imagen), ejecutamos:

```
oc new-app httpd~https://github.com/josedom24/osv4_html.git --name=app1
```

Como vemos se han creado varios recursos:

* Un ImageStream app1 que apuntará a la nueva imagen que vamos a generar, y que se utilizará en la definición del despliegue.
* Un BuildConfig, que guardará la configuración necesaria para construir la nueva imagen. De forma automática creará un objeto Build responsable de crear la imagen. Este proceso se realizará en un build pod.
* Un recurso Deployment responsable de desplegar los recursos necesario para ejecutar los Pods.
* Un recurso Service que nos posibilita el acceso a la aplicación.
* No ha creado un recurso Route para el acceso por medio de una URL, pero nos ha indicado el comando necesario para crearlo.
  
Podemos comprobar todos los recursos que se han creado, ejecutando:

```
oc status
```

También podemos ver los recursos que hemos creado, ejecutando:

```
oc get all
```

Finamente, creamos el recurso Route:

```
oc expose service app1
```

Y accedemos a la aplicación.

**Despliegue de una página estática con un servidor nginx**

Si queremos crear una aplicación con la página web de nuestro repositorio con una imagen basada en nginx, podemos buscar los recursos Images Stream que tenemos en el catálogo:

```
oc new-app -S nginx
```

Y posteriormente creamos la aplicación con el comando:

```
oc new-app nginx~https://github.com/josedom24/osv4_html.git --name=app2
```

**Detección automática del Builder Image**

OpenShift examina el repositorio y según los ficheros que tengamos es capaz de determinar con que lenguaje está escrito y te sugiere un Builder Image .

Si añadimos un fichero index.php a nuestro repositorio:

```
echo '<?php phpinfo();?>'> index.php
git add index.php 
git commit -am "php"
git push
```  

Y ahora creamos la aplicación sin indicar la Builder Image:

```  
oc new-app https://github.com/josedom24/osv4_html.git --name=app3
```  

Comprobamos que ha detectado que la aplicación está escrita en PHP, y nos ha seleccionado la Builder Image php:8.0-ubi8. Una vez desplegada la aplicación, creamos el objeto Route y accedemos al fichero index.php:

```
oc expose service app3
```

Si queremos usar otra versión de PHP, tendríamos que indicar la versión de la builder image, para ello buscamos las distintas versiones que nos ofrecen:

```
oc new-app -S php
```

Y ahora creamos una nueva aplicación con la versión 7.3-ubi7:

```
oc new-app php:7.3-ubi7~https://github.com/josedom24/osv4_html.git --name=app4

oc expose service app4
```

**Eliminar la aplicación**

Por ejemplo, para eliminar la aplicación app1 tendríamos que eliminar todos los recursos generados:

```
oc delete deploy app1
oc delete service app1
oc delete route app1
oc delete is app1
oc delete bc app1
```

### Despliegue de aplicaciones desde código fuente desde la consola web

Vamos a realizar el mismo ejercicio pero desde la consola web. Para ello accedemos desde la vista Developer a la opción de +Add y elegimos el apartado Git Repository.

**Despliegue de una página estática con un servidor apache2**

Queremos construir una imagen con un servidor web a partir de un repositorio donde tenemos una página web estática, para ello vamos a configurar el despliegue.

Indicamos el repositorio donde se encuentra la aplicación (https://github.com/josedom24/osv4_html.git). Y vemos que nos detecta una Builder Image para construir la nueva imagen, pero no nos muestra la que nos recomienda. Como pasaba en el apartado anterior, OpenShift no puede determinar el lenguaje con el que está escrita la aplicación, por lo que tendremos que indicar la Builder Image que vamos a utilizar. Para ello pulsamos sobre la opción Edit Import Strategy.

Vemos que ha seleccionado la estrategia de construcción (Builder Image) pero, como indicábamos, no se ha seleccionado ninguna. Nosotros podemos seleccionar una imagen para construir la nueva imagen (en nuestro caso httpd) y podemos elegir la versión de la imagen que hemos escogido.

Una vez lo hemos hecho, seguimos con la configuración de forma similar al despliegue de una imagen.

### Despliegue de aplicaciones desde Dockerfile con oc

Sigamos trabajando con el mismo repositorio y ahora vamos a suponer que queremos ejecutar nuestra aplicación con otra imagen base y hacer una configuración extra en la creación de la imagen. Tendríamos que crear un fichero Dockerfile para especificar los pasos de creación de la imagen. Para ello, creamos un fichero Dockerfile en el repositorio con el siguiente contenido:

```
FROM bitnami/nginx
WORKDIR /app
COPY . /app
```

Evidentemente, este fichero puede ser más complejo si la construcción de la imagen lo requiere. Ahora guardamos el fichero en el repositorio:

```
git add Dockerfile
git commit -am "php"
git push
```

Y ahora al intentar crear una nueva aplicación, OpenShift detectará que hay un fichero Dockerfile en el repositorio y lo utilizará para la creación automática de la imagen:

```
oc new-app https://github.com/josedom24/osv4_html.git --name=app1
```

Si queremos que la construcción se vuelva a realizar usando el mecanismo de Source-to-Image, tendremos que indicar la estrategia específicamente:

```
oc new-app https://github.com/josedom24/osv4_html.git --name=app2 --strategy=source
```

Y volverá a usar el mecanismo anterior.

### Despliegue de aplicaciones desde el catálogo con oc

**Despliegue de Templates en OpenShift**

Los Templates que nos ofrece OpenShift y que están guardados en el catálogo de aplicaciones, lo podemos encontrar en el proyecto openshift, para listar los que tenemos a nuestra disposición:

```
oc get templates -n openshift
```

Por ejemplo, vamos a desplegar una base de datos mariadb sin almacenamiento persistente, por lo tanto, vamos a usar el Template mariadb-ephemeral.

Los Templates tienen definidos una serie de parámetros que podemos utilizar para configurar el despliegue. Al instanciar un Template algunos parámetros hay que indicarlos de forma obligatoria, y otros, si no se ponen, se inicializan con valores por defecto.

Para ver todas las características de un Template, incluso los parámetros podemos ejecutar:

```
oc describe template mariadb-ephemeral -n openshift
```

Si queremos ver en particular los parámetros, ejecutamos:

```
oc process --parameters mariadb-ephemeral -n openshift
```

Donde observamos el valor de los parámetros, su descripción y su valor por defecto, si no se indican.

Para crear la aplicación desde el Template ejecutamos:

``` 
oc new-app mariadb-ephemeral -p MYSQL_USER=jose -p MYSQL_PASSWORD=asdasd -p MYSQL_ROOT_PASSWORD=asdasd -p MYSQL_DATABASE=blog
```

Como vemos se han creado tres recursos: un Secret, un Service y un DeploymentConfig, con los valores de configuración que hemos indicado:

```
oc get secret,service,dc,rc,pod
```

Vamos a comprobar que podemos acceder a la base de datos:

``` 
oc exec pod/mariadb-1-pb49r -it pod/mariadb-1-pb49r -- mysql -u jose -pasdasd -h mariadb blog
```





