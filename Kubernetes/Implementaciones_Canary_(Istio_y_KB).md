# Cómo realizar implementaciones Canary con Istio y Kubernetes


## 1. Introducción

Al introducir nuevas versiones de un servicio, a menudo es deseable cambiar un porcentaje controlado del tráfico de usuarios a una versión más nueva del servicio en el proceso de eliminación gradual de la versión anterior. Esta técnica se llama despliegue canario.

Los operadores de clústeres de Kubernetes pueden organizar implementaciones canary de forma nativa mediante etiquetas e implementaciones. Sin embargo, esta técnica tiene ciertas limitaciones: la distribución del tráfico y el recuento de réplicas están acoplados, lo que en la práctica significa que las proporciones de réplicas deben controlarse manualmente para limitar el tráfico a la versión canary. En otras palabras, para dirigir el 10% del tráfico a una implementación canary, necesitaría tener un grupo de diez pods, donde un pod recibiría el 10% del tráfico de usuarios y los otros nueve recibirían el resto.

La implementación con una malla de servicios de Istio puede solucionar este problema al permitir una separación clara entre el recuento de réplicas y la gestión del tráfico. La malla de Istio permite un control de tráfico detallado que desacopla la distribución y gestión del tráfico del escalado de réplicas. En lugar de controlar manualmente las proporciones de réplicas, puede definir porcentajes y objetivos de tráfico, e Istio gestionará el resto.

En este tutorial, creará una implementación canary utilizando Istio y Kubernetes. Implementará dos versiones de una aplicación de demostración Node.js y utilizará los recursos de servicio virtual y regla de destino para configurar el enrutamiento del tráfico tanto para la versión más nueva como para la anterior. Este será un buen punto de partida para desarrollar futuras implementaciones canary con Istio.


## 2. Requisitos previos

* Un clúster de Kubernetes 1.10+ con control de acceso basado en roles (RBAC) habilitado. Recomendamos encarecidamente un clúster con al menos 8 GB de memoria disponible y 4 CPU para esta configuración. 

* La kubectl herramienta de línea de comandos instalada en un servidor de desarrollo y configurada para conectarse a su clúster. 

* Docker instalado en su servidor de desarrollo. Asegúrese de agregar su usuario no root al docker grupo.
  
* Una cuenta de Docker Hub. 
  
* Istio se instaló y configuró siguiendo las instrucciones de Cómo instalar y usar Istio con Kubernetes. También debe tener habilitado y configurado el complemento de telemetría de Grafana para acceso externo.


## 3. Empaquetar la solicitud

En el tutorial de requisitos previos, Cómo instalar y usar Istio con Kubernetes, creó una node-demo imagen de Docker para ejecutar una aplicación de información sobre tiburones y envió esta imagen a Docker Hub. En este paso, creará otra imagen: una versión más nueva de la aplicación que utilizará para su implementación canary.

Nuestra aplicación de demostración original enfatizó algunos datos interesantes sobre los tiburones en su página Shark Info.

Pero hemos decidido en nuestra nueva versión canaria enfatizar algunos hechos más aterradores.

Nuestro primer paso será clonar el código de esta segunda versión de nuestra aplicación en un directorio llamado node_image. Usando el siguiente comando, clone el repositorio nodejs-canary-app de la cuenta de GitHub de DigitalOcean Community. Este repositorio contiene el código de la segunda versión, más aterradora, de nuestra aplicación:

``` 
git clone https://github.com/do-community/nodejs-canary-app.git node_image
``` 

Navegue al node_image directorio:

``` 
cd node_image
``` 

Este directorio contiene archivos y carpetas para la versión más reciente de nuestra aplicación de información sobre tiburones, que ofrece a los usuarios información sobre tiburones, como la aplicación original, pero con énfasis en hechos más aterradores. Además de los archivos de la aplicación, el directorio contiene un Dockerfile con instrucciones para crear una imagen de Docker con el código de la aplicación.

Para probar que el código de la aplicación y Dockerfile funcionan como se espera, puede crear y etiquetar la imagen usando el docker build comando y luego usar la imagen para ejecutar un contenedor de demostración. Usar la -t bandera con docker build le permitirá etiquetar la imagen con su nombre de usuario de Docker Hub para que pueda enviarla a Docker Hub una vez que la haya probado.

