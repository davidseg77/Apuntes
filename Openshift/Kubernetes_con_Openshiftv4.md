# Info del curso de Kubernetes con Openshift V4 de OW

## 1.Instalación de la herramienta OC

Lo descargamos para el SO requerido y en la terminal nos aseguramos de contar con el archivo. Hecho esto, lo descomprimimos.

```
tar xvf oc.tar
```

Despues llevamos el archivo binario oc al path donde nos permitirá que pueda ser ejecutado:

```
sudo install oc /usr/local/bin
```

Para comprobarlo

```
oc version
```

## 2. Configuración de oc para Developer Sandbox

Habremos de autenticarnos mediante el token proporcionado por Openshift. Lo hallamos en la parte superior a la derecha. Esto nos dará el token junto con el comando que permite su inserción:

```
oc login --token=xxxx --server=xxxxx
```

Para ver aspectos como el usuario, el servidor... en el cual estamos trabajando podemos verlo en .kube/config.
Y para ver los proyectos que tenemos activos:

```
oc get project
```

Para ver los namespaces que tenemos en nuestros proyectos

```
oc get ns
```

## 3. CRC

### Instalación en local

Instalamos los paquetes kvm. Estos comandos vienen en la docu de instalar CRC en ubuntu. 

```
sudo apt install qemu-kvm libvirt-daemon libvirt-daemon-system network-manager
```

Hemos de asegurarnos de que el usuario con el que estemos ejecutando estos pasos pertenezca al grupo libvirt:

```
sudo adduser usuario libvirt
```

A continuación tienes que bajarte la última versión de CRC, eligiendo la versión del sistema operativo que estés usando, desde la página oficial de descarga (para ello tendrás que hacer login con un cuenta de Red Hat): https://console.redhat.com/openshift/create/local

Además de bajarte la versión de CRC, tendrás que bajarte o copiar en el portapapeles un token que durante la instalación tendrás que introducir. (Download pull secret)

Una vez descargado el paquete lo descomprimimos y lo copiamos a un directorio del PATH para poder ejecutarlo:

```
tar -xf crc-linux-amd64.tar.xz
cd crc-linux-2.17.0-amd64
sudo install crc /usr/local/bin

crc version
CRC version: 2.17.0+44e15711
OpenShift version: 4.12.9
Podman version: 4.4.1
```

Para preparar el entorno, creando entre otras cosas la red para kvm.

```
crc setup
```

A continuación, aunque no es necesario, si necesitamos aumentar los recursos de la máquina virtual que vamos a crear podemos hacerlo de la siguiente manera:

```
crc config set cpus 8
crc config set memory 20480
```

Esto también es opcional, pero si queremos que durante la instalación se instalen los componentes de telemetría para mostrar las métricas de los recursos que se están utilizando, debes realizar la siguiente configuración:

```
crc config set enable-cluster-monitoring true
```

Finalmente creamos la máquina virtual, que ejecutará el clúster de OpenShift v4:

```
crc start
```

Durante el proceso nos pedirán que introduzcamos el token que hemos bajado o copiado:

? Please enter the pull secret  

Después de unos minutos, el clúster estará preparado y nos dará información para acceder.

No es necesario instalar la herramienta oc, durante el proceso de instalación se ha descargado, lo único que tenemos que hacer es configurar el PATH para que podamos acceder a ella, para ello ejecutamos:

```
eval $(crc oc-env)
```

### Algunos detalles de la instalación

Como hemos dicho, todos los ficheros relacionados con CRC se guardan en el directorio ~/.crc. La configuración de acceso al clúster, al igual que en kubernetes se guarda en el fichero ~/.kube/config. En nuestro caso:

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tL..
    server: https://api.crc.testing:6443
  name: api-crc-testing:6443
contexts:
- context:
    cluster: api-crc-testing:6443
    namespace: default
    user: kubeadmin
  name: crc-admin
- context:
    cluster: api-crc-testing:6443
    namespace: default
    user: developer
  name: crc-developer
current-context: crc-admin
kind: Config
preferences: {}
users:
- name: developer
  user:
    token: sha256~gEE6O8LrHV444o44W6kryQSR8pDGKMxnNdkblvX1P9M
- name: kubeadmin
  user:
    token: sha256~qdvyZgGGYo32tdpGh1adh8eu_-NaP5ESgoJTmD2LA1Y
