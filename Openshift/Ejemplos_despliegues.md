# Info del curso de Openshift V4 como Paas de OW. Ejemplos finales

## 1. Despliegue de aplicación Citas en OpenShift v4

La aplicación Citas nos muestra citas celebres de distintos autores en una página web. Esta formada por dos microservicios (citas-backend y citas-frontend) y un servicio de base de datos mysql. La aplicación funciona de la siguiente manera:

* citas-backend: Es una API RESTful que devuelve información sobre citas famosas de distintos autores famosos. La versión 1 devuelve información de 6 citas que tiene incluidas en el programa. La versión 2 lee la información de las citas de una base de datos guardada en un servidor mysql. La aplicación está construida en python 3.9 y ofrece el servicio en el puerto TCP/10000.
* citas-frontend: Es una aplicación python flask que crea una página web dinámica con una cita aleatoria que lee de citas-backend para ello conecta al servicio API RESTful usando el nombre y el puerto del servidor indicado en la variable de entorno CITAS_SERVIDOR.
  
### 1.1 Despliegue de citas-backend

Para realizar el despliegue y posteriormente actualizar a una nueva versión, lo primero que vamos a hacer es crear un objeto ImageStream que apunte a las imágenes de la aplicación que vamos a desplegar, para ello:

```
oc create is citas-backend
oc import-image citas-backend:v1 --from=docker.io/josedom24/citas-backend:v1
oc import-image citas-backend:v2 --from=docker.io/josedom24/citas-backend:v2
``` 

Vamos a crear la etiqueta prod apuntando a la versión que vamos a desplegar:

``` 
oc tag citas-backend:v1 citas-backend:prod
``` 

Vamos a crear el despliegue de la siguiente manera:

``` 
oc new-app citas-backend:prod -l app.kubernetes.io/part-of=citas-app --name=citas-backend 
``` 

Podemos crear un objeto Route para comprobar que está funcionando:

``` 
oc expose service citas-backend --port=10000

oc get routes
``` 

Para mostrar todas las citas:

```
curl http://citas-backend-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/quotes
```

Para mostrar una ruta aleatoria:

``` 
curl http://citas-backend-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/quotes/random
``` 

Una vez comprobado que funciona, borramos la ruta:

``` 
oc delete route/citas-backend
```

### 1.2 Despliegue de citas-frontend

Para desplegar el componente citas-frontend vamos a usar un Deployment Serverless. Vamos a realizar la configuración desde la consola web. Indicamos la imagen josedom24/citas-frontend y el icono de la aplicación.

Indicamos el nombre de la aplicación: citas-app, el nombre del despliegue: citas-frontend y el puerto que será el 5000.

Elegimos el recurso Deployment Serverless como Resource type y creamos la variable de entorno CITAS_SERVIDOR con el valor del nombre del host para acceder a la aplicación citas-backend y el puerto que está utilizando. En nuestro caso: citas-backend:10000. Es decir, indicaremos el nombre del recurso Service que hemos creado para acceder a la aplicación citas-backend.

Accedemos a la aplicación y comprobamos que funciona de forma correcta. 

Como hemos desplegado citas-frontend en un Deployment Serverless, pasados unos segundos sin acceder a la página veremos que escala a 0. El componente citas-backend sigue funcionando continuamente, y podemos tener otros componentes que acceden a él sin problemas.

### 1.3 Despliegue de la base de datos mysql

Para hacer el despliegue de la base de datos persistente vamos a usar la plantilla mysql-persistent. Veamos los parámetros que podemos configurar:

