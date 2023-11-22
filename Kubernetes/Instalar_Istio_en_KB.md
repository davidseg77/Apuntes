# Cómo instalar y usar Istio con Kubernetes


## 1. Introducción

Una malla de servicios es una capa de infraestructura que le permite gestionar la comunicación entre los microservicios de su aplicación. A medida que más desarrolladores trabajan con microservicios, las mallas de servicios han evolucionado para hacer ese trabajo más fácil y efectivo al consolidar tareas administrativas y de gestión comunes en una configuración distribuida.

El uso de una malla de servicios como Istio puede simplificar tareas como el descubrimiento de servicios, la configuración de enrutamiento y tráfico, el cifrado y la autenticación/autorización, y el monitoreo y la telemetría. Istio, en particular, está diseñado para funcionar sin cambios importantes en el código de servicio preexistente. Cuando se trabaja con Kubernetes, por ejemplo, es posible agregar capacidades de malla de servicios a las aplicaciones que se ejecutan en su clúster mediante la creación de objetos específicos de Istio que funcionan con los recursos de las aplicaciones existentes.

En este tutorial, instalará Istio utilizando el administrador de paquetes Helm para Kubernetes. Luego utilizará Istio para exponer una aplicación Node.js de demostración al tráfico externo mediante la creación de recursos de puerta de enlace y servicio virtual. Finalmente, accederá al complemento de telemetría de Grafana para visualizar los datos de tráfico de su aplicación.

## 2. Requisitos previos

* Clúster de Kubernetes con control de acceso basado en roles (RBAC) habilitado. Recomendamos encarecidamente un clúster con al menos 8 GB de memoria disponible y 4 vCPU para esta configuración.

* La kubectl herramienta de línea de comandos instalada en un servidor de desarrollo y configurada para conectarse a su clúster. 
  
* Helm instalado en su servidor de desarrollo y Tiller instalado en su clúster.
  
* Docker instalado en su servidor de desarrollo. Asegúrese de agregar su usuario no root al docker grupo.

* Una cuenta de Docker Hub.


## 3. Empaquetar la solicitud

Para utilizar nuestra aplicación de demostración con Kubernetes, necesitaremos clonar el código y empaquetarlo para que el kubelet agente pueda extraer la imagen.

Nuestro primer paso será clonar el repositorio nodejs-image-demo de la cuenta de GitHub. 

Para comenzar, clone el repositorio nodejs-image-demo en un directorio llamado istio_project:

git clone https://github.com/do-community/nodejs-image-demo.git istio_project

Navegue al istio_project directorio:

``` 
cd istio_project
``` 

Este directorio contiene archivos y carpetas para una aplicación de información sobre tiburones que ofrece a los usuarios información básica sobre los tiburones. Además de los archivos de la aplicación, el directorio contiene un Dockerfile con instrucciones para crear una imagen de Docker con el código de la aplicación.

Para probar que el código de la aplicación y Dockerfile funcionan como se espera, puede crear y etiquetar la imagen usando el docker build comando y luego usar la imagen para ejecutar un contenedor de demostración. Usar la -t bandera con docker build le permitirá etiquetar la imagen con su nombre de usuario de Docker Hub para que pueda enviarla a Docker Hub una vez que la haya probado.

Construya la imagen con el siguiente comando:

``` 
docker build -t your_dockerhub_username/node-demo .
``` 

El . comando especifica que el contexto de compilación es el directorio actual. Le hemos puesto un nombre a la imagen node-demo, pero eres libre de ponerle otro nombre.

Una vez que se completa el proceso de compilación, puede enumerar sus imágenes con docker images:

``` 
docker images
```

Verá el siguiente resultado que confirma la creación de la imagen:

``` 
Output
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/node-demo   latest              37f1c2939dbf        5 seconds ago       77.6MB
node                                10-alpine           9dfa73010b19        2 days ago          75.3MB
``` 

A continuación, utilizará docker run para crear un contenedor basado en esta imagen. Incluiremos tres banderas con este comando:

