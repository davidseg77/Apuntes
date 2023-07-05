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

## 2. ImageStreams: Gestión de imágenes en OpenShift v4

### ImageStream a imágenes del registro interno

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

**Uso de las ImageStream internas**

Como hemos indicado, usaremos las ImageStream para la creación de nuevas aplicaciones o la construcción de nuevas imágenes.

* *Creación de un nuevo despliegue*
* 
Por ejemplo, podríamos desplegar una nueva aplicación a partir de la ImageStream httpd:2.4:

```  
oc new-app httpd:2.4 --name=web1
```  

Una vez creado el despliegue, podríamos ver la imagen que se está usando para el despliegue:

```
oc describe deploy web1
```

Evidentemente su ID coincide con la imagen que vimos anteriormente y que estaba apuntada por httpd:2.4.

**Construcción de imágenes**

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

### Creación de ImageStream

Tenemos tres maneras de crear recursos ImageStream:

**1. Creación de ImageStream con new-app**

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

**2. Creación de ImageStream con import-image**

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

**3. Creación de ImageStream con fichero YAML**

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

### Gestión de las etiquetas en un ImageStream

**Crear nuevas etiquetas en un ImageStream**

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

**Ejemplo: Actualización de despliegue por cambio de imagen**

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

### Actualización automática de ImageStream

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

### Construcción de imágenes con estrategia Source-to-Image (S2I)

**Despliegue de la aplicación con la estrategia Source-to-image**

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

**Construcción de la imagen indicando una versión de la imagen constructora**

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


### Construcción de imágenes con estrategia Docker + repositorio Git

**Creación del BuildConfig con estrategia Docker**

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

**Otras operaciones**

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

### Definición del objeto BuildConfig

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

**Definición de los objetos BuildConfig definidos en los ejercicios anteriores**

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