```

### Algunos comando útiles de crc

Cada vez que empecemos a utilizar CRC iniciamos la máquina virtual con:

```
crc start
```

Cuando terminemos de trabajar, paramos la máquina con:

```
crc stop
```

Si necesitamos hacer una nueva instalación y eliminar la máquina virtual, ejecutamos:

```
crc delete
```

Podemos acceder a la consola web usando la URL https://console-openshift-console.apps-crc.testing o ejecutar el comando:

```
crc console
```

Si queremos la información para loguearnos con el comando oc podemos ejecutar:

```
crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p xxxxxxxxxxxxxxx https://api.crc.testing:6443'
```

### Configuración de oc para CRC

Por defecto la instalación de OpenShift en local no tiene ningún Operador instalado. Los Operadores nos permiten instalar componentes internos de OpenShift que añaden funcionalidades extras a nuestro clúster.

Para poder conectarnos a un terminal desde la consola web y tener a nuestra disposición la herramienta oc tenemos que instalar el operador WebTerminal (por dependencias se instará también el operador DevWorkspace Operator). Nos tenemos que conectar con el usuario administrador kubeadmin y en la vista Administrator accedemos a la opción Operators->OperatorHub y filtramos con el nombre del operador "WebTerminal"

Nos aparece una ventana con información del operador y pulsamos sobre el botón Install para comenzar la instalación, dejamos los valores por defecto, realizamos la instalación y comprobamos los operadores que hemos instalados en la opción Operators->Installed Operators.

### Consola web CRC

Para acceder a la consola web, usamos la URL: https://console-openshift-console.apps-crc.testing y nos pide que hagamos login.

Usamos el usuario developer o el usuario kubeadmin para acceder con un usuario normal o un usuario administrador. Las opciones serán las mismas, pero como hemos visto el usuario developer no tendrá acceso a algunos recursos.

También se puede acceder mediante:

```
crc console
```

## 4. Proyectos en Openshift

Un proyecto en OpenShift v4 es una agrupamiento lógico de recursos. En realidad es muy parecido al recurso namespace de Kubernetes pero puede guardar información adicional. Si eliminamos un proyecto se eliminarán todos los recursos que hemos creado en él.

Vamos a seguir trabajando con el usuario developer, que hemos usado para acceder al clúster de OpenShift:

```
oc login -u developer -p developer https://api.crc.testing:6443
```

Por lo tanto lo primero que vamos a hacer es crear un nuevo proyecto que utilizará este usuario:

```
oc new-project developer
```

Vemos que ahora está usando el proyecto developer. Para obtener la lista de proyectos disponibles:

```
oc get projects
```

Y para ver los detalles del proyecto, ejecutamos:

```
oc describe project developer
```

Si creamos un nuevo proyecto:

```
oc new-project developer2
```

Nos podemos posicionar en uno de ellos, ejecutando:

```
oc project developer
```

### Proyectos y namespaces

Como hemos comentado un objeto project es en realidad un namespace que puede guardar más información. De hecho, cada vez que creamos un proyecto se creará un namespace. Sin embargo el usuario developer no tiene permiso para acceder a los namespaces:

```
oc get ns
Error from server (Forbidden): namespaces is forbidden: User "developer" cannot list resource "namespaces" in API group "" at the cluster scope
```

Vamos a conectarnos como administrador para poder acceder a los namespaces:

```
oc login -u kubeadmin https://api.crc.testing:6443
```

El administrador tiene acceso a 68 proyectos, por defecto, está usando el último que se ha creado. Podemos ver todos los proyectos, ejecutando:

```
oc get projects
```

Ahora si podemos acceder a los namespaces, ejecutando:

```
oc get ns
```

Finalmente para borrar un proyecto, ejecutamos:

```
oc delete project developer2
```

## 5. OpenShift como distribución de Kubernetes

### Trabajando con pods

OpenShift configura por defecto una política de seguridad que sólo nos permite ejecutar contenedores no privilegiados, es decir, donde no se ejecuten procesos o acciones por el usuario con privilegio root (por ejemplo, no utilizan puertos no privilegiados, puertos menores a 1024, o no ejecuta operaciones con el usuario root).

Por esta razón, la mayoría de las imágenes que encontramos en el registro Docker Hub no funcionarán en OpenShift. Es por ello que es necesario usar imágenes de contenedores no privilegiados, por ejemplo creadas con imágenes constructoras (images builder) del propio OpenShift. En nuestro caso, en estos primeros ejemplos, vamos a usar imágenes generadas por la empresa Bitnami, que ya están preparadas para su ejecución en OpenShift.

De forma imperativa podríamos crear un Pod, ejecutando:

```
oc run pod-nginx --image=bitnami/nginx
```

Pero, normalmente necesitamos indicar más parámetros en la definición de los recursos. Además, sería deseable crear el Pod de forma declarativa, es decir indicando el estado del recurso que queremos obtener. Para ello, definiremos el recurso, en nuestro caso el Pod, en un fichero de texto en formato YAML. Por ejemplo, podemos tener el fichero pod.yaml:

```
apiVersion: v1 
kind: Pod 
metadata: 
 name: pod2-nginx 
 labels:
   app: nginx
   service: web
spec: 
 containers:
   - image: bitnami/nginx
     name: contenedor-nginx
     imagePullPolicy: Always
```

Veamos cada uno de los parámetros que hemos definido:

* apiVersion: v1: La versión de la API que vamos a usar.
* kind: Pod: La clase de recurso que estamos definiendo.
* metadata: Información que nos permite identificar unívocamente el recurso:
 -  name: Nombre del pod.
 -  labels: Las Labels nos permiten etiquetar los recursos de OpenShift (por ejemplo un pod) con información del tipo clave/valor.
* spec: Definimos las características del recurso. En el caso de un Pod indicamos los contenedores que van a formar el Pod (sección containers), en este caso sólo uno.
 - image: La imagen desde la que se va a crear el contenedor
 - name: Nombre del contenedor.
 - imagePullPolicy: Si creamos un contenedor, necesitamos tener descargada en un registro interno la imagen. Existe una política de descarga de estas imágenes:
  - La política por defecto es IfNotPresent, que se baja la imagen si no está en el registro interno.
  - Si queremos forzar la descarga desde el repositorio externo, indicaremos el valor Always.
  - Si estamos seguro que la imagen esta en el registro interno, y no queremos bajar la imagen del registro externo, indicamos el valor Never.
Ahora para crear el Pod a partir del fichero YAML, podemos usar dos subcomandos:

* create: Configuración imperativa de objetos, creamos el objeto (en nuestro caso el pod) pero si necesitamos modificarlo tendremos que eliminarlo y volver a crearlo después de modificar el fichero de definición.

```
  oc create -f pod.yaml
``` 

* apply: Configuración declarativa de objetos, el fichero indica el estado del recurso que queremos tener. Al aplicar los cambios, se realizarán todas las acciones necesarias para llegar al estado indicado. Por ejemplo, si no existe el objeto se creará, pero si existe el objeto, se modificará.

```
  oc apply -f pod.yaml