* **-p:** Esto publica el puerto en el contenedor y lo asigna a un puerto en nuestro host. Usaremos el puerto 80 en el host, pero no dude en modificarlo según sea necesario si tiene otro proceso ejecutándose en ese puerto. Para obtener más información sobre cómo funciona esto, consulte esta discusión en los documentos de Docker sobre enlace de puertos .
* **-d:** Esto ejecuta el contenedor en segundo plano.
* **--name:** Esto nos permite darle al contenedor un nombre personalizado.
* 
Ejecute el siguiente comando para construir el contenedor:

``` 
docker run --name node-demo -p 80:8080 -d your_dockerhub_username/node-demo
```

Inspeccione sus contenedores en ejecución con docker ps:

``` 
docker ps
``` 

Verá un resultado que confirma que el contenedor de su aplicación se está ejecutando.

Ahora puede visitar la IP de su servidor para probar su configuración. Su aplicación mostrará la siguiente página de inicio:http://your_server_ip

Ahora que ha probado la aplicación, puede detener el contenedor en ejecución. Úselo docker ps nuevamente para obtener su CONTAINER ID:

``` 
docker ps
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

Envíe la imagen de la aplicación a Docker Hub con el docker pushcomando . Recuerde reemplazarlo your_dockerhub_username con su propio nombre de usuario de Docker Hub:

``` 
docker push your_dockerhub_username/node-demo
``` 

Ahora tiene una imagen de la aplicación que puede extraer para ejecutar su aplicación con Kubernetes e Istio. A continuación, puede continuar con la instalación de Istio con Helm.


## 4. Instalar Istio con Helm

Aunque Istio ofrece diferentes métodos de instalación, la documentación recomienda utilizar Helm para maximizar la flexibilidad en la gestión de las opciones de configuración. Instalaremos Istio con Helm y nos aseguraremos de que el complemento Grafana esté habilitado para que podamos visualizar los datos de tráfico de nuestra aplicación.

Primero, agregue el repositorio de versiones de Istio:

``` 
helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.7/charts/
```

Esto le permitirá utilizar los gráficos de Helm en el repositorio para instalar Istio.

Comprueba que tienes el repositorio:

``` 
helm repo list
``` 

Deberías ver el istio.io repositorio en la lista:

``` 
Output
NAME            URL                                                                
stable          https://kubernetes-charts.storage.googleapis.com                   
local           http://127.0.0.1:8879/charts                                       
istio.io        https://storage.googleapis.com/istio-release/releases/1.1.7/charts/
``` 

A continuación, instale las definiciones de recursos personalizados (CRD) de Istio con el istio-init gráfico usando el helm install comando:

``` 
helm install --name istio-init --namespace istio-system istio.io/istio-init
Output
NAME:   istio-init
LAST DEPLOYED: Fri Jun  7 17:13:32 2019
NAMESPACE: istio-system
STATUS: DEPLOYED
...
``` 

Este comando envía 53 CRD a kube-api server, haciéndolos disponibles para su uso en la malla de Istio. También crea un espacio de nombres para los objetos de Istio llamados istio-system y utiliza la --name opción para nombrar la versión istio-init de Helm. Una versión en Helm se refiere a una implementación particular de un gráfico con opciones de configuración específicas habilitadas.

Para verificar que se hayan confirmado todos los CRD requeridos, ejecute el siguiente comando:

``` 
kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
``` 

Esto debería generar el número 53.

Ahora puede instalar el istio gráfico. Para asegurarnos de que el complemento de telemetría de Grafana esté instalado con el gráfico, usaremos la --set grafana.enabled=true opción de configuración con nuestro helm install comando. También usaremos el protocolo de instalación para nuestro perfil de configuración deseado: el perfil predeterminado. Istio tiene una serie de perfiles de configuración para elegir al instalar con Helm que le permiten personalizar los sidecars del plano de control y del plano de datos de Istio. El perfil predeterminado se recomienda para implementaciones de producción y lo usaremos para familiarizarnos con las opciones de configuración que usaríamos al pasar a producción.

Ejecute el siguiente helm install comando para instalar el gráfico:

``` 
helm install --name istio --namespace istio-system --set grafana.enabled=true istio.io/istio
```

``` 
Output
NAME:   istio
LAST DEPLOYED: Fri Jun  7 17:18:33 2019
NAMESPACE: istio-system
STATUS: DEPLOYED
...
``` 

Nuevamente, instalaremos nuestros objetos Istio en el istio-system espacio de nombres y nombraremos la versión; en este caso, istio.

Podemos verificar que los objetos de Servicio que esperamos para el perfil predeterminado se hayan creado con el siguiente comando:

``` 
kubectl get svc -n istio-system
``` 

Los Servicios que esperaríamos ver aquí incluyen istio-citadel, istio-galley, istio-ingressgateway, istio-pilot, istio-policy, istio-sidecar-injector, istio-telemetry y prometheus. También esperaríamos ver el grafana Servicio, ya que habilitamos este complemento durante la instalación:

``` 
Output
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                                                                                                                                      AGE
grafana                  ClusterIP      10.245.85.162    <none>            3000/TCP                                                                                                                                     3m26s
istio-citadel            ClusterIP      10.245.135.45    <none>            8060/TCP,15014/TCP                                                                                                                           3m25s
istio-galley             ClusterIP      10.245.46.245    <none>            443/TCP,15014/TCP,9901/TCP                                                                                                                   3m26s
istio-ingressgateway     LoadBalancer   10.245.171.39    174.138.125.110   15020:30707/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:30285/TCP,15030:31668/TCP,15031:32297/TCP,15032:30853/TCP,15443:30406/TCP   3m26s
istio-pilot              ClusterIP      10.245.56.97     <none>            15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       3m26s
istio-policy             ClusterIP      10.245.206.189   <none>            9091/TCP,15004/TCP,15014/TCP                                                                                                                 3m26s
istio-sidecar-injector   ClusterIP      10.245.223.99    <none>            443/TCP                                                                                                                                      3m25s
istio-telemetry          ClusterIP      10.245.5.215     <none>            9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       3m26s
prometheus               ClusterIP      10.245.100.132   <none>            9090/TCP                                                                                                                                     3m26s
```

También podemos verificar los Istio Pods correspondientes con el siguiente comando:

```
kubectl get pods -n istio-system
``` 

Los Pods correspondientes a estos servicios deben tener un STATUS of Running, que indica que los Pods están vinculados a nodos y que los contenedores asociados con los Pods se están ejecutando:

``` 
Output
NAME                                     READY   STATUS      RESTARTS   AGE
grafana-67c69bb567-t8qrg                 1/1     Running     0          4m25s
istio-citadel-fc966574d-v5rg5            1/1     Running     0          4m25s
istio-galley-cf776876f-5wc4x             1/1     Running     0          4m25s
istio-ingressgateway-7f497cc68b-c5w64    1/1     Running     0          4m25s
istio-init-crd-10-bxglc                  0/1     Completed   0          9m29s
istio-init-crd-11-dv5lz                  0/1     Completed   0          9m29s
istio-pilot-785694f946-m5wp2             2/2     Running     0          4m25s
istio-policy-79cff99c7c-q4z5x            2/2     Running     1          4m25s
istio-sidecar-injector-c8ddbb99c-czvwq   1/1     Running     0          4m24s
istio-telemetry-578b6f967c-zk56d         2/2     Running     1          4m25s
prometheus-d8d46c5b5-k5wmg               1/1     Running     0          4m25s
``` 

El READY campo indica cuántos contenedores se están ejecutando en un Pod. 

**Nota:** Si ve fases inesperadas en la STATUS columna, recuerde que puede solucionar problemas de sus Pods con los siguientes comandos:

``` 
kubectl describe pods your_pod -n pod_namespace
kubectl logs your_pod -n pod_namespace
``` 

El último paso en la instalación de Istio permitirá la creación de proxies Envoy, que se implementarán como complementos para los servicios que se ejecutan en la malla.

Los sidecars se suelen utilizar para agregar una capa adicional de funcionalidad en entornos de contenedores existentes. La arquitectura de malla de Istio se basa en la comunicación entre los sidecars de Envoy, que comprenden el plano de datos de la malla, y los componentes del plano de control. Para que la malla funcione, debemos asegurarnos de que cada Pod en la malla también ejecute un sidecar Envoy.

Hay dos formas de lograr este objetivo: inyección manual con sidecar e inyección automática con sidecar. Habilitaremos la inyección automática de sidecar etiquetando el espacio de nombres en el que crearemos los objetos de nuestra aplicación con la etiqueta istio-injection=enabled. Esto garantizará que el controlador MutatingAdmissionWebhook pueda interceptar solicitudes kube-api server y realizar una acción específica; en este caso, garantizará que todos los Pods de nuestra aplicación comiencen con un sidecar.

Usaremos el default espacio de nombres para crear los objetos de nuestra aplicación, por lo que aplicaremos la istio-injection=enabled etiqueta a ese espacio de nombres con el siguiente comando:

``` 
kubectl label namespace default istio-injection=enabled
``` 

Podemos verificar que el comando funcionó según lo previsto ejecutando:

``` 
kubectl get namespace -L istio-injection
``` 

Verá el siguiente resultado:

``` 
Output
AME              STATUS   AGE   ISTIO-INJECTION
default           Active   47m   enabled
istio-system      Active   16m   
kube-node-lease   Active   47m   
kube-public       Active   47m   
kube-system       Active   47m   
``` 

Con Istio instalado y configurado, podemos pasar a crear los objetos de Servicio e Implementación de nuestra aplicación.


## 5. Crear objetos de aplicación

Con la malla de Istio en su lugar y configurada para inyectar Pods sidecar, podemos crear un manifiesto de aplicación con especificaciones para nuestros objetos de Servicio e Implementación. Las especificaciones en un manifiesto de Kubernetes describen el estado deseado de cada objeto.

Nuestro Servicio de aplicación garantizará que los Pods que ejecutan nuestros contenedores permanezcan accesibles en un entorno dinámico, a medida que se crean y destruyen Pods individuales, mientras que nuestra Implementación describirá el estado deseado de nuestros Pods.

Abra un archivo llamado node-app.yaml con nanoo su editor favorito:

``` 
nano node-app.yaml
``` 

Primero, agregue el siguiente código para definir el nodejs Servicio de la aplicación:

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
``` 