``` 
oc process --parameters mysql-persistent -n openshift
NAME                    DESCRIPTION                                                             GENERATOR           VALUE
MEMORY_LIMIT            Maximum amount of memory the container can use.                                             512Mi
NAMESPACE               The OpenShift Namespace where the ImageStream resides.                                      openshift
DATABASE_SERVICE_NAME   The name of the OpenShift Service exposed for the database.                                 mysql
MYSQL_USER              Username for MySQL user that will be used for accessing the database.   expression          user[A-Z0-9]{3}
MYSQL_PASSWORD          Password for the MySQL connection user.                                 expression          [a-zA-Z0-9]{16}
MYSQL_ROOT_PASSWORD     Password for the MySQL root user.                                       expression          [a-zA-Z0-9]{16}
MYSQL_DATABASE          Name of the MySQL database accessed.                                                        sampledb
VOLUME_CAPACITY         Volume space available for data, e.g. 512Mi, 2Gi.                                           1Gi
MYSQL_VERSION           Version of MySQL image to be used (8.0-el7, 8.0-el8, or latest).                            8.0-el8
``` 

Creamos el despliegue ejecutando:

```
oc new-app mysql-persistent -p MYSQL_USER=usuario \
                            -p MYSQL_PASSWORD=asdasd \
                            -p MYSQL_DATABASE=citas \
                            -p MYSQL_ROOT_PASSWORD=asdasd \
                            -p VOLUME_CAPACITY=5Gi \
                            -l app.kubernetes.io/part-of=citas-app \
                            -l app.openshift.io/runtime=mysql-database
``` 

Y comprobamos los recursos que hemos creado.

A continuación, nos queda inicializar la base de datos, para ello vamos a copiar el fichero citas.sql al Pod. Para facilitar el copiado de ficheros vamos a guardar el nombre del Pod en una variable de entorno:

```
export PODNAME="mysql-2-wzqv4"
``` 

A continuación copiamos el fichero:

``` 
oc cp citas.sql $PODNAME:/tmp/
``` 

Y finalmente ejecutamos el fichero sql:

``` 
oc exec dc/mysql -- bash -c "mysql -uroot -pasdasd -h 127.0.0.1 < /tmp/citas.sql"
``` 

Finalmente comprobamos que hemos guardado las citas en la tabla quotes:

``` 
oc exec -it dc/mysql -- bash -c "mysql -u usuario -pasdasd -h 127.0.0.1 citas"
...
mysql> select * from quotes;
+----+---------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
| Id | quotation                                                                                                                                                     | author              |
+----+---------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
|  1 | Yeah, well, that's just like, your opinion, man.                                                                                                              | The Dude            |
|  2 | It is not only what you do but also the attitude you bring to it, that makes you a success.                                                                   | Don Schenck         |
...
``` 

### 1.4 Actualización de la aplicación citas-backend

Para terminar este ejercicio, vamos a actualizar citas-backend a la versión 2. Esta versión conecta con la base de datos mysql, para obtener las citas, por lo que tendremos que configurar las credenciales de acceso a la base de datos utilizando variables de entorno.

Por lo tanto en el despliegue de citas-backend vamos a crear las variables de entorno necesarias. Para realizar esta operación, vamos a crear un recurso ConfigMap con las variables de entorno que posteriormente cargaremos en el despliegue:

```
oc create cm citas-mysql --from-literal=USER_DB=usuario \
                      --from-literal=PASSWORD_DB=asdasd     \
                      --from-literal=HOST_DB=mysql 
``` 

Y configuramos el objeto Deployment citas-backend con las variables que hemos creado en el ConfigMap con el parámetro --from:

```
oc set env deploy/citas-backend --from=cm/citas-mysql
``` 

Por último, actualizamos la etiqueta del ImageStream para que prod apunte a v2:

``` 
oc tag -d citas-backend:prod
oc tag citas-backend:v2 citas-backend:prod
``` 

Se produce de forma automática la actualización del despliegue y podemos acceder de nuevo a la página web y comprobamos que el servicio que está devolviendo la información de la citas es citas-backend versión 2 y además comprobamos que tenemos más citas (en la tabla hay 16 citas), la versión 1 tenía sólo 6 citas.


## 2. Despliegue de aplicación Parksmap en OpenShift v4