```

**Otras operaciones con Pods**

Podemos ver el estado en el que se encuentra y si está o no listo:

```
oc get pod
```

Si queremos ver más información sobre los Pods, como por ejemplo, saber en qué nodo del cluster se está ejecutando:

```
oc get pod -o wide
```

Para obtener información más detallada del Pod:

```
oc describe pod pod-nginx
```

Podríamos editar el Pod y ver todos los atributos que definen el objeto, la mayoría de ellos con valores asignados automáticamente por el propio OpenShift y podremos actualizar ciertos valores:

```
oc edit pod pod-nginx
```

Normalmente no se interactúa directamente con el Pod a través de una shell, pero sí se obtienen directamente los logs al igual que se hace en docker:

```
oc logs pod/pod-nginx
```

En el caso poco habitual de que queramos ejecutar alguna orden adicional en el Pod, podemos utilizar el comando exec, por ejemplo, en el caso particular de que queremos abrir una shell de forma interactiva:

```
oc exec -it pod-nginx -- /bin/bash
```

Otra manera de acceder al pod:

```
oc rsh pod-nginx
```

Podemos acceder a la aplicación, redirigiendo un puerto de localhost al puerto de la aplicación:

```
oc port-forward pod-nginx 8080:8080
```

Y accedemos al servidor web en la url http://localhost:8080.

Para obtener las etiquetas de los Pods que hemos creado:

```
oc get pod --show-labels
```

Las etiquetas las hemos definido en la sección metadata del fichero YAML, pero también podemos añadirlos a los Pods ya creados:

```
oc label pod pod-nginx service=web --overwrite=true
```

Las etiquetas son muy útiles, ya que permiten seleccionar un recurso determinado (en el clúster puede haber cientos o miles de objetos). Por ejemplo para visualizar los Pods que tienen una etiqueta con un determinado valor:

```
oc get pod -l service=web
```

También podemos visualizar los valores de las etiquetas como una nueva columna:

```
oc get pod -Lservice
```

Y por último, eliminamos el Pod mediante:

```
oc delete pod pod-nginx
```

### Pod multicontenedor

La razón principal por la que los Pods pueden tener múltiples contenedores es para admitir aplicaciones auxiliares que ayudan a una aplicación primaria. Ejemplos típicos de estas aplicaciones pueden ser las que envían o recogen datos externos (por ejemplo de un repositorio) y los servidores proxy. El ayudante y las aplicaciones primarias a menudo necesitan comunicarse entre sí. Normalmente, esto se realiza a través de un sistema de archivos compartido o mediante la interfaz loopback (localhost).

Veamos dos ejemplos concretos:

Un servidor web junto con un programa auxiliar que sondea un repositorio Git en busca de nuevas actualizaciones.
Un servidor web con un servidor de aplicaciones PHP-FPM, lo podemos implementar en un Pod, y cada servicio en un contenedor. Además tendría un volumen interno que se montaría en el DocumentRoot para que el servidor web y el servidor de aplicaciones puedan acceder a la aplicación.
Veamos un pequeño ejemplo de un pod multicontenedor. Tenemos la definición del Pod en el fichero pod-multicontenedor.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-multicontenedor
spec:
  volumes:
  - name: html
    emptyDir: {}
  containers:
  - name: contenedor1
    image: bitnami/nginx
    volumeMounts:
    - name: html
      mountPath: /app
  - name: contenedor2
    image: debian
    volumeMounts:
    - name: html
      mountPath: /html
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date >> /html/index.html;
          sleep 1;
        done
```

Estudiemos la definición del Pod:

* El Pod se llama pod-multicontenedor y en el apartado spec vemos que está formado por un volumen (llamado html y de tipo emptyDir, que estudiaremos más adelante, pero que básicamente es un directorio que vamos a montar en los contenedores) y dos contenedores (llamados contenedor1 y contenedor2).
  
* El contenedor1 se crea a partir de la imagen bitnami/nginx, es el contenedor principal, encargado de servir la web. En este contenedor montamos el volumen html en su DocumentRoot (/app). Va a servir el fichero index.html que está modificando el otro contenedor.
  
* El contenedor2 es el auxiliar. En este caso se monta el volumen html en el directorio html donde va modificando el fichero index.html con la fecha y hora actuales cada un segundo (parámetro command y args).
Como los dos contenedores tienen montado el volumen, el fichero index.html que va modificando el contenedor2, es el fichero que sirve el contenedor1.

Vamos a realizar los siguientes pasos:

Creamos el Pod.

```
 oc apply -f pod-multicontenedor.yaml
```

Veamos la información del pod y vemos que está formado por dos contenedores y un volumen:

```
 oc describe pod pod-multicontenedor
```

Podemos acceder desde la consola web al detalle del pod, y vemos también la misma información.

Podemos acceder al primer contenedor para ver el contenido del fichero index.html:

```
 oc exec pod-multicontenedor -c contenedor1 -- /bin/cat /app/index.html
```

En esta ocasión hay que indicar el contenedor (opción -c) para indicar donde vamos a ejecutar la instrucción.

Para mostrar el contenido del fichero index.html en el segundo contenedor, ejecutamos:

```
 oc exec pod-multicontenedor -c contenedor2 -- /bin/cat /html/index.html
```

Si queremos acceder a un contenedor determinado o ver los logs de un contenedor hay que indicar el contenedor. Por ejemplo:

```
 oc logs pod/pod-multicontenedor -c contenedor1
 oc exec -it pod/pod-multicontenedor -c contenedor1 -- bash
 oc rsh -c contenedor2 pod/pod-multicontenedor 
```

Podemos ejecutar un "port forward" para acceder al Pod en el puerto 8080 de localhost, sabiendo que el servicio usa el puerto 8080.

```
 oc port-forward pod/pod-multicontenedor 8080:8080
```

### Tolerancia a fallos, escalabilidad, balanceo de carga: ReplicaSet

Para trabajar con los objetos ReplicaSet vamos a seguir trabajando con el usuario developer pero vamos a crear un nuevo proyecto:

```
oc new-project proyecto-rs
```

En este caso también vamos a definir el recurso de ReplicaSet en un fichero replicaset.yaml, por ejemplo como este:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: bitnami/nginx
          name: contenedor-nginx
```

Algunos de los parámetros definidos ya lo hemos estudiado en la definición del Pod. Los nuevos parámetros de este recurso son los siguientes:

* replicas: Indicamos el número de Pods que siempre se deben estar ejecutando.
* selector: Seleccionamos los Pods que va a controlar el ReplicaSet por medio de las etiquetas. Es decir este ReplicaSet controla los Pods cuya etiqueta app es igual a nginx.
* template: El recurso ReplicaSet contiene la definición de un Pod. Fíjate que el Pod que hemos definido en la sección template tiene indicado la etiqueta necesaria para que sea seleccionado por el ReplicaSet (app: nginx).

**Creación del ReplicaSet**

De forma declarativa creamos el ReplicaSet ejecutando:

```
oc apply -f replicaset.yaml
```

Y podemos ver los recursos que se han creado con:

```
oc get rs,pod
```

Observamos que queríamos crear 2 replicas del Pod, y efectivamente se han creado.

Si queremos obtener información detallada del recurso ReplicaSet que hemos creado:

```
oc describe rs replicaset-nginx
```

**Política de seguridad del Pod**

Al crear el RepicaSet nos da un warning, de este estilo:

Warning: would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "httpd" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "httpd" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "httpd" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "httpd" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")

Este warning indica que el contenedor contenedor-nginx no cumple con algunas de las restricciones de seguridad establecidas en la política de seguridad.

**Solución 1: Actualizar la definición del Pod para indicar el contento de seguridad**

Para resolver este warning, debe actualizar la definición del Pod o del contenedor para cumplir con estas restricciones de seguridad.

```
    spec:
      containers:
        - image: bitnami/nginx
          name: contenedor-nginx
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            seccompProfile:
              type: RuntimeDefault
            capabilities:
              drop:
              - ALL
