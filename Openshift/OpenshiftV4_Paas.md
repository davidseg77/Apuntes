# Info del curso de Openshift V4 como Paas de OW

## 1. Despliegue de aplicaciones en OpenShift v4

### 1.1 Despliegue de aplicaciones desde imágenes con oc

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

### 1.2 Despliegue de aplicaciones desde código fuente con oc

#### 1.2.1 Despliegue de una página estática con un servidor apache2

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

#### 1.2.2 Despliegue de una página estática con un servidor nginx

Si queremos crear una aplicación con la página web de nuestro repositorio con una imagen basada en nginx, podemos buscar los recursos Images Stream que tenemos en el catálogo:

```
oc new-app -S nginx
```

Y posteriormente creamos la aplicación con el comando:

```
oc new-app nginx~https://github.com/josedom24/osv4_html.git --name=app2
```

#### 1.2.3 Detección automática del Builder Image

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

#### 1.2.4 Eliminar la aplicación

Por ejemplo, para eliminar la aplicación app1 tendríamos que eliminar todos los recursos generados:

```
oc delete deploy app1
oc delete service app1
oc delete route app1
oc delete is app1
oc delete bc app1
```

### 1.3 Despliegue de aplicaciones desde código fuente desde la consola web

Vamos a realizar el mismo ejercicio pero desde la consola web. Para ello accedemos desde la vista Developer a la opción de +Add y elegimos el apartado Git Repository.

Despliegue de una página estática con un servidor apache2**

Queremos construir una imagen con un servidor web a partir de un repositorio donde tenemos una página web estática, para ello vamos a configurar el despliegue.