Construya la imagen con el siguiente comando:

``` 
docker build -t your_dockerhub_username/node-demo-v2 .
```

El . comando especifica que el contexto de compilación es el directorio actual. Le hemos nombrado la imagen node-demo-v2 para hacer referencia a la node-demo imagen que creamos en Cómo instalar y usar Istio con Kubernetes.

Una vez que se completa el proceso de compilación, puede enumerar sus imágenes con docker images:

``` 
docker images
```

Verá el siguiente resultado que confirma la creación de la imagen:

```
Output
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/node-demo-v2  latest              37f1c2939dbf        5 seconds ago       77.6MB
node                                  10-alpine           9dfa73010b19        2 days ago          75.3MB
``` 

A continuación, utilizará docker run para crear un contenedor basado en esta imagen. Incluiremos tres banderas con este comando:

* **-p:** Esto publica el puerto en el contenedor y lo asigna a un puerto en nuestro host. Usaremos el puerto 80 en el host, pero no dude en modificarlo según sea necesario si tiene otro proceso ejecutándose en ese puerto. 
* **-d:** Esto ejecuta el contenedor en segundo plano.
* **--name:** Esto nos permite darle al contenedor un nombre personalizado.
* 
Ejecute el siguiente comando para construir el contenedor:

``` 
docker run --name node-demo-v2 -p 80:8080 -d your_dockerhub_username/node-demo-v2
``` 

Inspeccione sus contenedores en ejecución con docker ps:

``` 
docker ps
``` 

Verá un resultado que confirma que el contenedor de su aplicación se está ejecutando:

``` 
Output
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS                  NAMES
49a67bafc325        your_dockerhub_username/node-demo-v2   "docker-entrypoint.s…"   8 seconds ago       Up 6 seconds        0.0.0.0:80->8080/tcp   node-demo-v2
```

Ahora puede visitar la IP de su servidor en su navegador para probar su configuración. Su aplicación mostrará la siguiente página de inicio:http://your_server_ip

Haga clic en el botón Obtener información sobre tiburones para acceder a la información más aterradora sobre tiburones.

Ahora que ha probado la aplicación, puede detener el contenedor en ejecución. Úselo docker psnuevamente para obtener su CONTAINER ID:

``` 
docker ps
``` 

``` 
Output
CONTAINER ID        IMAGE                                  COMMAND                  CREATED              STATUS              PORTS                  NAMES
49a67bafc325        your_dockerhub_username/node-demo-v2   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:80->8080/tcp   node-demo-v2
``` 

Detenga el contenedor con docker stop. Asegúrese de reemplazar la CONTAINER ID que aparece aquí con su propia aplicación CONTAINER ID:

``` 
docker stop 49a67bafc325
``` 

Ahora que ha probado la imagen, puede enviarla a Docker Hub. Primero, inicie sesión en la cuenta de Docker Hub que creó en los requisitos previos:

``` 
docker login -u your_dockerhub_username 
``` 

Cuando se le solicite, ingrese la contraseña de su cuenta Docker Hub. Iniciar sesión de esta manera creará un ~/.docker/config.json archivo en el directorio de inicio de su usuario no root con sus credenciales de Docker Hub.

Envíe la imagen de la aplicación a Docker Hub con el docker pushcomando. Recuerde reemplazarlo 
your_dockerhub_username con su propio nombre de usuario de Docker Hub:

``` 
docker push your_dockerhub_username/node-demo-v2
``` 

Ahora tiene dos imágenes de aplicaciones guardadas en Docker Hub: la node-demo imagen y el archivo node-demo-v2. Ahora modificaremos los manifiestos que creó en el tutorial de requisitos previos Cómo instalar y usar Istio con Kubernetes para dirigir el tráfico a la versión canary de su aplicación.


## 4. Modificar la implementación de la aplicación

En Cómo instalar y usar Istio con Kubernetes, creó un manifiesto de aplicación con especificaciones para los objetos de servicio e implementación de su aplicación. Estas especificaciones describen el estado deseado de cada objeto. En este paso, agregará una implementación para la segunda versión de su aplicación a este manifiesto, junto con etiquetas de versión que permitirán a Istio administrar estos recursos.