```

**Solución 2: Otorgar privilegios para ejecutar Pod privilegiados**

Muchas de las interacciones que se hacen sobre la API de OpenShift se realizan por el usuario final, pero muchas otras se hacen internamente. Para hacer estas últimas peticiones a la API se utiliza una cuenta especial de usuario, que se llaman Service Account.

OpenShift crea automáticamente algunas Service Account en cada proyecto. Por ejemplo, hay una Service Account que se llama default y será la responsable de ejecutar los Pods.

Vamos a modificar los privilegios de ejecución al Service Account llamado default, para ello:

```
oc login -u kubeadmin https://api.crc.testing:6443
oc project proyecto-rs
oc adm policy add-scc-to-user anyuid -z default
```

Esta instrucción agrega el restricción de seguridad llamada anyuid al ServiceAccount predeterminado (default) en tu proyecto actual de OpenShift.
La restricción anyuid permite a los contenedores en el Pod ejecutarse con privilegios.
Esta segunda opción la utilizaremos más adelante para poder ejecutar Pods privilegiados.

**Tolerancia a fallos**

¿Qué pasaría si borro uno de los Pods que se han creado? Inmediatamente se creará uno nuevo para que siempre estén ejecutándose los Pods deseados, en este caso 2:

```
oc delete pod <nombre_del_pod>
oc get pod
```

**Escalabilidad**

Para escalar el número de Pods:

```
oc scale rs replicaset-nginx --replicas=5
oc get pod
```

Otra forma de hacerlo sería cambiando el parámetro replicas de fichero YAML, y volviendo a ejecutar:

```
oc apply -f nginx-rs.yaml
```

La escalabilidad puede ser para aumentar el número de Pods o para reducirla:

```  
oc scale rs replicaset-nginx --replicas=1
```

**Eliminando el ReplicaSet**

Por último, si borramos un ReplicaSet se borrarán todos los Pods asociados:

```
oc delete rs replicaset-nginx
```

Otra forma de borrar el recurso, es utilizar el fichero YAML:

```
oc delete -f replicaset.yaml
```

### Desplegando aplicaciones: Deployment

Para trabajar con los objetos Deployment vamos a seguir trabajando con el usuario developer pero vamos a crear un nuevo proyecto:

```
oc new-project proyecto-deploy
```

Podemos crear un Deployment de forma imperativa utilizando un comando como el siguiente:

```
oc create deployment nginx --image bitnami/nginx
```

Nosotros, sin embargo, vamos a seguir describiendo los recursos en un fichero YAML. En este caso para describir un Deployment de nginx podemos escribir un fichero deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: bitnami/nginx
        name: contendor-nginx
        ports:
        - name: http
          containerPort: 8080
```

La creación de un Deployment crea un ReplicaSet y los Pods correspondientes. Por lo tanto en la definición de un Deployment se define también el ReplicaSet asociado (los parámetros replicas, selector y template). Los atributos relacionados con el Deployment que hemos indicado en la definición son:

* revisionHistoryLimit: Indicamos cuántos ReplicaSets antiguos deseamos conservar, para poder realizar rollback a estados anteriores. Por defecto, es 10.
* strategy: Indica el modo en que se realiza una actualización del Deployment. Es decir, cuando modificamos la versión de la imagen del Deployment, se crea un ReplicaSet nuevo y ¿qué hacemos con los Pods?:
  
 - Recreate: elimina los Pods antiguos y crea los nuevos.
 - RollingUpdate: va creando los nuevos Pods, comprueba que funcionan y se eliminan los antiguos; es la opción por defecto.
  
Además, hemos introducido un nuevo parámetro al definir el contenedor del pod: con el parámetro ports hemos indicado el puerto que expone el contenedor (containerPort) y le hemos asignado un nombre (name).

Para crear el deployment desde el dichero, ejecutamos:

```
oc apply -f deployment.yaml
oc get deploy,rs,pod
```

Para ver los recursos que hemos creado también podemos utilizar la instrucción:

```
oc get all
```

**Política de seguridad del Pod**

Al crear el Deployment nos da un warning igual que en el apartado anterior, al crear un ReplicaSet. De la misma manera que vimos en el apartado anterior, tenemos dos posibles soluciones:

Actualizar la definición del Pod para indicar el contexto de seguridad.

Otorgar privilegios para ejecutar Pod privilegiados, para ello:

```
 oc login -u kubeadmin https://api.crc.testing:6443
 oc project proyecto-deploy
 oc adm policy add-scc-to-user anyuid -z default
```

**Escalado de los Deployments**

Como ocurría con los ReplicaSets, los Deployments también se pueden escalar, aumentando o disminuyendo el número de Pods asociados. Al escalar un Deployment estamos escalando el ReplicaSet asociado en ese momento:

```
oc scale deployment/deployment-nginx --replicas=4
```

**Otras operaciones**

Si queremos acceder a la aplicación, podemos utilizar la opción de port-forward sobre el despliegue (de nuevo recordamos que no es la forma adecuada para acceder a un servicio que se ejecuta en un Pod, pero de momento no tenemos otra). En este caso si tenemos asociados más de un Pod, la redirección de puertos se hará sobre un solo Pod (no habrá balanceo de carga):

```
oc port-forward deployment/deployment-nginx 8080:80
```

Si queremos ver los logs generados en los Pods de un Deployment:

```
oc logs deployment/deployment-nginx
```

Si queremos obtener información detallada del recurso Deployment que hemos creado:

```
oc describe deployment/deployment-nginx
```

**Eliminando el Deployment**

Si eliminamos el Deployment se eliminarán el ReplicaSet asociado y los Pods que se estaban gestionando.

```
oc delete deployment/deployment-nginx
```

O también, usando el fichero:

```
oc delete -f deployment.yaml
```

### Ejecución de Pods privilegiados

Como hemos indicado anteriormente, OpenShift configura por defecto una política de seguridad que sólo nos permite ejecutar contenedores no privilegiados, es decir, donde no se ejecuten procesos o acciones por el usuario con privilegio root (por ejemplo, no utilizan puertos no privilegiados, puertos menores a 1024, o no ejecuta operaciones con el usuario root).

Hemos estado usando imágenes proporcionada por la empresa Bitnami, que están preparada para ejecutar contenedores no privilegiados. En este apartado vamos a configurar la seguridad de nuestro proyecto para permitir la ejecución de Pods privilegiados, y de esta manera poder usar todas las imágenes que encontramos en Docker Hub.

Vamos a trabajar con el usuario developer, creando un nuevo proyecto:

```
oc new-project proyecto-nginx
```

Veamos un ejemplo, si tenemos un fichero de despliegue donde se define un recurso Deployment, usando la imagen de Docker Hub nginx:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: contenedor-nginx
        ports:
        - name: http
          containerPort: 80