Esta definición de Servicio incluye un selector que hará coincidir los Pods con la app nodejs etiqueta correspondiente. También hemos especificado que el Servicio se dirigirá al puerto 8080 de cualquier Pod con la etiqueta coincidente.

También estamos nombrando el puerto de Servicio, de conformidad con los requisitos de Istio para Pods y Servicios. El http valor es uno de los valores que Istio aceptará para el name campo.

A continuación, debajo del Servicio, agregue las siguientes especificaciones para la Implementación de la aplicación. Asegúrese de reemplazar la image que aparece en la containers especificación con la imagen que creó y envió a Docker Hub en el Paso 1:

``` 
...
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

Las especificaciones para esta Implementación incluyen el número de replicas(en este caso, 1), así como un selector que define qué Pods administrará la Implementación. En este caso gestionará los Pods con la app: nodejs etiqueta.

El template campo contiene valores que hacen lo siguiente:

* **Aplicar la app:** nodejs etiqueta a los Pods administrados por el Deployment. Istio recomienda agregar la app etiqueta a las especificaciones de implementación para proporcionar información contextual para las métricas y la telemetría de Istio.
* **Aplique una version etiqueta** para especificar la versión de la aplicación que corresponde a este Deployment. Al igual que con la app etiqueta, Istio recomienda incluir la version etiqueta para proporcionar información contextual.
* **Defina las especificaciones para los contenedores que ejecutarán los Pods**, incluidos el contenedor name y el archivo image. Aquí image está la imagen que creó en el Paso 1 y la envió a Docker Hub. Las especificaciones del contenedor también incluyen una container Port configuración para señalar el puerto en el que escuchará cada contenedor. Si los puertos permanecen sin enumerar aquí, omitirán el proxy de Istio. Tenga en cuenta que este puerto 8080, corresponde al puerto de destino nombrado en la definición del Servicio.
  
Guarde y cierre el archivo cuando haya terminado de editarlo.

Con este archivo en su lugar, podemos pasar a editar el archivo que contendrá definiciones para objetos de puerta de enlace y servicio virtual, que controlan cómo el tráfico ingresa a la malla y cómo se enruta una vez allí.


## 6. Crear objetos de Istio

Para controlar el acceso a un clúster y el enrutamiento a los servicios, Kubernetes utiliza controladores y recursos de ingreso. Los recursos de entrada definen reglas para el enrutamiento HTTP y HTTPS a los servicios del clúster, mientras que los controladores equilibran la carga del tráfico entrante y lo enrutan a los servicios correctos.

Istio utiliza un conjunto diferente de objetos para lograr fines similares, aunque con algunas diferencias importantes. En lugar de utilizar un controlador para equilibrar la carga del tráfico, la malla de Istio utiliza una puerta de enlace, que funciona como un equilibrador de carga que maneja las conexiones HTTP/TCP entrantes y salientes. Luego, la puerta de enlace permite que se apliquen reglas de monitoreo y enrutamiento al tráfico que ingresa a la malla. En concreto, la configuración que determina el enrutamiento del tráfico se define como Servicio Virtual. Cada Servicio Virtual incluye reglas de enrutamiento que coinciden con criterios con un protocolo y destino específicos.

Aunque los recursos/controladores de ingreso de Kubernetes y las puertas de enlace/servicios virtuales de Istio tienen algunas similitudes funcionales, la estructura de la malla introduce diferencias importantes. Los controladores y recursos de ingreso de Kubernetes ofrecen a los operadores algunas opciones de enrutamiento, por ejemplo, pero las puertas de enlace y los servicios virtuales ofrecen un conjunto más sólido de funcionalidades, ya que permiten que el tráfico ingrese a la malla. En otras palabras, las capacidades limitadas de la capa de aplicación que los controladores y recursos de ingreso de Kubernetes ponen a disposición de los operadores de clústeres no incluyen las funcionalidades (incluidos enrutamiento, seguimiento y telemetría avanzados) proporcionadas por los sidecars en la malla de servicios de Istio.

Para permitir el tráfico externo en nuestra malla y configurar el enrutamiento a nuestra aplicación Node, necesitaremos crear una puerta de enlace Istio y un servicio virtual. Abra un archivo llamado node-istio.yaml para el manifiesto:

``` 
nano node-istio.yaml
``` 

Primero, agregue la definición para el objeto Gateway:

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
``` 