Cuando siguió las instrucciones de configuración en el tutorial de requisitos previos, creó un directorio llamado istio_project y dos yaml manifiestos: node-app.yaml, que contiene las especificaciones para sus objetos de Servicio e Implementación, y node-istio.yaml, que contiene especificaciones para sus recursos de Puerta de Enlace y Servicio Virtual de Istio.

Navegue al istio_project directorio ahora:

``` 
cd
cd istio_project
``` 

Ábre el node-app.yaml con nano o con tu editor favorito para realizar cambios en el manifiesto de tu aplicación:

``` 
nano node-app.yaml
``` 

Actualmente, el archivo tiene este aspecto:

``` 
apiVersion: v1
kind: Service
metadata:
  name: nodejs
  labels: 
    app: nodejs
spec:
  selector:
    app: nodejs
  ports:
  - name: http
    port: 8080 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs
  labels:
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
        version: v1
    spec:
      containers:
      - name: nodejs
        image: your_dockerhub_username/node-demo
        ports:
        - containerPort: 8080
``` 

Para obtener una explicación completa del contenido de este archivo, consulte el Paso 3 de Cómo instalar y usar Istio con Kubernetes.

Ya hemos incluido etiquetas de versión en nuestros campos Implementación metadata y template, siguiendo las recomendaciones de Istio para Pods y Servicios. Ahora podemos agregar especificaciones para un segundo objeto de implementación, que representará la segunda versión de nuestra aplicación, y realizar una modificación rápida en el namede nuestro primer objeto de implementación.

Primero, cambie el nombre de su objeto de implementación existente a :nodejs-v1

``` 
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-v1
  labels:
    version: v1
...
``` 

A continuación, debajo de las especificaciones de esta implementación, agregue las especificaciones para su segunda implementación. Recuerda agregar el nombre de tu propia imagen al image campo:

``` 
...
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-v2
  labels:
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
        version: v2
    spec:
      containers:
      - name: nodejs
        image: your_dockerhub_username/node-demo-v2
        ports:
        - containerPort: 8080
``` 

Al igual que la primera Implementación, esta Implementación utiliza una version etiqueta para especificar la versión de la aplicación que corresponde a esta Implementación. En este caso v2 distinguiremos la versión de la aplicación asociada a este Deployment v1 de la que corresponde a nuestro primer Deployment.

También nos hemos asegurado de que los Pods administrados por la v2 Implementación ejecutarán la node-demo-v2 imagen canary, que creamos en el Paso anterior.

Guarde y cierre el archivo cuando haya terminado de editarlo.

Una vez modificado el manifiesto de su aplicación, puede continuar y realizar cambios en su node-istio.yaml archivo.

## 5. ponderar el tráfico con servicios virtuales y agregar reglas de destino

En Cómo instalar y usar Istio con Kubernetes, creó objetos de puerta de enlace y servicio virtual para permitir el tráfico externo en la malla de Istio y enrutarlo al servicio de su aplicación. Aquí, modificará la configuración de su Servicio Virtual para incluir el enrutamiento a los subconjuntos de Servicio de su aplicación, v1 y v2. También agregará una regla de destino para definir políticas adicionales basadas en versiones de las reglas de enrutamiento que está aplicando al nodejs servicio de su aplicación.

Abre el node-istio.yaml archivo:

```
nano node-istio.yaml
``` 

Actualmente, el archivo tiene este aspecto:

``` 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: nodejs-gateway
spec:
  selector:
    istio: ingressgateway 
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nodejs
spec:
  hosts:
  - "*"
  gateways:
  - nodejs-gateway
  http:
  - route:
    - destination:
        host: nodejs
``` 

Para obtener una explicación completa de las especificaciones de este manifiesto, consulte el Paso 4 de Cómo instalar y usar Istio con Kubernetes.

Nuestra primera modificación será al Servicio Virtual. Actualmente, este recurso dirige el tráfico que ingresa a la malla a través de nuestro nodejs-gateway Servicio nodejs de aplicación. Lo que nos gustaría hacer es configurar una regla de enrutamiento que enviará el 80% del tráfico a nuestra aplicación original y el 20% a la versión más nueva. Una vez que estemos satisfechos con el rendimiento del canary, podemos reconfigurar nuestras reglas de tráfico para enviar gradualmente todo el tráfico a la versión más nueva de la aplicación.