```

Creamos el recurso ejecutando:

```
oc apply -f nginx.yaml
```

Warning: would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "contenedor-nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "contenedor-nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "contenedor-nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "contenedor-nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")

Nos aparece un aviso, de que no estamos cumpliendo las restricciones de seguridad, pero los recursos se han creado. Cuando vemos el estado del Pod que hemos creado, nos encontramos:

NAME                     READY   STATUS             RESTARTS      AGE
nginx-8565794bdc-lzf65   0/1     CrashLoopBackOff   4 (14s ago)   110s

Está dando error y se está continuamente reiniciando. Si vemos los logs del pod:

```
oc logs pod/nginx-8565794bdc-lzf65
```

nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
Significa que el contenedor está intentando crear un directorio como root, y no tiene permiso para ello. 
Terminamos eliminando los recursos:

```
oc delete deploy/nginx
```

**Cómo podemos ejecutar este despliegue**

La solución ya la hemos usado en los apartados anteriores. Tenemos que modificar los privilegios de ejecución de los Pods, para ello tenemos que añadir un privilegio al Service Account default.

Por lo tanto como administrador del clúster:

```
oc login -u kubeadmin https://api.crc.testing:6443
oc project proyecto-nginx
oc adm policy add-scc-to-user anyuid -z default
oc login -u developer -p developer https://api.crc.testing:6443
```

Esta instrucción agrega la restricción de seguridad llamada anyuid al ServiceAccount predeterminado (default) en tu proyecto actual de OpenShift.

La restricción anyuid permite a los contenedores en el pod ejecutarse con privilegios.
Por lo tanto, ahora ejecutamos la instrucción:

```
oc apply -f nginx.yaml
```
```
oc get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-8565794bdc-lm7dp   1/1     Running   0          25s
```

Y comprobamos que funciona:

``` 
oc port-forward deploy/nginx 8080:80
```

### Actualización de un Deployment (rollout y rollback)

Una vez que hemos creado un Deployment a partir de una imagen de una versión determinada, tenemos los Pods ejecutando la versión indicada de la aplicación.

¿Cómo podemos actualizar a una nueva versión de la aplicación?. Se seguirán los siguientes pasos:

1. Tendremos que modificar el valor del parámetro image para indicar una nueva imagen, especificando la nueva versión mediante el cambio de etiqueta.
2. En ese momento el Deployment se actualiza, es decir, crea un nuevo ReplicaSet que creará nuevos Pods de la nueva versión de la aplicación.
3. Según la estrategia de despliegue indicada, se irán borrando los antiguos Pods y se crearán lo nuevos.
4. El Deployment guardará el ReplicaSet antiguo, por si en algún momento queremos volver a la versión anterior.
   
Veamos este proceso con más detalles estudiando un ejemplo de despliegue:

**Desplegando la aplicación test_web**

Con el usuario developer creamos un nuevo proyecto:

```
oc new-project test-web
```

Vamos a partir del fichero deployment.yaml:

```
kind: Deployment
metadata:
  name: test-web
  labels:
    app: test-web
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: test-web
  template:
    metadata:
      labels:
        app: test-web
    spec:
      containers:
      - image: josedom24/test_web:v1
        name: contenedor1
        ports:
        - name: http
          containerPort: 8080
        imagePullPolicy: Always
        securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            seccompProfile:
              type: RuntimeDefault
            capabilities:
              drop:
              - ALL
```

Vamos a desplegar la versión v1 de la aplicación test_web. Para ello, ejecutamos:

```
oc apply -f deployment.yaml
```

A continuación podemos "anotar" en el despliegue la causa del nuevo despliegue, de esta forma al visualizar el historial de modificaciones veremos las causas que han provocado cada actualización. Para ello:

```
oc annotate deployment/test-web kubernetes.io/change-cause="Primer despliegue. Desplegamos versión 1"
```

Podemos comprobar los recursos que hemos creado:

```
oc get all
```

Y si accedemos al Pod con un port-forward comprobamos que la versión actual es la Versión 1:

```
oc port-forward deployment/test-web 8080:8080
```

**Actualizar un Deployment**

A continuación queremos desplegar una versión más reciente de la aplicación. Para ello tenemos que modificar el campo image de nuestro Deployment, esta operación la podemos hacer de dos formas:

Modificando el fichero YAML y volviendo a ejecutar un oc apply.

Ejecutando la siguiente instrucción:

```
 oc set image deployment/test-web contenedor1=josedom24/test_web:v2
```

Al ejecutar la actualización del Deployment podemos observar que se ha creado un nuevo ReplicaSet, que creará los nuevos Pods a partir de la versión modificada de la imagen. ¿Cómo se crean los nuevos Pods y se destruyen los antiguos? Dependerá de la estrategia de despliegue:

* Recreate que elimina los Pods antiguos y crea los nuevos.
* RollingUpdate, se van creando los nuevos Pods, se comprueba que funcionan y se eliminan los antiguos. Es la opción por defecto.
  
A continuación indicamos el motivo del cambio del despliegue con una anotación:

```
oc annotate deployment/test-web kubernetes.io/change-cause="Segundo despliegue. Desplegamos versión 2"
```

Veamos los recursos que se han creado en la actualización:

```
oc get all
```

Kubernetes y OpenShift utilizan el término rollout para la gestión de diferentes versiones de despliegues. Podemos ver el historial de actualizaciones que hemos hecho sobre el despliegue:

```
oc rollout history deployment/test-web
```

Y nos aparecen las anotaciones que hemos hecho de cada despliegue:

```
deployment.apps/test-web 
REVISION  CHANGE-CAUSE
1         Primer despliegue. Desplegamos versión 1
2         Segundo despliegue. Desplegamos versión 2
```

Y volvemos a acceder a la aplicación con un port-forward para comprobar que realmente se ha desplegado la Versión 2.

**Rollback del Deployment**

A ese proceso de volver a una versión anterior de la aplicación es lo que llamamos rollback, o de forma concreta en Kubernetes y en OpenShift, "deshacer" un rollout. Veremos en este ejemplo un mecanismo sencillo de volver a versiones anteriores. Como hemos comentado, las actualizaciones de los Deployment van creando nuevos ReplicaSets, y se va guardando el historial de ReplicaSets anteriores. Deshacer un Rollout será tan sencillo como activar uno de los ReplicaSets antiguos.

Ahora vamos a desplegar una versión que nos da un error (la versión 3 de la aplicación tiene un problema con la hoja de estilo). ¿Podremos volver al despliegue anterior?

```
oc set image deployment/test-web contenedor1=josedom24/test_web:v3
```

Y realizamos la anotación:

```
oc annotate deployment/test-web kubernetes.io/change-cause="Tercer despliegue. Desplegamos versión 3"
```

Comprobamos el historial de despliegues:

```
oc rollout history deployment/test-web
deployment.apps/test-web 
REVISION  CHANGE-CAUSE
1         Primer despliegue. Desplegamos versión 1
2         Segundo despliegue. Desplegamos versión 2
3         Tercer despliegue. Desplegamos versión 3
```

Volvemos a acceder a la aplicación haciendo un port-forward y comprobamos que tiene un problema con la hoja de estilo:

Se puede volver a la versión anterior del despliegue mediante rollout:

```
oc rollout undo deployment/test-web
oc get all
```

Y terminamos comprobando el historial de actualizaciones:

```
oc rollout history deployment/test-web
deployment.apps/test-web
REVISION  CHANGE-CAUSE
1         Primer despliegue. Desplegamos versión 1
3         Tercer despliegue. Desplegamos versión 3
4         Segundo despliegue. Desplegamos versión 2
```

Finalmente, podemos acceder de nuevo con un port-forward y comprobamos que hemos vuelto a la versión 2.

## 6. Acceso a las aplicaciones

### Trabajando con Services

Suponemos que tenemos desplegado la aplicación test-web del capítulo anterior. Tenemos dos Pods ofreciendo el servidor web nginx, a los que queremos acceder desde el exterior y que se balancee la carga entre ellos.

**Service ClusterIP**

Podríamos crear un recurso Service desde la línea de comandos:

```
oc expose deployment/test-web
```

También podemos describir las características del Service en un fichero YAML service.yaml:

```
kind: Service
apiVersion: v1
metadata:
  name: test-web
  labels:
    app: test-web
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
    app: test-web