Además de especificar un name para la puerta de enlace en el metadatacampo, hemos incluido las siguientes especificaciones:

* Un **selector** que hará coincidir este recurso con el controlador Istio IngressGateway predeterminado que se habilitó con el perfil de configuración que seleccionamos al instalar Istio.
* Una **servers especificación** que especifica lo que portse va a exponer para el ingreso y el host expuesto por la puerta de enlace. En este caso, especificamos todo hosts con un asterisco ( *) ya que no estamos trabajando con un dominio seguro específico.
  
Debajo de la definición de Gateway, agregue especificaciones para el Servicio Virtual:

``` 
...
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

Además de proporcionar un name servicio virtual, también incluimos especificaciones para este recurso que incluyen:

* Un **hosts campo** que especifica el host de destino. En este caso, volvemos a utilizar un valor comodín ( *) para permitir el acceso rápido a la aplicación en el navegador, ya que no estamos trabajando con un dominio.
* Un **gateways campo** que especifica la puerta de enlace a través de la cual se permitirán las solicitudes externas. En este caso, es nuestro nodejs-gateway Gateway.
* El **http campo** que especifica cómo se enrutará el tráfico HTTP.
* Un **destination campo** que indica dónde se enrutará la solicitud. En este caso, se enrutará al nodejs servicio, que implícitamente se expande al nombre de dominio completo (FQDN) del servicio en un entorno de Kubernetes: nodejs.default.svc.cluster.local. Sin embargo, es importante tener en cuenta que el FQDN se basará en el espacio de nombres donde se define la regla, no en el Servicio, así que asegúrese de usar el FQDN en este campo cuando el Servicio de su aplicación y el Servicio virtual estén en espacios de nombres diferentes.
  
Guarde y cierre el archivo cuando haya terminado de editarlo.

Con sus yaml archivos en su lugar, puede crear el servicio y la implementación de su aplicación, así como los objetos de puerta de enlace y servicio virtual que permitirán el acceso a su aplicación.


## 7. Crear recursos de aplicaciones y habilitar el acceso a telemetría

Una vez que haya creado los objetos de servicio e implementación de su aplicación, junto con una puerta de enlace y un servicio virtual, podrá generar algunas solicitudes para su aplicación y ver los datos asociados en sus paneles de Istio Grafana. Sin embargo, primero deberá configurar Istio para exponer el complemento Grafana y poder acceder a los paneles en su navegador.

Habilitaremos el acceso a Grafana con HTTP, pero cuando trabaje en producción o en entornos sensibles, se recomienda encarecidamente que habilite el acceso con HTTPS.

Debido a que configuramos la --set grafana.enabled=true opción de configuración al instalar Istio en el Paso 2, tenemos un servicio y un pod de Grafana en nuestro istio-system espacio de nombres, lo cual confirmamos en ese paso.

Con esos recursos ya implementados, nuestro siguiente paso será crear un manifiesto para una puerta de enlace y un servicio virtual para que podamos exponer el complemento Grafana.

Abra el archivo para el manifiesto:

``` 
nano node-grafana.yaml
``` 

Agregue el siguiente código al archivo para crear una puerta de enlace y un servicio virtual para exponer y enrutar el tráfico al servicio Grafana:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: grafana-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 15031
      name: http-grafana
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: grafana-vs
  namespace: istio-system
spec:
  hosts:
  - "*"
  gateways:
  - grafana-gateway
  http:
  - match:
    - port: 15031
    route:
    - destination:
        host: grafana
        port:
          number: 3000
``` 