Este ejercicio esta basado y es una adaptación del ejemplo que se muestra en la guía OpenShift Starter Guides.

### 2.1 Despliegue de Parksmap

Parksmap es la aplicación frontend que visualizará en un mapa las coordenados de los Parques Nacionales. Esta aplicación está escrita con el framework de Java Spring-boot y vamos a desplegarla usando la imagen quay.io/openshiftroadshow/parksmap:latest desde la consola web. Hemos escogido en la vista Developer, la opción +Add -> Conatiner Images, e indicamos la imagen y el icono de spring-boot.

A continuación, indicamos el nombre de la aplicación workshop, el nombre del despliegue parksmap y creamos un recurso Route.

Por último indicamos que vamos a desplegar la aplicación como un objeto Deployment e indicamos las siguientes etiquetas:

* app=workshop
* component=parksmap
* role=frontend

Creamos el despliegue, y al cabo de unos segundos comprobamos los recursos creados en la topología. Accedemos a la URL del objeto Route y comprobamos que la aplicación está funcionando, aunque todavía no puede mostrar la localización de los puertos naturales.

### 2.2 Permisos

Todas las interacciones que hacemos sobre la API de OpenShift son autenticadas (¿Quién eres?) y autorizadas (¿Estás autorizado a hacer esta operación?).

Muchas de las interacciones que se hacen sobre la API de OpenShift se realizan por el usuario final, pero muchas otras se hacen internamente. Para hacer estas últimas peticiones a la API se utilizan unas cuentas especiales de usuario, que se llaman Service Account.

OpenShift crea automáticamente algunas Service Account en cada proyecto. Por ejemplo, hay una Service Account que se llama default y será la responsable de ejecutar los Pods.

Puede ver los permisos actuales en la consola web: en la vista Administrator, escogiendo la opción User Management -> RoleBindings

Más adelante, veremos que la aplicación Parksmap necesita hacer una petición a la API OpenShift para preguntar sobre la configuración de otros recursos.

Por lo tanto, tenemos que otorgar el permiso view al Service Account default, para que el Pod pueda consultar sobre los recursos que se encuentran dentro del proyecto. Para ello, pulsamos sobre le botón Create binding de la pantalla anterior y configuramos el permiso indicando el nombre del permiso, el proyecto y el Role view que se va a asignar a un Service Account llamado default en nuestro proyecto.

Si queremos hacerlo desde la terminal, ejecutamos:

```
oc policy add-role-to-user view -z default
```

Finalmente actualizamos el despliegue:

```
oc rollout restart deployment/parksmap
``` 

## 3. Despliegue de aplicación Nationalparks en OpenShift v4 (2ª parte)

En este apartado vamos a desplegar la aplicación escrita en Java Nationalparks, es el servicio backend, que expondrá varios endpoints que la aplicación Parksmap utilizará para obtener la información de los Parques Nacionales que visualizará en el mapa. Este servicio guarda la información de los Parques Nacionales en una base de datos MongoDB. Para hacer las peticiones a este servicio se utilizará una URL proporcionada por el objeto Route que crearemos a continuación.

Vamos a usar la estrategia de construcción de imágenes Source-to-Image, utilizando el código de la aplicación que se encuentra en el repositorio https://github.com/openshift-roadshow/nationalparks.git.

Vamos a crear el despliegue desde la terminal, para ello:

``` 
oc new-app java:openjdk-11-ubi8~https://github.com/openshift-roadshow/nationalparks.git --name=nationalparks --strategy=source -l 'app=workshop,component=nationalparks,role=backend,app.kubernetes.io/part-of=workshop,app.openshift.io/runtime=java'
                
oc expose service/nationalparks
```

Una vez creado los recursos accedemos a la topología y comprobamos los que hemos creado.

Para comprobar que la aplicación está funcionando podemos acceder a la URL /ws/info y nos debe aparecer un JSON de ejemplo:

```
curl http://nationalparks-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/ws/info   
{"id":"nationalparks","displayName":"National Parks","center":{"latitude":"47.039304","longitude":"14.505178"},"zoom":4}
``` 

### 3.1 Conexión a la base de datos

Es el momento de desplegar una base de datos MongoDB para guardar la información de los Parques Naturales. Para realizar la instalación de MongoDB vamos a usar un Template que vamos a crear en nuestro proyecto:

``` 
oc create -f https://raw.githubusercontent.com/openshift-labs/starter-guides/ocp-4.8/mongodb-template.yaml
```

Hay que indicar que el Template que hemos creado intenta crear el despliegue de MongoDB desde una imagen mongodb:3.6 que busca por defecto en el proyecto openshift. Como en Red Hat OpenShift Dedicated Developer Sandbox no tenemos esa imagen, vamos a crear un objeto ImageStream en nuestro proyecto que apunte a una imagen de MongoDB externa, para ello:

```
oc import-image mongodb:3.6 --from=registry.access.redhat.com/rhscl/mongodb-36-rhel7 --confirm
``` 

Añadimos el despliegue desde el catálogo de aplicaciones, donde buscamos la plantilla que acabamos de crear.

Y realizamos la siguiente configuración:

* Indicamos nuestro proyecto como namespace, para que busque de manera adecuada el ImageStream con el que estamos trabajando.
* El nombre del servicio lo configuramos como mongodb-nationalparks.
* Las credenciales de la base de datos, usuario, contraseña, base de datos, contraseña del administrador lo configuramos com mongodb.
* Y la versión de la imagen dejamos la 3.6.
  
Una vez creada la base de datos, podemos ver los recursos que tenemos en nuestro proyecto desde la topología.

Ahora tenemos que configurar la aplicación Nationalparks con las credenciales de acceso a la base de datos para que pueda conectar al MongoDB, para ello vamos a usar el recurso Secret que se ha creado con la plantilla y vamos a configurar las variables de entorno de Nationalparks con los valores del Secret. Para ello, accede a la opción Secrets y elige el que se ha creado con la plantilla.

A continuación, vamos a usarlo para configurar el despliegue de Nationalparks, para ello pulsa sobre el botón Add Secret to workload.

Elegimos el Deployment Nationalparks y lo configuramos con estas nuevas variables de entorno. La configuración del despliegue a cambiado por lo que se producirá una actualización del despliegue, creando un nuevo ReplicaSet y un nuevo conjunto de Pods.

A continuación etiquetamos los recursos de MongoDB de forma adecuada:

```
oc label dc/mongodb-nationalparks svc/mongodb-nationalparks app=workshop component=nationalparks role=database --overwrite
```

Para cargar los datos sobre los Parques Nacionales a la base de datos, tenemos que ejecutar la URL /ws/data/load de la aplicación Nationalparks, para ello:

``` 
curl http://nationalparks-josedom24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com/ws/data/load
Items inserted in database: 2893
```

Ahora si accedemos a la URL /ws/data/all veremos un JSON con todos los datos de los Parques Nacionales, que esta leyendo de la base de datos.

Finalmente, veamos como la aplicación Parksmap se puede comunicar con la aplicación Nationalparks. En otros ejemplos configuramos el frontend con el nombre del recurso Service del backend para que pudiera tener acceso al servicio.

En este caso, Parksmap está consultando la API de OpenShift y preguntando por las rutas y servicios del proyecto. Si alguno de ellos tiene una etiqueta que sea type=parksmap-backend, la aplicación sabe que debe hacer peticiones a esa URL para buscar los datos del mapa. Por lo tanto, si queremos que Parksmap haga peticiones a Nationalparks para pedir información de los Parques Nacionales y poder visualizarlos tenemos que asignar la etiqueta indicada al recurso Route de Nationalparks, para ello:

``` 
oc label route nationalparks type=parksmap-backend
```

Y ahora si accedemos a la URL de Parksmap, veremos la visualización de los Parques Nacionales.