```

Veamos la descripción:

* Vamos a crear un recurso Service (parámetro kind) y lo nombramos como nginx (parámetro name). Este nombre será importante para la resolución dns.
* En la especificación del recurso indicamos el tipo de Service (parámetro type).
* A continuación, definimos el puerto por el que va a ofrecer el Service y lo nombramos (dentro del apartado port: el parámetro port y el parámetro name). Además, debemos indicar el puerto en el que los Pods están ofreciendo el Service (parámetro targetPort)
* Por ultimo, seleccionamos los Pods a los que vamos acceder y vamos a balancear la carga seleccionando los Pods por medio de sus etiquetas (parámetro selector).

Podemos ver la información más detallada del Service que acabamos de crear:

```
oc describe service/test-web
Name:              test-web
...
Selector:          app=test-web
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                172.30.211.73
IPs:               172.30.211.73
Port:              http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.43.128:8080,10.128.51.189:8080
...
```

* Podemos ver la etiqueta de los Pods a los que accede (Selector).
* El tipo de Service (Type). La IP virtual que ha tomado (CLUSTER-IP) y que es accesible desde el cluster (IP).
* El puerto por el que ofrece el Service (Port).
* El puerto de los Pods a los que redirige el tráfico (TargetPort).
* Y por último, podemos ver las IPs de los Pods que ha seleccionado y sobre los que balanceará la carga (Endpoints).

**Services NodePort**

La definición de un Service de tipo NodePort sería exactamente igual, pero cambiando el parámetro type. Por ejemplo, lo tenemos definido en el fichero service-np.yaml:

``` 
kind: Service
apiVersion: v1
metadata:
  name: test-web-np
  labels:
    app: test-web
spec:
  type: NodePort
  ports:
    - name: http
      port: 8080
      targetPort: 8080
  selector:
    app: test-web
```

Creamos el recurso:

```
oc apply -f service-np.yaml
```

Para ver los Services que tenemos creado:

```
oc get services

NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
test-web      ClusterIP   10.217.4.144   <none>        8080/TCP         7s
test-web-np   NodePort    10.217.5.209   <none>        8080:30737/TCP   4s
```

Recuerda que si usamos oc get all también se mostrarán los Services.

Vamos a acceder a la aplicación, necesitamos saber la dirección IP del nodo master del clúster, para ello ejecuto:

```
crc ip
192.168.130.11
```

Por lo tanto para acceder necesito esa dirección IP y el puerto que se ha asignado al Service NodePort, en nuestro caso 30737.

Para eliminar el Service, ejecutamos:

``` 
oc delete service/test-web-np
```

### Accediendo a las aplicaciones: ingress y routes

**Ingress**

Aunque podríamos utilizar la definición de un recurso ingress para el acceso a la aplicación usando una URL, por ejemplo con un fichero ingress.yaml:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-test-web
spec:
  rules:
  - host: www.example.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: test-web
            port:
              number: 8080
```

En la definición hemos iniciado los siguientes parámetros:

* host: Indicamos el nombre de host que vamos a usar para el acceso. Este nombre debe apuntar a la ip de un nodo del clúster.
* path: Indicamos el path de la url que vamos a usar, en este caso sería la ruta raíz: /.
* pathType: No es importante, nos permite indicar cómo se van a trabajar con las URL.
* backend: Indicamos el Service al que vamos a acceder. En este caso indicamos el nombre del Service (service/name) y el puerto del Service (service/port/number).
  
Y podríamos crear el recurso, ejecutando:

``` 
oc apply -f ingress.yaml
```

*Nota:* Si estamos usando OpenShift Dedicated Developer Sandbox, no tenemos acceso a la dirección IP del nodo master del clúster, por lo que en nuestro servidor DNS no podemos asociar la URL con una dirección IP. De la misma manera, no podremos usar un recurso Service de tipo NodePort.

En OpenShift se recomienda el uso de recursos Routes, que nos asignan de forma automática una URL que podemos usar directamente (está dada de alta en un servidor DNS).

**Route**

La manera más sencilla de crear un recurso Route en OpenShift es ejecutando:

```
oc expose service/test-web
```

También podemos definir el recurso en un fichero route.yaml, para crearlo a continuación con oc apply:

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: test-web
  labels:
    app: test-web
spec:
  to:
    kind: Service
    name: test-web
  port:
    targetPort: http
```

Ahora podemos ver los objetos routes que tenemos creados, ejecutando:

```
oc get routes
```

Y obtener información de la ruta creada con el comando:

```
oc describe route/test-web
Name:			test-web
Namespace:		test-web
Created:		About a minute ago
Labels:			app=test-web
Annotations:		openshift.io/host.generated=true
Requested Host:		test-web-test-web.apps.sandbox-m3.1530.p1.openshiftapps.com