Nuestras especificaciones de Grafana Gateway y Virtual Service son similares a las que definimos para nuestra aplicación Gateway y Virtual Service en el Paso 4. Sin embargo, existen algunas diferencias:

* Grafana se expondrá en el http-grafana puerto designado (puerto 15031) y se ejecutará en el puerto 3000 del host.
* Tanto la puerta de enlace como el servicio virtual están definidos en el istio-system espacio de nombres.
* En host este Servicio Virtual es el grafana Servicio en el istio-system espacio de nombres. Dado que estamos definiendo esta regla en el mismo espacio de nombres en el que se ejecuta el servicio Grafana, la expansión FQDN volverá a funcionar sin conflictos.

Guarde y cierre el archivo cuando haya terminado de editarlo.

Crea tus recursos de Grafana con el siguiente comando:

``` 
kubectl apply -f node-grafana.yaml
``` 

El kubectl apply comando le permite aplicar una configuración particular a un objeto en el proceso de creación o actualización. En nuestro caso, estamos aplicando la configuración que especificamos en el node-grafana.yaml archivo a nuestros objetos Gateway y Servicio virtual en el proceso de creación.

Puede echar un vistazo a la puerta de enlace en el istio-systemespacio de nombres con el siguiente comando:

``` 
kubectl get gateway -n istio-system
``` 

