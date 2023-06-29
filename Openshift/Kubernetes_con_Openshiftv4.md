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



