Indicamos el repositorio donde se encuentra la aplicación (https://github.com/josedom24/osv4_html.git). Y vemos que nos detecta una Builder Image para construir la nueva imagen, pero no nos muestra la que nos recomienda. Como pasaba en el apartado anterior, OpenShift no puede determinar el lenguaje con el que está escrita la aplicación, por lo que tendremos que indicar la Builder Image que vamos a utilizar. Para ello pulsamos sobre la opción Edit Import Strategy.

Vemos que ha seleccionado la estrategia de construcción (Builder Image) pero, como indicábamos, no se ha seleccionado ninguna. Nosotros podemos seleccionar una imagen para construir la nueva imagen (en nuestro caso httpd) y podemos elegir la versión de la imagen que hemos escogido.

Una vez lo hemos hecho, seguimos con la configuración de forma similar al despliegue de una imagen.

### 1.4 Despliegue de aplicaciones desde Dockerfile con oc

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

### 1.5 Despliegue de aplicaciones desde el catálogo con oc

#### 1.5.1 Despliegue de Templates en OpenShift

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

## 2. ImageStreams: Gestión de imágenes en OpenShift v4

### 2.1 ImageStream a imágenes del registro interno

Las ImageStreams que apuntan a imágenes internas, la podemos encontrar en el catálogo, por ejemplo si buscamos por la palabra "httpd", podemos encontrar:

```
oc new-app -S httpd
```

Realmente los ImageStreams que apuntan a imágenes internas se encuentran en el proyecto openshift, podemos verlos todos, ejecutando:

``` 
oc get is -n openshift
```

Por ejemplo, si nos centramos en el ImageStream httpd:

```
oc get is httpd -n openshift
```

Vemos el nombre, la ruta a dicha imagen y las etiquetas que tiene definidas.

Tenemos además los recursos ImageStreamTags que representan las etiquetas de un ImageStream y son las que verdaderamente apuntan a una imagen, por ejemplo:

```  
oc get istag httpd:2.4 -n openshift
```

Vemos como la etiqueta 2.4 de la ImageStream httpd está apuntado a una imagen en el registro interno (image-registry.openshift-image-registry.svc:5000) con un nombre y un determinado identificador. Esto me asegura que cada vez que use httpd:2.4 estaré usando la misma imagen.

Finalmente indicar que con el comando oc get images ejecutado como administrador del clúster puedo acceder a las imágenes del registro interno, por ejemplo vamos a buscar la imagen anterior en el registro:

```
oc get images | grep httpd
```

#### 2.1.1 Uso de las ImageStream internas

Como hemos indicado, usaremos las ImageStream para la creación de nuevas aplicaciones o la construcción de nuevas imágenes.

* *Creación de un nuevo despliegue*
  
Por ejemplo, podríamos desplegar una nueva aplicación a partir de la ImageStream httpd:2.4:

```  
oc new-app httpd:2.4 --name=web1
```  

Una vez creado el despliegue, podríamos ver la imagen que se está usando para el despliegue:

```
oc describe deploy web1
```

Evidentemente su ID coincide con la imagen que vimos anteriormente y que estaba apuntada por httpd:2.4.

#### 2.1.2 Construcción de imágenes

Del mismo modo vamos a usar la ImageStream httpd:2.4 como base para crear otra imagen:

```
oc new-app httpd:2.4~https://github.com/josedom24/osv4_html --name=web2
```

Evidentemente se creará un nuevo recurso ImageStream que apunta a la nueva imagen que estamos construyendo:

```
oc get is

oc describe is web2
```

Y comprobamos que se ha creado esa imagen en el registro interno:

```
oc get images | grep web2
```

### 2.2 Creación de ImageStream

Tenemos tres maneras de crear recursos ImageStream:

#### 2.2.1 Creación de ImageStream con new-app

Cada vez que creamos un nuevo despliegue con la instrucción oc new-app se crea una objeto ImageStream que apunta a la imagen utilizada o construida. Por ejemplo:

```
oc new-app bitnami/nginx --name=web3
``` 

Si obtenemos los recursos ImageStream:

```
oc get is
```

Como vemos el nuevo ImageStream ha sido nombrado con el nombre que hemos puesto a la aplicación, y en este caso lo ha etiquetado con el mismo nombre de etiqueta (latest) que la imagen original:

Si obtenemos información del recurso:

```
oc describe is web3
```

Observamos que tenemos una imagen con una etiqueta y encontramos el nombre de la imagen a la que estamos apuntando, cuyo ID se corresponde con la imagen original.

Finalmente podemos comprobar que tenemos la imagen en nuestro registro interno:

``` 
oc get images|grep nginx
```

#### 2.2.2 Creación de ImageStream con import-image

Otra manera de crear recursos ImageStream es usando el comando import-image. En este caso indicamos el nombre de la ImageStream y la imagen a la que apunta. En este ejemplo vamos a utilizar el mismo nombre y vamos a apuntar a otra imagen indicando otra etiqueta:

```
oc import-image web4:latest --from=docker.io/bitnami/apache --confirm
``` 

Es necesario poner la opción --confirm cuando creamos por primera vez un objeto ImageStream.

Otra forma de hacerlo sería creando antes un nuevo objeto ImageStream y luego importando la imagen:

```
oc create is web4
oc import-image web4:latest --from=docker.io/bitnami/apache
``` 

En este caso hemos creado el ImageStream web4:latest:

```
oc describe is web4
```

Si miramos los ID de las imágenes bitnami/apache en Docker Hub podemos comprobar que hemos apuntado la versión latest. Además vemos que la versión 2.4.57 es la misma imagen.

Podríamos crear un nuevo tag apuntando a la misma imagen dentro de la ImageStream web4:

```
oc import-image web4:2.4.57 --from=docker.io/bitnami/apache:2.4.57

oc describe is web4
```

Y también podríamos crear un tag apuntando a otra versión que corresponde a una imagen diferente:

```
oc import-image web4:2.4.56 --from=docker.io/bitnami/apache:2.4.56

oc describe is web4
```

Para ver las imágenes que tenemos en registro interno, ejecutamos:

```
oc get images|grep apache
```

Para terminar, podríamos usar este ImageStream para desplegar una nueva aplicación:

```
oc new-app web4:2.4.57 --name apache1
```

#### 2.2.3 Creación de ImageStream con fichero YAML

La definición de un objeto ImageStream es muy sencilla. Por ejemplo, podemos tener guardada la definición en el fichero is.yaml:

```
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: citas
spec:
  tags:
  - name: "1.0"
    from:
      kind: DockerImage
      name: josedom24/citas-backend:v1
```

Y simplemente lo creamos, ejecutando:

```
oc apply -f is.yaml

oc get is
```

### 2.3 Gestión de las etiquetas en un ImageStream

#### 2.3.1 Crear nuevas etiquetas en un ImageStream

Para hacer este ejercicio vamos a crear una ImageStream que tendrá dos etiquetas que apuntan a las dos versiones de la imagen josedom24/citas-backend. Para ello, ejecutamos:

```
oc create is citas
oc import-image citas:v1 --from=docker.io/josedom24/citas-backend:v1
oc import-image citas:v2 --from=docker.io/josedom24/citas-backend:v2
```

Si comprobamos la descripción del ImageStream citas podemos comprobar que tenemos dos etiquetas que apuntan a dos imágenes distintas:

```
oc describe is citas
``` 

Podemos, por ejemplo, crear una etiqueta que nos permita apuntar a la imagen más nueva:

```
oc tag citas:v2 citas:latest
```

Otro ejemplo, podemos crear una etiqueta para indicar la versión que queremos desplegar en producción:

```
oc tag citas:v1 citas:prod
```

Ahora tendríamos un ImageStream con 4 etiquetas apuntando a 2 imágenes distintas:

```
oc describe is citas
``` 

Para eliminar una etiqueta de un ImageStream podemos hacerlo de dos formas:

```
oc tag citas:latest -d
oc delete istag citas:latest
```

#### 2.3.2 Ejemplo: Actualización de despliegue por cambio de imagen

Vamos a crear un despliegue utilizando el ImageStream citas:prod (que actualmente apunta a la versión 1 de la imagen). Hay que tener en cuenta que la imagen usa el puerto TCP/10000 para ofrecer el servicio:

```  
oc new-app citas:prod --name=app-citas
oc expose service app-citas --port=10000
```

Se ha creado un despliegue:

```
oc get deploy,rs,pod
```

Y comprobamos que hemos desplegado la versión 1 de la aplicación:

```
curl http://app-citas-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/version
v1
```

¿Qué ocurrirá si hacemos que la etiqueta prod del ImageStream referencie a la segunda versión de la imagen?. Habrá un cambio en la imagen que se ha utilizado en el despliegue, y esto hará que se actualice el despliegue creando un nuevo ReplicaSet que creará un nuevo Pod con la nueva versión de la imagen:

```
oc tag citas:prod -d
oc tag citas:v2 citas:prod
```

Comprobamos que se ha actualizado el despliegue:

```
oc get deploy,rs,pod
```

Y que ahora tenemos la segunda versión de la imagen desplegada:

```
curl http://app-citas-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/version
v2
```  

### 2.4 Actualización automática de ImageStream

Cuando creamos un nuevo ImageStream que apunta a una imagen, podemos activar una funcionalidad que periódicamente comprueba si la imagen ha cambiado, y en caso afirmativo, actualiza el ImageStream para que apunte a la nueva imagen. Por defecto, el periodo de comprobación es de 15 minutos.

En primer lugar, vamos a crear una imagen Docker y la voy a subir a mi cuenta de Docker Hub. Para ello vamos a usar el fichero Dockerfile:

```
FROM centos:centos7
CMD echo 'Hola, esta es la versión 1 de la imagen' && exec sleep infinity
```

Para crear la imagen y subirla ejecuto las siguientes instrucciones:

```
docker login
docker build -t josedom24/imagen-prueba .
docker push josedom24/image-prueba
```

A continuación creamos el ImageStream apuntando a dicha image, con la opción --scheduled=true que es el parámetro que nos permite monitorizar la imagen original:

```
oc import-image imagen-prueba:latest --from=docker.io/josedom24/imagen-prueba:latest --scheduled=true --confirm
```

Comprobamos el id de la imagen a la que apunta:

```
oc describe is imagen-prueba
```

Ahora desplegamos esta imagen y comprobamos la versión:

```
oc new-app imagen-prueba --name=app-prueba

oc logs deploy/app-prueba
```

Ahora vamos a modificar el fichero Dockerfile y vamos a volver a subir la imagen con el mismo nombre y la misma etiqueta. Cambiamos el mensaje del fichero Dockerfile para que ponga versión 2, y volvemos a generar la imagen y subirla:

```  
docker build -t josedom24/imagen-prueba .
docker pull josedom24/image-prueba   
```

Cuando se ha subido la imagen ya vemos que se le ha asignado otro id:

``` 
latest: digest: sha256:20ed24d87a2f66eab7ddae859fe43ad101bea5b28b2101947e1468f24c52470d size: 529
```

Y esperamos los 15 minutos...

Y podemos comprobar que el ImageStream se ha actualizado:

```
oc describe is imagen-prueba
```

Y que se ha actualizado el despliegue:

```
oc get deploy,rs,pod
```

Y por tanto tenemos desplegado la nueva versión de la imagen:

```
oc logs deploy/app-prueba
```

## 3. Builds: Construcción automática de imágenes

### 3.1 Construcción de imágenes con estrategia Source-to-Image (S2I)

#### 3.1.1 Despliegue de la aplicación con la estrategia Source-to-image

Para ello usamos el comando oc new-app indicando la imagen constructora que vamos a usar y el repositorio donde se encuentra el código. Si no indicamos la imagen constructora, OpenShift examinará los ficheros del repositorio y escogerá una imagen constructora dependiendo del lenguaje de programación que haya detectado.

Por lo tanto en la primera aplicación no vamos a indicar la imagen constructora y comprobamos que imagen nos sugiere, con nuestra aplicación PHP:

``` 
oc new-app https://github.com/josedom24/osv4_php --name=app1
```

En este caso se ha escogido la imagen php:8.0-ubi8 para realizar la construcción.

Una vez ejecutado el comando, podemos comprobar que efectivamente se ha creado un objeto BuildConfig:

```
oc get bc
```

Además, se ha comenzado el proceso de creación de la imagen, y para ello se ha creado un objeto Build (que está en ejecución):

```
oc get build
```

¿Dónde se ejecuta la construcción de la nueva imagen? En un Pod de construcción:

```
oc get pod
```

Si quieres ver las tareas que se están ejecutando en el proceso de construcción, ejecuta:

```
oc logs -f bc/app1
```

Una vez terminada la construcción:

```
oc get build
```

Podemos comprobar que se ha creado el objeto ImageStream apuntando a la nueva imagen que hemos creado:

```
oc get is
```

Y se ha creado el objeto Deployment responsable de la creación de los Pods de la aplicación, que se construyen a partir de la imagen construida:

```
oc get deploy,rs,pod
```

Podemos ver como cuando el Pod constructor (app1-1-build) termina de construir la imagen, se detiene y está en estado Completed.

Por último podemos crear el objeto Route:

```
oc expose service app1
```

Y acceder a la aplicación.

#### 3.1.2 Construcción de la imagen indicando una versión de la imagen constructora

En el ejercicio anterior, al no indicar la imagen constructora, OpenShift detectó el lenguaje de programación de la aplicación guardada en el repositorio y nos ofreció una imagen constructora, en este caso una imagen PHP.

Sin embargo, es posible que indiquemos una imagen constructora explícitamente al desplegar la aplicación. Podemos buscar en el catálogo, cuantas versiones de la imagen PHP tenemos a nuestra disposición:

```
oc new-app -S php
```

Además, en la ejecución de la aplicación nos hemos dado cuenta que la aplicación espera leer una variable de entrono que se llama INFORMACION. Para indicar una versión determinada de la imagen PHP y además crear la variable de entorno en el despliegue, podemos ejecutar la siguiente instrucción:

```
oc new-app php:7.4-ubi8~https://github.com/josedom24/osv4_php --name=app1-v2 -e INFORMACION="Versión 2"
```

Comprobamos que se ha creado otro objeto BuildConfig y que se ha empezado a realizar un nuevo build:

```
oc get bc
``` 

Una vez finalizado, creamos la ruta de este despliegue y accedemos a la aplicación.


### 3.2 Construcción de imágenes con estrategia Docker + repositorio Git

#### 3.2.1 Creación del BuildConfig con estrategia Docker

Vamos a seguir trabajando con el repositorio https://github.com/josedom24/osv4_php al que vamos a añadir un fichero Dockerfile con el siguiente contenido:

```
# https://github.com/sclorg/s2i-php-container/
FROM registry.access.redhat.com/ubi8/php-74
ENV INFORMACION="Estrategia Docker"
USER 0
COPY . /tmp/src
RUN chown -R 1001:0 /tmp/src
USER 1001
# Let the assemble script to install the dependencies
RUN /usr/libexec/s2i/assemble
# Run script uses standard ways to run the application
CMD /usr/libexec/s2i/run
```

Para ello:

```
git add Dockerfile
git commit -am "Añadimos Dockerfile"
git push
```

Ahora, vamos a usar el comando oc new-build para construir la imagen. Este comando detectará en el repositorio el fichero Dockerfile y realizará una construcción con estrategia Docker. Vamos a verlo:

```
oc new-build https://github.com/josedom24/osv4_php --name=app2
```

Como vemos se crean dos objetos ImageStream: uno que referencia a la imagen indicada en el fichero Dockerfile y otro que referencia a la imagen creada, y un objeto BuildConfig:

```
oc get bc

oc get build
```  

Si quieres ver las tareas que se están ejecutando en el proceso de construcción, ejecuta:

```
oc logs -f bc/app2
```

Una vez terminada, la construcción podemos ver los objetos ImageStream que se han generado:

```
oc get is
```

Sólo hemos creado la nueva imagen, pero no hemos realizado un despliegue, podemos desplegar nuestra nueva imagen utilizando el comando oc new-app indicando la imagen que hemos creado (el nombre del objeto ImageStream que lo referencia):

```
oc new-app app2 --name=app2
```

Como vemos ha encontrado una ImageStream llamada app2:latest, que será el que se utiliza para realizar el despliegue. En este caso, oc new-app sólo ha creado un recurso Deployment y otro Service.

```
oc get deploy,rs,pod 
```

Ya nos había ocurrido en un ejemplo anterior, vemos que se han creado dos recursos ReplicaSet. En realidad, en el proceso interno de creación del despliegue se crea un ReplicaSet pero no tiene indicada la imagen, por eso falla y a continuación, se vuelve a crear otro, con la imagen que hemos indicado, que ya si funciona y crea el Pod.

Creamos el objeto Route y accedemos a la aplicación.

#### 3.2.2 Otras operaciones

Si queremos crear la imagen y desplegarla, ejecutamos:

```
oc new-app https://github.com/josedom24/osv4_php --name=app2-v2
``` 

Si queremos, no utilizar el fichero Dockerfile para la construcción y volver a usar la estrategia de Source-to-Image, hay que indicarlo explícitamente:

```
oc new-build https://github.com/josedom24/osv4_php --name=app2-v4 --strategy=source
```

O si queremos también, desplegar la aplicación:

```
oc new-app https://github.com/josedom24/osv4_php --name=app2-v4 --strategy=source
```

### 3.3 Definición del objeto BuildConfig

Como cualquier recurso de OpenShift podemos tener su definición en un fichero YAML, y crearlo a partir de este fichero.

Tenemos un fichero bc.yaml con el siguiente contenido:

```  
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  labels:
    app: app3
  name: app3
spec:
  failedBuildsHistoryLimit: 5
  output:
    to:
      kind: ImageStreamTag
      name: imagen-app3:latest
  runPolicy: Serial
  source:
    git:
      uri: https://github.com/josedom24/osv4_php
    type: Git
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: php:latest
        namespace: openshift
    type: Source
  successfulBuildsHistoryLimit: 5
  triggers:
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

Veamos los distintos parámetros:

* failedBuildsHistoryLimit: Especifica la cantidad máxima de builds fallidos que se pueden mantener en el historial de builds de la aplicación.
* successfulBuildsHistoryLimit: Especifica la cantidad máxima de builds que se han ejecutado de manera correcta que se pueden mantener en el historial de builds de la aplicación.
* runPolicy: Como podemos indicar varias fuentes de entrada, este parámetro indica la manera de construirlos. Por defecto el valor es Serial indicando que se deben construir de manera consecutiva. En otros escenarios es posible hacer construcciones en paralelo (Parallel).
* output: Me permite indicar donde se guardará la imagen construida. En este caso, se creará un recurso ImageStream (campo kind) y el nombre y etiqueta asignada (name).
* source: Indica la fuente de información donde se toman los ficheros. En este caso es un repositorio Git (type), indicando la dirección git.uri.
* strategy: Nos define la estrategia de construcción. Indicamos la imagen constructora (from): indicamos el tipo (kind), el nombre (name) y el proyecto donde se encuentra (namespace). También indicamos el tipo de estrategia, con el parámetro type.
* triggers: Se definen los disparadores que comienza una nueva construcción de forma automática. En este caso tenemos dos posibles disparadores: que la configuración del recurso cambie (type: ConfigChange) o que la imagen de construcción cambie (type: ImageChange). Más adelante veremos que hay otro tipo de disparador.
  
Para crear el nuevo objeto BuildConfig, ejecutamos:

```
oc apply -f bc.yaml 

oc get bc
```

Y comprobamos que se ha comenzando una construcción:

```
oc get build
```

Vemos que la construcción ha dado un fallo: InvalidOutputReference. Esto es debido a que la ImageStream que habíamos configurado de salida: imagen-app3no existe. Para crear el objeto ImageStream ejecutamos:

```
oc create is imagen-app3
```

Una vez que la hemos creado, comprobamos que el build ya puede seguir ejecutándose:

```
oc get build
```

Si queremos ver las características de los recursos que hemos creado, podemos ejecutar:

```
oc describe bc app3
oc describe build app3-1
```

Cuando finalice la construcción de la imagen, podríamos desplegarla ejecutando:

```
oc new-app imagen-app3 --name=app3
```

#### 3.3.1 Definición de los objetos BuildConfig definidos en los ejercicios anteriores

Recordamos que teníamos un BuildConfig app1 que utilizaba la estrategia Source-to-Image (S2I) y obtenía los ficheros de un repositorio Git, podemos ver su definición ejecutando:

```
oc get bc app1 -o yaml
...    
spec:
  ...
  output:
    to:
      kind: ImageStreamTag
      name: app1:latest
  ...
  source:
    git:
      uri: https://github.com/josedom24/osv4_php
    type: Git
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: php:8.0-ubi8
        namespace: openshift
    type: Source
  triggers:
  - github:
      secret: 35iNue334T9PuT4Sp2sV
    type: GitHub
  - generic:
      secret: hwvVeSjQhcjyDSNdvGYt
    type: Generic
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

El BuildConfig app2 utilizaba la estrategia Docker y obtenía el fichero Dockerfile de un repositorio Git, podemos ver su definición ejecutando:

```
oc get bc app2 -o yaml
...
spec:
  ...
  output:
    to:
      kind: ImageStreamTag
      name: app2:latest
  ...
  source:
    git:
      uri: https://github.com/josedom24/osv4_php
    type: Git
  strategy:
    dockerStrategy:
      from:
        kind: ImageStreamTag
        name: php-74:latest
    type: Docker
  triggers:
  - github:
      secret: bb3ZKLHH7hTfQ53Wc5me
    type: GitHub
  - generic:
      secret: 3G0Sc0NhPSuuFhlNGEQ6
    type: Generic
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

### 3.4 Actualización manual de un build

En este apartado vamos a aprender los comandos que nos permiten gestionar los procesos de construcción a partir de un objeto BuildConfig. Para ello vamos a crear un BuildConfig usando la estrategia Docker y los ficheros necesarios se encuentra en el repositorio https://github.com/josedom24/osv4_python.

La aplicación muestra los municipios de una provincia, el nombre de la provincia se indica en la variable de entorno PROVINCIA. El fichero Dockerfile crea esta variable con una valor determinado.

Por lo tanto lo primero que vamos a hacer es crear el objeto BuildCondig, para ello:

```
oc new-build https://github.com/josedom24/osv4_python --name=app4

oc get bc 
```

Como vemos en el campo LATEST, se indica que se ha creado un build, que de forma automática ha llamado app4-1. Cuando termina la construcción, comprobamos que se ha creado un nuevo ImageStream apuntando a la nueva imagen:

```
oc get is -o name

imagestream.image.openshift.io/app4
```

Ahora, podríamos crear una nueva aplicación que utilizará esta nueva imagen que hemos generado (vamos a llamar a la aplicación con el mismo nombre que el build, aunque se podría llamar de forma distinta):

```
oc new-app app4 --name=app4
oc expose service app4
```

#### 3.4.1Primera modificación: Modificación de la aplicación

¿Qué pasa si mi equipo de desarrollo saca una nueva versión de la aplicación y queremos desplegar esta nueva versión?. Para ello vamos a modificar la aplicación y luego vamos a lanzar una nueva construcción:

Modificamos el fichero app/templates/base.html y cambiamos la línea: <h1>Temperaturas: {{prov}}</h1> por esta otra <h1>Temperaturas: {{prov}} Versión 2</h1>.

Guardamos los cambios en el repositorio:

```
 git commit -am "Versión 2"
 git push
```

Lanzamos manualmente una nueva construcción de la imagen:

```
 oc start-build app4
 build.build.openshift.io/app4-2 started

 oc get bc
 ```

 Vemos como tenemos en ejecución otro Pod constructor (app4-2-build) donde se está creando la nueva imagen.

¿Qué ha ocurrido al finalizar la construcción de la nueva imagen? El despliegue se ha actualizado, al cambiar la imagen de origen, y por tanto ha creado un nuevo recurso ReplicaSet que ha creado un nuevo Pod:

```
 oc get rs

 oc get pod
```

Si accedemos a la aplicación vemos la modificación.

#### 3.4.2 Segunda modificación: Modificación del valor de la variable de entorno

¿Qué pasa si modificamos el fichero Dockerfile, por ejemplo para cambiar el valor de la variable de entorno? De la misma forma, habrá que construir una nueva imagen, y desplegarla de nuevo.

Modifica el fichero Dockerfile y cambia el valor de la variable de entorno: ENV PROVINCIA=cadiz.

Guardamos los cambios en el repositorio:

``` 
 git commit -am "Modificación Dockerfile"
 git push
```

Lanzamos manualmente una nueva construcción de la imagen:

```
 oc start-build app4

 oc get bc

 oc get build

 oc get pod
```

Al terminar la construcción de la imagen, se ha actualizado el Deployment:

```
 oc get rs

 oc get pod
```

Si accedemos a la aplicación vemos la modificación.

#### 3.4.3 Otras operaciones

Podemos cancelar una construcción ejecutando la instrucción oc cancel-build:

```
oc start-build app4 
build.build.openshift.io/app4-5 started

oc cancel-build app4-5
build.build.openshift.io/app4-5 marked for cancellation, waiting to be cancelled
```

Por último si borramos el objeto BuildConfig se borrarán todas los objetos Builds y todos los Pods de construcción:

```
oc delete bc app4
buildconfig.build.openshift.io "app4" deleted

oc get build
No resources found in josedom24-dev namespace.

oc get pod
```

### 3.5 Construcción de imágenes desde ficheros locales

A este tipo de construcción de imágenes se la llama Binary Build, y nos permite cargar código fuente directamente en un build en lugar de indicar un repositorio Git.

#### 3.5.1 Ejemplo 1: Construcción de cambios locales del código fuente

Vamos a crear una aplicación a partir de un repositorio de ejemplo de OpenShift que nos despliega una aplicación construida con Python Flask: https://github.com/devfile-samples/devfile-sample-python-basic.git.

Para ello ejecutamos las siguientes instrucciones:

```
oc new-app https://github.com/devfile-samples/devfile-sample-python-basic.git --name=app5
oc expose service app5
```

Después de construir la imagen, se despliega y podemos acceder a la aplicación

A continuación nos clonamos el repositorio y hacemos un cambio en el código:

```
git clone https://github.com/devfile-samples/devfile-sample-python-basic.git
cd devfile-sample-python-basic/
```

Modificamos el fichero app.py y cambiamos el mensaje Hello World! por otro mensaje.

A continuación vamos a crear una nueva construcción, pero le vamos a indicar que coja los ficheros del directorio donde hemos realizado el cambio, para ello:

```
oc start-build app5 --from-dir="."

oc get bc

oc get build
```

Una vez terminada dicha construcción, accedemos de nuevo a la aplicación para asegurarnos que se ha modificado con el nuevo código. Una vez que probemos que los cambios son adecuados, podríamos guardarlos en el repositorio Git.

#### 3.5.2 Ejemplo 2: Construcción de imagen usando código privado

En este caso partimos de un código fuente que tenemos en local, no está en ningún repositorio Git.

En un directorio local tenemos un fichero Dockerfile con el siguiente contenido:

```
FROM centos:centos7
EXPOSE 8080
COPY index.html /var/run/web/index.html
CMD cd /var/run/web && python -m SimpleHTTPServer 8080
```

Y un dichero index.html:

```
<html lang="es">
  <head>
    <title>Local App</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  </head>
  <body>
    <h1>Curso OpenShift v4</h1>
    <p>Aplicación local</p>
  </body>
</html>
```

A continuación vamos a crear un BuildConfig de la siguiente manera:

```
oc new-build --strategy docker --binary  --name app6
```

Como vemos indicamos la estrategia de construcción, que usará el fichero Dockerfile e indicamos con el parámetro --binary que el código fuente habrá que inyectarlo en cada una de las construcciones. Podemos observar que al crear el objeto BuildConfig no se disparado ningún build:

```
oc get bc
```

Tenemos que iniciar la construcción de forma manual indicando donde está el código fuente:

```
oc start-build app6 --from-dir="."

oc get build
```

Una vez concluida la construcción comprobamos que se ha creado la imagen, y lanzamos una nueva aplicación:

```
$ oc get is -o name
imagestream.image.openshift.io/app6

oc new-app app6
oc expose service app6
```

Finalmente, accedemos a la aplicación.

### 3.6 Construcción de imágenes con Dockerfile en línea

En este tipo de construcción vamos a tener el contenido del fichero Dockerfile dentro de la definición del objeto BuildConfig.

Vamos a ver dos ejemplos:

#### 3.6.1 Ejemplo 1: BuildConfig con un Dockerfile inline

Partimos de la definición de un BuildConfig que tenemos en el fichero bc-dockerfile1.yaml:

```
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  labels:
    app: app7
  name: app7
spec:
  failedBuildsHistoryLimit: 5
  output:
    to:
      kind: ImageStreamTag
      name: imagen-app7:latest
  runPolicy: Serial
  source:
    dockerfile: |
      FROM centos:centos7
      CMD echo 'Hola, estás probando un dockerfile inline' && exec sleep infinity
    type: dockerfile
  strategy:
    dockerStrategy:
    type: Docker
  successfulBuildsHistoryLimit: 5
  triggers:
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

Creamos el objeto BuildConfig, creando en primer lugar la ImageStream que hemos indicado como salida:

```
oc create is imagen-app7
oc apply -f bc-dockerfile1.yaml 
```

```
oc get bc
oc get build
```

Una vez creada la nueva imagen, podemos desplegarla y comprobar la salida del Pod que se ha creado:

```
oc new-app imagen-app7 --name=app7

oc logs deploy/app7
Hola, estás probando un dockerfile inline
```

#### 3.6.2 Ejemplo 2: BuildConfig con un Dockerfile sustituido

En este caso partimos de un repositorio Git donde tenemos una aplicación y un Dockerfile para la creación de la imagen. Sin embargo, vamos a partir del código de ese repositorio pero vamos a sustituir el Dockerfile por uno que tenemos definido en el BuildConfig. Para ello partimos de la definición que tenemos en el fichero bc-dockerfile2.yaml:

```
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  labels:
    app: app8
  name: app8
spec:
  failedBuildsHistoryLimit: 5
  output:
    to:
      kind: ImageStreamTag
      name: imagen-app8:latest
  runPolicy: Serial
  source:
    type: Git
    git:
      uri: https://github.com/josedom24/osv4_python                    
    dockerfile: |
      FROM bitnami/python:3.7
      WORKDIR /app
      COPY app /app
      RUN pip install --upgrade pip
      RUN pip3 install --no-cache-dir -r requirements.txt
      ENV PROVINCIA=cadiz
      EXPOSE 8000
      CMD [ "python3", "app.py"]
  strategy:
    dockerStrategy:
    type: Docker
  successfulBuildsHistoryLimit: 5
  triggers:
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

No se va a ejecutar el fichero Dockerfile que se encuentra en el repositorio Git, se va a sustituir por el que hemos escrito en la definición anterior. Por la tanto la variable PROVINCIA no será sevilla, tendrá el valor de cadiz, además se ha utilizado otra versión de la imagen base para construir la imagen.

Y ejecutamos:

```
oc create is imagen-app8
oc apply -f bc-dockerfile2.yaml
```

``` 
oc new-app imagen-app8 --name=app8
oc expose service app8
```

Y accedemos a la aplicación.

### 3.7 Actualización automática de un build

En primer lugar vamos a crear un objeto ImageStream que apuntará a una imagen constructora de PHP, que luego utilizaremos para explicar el trigger ImageChange.

```
oc import-image mi-php:v1 --from=registry.access.redhat.com/ubi8/php-74 --confirm
oc new-build mi-php:v1~https://github.com/josedom24/osv4_php --strategy=source --name=app9

oc get build
```  

Si comprobamos la definición del objeto y nos centramos en la sección triggers:

```
oc get bc app9 -o yaml
...
triggers:
  - github:
      secret: -EPtf1InRvdMAwKLd-wY
    type: GitHub
  - generic:
      secret: ys1E_7ah4ghQYY_wUbI0
    type: Generic
  - type: ConfigChange
  - imageChange: {}
    type: ImageChange
```

También lo podríamos ver obteniendo información del objeto:

```
oc describe bc app9
```

Vemos que tenemos tres posibles disparadores:

* ConfigChange: Permite que se cree una nueva construcción de forma automática cuando se crea el objeto BuildConfig.
* ImageChange: Permite que se cree una nueva construcción de forma automática cuando está disponible una nueva versión de la imagen constructora.
* Webhook: Nos permite disparar una nueva construcción de forma automática cuando ocurre un evento (por ejemplo un push) en un servicio externo (por ejemplo, un repositorio GitHub). Este servicio hace una llamada a una URL que nosotros le proporcionamos que produce que se inicie el proceso de construcción.

#### 3.7.1 Ejemplo de actualización del build por ImageChange

Seguimos trabajando con el BuildConfig que hemos creado y que ha disparado el primer build al tener configurado el trigger ConfigChange.

```
oc get build
```

Comprobamos que se han creado dos ImageStreams, uno que se refiere a la imagen constructora y otro a la imagen generada:

``` 
oc get is -o name
imagestream.image.openshift.io/app9
imagestream.image.openshift.io/mi-php
```

La imagen constructora que utiliza es mi-php:v1. Si hacemos que esa etiqueta del ImageStream apunte a otra imagen, habremos cambiado la imagen constructora, y debido al trigger ImageChange se volverá a iniciar un nuevo build. En primer lugar voy a eliminar la etiqueta (el objeto ImageStreamTag):

```
oc delete istag mi-php:v1
```

Y voy a volver a hacer que referencie a otra imagen:

``` 
oc import-image mi-php:v1 --from=registry.access.redhat.com/ubi8/php-80
```

Y comprobamos que se ha iniciado otro build, que construir una nueva imagen:

```
oc get build
```

## 4. Deployment us DeploymentConfig

### 4.1 Creación de un DeployConfig al crear una aplicación

Por ejemplo, si queremos crear un despliegue a partir de la imagen josedom24/test_web:v1 y queremos hacerlo con un DeploymentConfig, ejecutaremos:

```
oc new-app josedom24/test_web:v1 --name test-web --as-deployment-config=true
```

Si comprobamos nuestro despliegue:

```
oc status
```

Y si vemos los recursos que se han creado:

```
oc get all
```

Podemos observar como se ha ejecutado el Pod pod/test-web-1-deploy responsable de crear los Pods del primer despliegue que hemos realizado con el recursos ReplicationController, controlado por el DeploymentConfig.

Podemos ver la descripción de los recursos creados:

```  
oc describe dc/test-web
oc describe rc/test-web-1
```

Por último, creamos el recurso Route y comprobamos el acceso a la aplicación:

```  
oc expose service/test-web
```

### 4.2 Definición de un recurso DeploymentConfig

Podemos ver la definición del recurso DeploymentConfig que hemos creado, ejecutando:

```
oc get dc/test-web -o yaml
```

Si eliminamos algunas líneas que no son importantes, nos queda una definición similar a esta:

```
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  labels:
    app: test-web
    app.kubernetes.io/component: test-web
    app.kubernetes.io/instance: test-web
  name: test-web
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    deploymentconfig: test-web
  strategy:
    type: Rolling
  template:
    metadata:
      labels:
        deploymentconfig: test-web
    spec:
      containers:
      - image: josedom24/test_web@sha256:99db6f7fdcd6aa338d80b5cd926dff8bae50062c49f82c79a3d67d048efb13a4
        imagePullPolicy: IfNotPresent
        name: test-web
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 8443
          protocol: TCP
        resources: {}
  triggers:
  - type: ConfigChange
  - imageChangeParams:
      automatic: true
      containerNames:
      - test-web
      from:
        kind: ImageStreamTag
        name: test-web:v1
        namespace: test-web
      lastTriggeredImage: josedom24/test_web@sha256:99db6f7fdcd6aa338d80b5cd926dff8bae50062c49f82c79a3d67d048efb13a4
    type: ImageChange
```

Podemos indicar algunos detalles importantes:

* La sección metadata define el nombre y las etiquetas para el objeto DeploymentConfig.
* La sección spec define el recurso ReplicationController que se creará, donde se incluye el número deseado de réplicas (replicas)(en este caso, 1), el límite de historial de revisiones (revisionHistoryLimit) y las etiquetas que seleccionan los Pods que se controlan (selector).
* La sección template describe la plantilla de Pod que se utilizará para crear nuevas réplicas. Como hemos creado el recurso DeploymentConfig con la instrucción oc new-app se ha creado un recurso ImageStream que es el que se utilizará para indicar la imagen desde la que se creará el contenedor.
*  La sección triggers define las condiciones bajo las cuales se desencadenará una nueva implementación. En este caso, se definen dos disparadores.
 - El primer disparador es de tipo ConfigChange, que provocará que se desencadene una nueva implementación si cambian algunos de los parámetros de configuración del DeploymentConfig.
 - El segundo disparador es de tipo ImageChange, que provocará que se desencadene una nueva implementación si cambia la imagen utilizada por el contenedor. El campo from especifica la fuente de la nueva imagen, en este caso un ImageStreamTag llamado test-web:v1. El campo automatic especifica si el cambio de imagen debe detectarse automáticamente o no.

#### 4.2.1 Escalado de los Deployments**

Como ocurría con los Deployments, los DeploymentConfig también se pueden escalar, aumentando o disminuyendo el número de Pods asociados. Al escalar un DeploymentConfig estamos escalando el ReplicationController asociado en ese momento:

```
oc scale dc/test-web --replicas=4
```

#### 4.2.2 Otras operaciones

Si queremos ver los logs generados en los Pods de un DeploymentConfig:

```  
oc logs dc/test-web
``` 

Si queremos obtener información detallada del recurso DeploymentConfig que hemos creado:

```  
oc describe dc/test-web
``` 

Si eliminamos el DeploymentConfig se eliminarán el ReplicationController asociado y los Pods que se estaban gestionando.

``` 
oc delete dc/test-web
```

### 4.3 Actualización de un DeploymentConfig (rollout)

Para comprobar los distintos motivos de actualización, vamos a lanzar un nuevo despliegue con un DeploymentConfig, para ello ejecutamos:

```
oc new-app httpd~https://github.com/josedom24/osv4_html --name=web1 --as-deployment-config=true
```

Se ha creado un nuevo BuildConfig que ha construido la nueva imagen usando Source-to-Image, y al finalizar se ha lanzado la aplicación, como veíamos en el apartado anterior:

```
oc get dc
``` 

Creamos el objeto Route y accedemos a la página:

```
oc expose service web1
```

#### 4.3.1 Actualización manual del despliegue

La primera actualización la vamos a hacer de forma manual, y simplemente vamos a hacer que se actualice el despliegue a partir de la última revisión (no va a existir ningún cambio), para ello:

```  
oc rollout latest dc/web1

oc get dc,rc,pod
```

Vemos que ya tenemos dos revisiones (dos actualizaciones), por lo tanto, tenemos dos objetos ReplicationController, dos Pod Deploy y el Pod de la aplicación ha cambiado y tiene un 2 indicando que corresponde a la segunda revisión.

Para ver el estado y el historial de revisiones ejecutamos:

``` 
oc rollout status dc/web1

oc rollout history dc/web1
```

Si queremos detalles de un revisión en concreto, ejecutamos:

```
oc rollout history dc/web1 --revision=2
``` 

#### 4.3.2 Actualización por ConfigChange

En este apartado, vamos a realizar una actualización del despliegue, cambiando la configuración, por ejemplo:

```  
oc edit dc/web1
``` 

Y cambiamos el parámetro terminationGracePeriodSeconds (especifica el tiempo que se le dará a un contenedor para finalizar sus tareas y cerrar las conexiones abiertas). Una vez realizado el cambio, el despliegue se empieza a actualizar:

```  
oc get dc,rc,pod

oc rollout history dc/web1
```

#### 4.3.3 Actualización por ImageChange

Por último, vamos a hacer un cambio en la aplicación, voy a generar una nueva imagen, y el despliegue se actualizará por el cambio de imagen. Para ello, cambiamos el fichero index.html del repositorio git. Una vez realizado el cambio guardo los cambios en el repositorio y genero de nuevo la imagen:

```
git commit -am "Cambio index.html"
git push
``` 

``` 
oc start-build web1
``` 

Una vez finalizada la construcción observamos que realmente se actualizado el despliegue:

```  
oc get dc,rc,pod

oc rollout history dc/web1
``` 

Podemos acceder a la página para comprobar que se ha modificado.

### 4.4 Rollback de un DeploymentConfig

Tenemos la posibilidad de volver al estado del despliegue que teníamos en una revisión anterior (Rollback). Para realizar este ejercicio vamos a desplegar una nueva aplicación a partir de una imagen que tenemos en Docker Hub. La imagen josedom24/test_web tiene tres versiones, identificadas por etiquetas.

En este ejercicio, vamos a crear un ImageStream antes de desplegar la aplicación, que nos permitirá ir cambiando de imagen con el uso de un tag, para ello:

```
oc create is is_test_web
oc import-image is_test_web:v1 --from=docker.io/josedom24/test_web:v1
oc import-image is_test_web:v2 --from=docker.io/josedom24/test_web:v2
oc import-image is_test_web:v3 --from=docker.io/josedom24/test_web:v3
```

Vamos a crear una nueva etiqueta prod que en primer lugar apuntará a la versión v1 y que utilizaremos para el despliegue, posteriormente iremos referenciando con esta etiqueta otras versiones de la imagen, por lo que se producirá el trigger ImageChange:

```
oc tag is_test_web:v1 is_test_web:prod
```

#### 4.4.1 Creación del DeploymentConfig

Creamos el DeploymentConfig con la siguiente instrucción:

```
oc new-app is_test_web:prod --name test-web --as-deployment-config=true
oc expose service test-web
```

```
oc get dc,rc,pod

oc rollout history dc/test-web
```

#### 4.4.2 Actualización del despliegue a la versión 2

Para producir la actualización vamos a cambiar la imagen, para ello vamos a referenciar el tag prod a la segunda versión:

```
oc tag is_test_web:prod -d
oc tag is_test_web:v2 is_test_web:prod
```

Inmediatamente se produce la actualización del despliegue:

```
oc get dc,rc,pod

oc rollout history dc/test-web
```

#### 4.4.3 Rollback del despliegue

Volvemos a realizar el cambio de imagen para desplegar la tercera versión:

```
oc tag is_test_web:prod -d
oc tag is_test_web:v3 is_test_web:prod
```

Se produce la actualización:

```  
oc get dc,rc,pod

oc rollout history dc/test-web
```

Pero ahora, al acceder a la aplicación nos damos cuenta de que tiene un error.

Para volver a la versión anterior podemos ejecutar:

```
oc rollout undo dc/test-web
```

Y comprobamos que hemos vuelto a la versión anterior

Realmente podemos volver a la revisión que queramos:

```
oc rollout undo dc/test-web --to-revision=1

oc rollout history dc/test-web
```

### 4.5 Estrategias de despliegues

Vamos a estudiar dos tipos de estrategias:

#### 4.5.1 Estrategia Rolling Update

En este tipo de estrategia utiliza una implementación gradual y en cascada para actualizar los Pods: se van creando los nuevos Pods, se comprueba que funcionan, y posteriormente se van eliminando los Pods antiguos. Es la estrategia por defecto. Veamos la configuración de esta estrategia, creando un DeploymentConfig a partir de la ImageStream is-example que tiene otras dos etiquetas apuntando a las distintas versiones:

```
oc create is is_example
oc import-image is_example:v1 --from=quay.io/openshifttest/deployment-example:v1
oc import-image is_example:v2 --from=quay.io/openshifttest/deployment-example:v2
oc tag is_example:v1 is_example:latest
```

```
oc new-app is_example:latest --as-deployment-config=true --name=example
```

Ahora nos fijamos en la configuración de la estrategia en la definición del objeto:

```
oc get dc example -o yaml
strategy:
    activeDeadlineSeconds: 21600
    resources: {}
    rollingParams:
      intervalSeconds: 1
      maxSurge: 25%
      maxUnavailable: 25%
      timeoutSeconds: 600
      updatePeriodSeconds: 1
    type: Rolling
```

* intervalSeconds: El tiempo de espera entre la comprobación del estado de despliegue después de la actualización.
* timeoutSeconds: El tiempo de espera máximo para cada actualización de Pod.
* updatePeriodSeconds: El tiempo de espera entre actualizaciones de Pods individuales.Por defecto, un segundo.
* maxSurge: El número máximo de Pods nuevos que se pueden crear por encima del número deseado de replicas.
* maxUnavailable: El número máximo de Pods que se pueden eliminar simultáneamente.
  
Vamos a escalar el despliegue y a comprobar el acceso a la aplicación:

```
oc scale dc/example --replicas=5
oc expose service/example
```

A continuación vamos a hacer la actualización del despliegue, para verlo bien puedes poner en una terminal la siguiente instrucción:

```
watch oc get pod
``` 
Y en otra terminal actualizamos la etiqueta latest del ImageStream Para provocar el nuevo despliegue:

```
oc tag -d is_example:latest
oc tag is_example:v2 is_example:latest
```

Finalmente podemos comprobar que tenemos desplegada la versión 2.

#### 4.5.2 Estrategia Recreate

En algunas circunstancias, podemos necesitar eliminar todos los Pods antiguos y posteriormente crear los nuevos. Este tipo de estrategia se denomina Recreate.

Vamos a modificar el tipo de estrategia en nuestro DeploymentConfig:

```
oc edit dc example
```

Y modificamos la sección strategy y la dejamos así:

```
strategy:
    type: Recreate
```

Ahora vamos a volver a actualizar el despliegue a la versión 1 (recuerda tener en una terminal ejecutando watch oc get pod para ver como se eliminan todos los Pods antes de crear los nuevos):

```
oc tag -d is_example:latest
oc tag is_example:v1 is_example:latest 
```

### 4.6 Estrategias de despliegues basadas en rutas

En este tipo de estrategia de despliegue configuramos el objeto Route para que enrute el tráfico a distintos Pods de distintos servicios.

Con esta funcionalidad podemos implementar una estrategia de despliegue Blue/Green, podemos ofrecer dos versiones de la aplicación: la nueva (la "verde") se pone a prueba y se evalúa, mientras los usuarios siguen usando la versión actual (la "azul"). El cambio entre versiones se va haciendo gradualmente. Si hay algún problema con la nueva versión, es muy fácil volver a la antigua versión.

#### 4.6.1 Modificación del objeto Route para servir otra aplicación

Este tipo de estrategia trabaja con dos objetos Deployment, pero se crea un sólo objeto Route que en un primer momento está conectado al Service del primer despliegue (versión actual de la aplicación). Vamos a crear dos despliegues y vamos a crear la ruta que apunta al primero de ellos:

```
oc new-app josedom24/citas-backend:v1 --name=app-blue
oc new-app josedom24/citas-backend:v2 --name=app-green
```

```
oc expose svc/app-blue --port=10000 --name=app
```

En otro terminal, podemos comprobar la versión de la aplicación a la que estamos accediendo, ejecutando:

```
while true; do curl http://app-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/version; done
```

Si queremos que la ruta nos sirva la nueva aplicación, simplemente tenemos que modificar el Service que tiene configurado.

```
oc patch route/app -p '{"spec":{"to":{"name":"app-green"}}}'
```

Vamos a servir la primera versión para continuar con el ejemplo:

``` 
oc patch route/app -p '{"spec":{"to":{"name":"app-blue"}}}'
```

#### 4.6.2 Despliegue Blue/Green basado en rutas

Ahora podemos configurar la ruta, para que vaya enrutando a los dos Services correspondientes a los dos Deployment. A cada servicio se le asigna un peso y la proporción de peticiones a cada servicio es el peso asignado dividido por la suma de los pesos.

Por ejemplo si queremos que el 75% de las peticiones siga sirviendo la versión actual y el 25% la nueva versión:

```
oc set route-backends app app-blue=75 app-green=25
```

Obtenemos información de la ruta y lo comprobamos:

``` 
oc get route app
```

Para que sirvan las dos versiones al 50%, podemos usar cualquiera de esta dos posibilidades:

```
oc set route-backends app app-blue=50 app-green=50
oc set route-backends app --equal
```

Y finalmente si sólo queremos servir la nueva versión:

```
oc set route-backends app app-blue=0 app-green=100
```

Los pesos se indican con un número de 0 a 256. Podemos indicar el peso para un servicio y hacer que el otro se ajuste de forma automática:

```
oc set route-backends app app-blue=30 --adjust 
oc get route app
```

También podemos indicar los pesos como porcentajes:

```
oc set route-backends app app-blue=25% --adjust 

oc get route app
```  

Los pesos también se pueden modificar de manera muy sencilla desde la consola web editando la definición del objeto Route.


## 5. Plantillas: empaquetando los objetos en OpenShift

### 5.1 Introducción a los Templates

También podemos obtener la lista de templates que se encuentran en el proyecto openshift, ejecutando la siguiente instrucción:

``` 
oc get templates -n openshift
```

Si queremos desplegar la aplicación ejemplo nodejs podemos usar la plantilla nodejs-postgresql-example. Para ver los parámetros que podemos configurar, ejecutamos:

``` 
oc process --parameters nodejs-postgresql-example -n openshift
```

Sólo vamos a definir el parámetro NAME para indicar el nombre de la aplicación durante la creación. Para ello, ejecutamos

``` 
oc new-app nodejs-postgresql-example -p NAME=app-nodejs
```  

Comprobamos los recursos que ha creado la plantilla:

``` 
oc get all -o name
pod/app-nodejs-1-build
pod/app-nodejs-1-deploy
pod/app-nodejs-1-lsdhb
pod/postgresql-1-deploy
pod/postgresql-1-x8kn6
replicationcontroller/app-nodejs-1
replicationcontroller/postgresql-1
service/app-nodejs
service/modelmesh-serving
service/postgresql
deploymentconfig.apps.openshift.io/app-nodejs
deploymentconfig.apps.openshift.io/postgresql
buildconfig.build.openshift.io/app-nodejs
build.build.openshift.io/app-nodejs-1
imagestream.image.openshift.io/app-nodejs
route.route.openshift.io/app-nodejs
``` 

Esperamos a que la imagen se construya, y accedemos a la aplicación.

### 5.2 Descripción de un objeto Template

Vamos a crear un objeto Template desde su definición en un fichero YAML. En este ejemplo vamos a hacer un Template muy sencillo, que nos va a permitir crear un recurso Deployment usando la imagen bitnami/mysql y como veremos posteriormente hemos creado varios parámetros para permitir su configuración. Partimos del fichero mysql-plantilla.yaml con el siguiente contenido:

```  
apiVersion: template.openshift.io/v1
kind: Template
labels:
  app: deployment-mysql
metadata:
  name: mysql-plantilla
  annotations:
    description: "Plantilla para desplegar un deployment de mysql"
    iconClass: "icon-mysql-database"
    tags: "database,mysql"
objects:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ${NOMBRE_APP}
  spec:
    replicas: ${{REPLICAS}}
    selector:
      matchLabels:
        app: mysql
    template:
      metadata:
        labels:
          app: mysql
      spec:
        containers:
          - name: ${NOMBRE_CONTENEDOR}
            image: bitnami/mysql  
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: ${ROOT_PASSWORD}
            - name: MYSQL_USER
              value: ${USER}
            - name: MYSQL_PASSWORD
              value: ${PASSWORD}
            - name: MYSQL_DATABASE
              value: ${DATABASE}
            ports:
            - containerPort: 6379
              protocol: TCP
parameters:
- name: REPLICAS
  description: Número de pods creados
  value: "1"
- name: NOMBRE_APP
  description: Nombre del pod que se va a crear
  from: 'mysql[0-9]{2}'
  generate: expression
- name: NOMBRE_CONTENEDOR
  description: Nombre del contenedor que se va a crear
  value: contenedor-mysql
- name: ROOT_PASSWORD
  description: Contraseña del root de mysql
  from: '[A-Z0-9]{8}'
  generate: expression
- name: USER
  description: Nombre del usuario mysql que se va acrear
  value: usuario
- name: PASSWORD
  description: Contraseña del usuario de mysql
  from: '[A-Z0-9]{8}'
  generate: expression
- name: DATABASE
  description: Nombre de la base de datos que se va a crear
  value: nueva_bd
```

Veamos cada uno de los apartados que tiene la configuración:

* labels: Indicamos las etiquetas que tendrán todos los objetos que vamos a crear.
* metadata: Tiene las secciones que normalmente tiene esta sección en la definición de cualquier objeto. En este caso nos vamos a fijar en las anotaciones (annotations):
 - description: Definimos una descripción de lo que hace la plantilla.
 - iconClass: Indicamos el icono que se mostrará en el catálogo de aplicaciones. Más iconos.
 - tags: Etiquetas asignadas a la plantilla, que facilitan su búsqueda en el catálogo.
Hay más posibles anotaciones que puedes estudiar en la documentación.
* objects: Definimos los objetos que se van a crear al instanciar la plantilla.
* parameters: Definimos los parámetros que hemos indicado en la definición de los objetos. A la hora de instanciar la plantilla estos parámetros se pueden sobreescribir. Algunos de los atributos que podemos poner de cada parámetro:
 - name: Nombre del parámetro.
 - description: Descripción del parámetro.
 - value: Valor por defecto. Si no indico el parámetro al crear los objetos del Template tomará el valor por defecto.
 - from: Expresión regular que se usa para generar un valor aleatorio, si no indicamos el valor del parámetro. Va acompañado del atributo generate: expression.
  
Como vemos los parámetros se pueden indicar de dos formas:

* ${NOMBRE_PARÁMETRO}: El valor se proporciona como una cadena de caracteres. Normalmente usamos esta forma.
* ${{NOMBRE_PARÁMETRO}}: El valor se puede proporcionar como un valor que no sea una cadena de caracteres. Lo hemos usado para indicar el número de replicas, que en la definición tiene que ser un número entero (no se entrecomilla).
  
Por último para crear el objeto Template a partir de su definición, ejecutamos:

```
oc apply -f mysql-plantilla.yaml
``` 

Y podemos ver que realmente la hemos creado, desde la línea de comandos:

```
oc get templates
```

### 5.3 Crear objetos desde un Template

Si queremos ver los parámetros que podemos configurar en la plantilla, ejecutamos:

```
oc process --parameters mysql-plantilla
``` 

Podemos generar la definición de los objetos que crea un Template. La generación de la definición de los objetos no implica su creación. Veamos algunos ejemplos:

Genero la definición en formato YAML de los objetos sin indicar ningún parámetro (se cogen los valores por defecto):

```
 oc process mysql-plantilla -o yaml
```

Podemos indicar algunos parámetros en la generación de la definición. Los parámetros que no indiquemos cogerán sus valores por defecto:

```  
 oc process mysql-plantilla -o yaml -p REPLICAS=2 -p NOMBRE_APP=mimysql -p ROOT_PASSWORD=asdasd1234
```

Si tenemos muchos parámetros podemos guardar los parámetros en un fichero, por ejemplo en parametros-mysql.txt:

``` 
 REPLICAS=3
 NOMBRE_APP=otra_mysql
 NOMBRE_CONTENEDOR=contenedor1
 ROOT_PASSWORD=asdasd1234
 USER=usuario
 PASSWORD=mypassword
 DATABASE=mi_bd
```  

Podemos usar este fichero para indicar los parámetros de la siguiente forma:

``` 
 oc process mysql-plantilla -o yaml --param-file=parametros-mysql.txt
``` 

Además, de los parámetros, al generar la definición de los objetos podemos indicar las labels que tendrán todos los objetos generados:

```
 oc process mysql-plantilla -o yaml -l type=database
```

Evidentemente, podemos definir parámetros y etiquetas:

```  
 oc process mysql-plantilla -o yaml -p REPLICAS=2 -l type=database
```

Una vez que sabemos como generar la definición de los objetos que están definido en una plantilla, tenemos varias formas para crearlos:

1. Generar la definición de los objetos de la plantilla (si es necesario indicado parámetros y etiquetas) y ejecutando oc apply sobre la definición generada, por ejemplo:

```  
 oc process mysql-plantilla -o yaml -p REPLICAS=2 -p NOMBRE_APP=mimysql | oc apply -f -
```

2. Guardar la generación de la definición de los objetos en un fichero YAML, y utilizar este fichero para crear los objetos. Esta opción tiene dos ventajas: que podemos reproducir el despliegue y que podemos eliminar los objetos utilizando el fichero YAML. Por ejemplo:

```  
 oc process mysql-plantilla -o yaml -p NOMBRE_APP=app1 -p REPLICAS=3 -l type=database > deploy_mysql.yaml        
 oc apply -f deploy_mysql.yaml
```

3. Utilizando el comando oc new-app:

```
 oc new-app mysql-plantilla -p NOMBRE_APP=new-mysql
```

#### 5.3.1 Prueba de funcionamiento

Creamos un Deployment a partir del Template. Para ello, indicamos sólo dos parámetros:

```
oc process mysql-plantilla -p NOMBRE_APP=mysql -p PASSWORD=asdasd | oc apply -f -
```

A continuación intentamos acceder a la base de datos, ejecutando:

```
oc exec -it deploy/mysql -- mysql -u usuario -pasdasd nueva_bd -h localhost
...
mysql>
```

### 5.4 Creación de plantillas a partir de objetos existentes

#### 5.4.1 Crear Templates a partir de Templates existentes

Esta operación es muy sencilla, y simplemente consiste en copiar la definición YAML de un Template en un fichero y posteriormente hacer las modificaciones que necesitemos. Por ejemplo:

```
oc get template mysql-plantilla -o yaml > nueva_plantilla.yaml
```

Otro ejemplo que nos permite hacer una copia de una plantilla del catálogo de aplicaciones de de OpenShift:

```
oc get template mariadb-ephemeral -n openshift -o yaml > otra-plantilla.yaml
```

#### 5.4.2 Crear Templates a partir de objetos existentes

Vamos a imaginar que hemos desplegado una aplicación PHP que tenemos guardada en un repositorio. Para ello hemos ejecutado el comando:

```
oc new-app php~https://github.com/josedom24/osv4_php --name=app-php --as-deployment-config=true
oc expose service app-php
```

Como ya sabemos estas dos instrucciones han creado varios tipos de objetos: DeploymentConfig, BuildConfig, ImageStream, Service y Route.

Ahora queremos diseñar un Template que nos permita desplegar aplicaciones PHP que estén guardadas en repositorios GitHub. A partir de la definición de los objetos que hemos creado, podemos crear la definición de un Tempalate, estableciendo los parámetros que queramos configurar posteriormente. Para ello podemos ejecutar la instrucción:

```
oc get -o yaml all > php-plantilla.yaml
```

En el fichero php-plantilla.yaml tendremos la lista de las definiciones de los objetos, en el próximo apartado veremos cómo lo convertimos en la definición de un Template.


### 5.5 Uso de Helm en OpenShift desde la línea de comandos

#### 5.5.1 Instalación de Helm chart

Si accedemos a la consola web, en la parte superior derecha lel botón ? y la opción Command line tools, nos aparece la página web de descarga del Helm CLI: Download Helm.

Podemos descargarnos la versión que necesitemos y copiar el binario en un directorio del PATH, por ejemplo en Linux:

```
wget -O helm https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/helm/latest/helm-linux-amd64 
sudo install helm /usr/local/bin

helm version
```  

Desde el cliente helm podemos gestionar el ciclo de vida de las aplicaciones instaladas en nuestro clúster:

```
helm ls
```

Los repositorios de charts son independientes, es decir, no están sincronizados. Por ejemplo, cuando acabamos de instalar helm no tiene repositorios instalados:

```
helm repo list
Error: no repositories to show
```

Para añadir el repositorio de charts de OpenShift y actualizarlo, ejecutamos:

```
helm repo add openshift https://charts.openshift.io/

helm repo update
```

Y ahora podemos buscar los charts ejecutando:

```
helm search repo jenkins
```

Y para buscar información acerca de ese chart:

```
helm show all openshift/redhat-jenkins
```

#### 5.5.2 Instalación de un chart helm desde la línea de comandos

Podemos buscar más repositorios de charts explorando la página Artifact Hub, por ejemplo podemos añadir el repositorio de charts de Bitnami de la siguiente manera:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

Y podemos comprobar que hemos añadido un nuevo repositorio:

```
helm repo list
```

Actualizamos la lista de charts ofrecidos por los repositorios:

```
helm repo update
```

Como hemos comentado anteriormente, los charts los podemos buscar en la página Artifact Hub o los podemos buscar desde la línea de comandos, por ejemplo si queremos buscar un chart relacionado con nginx:

```
helm search repo nginx
```

¿Y cómo sabemos los parámetros que tiene definido cada chart y sus valores por defecto?. Estudiando la documentación del chart en Artifact Hub. En concreto para el chart con el que estamos trabajando, accediendo a su página de de documentación. También podemos obtener esta información ejecutando el siguiente comando:

```
helm show all bitnami/nginx
```

Vamos a desplegar este chart modificando el parámetro service.type como ClusterIP ya que luego crearemos un recurso Route para acceder a él.

```
helm install web bitnami/nginx --set service.type=ClusterIP 
```

Podemos listar los charts que tenemos instalados:

```
helm ls
```

Los nombres de los recursos que se han creado son del tipo `-, por ejemplo:

```
oc get all -o name
service/web-nginx
deployment.apps/web-nginx
...
```

Podemos crear la ruta de acceso:

```  
oc expose service/web-nginx
```

Y acceder para comprobar que funciona. Finalmente podemos borrar la aplicación desplegada con:

```
helm uninstall web
```

## 6. Almacenamiento en OpenShift v4

### 6.1 Ejemplo 1: Gestión de almacenamiento desde la consola web: phpsqlitecms 

En este ejemplo, vamos a instalar un CMS PHP llamado phpSQLiteCMS que utiliza una base de datos SQLite. Para ello vamos a utilizar el código de la aplicación que se encuentra en el repositorio: https://github.com/ilosuna/phpsqlitecms.

Vamos a realizar el despliegue desde la línea de comandos:

```
oc new-app php:7.3-ubi7~https://github.com/ilosuna/phpsqlitecms --name=phpsqlitecms
oc expose service/phpsqlitecms
```

Se han creado los recursos y podemos acceder a la aplicación.

#### 6.1.1 Modificación de la aplicación

A continuación, vamos a entrar en la zona de administración, en la URL /cms, y con el usuario y contraseña: admin - admin vamos a realizar un cambio (por ejemplo el nombre de la página) que se guardará en la base de datos SQLite.

#### 6.1.2 Volúmenes persistentes

Necesitamos un volumen para guardar los datos de la base de datos. Vamos a crear un volumen y lo vamos a montar en le directorio /opt/app-root/src/cms/data, que es donde se encuentra la base de datos. Para ello vamos a crear un objeto PersistentVolumenClaim que nos permitirá crear un PersistentVolumen que asociaremos al Deployment. Lo vamos a hacer desde la consola web, desde la vista Administrator, escogemos la opción Storage -> PersistentVolumenClaims y creamos un nuevo objeto.

A continuación, añadimos almacenamiento al despliegue, indicando el objeto PersistentVolumenClaim que hemos creado, y el directorio donde vamos a montar el volumen. Se ha actualizado el despliegue, se ha creado un nuevo Pod con la nueva versión (el volumen montado en el directorio) y podemos comprobar que el PersistentVolumenClaim se ha asociado con un PersistentVolumen.

La aplicación no está funcionando bien. ¿Qué ha pasado?. Al montar el volumen en el directorio /opt/app-root/src/cms/data, el contenido anterior, correspondiente a los ficheros de la base de datos se ha perdido. Tenemos que copiar en este directorio (en realidad en el volumen) los ficheros necesarios, para ello vamos a copiarlos desde el repositorio:

```
git clone https://github.com/ilosuna/phpsqlitecms
cd phpsqlitecms/cms

oc get pod
``` 

```
oc cp data phpsqlitecms-687b8ff8cd-sqcdm:/opt/app-root/src/cms
```

Y volvemos a comprobar si está funcionando la aplicación.

#### 6.1.3 Estrategias de despliegue y almacenamiento

¿Qué ocurrirá si volvemos actualizar el despliegue, creando un nuevo Pod? Lo vamos a realizar desde el entorno web seleccionando la acción Restart rollout. Se crea un nuevo Pod, pero no termina de estar en estado de ejecución.

Si vemos los eventos del pod, nos aclara el problema que ha existido. El problema es el siguiente:

* El volumen que se ha creado no permite que dos Pods estén conectados simultáneamente a él Esto es debido al tipo de volumen, en nuestro caso: AWS Elastic Block Store (EBS).
* La estrategia de despliegue **RolligUpdate** crea el nuevo Pod, comprueba que funciona para posteriormente eliminar el viejo. Pero en este caso, no puede terminar de crear el nuevo Pod, porque no se puede conectar al volumen mientras el antiguo Pod este conectado a él.
  
La solución es configurar la estrategia de despliegue a Recreate, al eliminar el Pod antiguo, el Pod nuevo se puede conectar al volumen sin problemas. Para ello:

```
oc edit deploy/phpsqlitecms
...
spec:
...
  strategy:
    type: Recreate
```

Y volvemos realizar la actualización del despliegue:

```  
oc rollout restart deploy/phpsqlitecms

oc get pod
```

Como vemos se ha creado un nuevo Pod sin problemas.

#### 6.1.4 Escalado y almacenamiento

Como hemos indicado el almacenamiento ofrecido por Red Hat OpenShift Dedicated Developer Sandbox no permite que varios Pods estén simultáneamente conectado a un mismo volumen, no proporciona almacenamiento compartido.

De la misma manera, el nuevo Pod que se está creando no termina de crearse por que no se puede conectar al volumen, que ya está conectado al primer Pod. Por lo tanto concluimos, que con este tipo de almacenamiento, no podemos escalar los despliegues.

### 6.2 Ejemplo 2: Gestión de almacenamiento desde la línea de comandos: GuestBook

En esta tarea vamos a desplegar una aplicación web que requiere de dos servicios para su ejecución. La aplicación se llama GuestBook y necesita los siguientes servicios:

* La aplicación GuestBook es una aplicación web desarrollada en python que tenemos guardada en el repositorio https://github.com/josedom24/osv4_guestbook.git.
* Esta aplicación guarda la información en una base de datos no relacional redis, que utiliza el puerto 6379/tcp para recibir las conexiones. Usaremos la imagen bitnami/redis.
  
Por lo tanto para desplegar las aplicaciones vamos a ejecutar los siguientes comandos:

```
oc new-app https://github.com/josedom24/osv4_guestbook.git --name=guestbook
oc new-app bitnami/redis  -e REDIS_PASSWORD=mypass --name=redis
```

A continuación, creamos la ruta para acceder a GuestBook y comprobamos que funciona:

```
oc expose service/guestbook
```

#### 6.2.1 Persistencia de la información

En primer lugar vamos a crear un objeto PersistentVolumeClaim que nos va permitir solicitar la creación de un PersistentVolume, para ello usamos la definición del objeto que tenemos en el fichero pvc-redis.yaml:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: my-pvc-redis
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Y creamos el objeto, ejecutando:

```
oc apply -f pvc-redis.yaml
```

Podemos ver que se ha creado el objeto, pero que no se va asociar a un volumen hasta que no se utilice:

```
oc get pvc
```

A continuación tenemos que modificar el despliegue de la aplicación GuestBook, para cambiar la estrategia de despliegue, asociar el volumen e indicar el directorio de montaje, para ello editamos el Deployment redis y lo dejamos con las siguientes modificaciones:

```
oc edit deploy/redis

spec:
...
  strategy:
    type: Recrete
template:
...
  spec:
    containers:
    ...
      volumeMounts:
        - mountPath: /bitnami/redis/data
          name: my-volumen
      ...
    volumes:
      - name: my-volumen
        persistentVolumeClaim:
          claimName: my-pvc-redis
``` 

Cuando modificamos el Deployment, se produce una actualización: se creará un nuevo ReplicaSet que creará un nuevo Pod con la nueva configuración.

```  
oc get rs

oc get pod
```

Puedes ver las características del Pod, ejecutando:

```
oc describe pod/redis-7f59bf9479-mm76b
```

Accede de nuevo a la aplicación, introduce algunos mensajes, y vamos a simular la eliminación del Pod:

``` 
oc delete pod/redis-7f59bf9479-mm76b
``` 

Inmediatamente se creará un nuevo Pod, volvemos acceder y comprobar que la información no se ha perdido.


### 6.3 Ejemplo 3: Haciendo persistente la aplicación Wordpress

#### 6.3.1 Base de datos persistente

Para obtener una base de datos persistente vamos a crear una aplicación de base de datos a partir de una plantilla que cree y configure un volumen. Por ejemplo, vamos a usar la plantilla mariadb-persistent:

```
oc process --parameters mariadb-persistent -n openshift
```

Y creamos el despliegue, ejecutando:

```
oc new-app mariadb-persistent -p MYSQL_USER=usuario \ 
                              -p MYSQL_PASSWORD=asdasd \
                              -p MYSQL_DATABASE=wordpress \
                              -p MYSQL_ROOT_PASSWORD=asdasd \
                              -p VOLUME_CAPACITY=5Gi
```

Podemos comprobar que se ha creado un objeto PersistentVolumeClaim asociado a un PersistentVolume:

```
oc get pvc
```

Y que efectivamente está montado en un directorio de los Pods:

```
oc describe dc/mariadb
```

#### 6.3.2 Wordpress persistente

Vamos a desplegar Wordpress y posteriormente, crearemos un nuevo volumen para guardar los datos del blog. Este volumen habrá que montarlo en el directorio /bitnami/wordpress.

Para desplegar el Wordpress usando un DeploymentConfig:

```
oc new-app bitnami/wordpress -e WORDPRESS_DATABASE_NAME=wordpress -e  WORDPRESS_DATABASE_HOST=mariadb -e WORDPRESS_DATABASE_USER=usuario -e WORDPRESS_DATABASE_PASSWORD=asdasd --name wordpress --as-deployment-config=true
```

En primer lugar cambiamos la estrategia de despliegue, para evitar el problema que hemos visto en ejemplos anteriores:

```
oc patch dc/wordpress --patch '{"spec":{"strategy":{"type":"Recreate"}}}'
```

A continuación, creamos un PersistentVolumeClaim usando la definición que tenemos en el fichero pvc-wordpress.yaml:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: my-pvc-wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Creamos el PersistentVolumeClaim y añadimos el almacenamiento al despliegue:

```
oc apply -f pvc-wordpress.yaml

oc set volumes dc/wordpress --add -m /bitnami/wordpress --name=vol-wordpress -t pvc --claim-name=my-pvc-wordpress --overwrite
```

Comprobamos los dos volúmenes que hemos creado:

``` 
oc get pvc
``` 

Y que efectivamente hemos asociado un volumen al DeploymentConfig montado en el directorio /bitnami/wordpress:

```
oc describe dc/wordpress
``` 

Finalmente creamos la ruta y accedemos a la aplicación:

```
oc expose service/wordpress
```

Ahora podemos entrar en la zona de administración (en la URL /wp-admin) y usando las credenciales por defecto, usuario y contraseña: user - bitnami, podemos crear una nueva entrada con una imagen.

Finalmente podemos actualizar los dos despliegues:

```
oc rollout latest dc/mariadb
oc rollout latest dc/wordpress
```

Y volvemos acceder a la aplicación para comprobar que no hemos perdido la información.

### 6.4 Instantáneas de volúmenes

Un recurso VolumeSnapshot representa una instantánea de un volumen en un sistema de almacenamiento. Las instantáneas de volumen nos proporciona una forma estandarizada de copiar el contenido de un volumen en un momento determinado sin crear un volumen completamente nuevo.

En la consola web, en la vista de Administrator, en el apartado Storage -> VolumeSnapshotClasses, podemos ver los recursos VolumeSnapshotClasses que están definidos en este clúster.

Para realizar el ejercicio, vamos a desplegar un servidor web nginx asociado a un volumen. La definción de la solicutd del volumen la encontramos en el fichero pvc.yaml con el siguiente contenido:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: my-pvc
spec:
  storageClassName: gp2-csi
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Nos aseguramos con el parámetro storageClassName que se utilicen volúmenes del tipo gp2-sci para que el VolumeSnapshotClasses pueda realizar de manera adecuada los snpashots. Ahora, ejecutamos las siguientes instrucciones:

```
oc apply -f pvc.yaml
oc new-app bitnami/nginx --name nginx
oc expose service/nginx
oc set volumes deploy/nginx --add -m /app --name=my-vol -t pvc --claim-name=my-pvc --overwrite
oc exec deploy/nginx -- bash -c "echo '<h1>Probando los SnapShots</h1>' > /app/index.html"
```

A continuación, vamos a crear una instantánea de ese volumen, para ello entramos en la sección Storage -> VolumeSnapshots y pulsamos sobre el botón Create VolumeSnapshot. Indicamos el recurso PersistentVolumeClaim al que queremos crear la instantánea y el nombre de la misma. Y transcurridos unos segundos, podremos ver la lista de las instantáneas de volúmenes que hemos creado.

A partir de la instantánea podemos crear un nuevo volumen con la misma información, para ello escogemos la opción, indicando las propiedades del recurso PersistentVolumeClaim. 

Volvemos a crear un nuevo Deployment:

```
oc new-app bitnami/nginx --name nginx2
oc expose service/nginx2
oc set volumes deploy/nginx2 --add -m /app --name=my-vol -t pvc --claim-name=my-pvc2 --overwrite
```

Y comprobamos que podemos acceder al fichero index.html, que en esta ocasión no hemos tenido que crear porque se ha restaurado desde la instantánea de volumen.


## 7. OpenShift Pipelines

### 7.1 Despliegue de una aplicación con OpenShift Pipeline

#### 7.1.1 Tekton CLI

Vamos a usar una herramienta de línea de comandos llamada tkn, puedes seguir las siguientes instrucciones para su instalación.

#### 7.1.2 Aplicación de ejemplo

Vamos a desplegar una aplicación muy sencilla de votaciones:

* frontend: Aplicación construida con Python Flask que nos permite votar.
* backend: Aplicación escrita en Go que nos permite guardar las votaciones.
En el directorio k8s del repositorio se encuentran los ficheros YAML que posibilitan el despliegue de la aplicación.

#### 7.1.3 Instalación de Tasks

Las Tasks consisten en una serie de pasos que se ejecutan de forma secuencial. Las tareas se ejecutan mediante la creación de TaskRuns. Un TaskRun creará un Pod y cada paso se ejecuta en un contenedor independiente dentro del mismo Pod. Podemos definir entradas y salidas para interactuar con otras tareas en el pipeline.

Vamos a instalar dos tareas:

- apply-manifests: responsable de ejecutar los ficheros YAML que se encuentran en el directoriok8s y por lo tanto aplicar los posibles cambios en los recurso que estamos creando.
- update-deployment: responsable de actualizar el objeto Deployment, en concreto cambiar el nombre y la imagen.
- 
Todos los ficheros que vamos a usar están en el repositorio pipelines-tutorial.

Para ello tenemos el fichero 01_apply_manifest_task.yaml con el siguiente contenido y el fichero 02_update_deployment_task.yaml con este contenido.

Creamos las tareas ejecutando:

```
oc create -f https://raw.githubusercontent.com/josedom24/pipelines-tutorial/master/01_pipeline/01_apply_manifest_task.yaml
oc create -f https://raw.githubusercontent.com/josedom24/pipelines-tutorial/master/01_pipeline/02_update_deployment_task.yaml
```

Puedes ver la lista de tareas creadas, ejecutando una de estas dos instrucciones:

```
oc get tasks
tkn task ls
```

Hay tareas predefinidas en el clúster de OpenShift, para obtener la lista podemos ejecutar una de estas dos instrucciones:

```
oc get clustertasks
tkn clustertasks ls
```

#### 7.1.4 Creando el pipeline

En este ejemplo, vamos a crear un pipeline que toma el código fuente de la aplicación de GitHub y luego lo construye y despliega en OpenShift.

El fichero 04_pipeline.yaml tiene la definición YAML del pipeline:

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  workspaces:
  - name: shared-workspace
  params:
  - name: deployment-name
    type: string
    description: name of the deployment to be patched
  - name: git-url
    type: string
    description: url of the git repo for the code of deployment
  - name: git-revision
    type: string
    description: revision to be used from repo of the code for deployment
    default: master
  - name: IMAGE
    type: string
    description: image to be build from the code
  tasks:
  - name: fetch-repository
    taskRef:
      name: git-clone
      kind: ClusterTask
    workspaces:
    - name: output
      workspace: shared-workspace
    params:
    - name: url
      value: $(params.git-url)
    - name: subdirectory
      value: ""
    - name: deleteExisting
      value: "true"
    - name: revision
      value: $(params.git-revision)
  - name: build-image
    taskRef:
      name: buildah
      kind: ClusterTask
    params:
    - name: IMAGE
      value: $(params.IMAGE)
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - fetch-repository
  - name: apply-manifests
    taskRef:
      name: apply-manifests
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - build-image
  - name: update-deployment
    taskRef:
      name: update-deployment
    params:
    - name: deployment
      value: $(params.deployment-name)
    - name: IMAGE
      value: $(params.IMAGE)
    runAfter:
    - apply-manifests
```

Veamos las diferentes tareas que se ejecutan en el pipeline:

* Clona el código fuente de la aplicación desde un repositorio git (git-url y git-revision param) usando el ClusterTask git-clone.
* Construye la imagen del contenedor de la aplicación utilizando la ClusterTask buildah que utiliza Buildah para construir la imagen.
* La imagen de la aplicación se envía a un registro de imágenes (parámetro image).
* La nueva imagen de la aplicación se despliega en OpenShift utilizando las tareas apply-manifests y update-deployment.
  
Es posible que haya notado que no hay referencias al repositorio git o al registro de imágenes que se utiliza en el pipeline. Esto se debe a que los pipelines en Tekton están diseñados para ser genéricos y reutilizables. Al activar un pipeline, puedes proporcionar diferentes repositorios git y registros de imágenes para ser utilizados durante la ejecución del pipeline.

En concreto los parámetros que hay que proporcionar son:

* deployment-name: Nombre del despliegue que tiene que coincidir con el que se ha indicado como nombre del Deployment en el fichero YAML correspondiente en el repositorio k8s.

* git-url: URL del repositorio GitHub que queremos desplegar.

* git-revision: Rama del repositorio GitHub, por defecto es master.

* IMAGE: Nombre de la imagen que vamos a construir. En nuestro caso indicaremos el registro interno de OpenShift. Recuerda que el registro interno tiene la siguiente URL, donde hay que indicar el nombre del proyecto, en mi caso:

``` 
  `image-registry.openshift-image-registry.svc:5000/josedom24-dev`
```

El orden de ejecución de las tareas está determinado por las dependencias que se definen entre las tareas a través de las entradas y salidas, así como las órdenes explícitas que se definen a través de runAfter.

El campo Workspaces permite especificar uno o más volúmenes que cada Task en el Pipeline requiere durante la ejecución para intercambiar información.

Para crear el pipeline, ejecutamos:

```  
oc create -f https://raw.githubusercontent.com/josedom24/pipelines-tutorial/master/01_pipeline/04_pipeline.yaml
```  

Y puedes ver los objetos Pipelines que has creado, ejecutando una de estas dos instrucciones:

```
oc get pipelines
tkn pipelines ls
``` 

Si accedemos a la sección Pipeline de la consola web podemos ver la lista de Pipelines. Y si pulsamos sobre el Pipeline obtenemos información detallada del mismo.

### 7.2 Gestión de OpenShift Pipeline desde el terminal

#### 7.2.1 Disparar el pipeline

Como hemos indicado anteriormente, para que las distintas tareas del pipeline compartan información en el Workspaces, necesitamos asociar un volumen para que se guarde la información de manera compartida entre los distintos Pods que ejecutan las tareas. Para crear el volumen vamos a usar un objeto PersistentVolumeClaim que esta definido en el fichero 03_persistent_volume_claim.yaml y que tiene este contenido.

Para disparar la primera ejecución del Pipeline que creará el primer PipelineRun vamos a usar la herramienta tk, ejecutando la siguiente instrucción para la aplicación backend:

``` 
tkn pipeline start build-and-deploy \
    -w name=shared-workspace,volumeClaimTemplateFile=https://raw.githubusercontent.com/josedom24/pipelines-tutorial/master/01_pipeline/03_persistent_volume_claim.yaml \
    -p deployment-name=pipelines-vote-api \
    -p git-url=https://github.com/josedom24/pipelines-vote-api.git \
    -p IMAGE=image-registry.openshift-image-registry.svc:5000/josedom24-dev/pipelines-vote-api \
    --use-param-defaults
```

Y ejecutando la siguiente instrucción indicando el nombre del PipelineRun podemos ver los logs de las distintas tareas:

```
tkn pipelinerun logs build-and-deploy-run-85whr -f -n josedom24-dev
```

De forma similar, para desplegar el frontend:

```
tkn pipeline start build-and-deploy \
    -w name=shared-workspace,volumeClaimTemplateFile=https://raw.githubusercontent.com/josedom24/pipelines-tutorial/master/01_pipeline/03_persistent_volume_claim.yaml \
    -p deployment-name=pipelines-vote-ui \
    -p git-url=https://github.com/josedom24/pipelines-vote-ui.git \
    -p IMAGE=image-registry.openshift-image-registry.svc:5000/josedom24-dev/pipelines-vote-ui \
    --use-param-defaults
```

Vemos el Pipeline que hemos creado:

``` 
tkn pipeline list
```

Si queremos ver las ejecuciones que se han realizado, es decir los objetos PipelineRun:

```
tkn pipelinerun list
``` 

Y sus logs, lo podemos ver:

```
tkn pipeline logs -f
```

Podemos ver los recursos que hemos creado, y si accedemos a la aplicación, vemos que está funcionando.

Si haces algún cambio en las aplicaciones y quieres volver a lanzar el despliegue, tendrías que ejecutar:

```
tkn pipeline start build-and-deploy --last
``` 

### 7.3 Instalación de OpenShift Pipeline en CRC

Por defecto la instalación de OpenShift en local no tiene ningún Operador instalado. Los Operadores nos permiten instalar componentes internos de OpenShift que añaden funcionalidades extras a nuestro clúster.

Instalación del operador OpenShift Pipelines desde la consola web
En la vista Administrator, escogemos la opción Operators->OperatorHub y filtramos con el nombre del operador "OpenShift Pipelines". Nos aparece una ventana con información del operador y pulsamos sobre el botón Install para comenzar la instalación.

Instalamos la última versión del operador etiquetada con latest.
Al escoger la opción All namespaces on the cluster (default) hacemos que el operador se pueda usar en todos los proyectos.
Se va a crear un namespace llamado openshift-operators donde se crearán los recursos que va a instalar el operador.
Se activa la opción de actualizaciones automáticas.

Una vez instalado podemos comprobar que lo tenemos instalado en la opción Operators->Installed Operators. Puedes ver los recursos que se han creado ejecutando:

```  
oc get all -n openshift-operators
```

Y puedes comprobar que ya aparecen la opciones de Pipelines.


## 8. OpenShift Serverless

### 8.1 Ejemplo de Serverless Serving

#### 8.1.1 Knative CLI

En estos ejemplos, vamos a usar la herramienta de línea de comando kn para manejar las aplicaciones Serverless. Para bajar esta herramienta, accedemos a la consola web de OpenShift y elegimos la opción Command line tools en el icono de ayuda (?).

También lo puedes bajar en la página de la documentación oficial.

Para la instalación en Linux Debian/Ubuntu, descomprimimos y copiamos a un directorio que tengamos en el PATH:

```
tar -xf kn-linux-amd64.tar.gz
sudo install kn /usr/local/bin

kn version
Version:      v1.7.1
Build Date:   2023-03-29 06:35:36
...
```

#### 8.1.2 Desplegando una aplicación Serverless

Tenemos varias formas de realizar el despliegue:

1. Usando la definición del servicio en un fichero YAML
   
La definición del despliegue Serverless lo tenemos guardado en el fichero serverless.yaml:

``` 
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello 
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-samples/helloworld-go
          env:
            - name: TARGET 
              value: "Serverless! v1"
```

En ella hemos indicado la imagen que vamos a desplegar, con una variable de entrono que será el mensaje que devuelve la aplicación. Para crear el despliegue:

``` 
oc apply -f serverless.yaml
```

2. Usando el CLI kn
   
Si usamos la herramienta kn, ejecutamos:

```
kn service create hello \
--image gcr.io/knative-samples/helloworld-go \
--port 8080 \
--env TARGET="Serverless! v1"
``` 

3. Desde la consola web de OpenShift

Añadimos un nuevo despliegue desde una imagen. Indicamos la imagen, el nombre y nos aseguramos que en el tipo de despliegue (Resource type) está configurado como Serverless Deployment y creamos la variable de entorno. 

#### 8.1.3 Comprobación de los recursos que se han creado

Si creamos la aplicación con cualquiera de las tres alternativas, podemos comprobar los recursos que se han creado:

```
oc get ksvc,revision,configuration,route.serving.knative
``` 

Puedes ver estos recursos también con la herramienta kn:

```
kn service list
kn revision list
kn route list
```

Realmente podemos comprobar que estos objetos han creado otros objetos, para que la aplicación este desplegada. Si obtenemos los recursos en el terminal:

``` 
oc get all -o name
pod/hello-00001-deployment-57d4c44b69-6wb6z
service/hello
service/hello-00001
service/hello-00001-private
service/modelmesh-serving
deployment.apps/hello-00001-deployment
replicaset.apps/hello-00001-deployment-57d4c44b69
imagestream.image.openshift.io/hello
route.serving.knative.dev/hello
service.serving.knative.dev/hello
revision.serving.knative.dev/hello-00001
configuration.serving.knative.dev/hello
```

#### 8.1.4 Autoescalado

Podemos comprobar que la aplicación funciona:

```
curl https://hello-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
Hello Serverless! v1!
``` 

Comprobamos que hay un Pod ejecutando la aplicación:

``` 
oc get pod
```  

Si esperamos unos segundos veremos que el despliegue Serverless escala a 0, eliminando el Pod mientras no accedemos a la aplicación.

```
oc get pod
```

Puedes ejecutar la instrucción watch oc get pod y comprobar como se crean y eliminan los Pods si hay o no tráfico hacia la aplicación.

#### 8.1.5 Distribución de tráfico hacía una aplicación Serverless

Esta característica la podemos usar para realizar distintas estrategias de despliegue, por ejemplo una estrategia Blue/Green. Para ello vamos a crear una nueva revisión, creando un nuevo objeto Revision modificando la configuración del servicio, por ejemplo cambiando la variable de entorno. Para ello podemos realizar la modificación como hemos visto con otros objetos de OpenShift:

```
oc edit ksvc/hello
...
containers:
  - env:
    - name: TARGET
      value: Serverless! v2
```

Esta modificación también se puede hacer con la herramienta kn:

``` 
kn service update hello --env TARGET="Serverless! v2"
```  

Ahora hemos creado una segunda revisión de la aplicación:

``` 
oc get revision
NAME          CONFIG NAME   K8S SERVICE NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
hello-00001   hello                            1            True             0                 0
hello-00002   hello                            2            True             0                 0
```  

El objeto Revision que hemos creado ha creado un nuevo Deployment que controlará los Pods de la nueva versión de la aplicación. Si accedemos ahora a la ruta de la aplicación, veremos que nos redirige a los Pods de la nueva revisión:

```  
curl https://hello-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
Hello Serverless! v2!
```  

Sin embargo, de una manera muy sencilla podemos redistribuir el tráfico entre cualquier revisión del despliegue, asignando pesos a cada uno de ellos y de esta manera podemos implementar una estrategia de despliegue Blue/Green.

Por ejemplo, desde la consola web, elegimos la opción Set traffic distribution, e indicamos el peso que asignamos a cada revisión para que el tráfico se distribuye con esa proporción. 

Ahora podemos probar el acceso de la aplicación ejecutando el siguiente bucle:

```
while true; do curl https://hello-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com; done
Hello Serverless! v2!
Hello Serverless! v1!
...
```

La distribución entre distintas revisiones se puede realizar también con la herramienta kn:

```
kn service update hello --traffic hello-00001=25 --traffic @latest=75
```

### 8.2 Ejemplo de Serverless Eventing

En este ejemplo vamos a crear una aplicación Serverless muy sencilla, que responde a un evento periódico que vamos a producir con un componente (Event sources) llamado PingSource.

#### 8.2.1 Creación de la aplicación Serverless desde la consola web

Para crear la aplicación Serverless vamos a crear una nueva aplicación usando la imagen quay.io/openshift-knative/knative-eventing-sources-event-display:latest.

Recuerda que el parámetro Resource type debe estar definido con la opción Serverless Aplicattion.

A continuación, vamos a crear el generador de eventos de tipo PingSource, para ello desde el catálogo de aplicaciones lo vamos a instalar y realizamos la configuración.

#### 8.2.2 Creación de la aplicación Serverless con la herramienta kn

Para crear la aplicación Serverless ejecutamos:

``` 
kn service create event-display --image quay.io/openshift-knative/knative-eventing-sources-event-display:latest
```

Y para crear el generador de eventos de tipo PingSource, ejecutamos:

```
 kn source ping create test-ping-source \
--schedule "*/2 * * * *" \
--data '{"message": "Hello world!"}' \
--sink ksvc:event-display
``` 

#### 8.2.3 Comprobación del funcionamiento

Podemos ejecutar en un terminal la siguiente instrucción:

``` 
watch oc get pod
``` 

Para comprobar que cada 2 minutos se crea un Pod tras recibir el evento, pasado unos segundo el Pod se elimina. Cuando tenemos un Pod en ejecución podemos ver sus logs para comprobar que efectivamente ha recibido la información del evento:

``` 
oc logs $(oc get pod -o name | grep event-display) -c event-display

☁️  cloudevents.Event
Validation: valid
Context Attributes,
  specversion: 1.0
  type: dev.knative.sources.ping
  source: /apis/v1/namespaces/josedom24-dev/pingsources/ping-source
  id: b49e0e3d-1cfc-4e07-98a3-db6882cc3d25
  time: 2023-04-27T17:14:00.041380151Z
Data,
  {"message": "Hello world!"}
```

### 8.3 Ejemplo de Serverless Function

Serverless Functions nos permite crear e implementar funciones Serverless basadas en eventos como un servicio Knative. Una función Serverless, también conocida como función sin servidor, es un modelo de programación en la nube en el que nos centramos en escribir funciones que se ejecutan en una infraestructura que no tenemos que mantener, en nuestro caso OpenShift la ejecuta como aplicación Serverless.

Podemos crear funciones Serverless en distintos lenguajes y framework: Qurakus, Go, Node.js, TypeScript, Python,...

#### 8.3.1 Ejemplo de función Serverless escrita en Python

Lo primero que vamos a hacer es crear una aplicación en nuestro entorno de desarrollo basada en python, para ello ejecutamos:

```
cd python
kn func create -l python
```

Nos ha creado un esqueleto de aplicación python. Los ficheros que se han creado son los siguientes:

``` 
ls
app.sh  func.py  func.yaml  Procfile  README.md  requirements.txt  test_func.py
``` 

Nuestra aplicación debe estar en el fichero func.py por lo que vamos a modificar este fichero para escribir una función principal muy sencilla, que devolverá un json:

``` 
def main(context: Context):
    body = { "mensaje": "Funcion Serverless" }
    headers = { "content-type": "application/json" }
    return body, 200, headers
``` 

A continuación, debemos crear una imagen con el código que hemos desarrollado, está imagen se guardará en el registro interno de OpenShift, para ello ejecutamos:

``` 
kn func build
```

Si queremos comprobar el funcionamiento de nuestra función en nuestro entorno de desarrollo, ejecutamos:

``` 
kn func run
...
Function started on port 8080
```

Si accedemos a localhost al puerto 8080 podemos ver la aplicación funcionando antes de desplegarla.

Por último, para desplegar nuestra función en una aplicación Serverless en OpenShift, ejecutamos:

``` 
kn func deploy
``` 

Una vez desplegada, podemos ver el esquema de recursos creados en la topología. Y comprobamos que hemos creado un servicio Knative:

```
kn service ls
``` 

Finalmente, si accedemos a la aplicación, comprobamos que funciona de manera adecuada:

``` 
curl https://python-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
{"mensaje":"Funcion Serverless"}
```

### 8.4 Instalación de OpenShift Serverless en CRC

#### 8.4.1 Instalación del operador OpenShift Pipelines desde la consola web

En la vista Administrator, escogemos la opción Operators->OperatorHub y filtramos con el nombre del operador "OpenShift Serverless".

Nos aparece una ventana con información del operador y pulsamos sobre el botón Install para comenzar la instalación. 

* Instalamos la última versión del operador etiquetada con latest.
* Al escoger la opción All namespaces on the cluster (default) hacemos que el operador se pueda usar en todos los proyectos.
* Se va a crear un namespace llamado openshift-serverless donde se crearán los recursos que va a instalar el operador.
* Se activa la opción de actualizaciones automáticas.
  
Una vez instalado podemos comprobar que lo tenemos instalado en la opción Operators->Installed Operators.

#### 8.4.2 Instalar Knative Serving

Instalar Knative Serving le permite crear servicios y funciones Knative en su clúster. Para realizar la instalación pulsamos sobre la opción Knative Serving del operador OpenShift Serverless que hemos instalado y pulsamos sobre el botón Create KnativeServing. Y dejamos todas las opciones por defecto, pero en la vista YAML nos aseguramos de configurar el namespace con el valor knative-serving, que es el valor esperado y es el proyecto donde se crearan los recurso del operador.

#### 8.4.3 Instalar Knative Eventing

De manera similar, vamos a instalar Knative Eventing, que nos permite el uso de la arquitectura basada en eventos. Para realizar la instalación pulsamos sobre la opción Knative Eventing del operador OpenShift Serverless que hemos instalado

Y dejamos todas las opciones por defecto, pero en la vista YAML nos aseguramos de configurar el namespace con el valor knative-eventing, que es el valor esperado y es el proyecto donde se crearan los recurso del operador. Y puedes comprobar que ya aparecen las opciones de Serverless.




















