Verá el siguiente resultado:

``` 
Output
NAME              AGE
grafana-gateway   47s
``` 

Puedes hacer lo mismo para el Servicio Virtual:

``` 
kubectl get virtualservice -n istio-system
Output
NAME         GATEWAYS            HOSTS   AGE
grafana-vs   [grafana-gateway]   [*]     74s
``` 

Con estos recursos creados, deberíamos poder acceder a nuestros paneles de Grafana en el navegador. Sin embargo, antes de hacer eso, creemos nuestra aplicación Servicio e Implementación, junto con nuestra aplicación Puerta de Enlace y Servicio Virtual, y verifiquemos que podemos acceder a nuestra aplicación en el navegador.

Cree la aplicación Servicio e Implementación con el siguiente comando:

``` 
kubectl apply -f node-app.yaml
``` 

Espere unos segundos y luego verifique los Pods de su aplicación con el siguiente comando:

``` 
kubectl get pods
Output
NAME                      READY   STATUS    RESTARTS   AGE
nodejs-7759fb549f-kmb7x   2/2     Running   0          40s
``` 

Los contenedores de su aplicación se están ejecutando, como puede ver en la STATUS columna, pero ¿por qué aparece la READY columna 2/2 si el manifiesto de la aplicación del Paso 3 solo especifica 1 réplica?