En lugar de enrutar a un solo archivo destination, como hicimos en el manifiesto original, agregaremos destination campos para ambos subconjuntos de nuestra aplicación: la versión original ( v1) y el canario ( v2).

Realice las siguientes adiciones al Servicio virtual para crear esta regla de enrutamiento:

``` 
...
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nodejs
spec:
  hosts:
  - "*"
  gateways:
  - nodejs-gateway
  http:
  - route:
    - destination:
        host: nodejs
        subset: v1
      weight: 80
    - destination:
        host: nodejs
        subset: v2
      weight: 20
``` 

La política que hemos agregado incluye dos destinos: el subset de nuestro nodejs Servicio que ejecuta la versión original de nuestra aplicación v1 y el subset que ejecuta el canario v2. El subconjunto uno recibirá el 80% del tráfico entrante, mientras que el canario recibirá el 20%.

A continuación, agregaremos una regla de destino que aplicará reglas al tráfico entrante después de que ese tráfico se haya enrutado al servicio apropiado. En nuestro caso, configuraremos subset campos para enviar tráfico a Pods con las etiquetas de versión apropiadas.

Agregue el siguiente código debajo de su definición de Servicio Virtual:

``` 
...
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: nodejs
spec:
  host: nodejs
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
``` 

Nuestra regla garantiza que el tráfico a nuestros subconjuntos de servicios v1 y v2 llegue a los pods con las etiquetas adecuadas: version: v1 y version: v2. Estas son las etiquetas que incluimos en las especificaciones de implementación de nuestra aplicación.

Sin embargo, si quisiéramos, también podríamos aplicar políticas de tráfico específicas a nivel de subconjunto, lo que permitiría una mayor especificidad en nuestras implementaciones canary. Para obtener información adicional sobre cómo definir políticas de tráfico en este nivel, consulte la documentación oficial de Istio.

Guarde y cierre el archivo cuando haya terminado de editarlo.

Una vez revisados ​​los manifiestos de su aplicación, está listo para aplicar los cambios de configuración y examinar los datos de tráfico de su aplicación utilizando el complemento de telemetría de Grafana.


## 6. Aplicar cambios de configuración y acceder a datos de tráfico

Los manifiestos de la aplicación se actualizan, pero aún debemos aplicar estos cambios a nuestro clúster de Kubernetes. Usaremos el kubectl apply comando para aplicar nuestros cambios sin sobrescribir completamente la configuración existente. Después de hacer esto, podrá generar algunas solicitudes para su aplicación y ver los datos asociados en sus paneles de Istio Grafana.

Aplique su configuración a los objetos de Servicio e Implementación de su aplicación:

``` 
kubectl apply -f node-app.yaml
```

Verá el siguiente resultado:

``` 
Output
service/nodejs unchanged
deployment.apps/nodejs-v1 created
deployment.apps/nodejs-v2 created
``` 

A continuación, aplique las actualizaciones de configuración que realizó node-istio.yaml, que incluyen los cambios en el Servicio Virtual y la nueva Regla de Destino:

```
kubectl apply -f node-istio.yaml
```

Verá el siguiente resultado:

``` 
Output
gateway.networking.istio.io/nodejs-gateway unchanged
virtualservice.networking.istio.io/nodejs configured
destinationrule.networking.istio.io/nodejs created
``` 

Ahora está listo para generar tráfico a su aplicación. Sin embargo, antes de hacer eso, primero verifique que tenga el grafana Servicio en ejecución:

``` 
kubectl get svc -n istio-system | grep grafana
Output
grafana                  ClusterIP      10.245.233.51    <none>           3000/TCP                                                                                                                                     4d2h
``` 

Compruebe también los Pods asociados:

``` 
kubectl get svc -n istio-system | grep grafana
Output
grafana-67c69bb567-jpf6h                 1/1     Running     0          4d2h
``` 

Finalmente, verifique el grafana-gateway Gateway y grafana-vs el Servicio Virtual:

``` 
kubectl get gateway -n istio-system | grep grafana
Output
grafana-gateway   3d5h
``` 

``` 
kubectl get virtualservice -n istio-system | grep grafana
Output
grafana-vs   [grafana-gateway]   [*]     4d2h
``` 