...
Service:	test-web
Weight:		100 (100%)
Endpoints:	10.128.43.128:8080, 10.128.51.189:8080
```

Podemos ver la URL que nos han asignado para el acceso (Requested Host), el servicio con el que esta conectado (Service) y cómo está balanceado la carga entre los Pods seleccionados por el servicio (Endpoints).

El formato de la URL que se ha generado es:

```
<nombre_despliegue>-<nombre_namespace>-<url de acceso al clúster de openshift>
```

Podemos usar la URL para acceder a la aplicación.

### Servicio DNS en OpenShift

Existe un componente en OpenShift que ofrece un servidor DNS interno para que los Pods puedan resolver diferentes nombres de recursos (Services, Pods, ...) a direcciones IP.

Cada vez que se crea un nuevo recurso Service se crea un registro de tipo A con el nombre:

<nombre_servicio>.<nombre_namespace>.svc.cluster.local.

**Comprobemos el servidor DNS**

Partimos del punto anterior donde tenemos creado un Service:

```
oc get services
test-web            ClusterIP   172.30.211.73   <none>        8080/TCP  
```

Para comprobar el servidor DNS de nuestro clúster y que podemos resolver los nombres de los distintos Services, vamos a usar un Pod, cuya definición esta en el fichero busybox.yaml creado desde una imagen busybox. Es una imagen muy pequeña pero con algunas utilidades que nos vienen muy bien:

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: contenedor
    image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
```

Creamos el pod:

```
oc apply -f busybox.yaml
```

¿Qué servidor DNS está configurado en los Pods que estamos creando? Podemos ejecutar la siguiente instrucción para comprobarlo:

``` 
oc exec -it busybox -- cat /etc/resolv.conf
search test-web.svc.cluster.local svc.cluster.local cluster.local crc.testing
nameserver 10.217.4.10
```

El servidor DNS tiene asignado la IP del clúster *10.217.4.10.*

Podemos utilizar el nombre corto del Service, porque buscará el nombre del host totalmente cualificado usando los dominios indicados en el parámetro search. Como vemos el primer nombre de dominio es el que se crea con los Services: test-web.svc.cluster.local svc.cluster.local (recuerda que el proyecto que estamos usando es test-web).

Vamos a comprobar que realmente se ha creado un registro A para el Service, haciendo consultas DNS:

```
oc exec -it busybox -- nslookup test-web
Server:		10.217.4.10
Address:	10.217.4.10:53
...
Name:	test-web.test-web.svc.cluster.local
Address: 10.217.4.144
```

Vemos que ha hecho la resolución del nombre test-web con la IP correspondiente a su servicio.

Podemos concluir que, cuando necesitemos acceder desde alguna aplicación desplegada en nuestro clúster a otro servicio ofrecido por otro despliegue, utilizaremos el nombre que hemos asignado a su Service de acceso.

## 7. Despliegues parametrizados

### Variables de entorno

**Configuración de aplicaciones usando variables de entorno**

Utilizaremos el fichero mysql-deployment-env.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: contenedor-mysql
          image: bitnami/mysql
          ports:
            - containerPort: 3306
              name: db-port
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: my-password
```

En el apartado containers hemos incluido la sección env donde vamos indicando, como una lista, el nombre de la variable (name) y su valor (value).

Vamos a trabajar con el usuario developer, creando un nuevo proyecto:

```
oc new-project mysql
```

Vamos a comprobar si realmente se ha creado el servidor de base de datos con esa contraseña del root:

```
oc apply -f mysql-deployment-env.yaml

oc get all
```

Comprobamos que se ha creado una variable del entorno en el contenedor:

```
oc exec -it deployment.apps/mysql-env -- env
...
MYSQL_ROOT_PASSWORD=my-password
```

Y finalmente realizamos un acceso a la base de datos:

```
oc exec -it deployment.apps/mysql-env -- bash -c "mysql -u root -p -h 127.0.0.1"
Enter password:
...
mysql>
```

### ConfigMaps

**Configuración de aplicaciones usando ConfigMaps**

ConfigMap permite definir un diccionario (clave,valor) para guardar información que se puede utilizar para configurar una aplicación.

Aunque hay distintas formas de indicar el conjunto de claves-valor de nuestro ConfigMap, en este caso vamos a usar literales, por ejemplo:

```
oc create cm mysql --from-literal=root_password=my-password \
                          --from-literal=mysql_usuario=usuario     \
                          --from-literal=mysql_password=password-user \
                          --from-literal=basededatos=test
```

En el ejemplo anterior, hemos creado un ConfigMap llamado mysql con cuatro pares clave-valor. Para ver los ConfigMaps que tenemos creados, podemos utilizar:

```
oc get cm
```

Y para ver los detalles del mismo:

```
oc describe cm mysql
```

Si queremos crear un fichero YAML para declarar el objeto ConfigMap, podemos ejecutar:

```
oc create cm mysql --from-literal=root_password=my-password \
                          --from-literal=mysql_usuario=usuario     \
                          --from-literal=mysql_password=password-user \
                          --from-literal=basededatos=test \
                          -o yaml --dry-run=client > configmap.yaml
```

Una vez que creado el ConfigMap se puede crear un despliegue donde las variables de entorno se inicializan con los valores guardados en el ConfigMap. Por ejemplo, un despliegue de una base de datos lo podemos encontrar en el fichero mysql-deployment-configmap.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-configmap
  labels:
    app: mysql2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql2
  template:
    metadata:
      labels:
        app: mysql2
    spec:
      containers:
        - name: contenedor-mysql
          image: bitnami/mysql
          ports:
            - containerPort: 3306
              name: db-port
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: mysql
                  key: root_password
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: mysql
                  key: mysql_usuario
            - name: MYSQL_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: mysql
                  key: mysql_password
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: mysql
                  key: basededatos
```

Creamos el despliegue, comprobamos que las variables se han creado y accedemos a la base de datos con el usuario que hemos creado:

```
oc apply -f mysql-deployment-configmap.yaml

oc exec -it deploy/mysql-configmap -- env
...
MYSQL_ROOT_PASSWORD=my-password
MYSQL_USER=usuario
MYSQL_PASSWORD=password-user
MYSQL_DATABASE=test

oc exec -it deployment.apps/mysql-configmap -- bash -c "mysql -u usuario -p -h 127.0.0.1"
Enter password: 
...
mysql> show databases;
```

### Secrets

Los Secrets permiten guardar información sensible que será codificada o cifrada.