Este segundo contenedor es el sidecar Envoy, que puedes inspeccionar con el siguiente comando. Asegúrese de reemplazar el pod que aparece aquí con el NAME de su propio nodejs Pod:

``` 
kubectl describe pod nodejs-7759fb549f-kmb7x
``` 

``` 
Output
Name:               nodejs-7759fb549f-kmb7x
Namespace:          default
...
Containers:
  nodejs:
  ...
  istio-proxy:
    Container ID:  docker://f840d5a576536164d80911c46f6de41d5bc5af5152890c3aed429a1ee29af10b
    Image:         docker.io/istio/proxyv2:1.1.7
    Image ID:      docker-pullable://istio/proxyv2@sha256:e6f039115c7d5ef9c8f6b049866fbf9b6f5e2255d3a733bb8756b36927749822 
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
    ...
``` 

A continuación, cree su aplicación Gateway y Servicio Virtual:

``` 
kubectl apply -f node-istio.yaml
``` 

Puede inspeccionar la puerta de enlace con el siguiente comando:

```
kubectl get gateway
Output
NAME             AGE
nodejs-gateway   7s
``` 

Y el Servicio Virtual:

``` 
kubectl get virtualservice
Output
NAME     GATEWAYS           HOSTS   AGE
nodejs   [nodejs-gateway]   [*]     28s
``` 

Ahora estamos listos para probar el acceso a la aplicación. Para ello necesitaremos la IP externa asociada a nuestro istio-ingress gateway Servicio, que es de tipo Servicio LoadBalancer .

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
prometheus               ClusterIP      10.245.100.132   <none>            9090/TCP                                                                                                                                     42m
```

Debe istio-ingressgateway ser el único Servicio con TYPE LoadBalancer y el único Servicio con una IP externa.

Navegue a esta IP externa en su navegador: .http://ingressgateway_ip

A continuación, genere algo de carga en el sitio haciendo clic en actualizar cinco o seis veces.

Ahora puede consultar el panel de Grafana para ver los datos de tráfico.

En su navegador, navegue hasta la siguiente dirección, nuevamente usando su istio-ingress gateway IP externa y el puerto que definió en su manifiesto de Grafana Gateway: .http://ingressgateway_ip:15031

Verá la siguiente página de inicio con Home Dashboard de Grafana.

Al hacer clic en Inicio en la parte superior de la página, accederá a una página con una carpeta istio. Para obtener una lista de opciones desplegables, haga clic en el icono de la carpeta istio.

En esta lista de opciones, haga clic en Istio Service Dashboard.

Esto lo llevará a una página de destino con otro menú desplegable.

Seleccione nodejs.default.svc.cluster.local de la lista de opciones disponibles.

Ahora podrá ver los datos de tráfico de ese servicio.

Ahora tiene una aplicación Node.js en funcionamiento ejecutándose en una malla de servicios de Istio con Grafana habilitado y configurado para acceso externo.

## 8. Conclusión

En este tutorial, instaló Istio usando el administrador de paquetes Helm y lo usó para exponer un servicio de aplicación Node.js usando objetos de puerta de enlace y servicio virtual. También configuró objetos de puerta de enlace y servicio virtual para exponer el complemento de telemetría de Grafana, a fin de ver los datos de tráfico de su aplicación.