Si no ve el resultado de estos comandos, consulte los pasos 2 y 5 de Cómo instalar y usar Istio con Kubernetes, que explican cómo habilitar el complemento de telemetría de Grafana al instalar Istio y cómo habilitar el acceso HTTP al servicio Grafana.

Ahora puede acceder a su aplicación en el navegador. Para hacer esto, necesitará la IP externa asociada con su istio-ingress gateway Servicio, que es un tipo de Servicio LoadBalancer. Emparejamos nuestra nodejs-gateway puerta de enlace con este controlador al escribir nuestro manifiesto de puerta de enlace en Cómo instalar y usar Istio con Kubernetes. Para obtener más detalles sobre el manifiesto de Gateway, consulte el Paso 4 de ese tutorial.

Obtenga la IP externa del istio-ingress gateway Servicio con el siguiente comando:

``` 
kubectl get svc -n istio-system
```

Verá un resultado como el siguiente:

``` 
Output
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                                                                                                                                      AGE
grafana                  ClusterIP      10.245.85.162    <none>            3000/TCP                                                                                                                                     42m
istio-citadel            ClusterIP      10.245.135.45    <none>            8060/TCP,15014/TCP                                                                                                                           42m
istio-galley             ClusterIP      10.245.46.245    <none>            443/TCP,15014/TCP,9901/TCP                                                                                                                   42m
istio-ingressgateway     LoadBalancer   10.245.171.39    ingressgateway_ip 15020:30707/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:30285/TCP,15030:31668/TCP,15031:32297/TCP,15032:30853/TCP,15443:30406/TCP   42m
istio-pilot              ClusterIP      10.245.56.97     <none>            15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       42m
istio-policy             ClusterIP      10.245.206.189   <none>            9091/TCP,15004/TCP,15014/TCP                                                                                                                 42m
istio-sidecar-injector   ClusterIP      10.245.223.99    <none>            443/TCP                                                                                                                                      42m
istio-telemetry          ClusterIP      10.245.5.215     <none>            9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       42m
prometheus               ClusterIP      10.245.100.132   <none>            9090/TCP      
``` 

Debe istio-ingress gateway ser el único Servicio con TYPE LoadBalancer y el único Servicio con una IP externa.

Navegue a esta IP externa en su navegador: .http://ingressgateway_ip

Haga clic en el botón Obtener información sobre tiburones. Verá una de las dos páginas de información sobre tiburones.

Haga clic en actualizar en esta página varias veces. Deberías ver la página de información sobre tiburones más amigable con más frecuencia que la versión más aterradora.

Una vez que haya generado algo de carga actualizando cinco o seis veces, puede dirigirse a sus paneles de Grafana.

En su navegador, navegue hasta la siguiente dirección, nuevamente usando su istio-ingress gateway IP externa y el puerto definido en el manifiesto de Grafana Gateway: .http://ingressgateway_ip:15031

Verá la siguiente página de inicio, la de Grafana Dashboard.

Al hacer clic en Inicio en la parte superior de la página, accederá a una página con una carpeta istio. Para obtener una lista de opciones desplegables, haga clic en el icono de la carpeta istio.

En esta lista de opciones, haga clic en Istio Service Dashboard. Esto lo llevará a una página de destino con otro menú desplegable.

Seleccione nodejs.default.svc.cluster.local de la lista de opciones disponibles.

Si navega hasta la sección Cargas de trabajo de servicio de la página, podrá ver las solicitudes entrantes por destino y código de respuesta.

Aquí verá una combinación de códigos de respuesta HTTP 200 y 304, que indican OK respuestas exitosas Not Modified. Las respuestas etiquetadas nodejs-v1 deben superar en número a las respuestas etiquetadas nodejs-v2, lo que indica que el tráfico entrante se enruta a nuestros subconjuntos de aplicaciones siguiendo los parámetros que definimos en nuestros manifiestos.

## 7. Conclusión

En este tutorial, implementó una versión canary de una aplicación de demostración Node.js usando Istio y Kubernetes. Creó recursos de servicio virtual y regla de destino que juntos le permitieron enviar el 80 % de su tráfico a su servicio de aplicación original y el 20 % a la versión más nueva. Una vez que esté satisfecho con el rendimiento de la versión más reciente de la aplicación, puede actualizar sus ajustes de configuración como desee.