Hay distintos tipos de Secret, en este curso vamos a usar los genéricos y los vamos a crear a partir de un literal. Por ejemplo para guardar la contraseña del usuario root de una base de datos, crearíamos un Secret de la siguiente manera:

```
oc create secret generic mysql --from-literal=password=my-password
```

Podemos obtener información de los Secrets que hemos creado con las instrucciones:

```
oc get secret
oc describe secret mysql
```

Si queremos crear un fichero YAML para declarar el objeto Secret, podemos ejecutar:

```
oc create secret generic mysql --from-literal=password=my-password \
                          -o yaml --dry-run=client > secret.yaml
```

Veamos a continuación cómo quedaría un despliegue que usa el valor de un Secret para inicializar una variable de entorno. Vamos a usar el fichero mysql-deployment-secret.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-secret
  labels:
    app: mysql3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql3
  template:
    metadata:
      labels:
        app: mysql3
    spec:
      containers:
        - name: contenedor-mysql
          image:  bitnami/mysql
          ports:
            - containerPort: 3306
              name: db-port
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql
                  key: password
```

Observamos como al indicar las variables de entorno (sección env) seguimos indicado el nombre (name) pero el valor se indica con un valor de un Secret (valueFrom: - secretKeyRef:), indicando el nombre del Secret (name) y la clave correspondiente (key).

### Agrupación de aplicaciones

**Agrupando despliegues en aplicaciones**

Cogemos uno de los despliegues que queremos agrupar, y elegimos la opción Edit application grouping.

A continuación podemos nombrar la aplicación (el agrupamiento) que estamos creando

En el otro despliegue escogemos la misma opción (Edit application grouping) y escogemos lel nombre de la aplicación que ya tenemos creada

En este momento los dos despliegues ya están agrupados, y podemos verlo visualmente en la topología

La agrupación ha creado un nuevo Label en cada uno de los Deployments implicados:

    app.kubernetes.io/part-of=wordpress

**Conexión entre despliegues**

Podemos indicar que existe una relación entre despliegues de una aplicación, para ello sólo tenemos que arrastrar la flecha que sale de uno de los despliegues encima de otro despliegue

En este caso queremos señalar que los Pods del despliegue Wordpress acceden a los Pods del despliegue MySql. Las conexiones se señalan en el recurso con una anotación, por ejemplo en el Deployment** Wordpress se ha realiza una nueva anotación:

    app.openshift.io/connects-to: [{"apiVersion":"apps/v1","kind":"Deployment","name":"mysql"}]

## 8. Almacenamiento en OpenShift v4

### Almacenamiento en CRC

Al usar la instalación local de OpenShift realizada con la herramienta CRC, tenemos todo el control del clúster, y con el usuario administrador tendremos acceso a todos los recursos relacionados con el almacenamiento.

Por lo tanto, el usuario administrador podrá gestionar los recursos PersitentVolumen y storageClass, mientras que los usuarios sin privilegios gestionarán los recursos PersistentVolumenClaim para realizar la petición de los volúmenes. Veamos esto con un ejemplo:

```
    oc login -u developer -p developer https://api.crc.testing:6443'.
    oc get pv
```

En el caso de nuestra instalación con CRC tenemos disponible un StorageClass de tipo hostpath, es decir cada volumen corresponde con un directorio en el host (al tener un sólo nodo en el clúster con este tipo de almacenamiento tenemos almacenamiento compartido entre todas las réplicas de un Pod):

```
    oc get storageclass
```

Podemos observar que la configuración del recurso Storage Class tiene los siguientes parámetros:

* Política de reciclaje Delete, es decir cuando el volumen se desasocie de su solicitud, se borrará.

* Modo de asociación WaitForFirstConsumer, es decir no se crea el objeto PersistentVolumen (PV) hasta que no se utilice el volumen por el contenedor.

### Volúmenes dentro de un pod

**Declaración de volúmenes en un pod**

Vamos a trabajar con la definición de un Pod que hemos definido en el fichero pod.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: writer-container
    image: busybox
    volumeMounts:
    - name: shared-data
      mountPath: /data
    - name: host-data
      mountPath: /dir
    command: ["sh", "-c", "echo 'Hello, world!' > /data/my-file.txt && sleep 3600"]
  - name: reader-container
    image: busybox
    volumeMounts:
    - name: shared-data
      mountPath: /data
    - name: host-data
      mountPath: /dir
    command: ["/bin/sh", "-c", "cat /data/my-file.txt; sleep 3600"]
    args: ["-w"]
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: host-data
    host-path:
      path: /tmp/datos
```

En el Pod hemos definido dos contenedores y dos volúmenes.

* El volumen (shared-data) es de tipo emptyDir y se monta en el directorio /data de los dos contenedores.
* El volumen (host-data) es de tipo hostPath y se monta en el directorio /dir de los dos contenedores.
* El contenedor writer-container escribe un fichero en el directorio /data.
* El contenedor reader-container lee el fichero guardado en el directorio /data.

Estamos trabajando con el usuario administrador en el proyecto default_

```
oc login -u kubeadmin https://api.crc.testing:6443
oc project default
```

```
oc apply -f pod.yaml
```

Y mostramos los logs del contenedor reader-container, para asegurarnos que está leyendo el fichero que ha creado el contenedor writer-container:

```
oc logs -c reader-container my-pod
Hello, world!
```

Si cambiamos el valor del fichero en el primer contenedor, cambiará en el segundo contenedor:

```
oc exec -c writer-container my-pod -- sh -c 'echo "Hola, mundo!" > /data/my-file.txt'
oc exec -c reader-container my-pod -- sh -c 'cat /data/my-file.txt'
Hola, mundo!
```

Si creamos un nuevo fichero en el directorio /dir del primer contenedor, se estará creando en el directorio /tmp/datos del host donde se está ejecutando el Pod. A continuación listaremos los ficheros del directorio /dir del segundo contenedor y se debe mostrar el fichero:

```
oc exec -c writer-container my-pod -- sh -c 'touch /dir/new-file.txt'
oc exec -c reader-container my-pod -- sh -c 'ls /dir'
new-file.txt
```

Si borramos el Pod se eliminará lo guardado en el directorio /data. Al crear un nuevo Pod, el volumen de tipo hostPath se creará de nuevo por lo que el directorio /tmp/datos se creará de nuevo y el directorio /dir del Pod también estará vacío.

Finalmente podemos obtener información de los volúmenes que tenemos en un Pod ejecutando:

```
oc describe pod/my-pod
```


















