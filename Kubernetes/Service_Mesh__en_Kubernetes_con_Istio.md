# Service Mesh en Kubernetes con Istio


## 1. Escenario que vamos a montar en Kubernetes.

En nuestro caso, vamos a montar un cluster de kubernetes con kubeadm. Hemos elegido este método, por su fácil instalación, que se llevará acabo en las siguientes maquinas:

• Nieve. Maquina Debian de 4GB de RAM. Sera el Master.
• Sansa. Maquina Debian de 2GB de RAM.
• Arya. Maquina Debian de 2GB de RAM.

La instalación se llevará acabo de la siguiente manera:

### 1.1. Instalación de Docker

Lo primero que haremos será instalar Docker en nuestras tres máquinas, al ser las tres máquinas Debian, la instalación se llevará acabo de la siguiente manera:

Instalamos los paquetes necesarios para poder instalar paquetes con apt mediante https.

``` 
sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common
``` 

Añadimos las claves GPG oficiales de Docker:

``` 
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
``` 

Añadimos el repositorio de Docker:

``` 
sudo add-apt-repository “deb [arch=amd64] https://download.docker.com/linux/debian \ $(lsb_release -cs) \ stable”
``` 

Actualizamos los repositorios e instalamos Docker:

``` 
sudo apt-get update sudo apt-get install docker-ce
```

### 1.2. Instalación de kubeadm

Ahora deberemos instalar kubeadm, kubelet y kubectl en las 3 máquinas. Para ello, seguimos los pasos que aparecen en la documentación de kubernetes, que se basa en añadir la clave GPG de kubernetes, su repositorio, actualizar los repositorios e instalar los paquetes necesarios: 

``` 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list 
deb http://apt.kubernetes.io/ kubernetes-xenial main 
EOF 
apt-get update 
apt-get install -y kubelet kubeadm kubectl
``` 

Ahora deberemos inicializar el nodo nieve como master, para ello ejecutamos: 

``` 
kubeadm init –pod-network-cidr=10.0.0.0/16
``` 

Si todo sale bien, se nos proporcionara un comando con la siguiente estructura: 

``` 
kubeadm join –token <token> <master_ip>:<master-port>
```

Esto nos servirá más adelante para añadir los otros nodos al master.
También nos proporcionara las siguientes líneas, que ejecutaremos para poder utilizar kubeadm con nuestro usuario:

``` 
mkdir -p $HOME/.kube sudo cp-i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOE/.kube/config
``` 

### 1.3. Instalación de addon de red interna

A continuación, instalaremos un addon para gestionar la red interna por la que se comunicaran los nodos. Entre la gran lista de opciones, nosotros nos hemos decantado por weave. Su instalación se realizará ejecutando la siguiente sentencia: 

``` 
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')”
```

Una vez instalado, podremos ver si su instalación es correcta ejecutando: 

``` 
root@nieve:/home/debian# kubectl get pods -n kube-system 
```

### 1.4 Añadir nodos al master

Solo nos faltaría añadir los nodos al master, para ello, ejecutamos el comando que mencionamos antes que nos proporcionó kubadm al convertir a nieve en master, en todos los nodos.

``` 
kubeadm join <ip_master:port> --token <token> --discovery-token-ca-cert-hash <hash>
``` 

Por último, ejecutamos el siguiente comando para ver si efectivamente se ha creado correctamente el cluster. Todo estará correcto si el “STATUS” está en Ready. 

``` 
root@nieve:/home/debian# kubectl get nodes
``` 


## 2. Instalar Istio en Kubernetes

La instalación de Istio es sencilla, y aunque se resume en pocos pasos, el tiempo que consume la descarga de paquetes y su instalación es relativamente grande. Veamos los pasos a seguir para instalar Istio, partiendo del entorno que ya hemos explicado un poco antes.

Lo primero que haremos será descargarnos los ficheros de instalación de Istio. Para ello ejecutaremos: 

``` 
curl -L https://git.io/getLatestIstio | sh -
``` 

Una vez ejecutado se nos descargará un directorio, con el nombre de Istio y la versión, en este caso, nuestro directorio se llamará Istio-1.1.7. Una vez descargado entraremos en la la siguiente ruta: 

``` 
cd istio-1.1.7/install/kubernetes/Helm/istio-init/files
``` 

Una vez que nos localizamos en esta ruta, deberemos proceder a la instalación de Istio en sí, para ello ejecutamos un kubectl apply por cada uno de los cuatro ficheros yaml que tenemos: 

```
kubectl apply -f crd-10.yaml 
kubectl apply -f crd-11.yaml 
kubectl apply -f crd-certmanager-10.yaml 
kubectl apply -f crd-certmanager-11.yaml
``` 

Tras unos 5-10 minutos (en mi caso), la instalación habrá concluido, y veremos que se ha creado un namespace llamado “istio-system”, para verificarlo, ejecutaremos primero un kubectl get svc -n istio-system: 

``` 
kubectl get pods,svc -n istio-system
``` 


## 3. Habilitar el uso de Istio

El uso de Istio se implantará habilitando Istio en un namespace, en mi caso, en el namespace default, que es el que crea kubernetes por defecto, para ello ejecutamos: 

``` 
kubectl label namespace default istio-injectio=enabled
``` 

Con esto, lo que hacemos es que cada vez que ejecutamos un kubectl apply, pasara primero por Istio, y este hará los cambios necesarios en el fichero yaml, para que Istio funcione en las aplicaciones que vamos a desplegar.


## 4. Escenario de Pruebas

Cuando nos descargamos todo lo necesario para la instalación de Istio, también se nos descargó una serie de escenarios para realizar pruebas. En concreto, en este caso vamos a usar el escenario llamado Bookinfo.

El escenario en cuestión se basa, en un ingress, conectado con una página en producción conectada a su vez con dos servicios, uno dedicado a Ruby, que contendrá los detalles de la “reviews”, y otro dedicado a Java, que mantiene desplegado 3 versiones distintas de un sistema de puntuación en 3 pods distintos, que a su vez tiene dos de sus pods conectados a un servicio de node.js, para ejecutar las reviews programadas en java.

La instalación de este escenario es muy simple. Una vez habilitado la istio-injection en el namespace que vamos a usar, ejecutaremos el siguiente comando: 

``` 
kubectl apply -f istio-1.1.7/simples/bookinfo/platform/kube/bookinfo.yaml
``` 

Una vez ejecutado, deberemos crear un nodeport con el en el deployment productpage: 

``` 
kubectl expose deployment productpage-v1 –type=NodePort
```

Una vez ejecutado, si hacemos un kubectl get services, vemos que tenemos la redirección activa, y que podremos acceder desde el puerto 30407: 

``` 
kubectl get services productpage-v1
``` 

Por último, si accedemos en cualquier navegador a la dirección 172.22.201.184:30407, nos mostrara la página funcionando.


## 5. Pruebas de Funcionamiento

Entre las numerosas pruebas de funcionamiento que se podrían hacer, hemos elegido 3 que pienso que son las más interesantes, fáciles de asimilar y que demuestran el funcionamiento puro de Istio. Estas pruebas van a ser las siguientes:

• Blue Green deployment.
• Canary Release.
• Request Timeouts

### 5.1 Blue Green Deployment

Un blue green deployment consiste en tener varias versiones de una aplicación desplegada, pero mandar todo el tráfico a través de una sola de ellas. Con Istio podemos implementar esta eficaz técnica de una manera fácil y limpia, sin tener que cambiar el código de las aplicaciones.

Una vez creado nuestro escenario de pruebas, y como vimos con la explicación de este, tenemos 3 versiones de una aplicación java. En este ejemplo de blue green deployment, haremos que todo el trafico vaya a la versión 1 de esta, para ello crearemos un fichero llamado blue-green.yaml que tendrá el siguiente contenido:

``` 
--- apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
  - reviews 
  http: 
  - route: 
    - destination: 
      host: reviews 
      subset: v1
``` 

Con este sencillo fichero, le estamos diciendo a Istio, que todas las peticiones que lleguen sean mandadas a V1. A continuación, ejecutamos el siguiente comando, y esperamos unos segundos para que la configuración tenga efecto: 

``` 
kubectl apply -f blue-green.yaml
``` 

Ahora si entramos en la página, veremos que no nos aparece ninguna estrella, ya que la v1, no tiene implementada el dibujo de las estrellas.

Con este sencillo paso, hemos redireccionado todo el tráfico a una única versión. Ahora imaginemos que hemos detectado un fallo en la versión V1, y debemos redireccionar el trafico a la versión V2. Para ello, solo deberemos hacer un pequeño cambio en el fichero blue-green.yaml, dejándolo así: 

``` 
--- apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
  - reviews 
  http: 
  - route: 
    - destination: 
      host: reviews 
      subset: v2
``` 

Aplicamos los cambios: 

``` 
kubectl apply -f blue-green.yaml
``` 

Y si comprobamos, veremos que ahora nos aparecen estrellas negras cada vez que recargamos la página, habiéndose completado la configuración.

Si deseamos deshacer los cambios, y que siga con el Round-robin propio de kubernetes bastara con ejecutar un: 

``` 
kubectl delete -f blue-gren.yaml
``` 

### 5.2 Canary release

Un canary release se basa tener una versión desplegada funcional, e introducir otra a la que le iremos incrementando el tráfico poco a poco, para ver como funciona esta a medida que se le va aumentando el número de peticiones.

Por ejemplo, tenemos una versión desplegada, a la que le llegan el 100% de las peticiones, pero hemos desarrollado otra, que queremos ir implementando poco a poco, asi que tendremos dos versiones, la primera V1 que ya no le llegara el 100% de las peticiones sino el 90% y una V2, que hemos implementado nueva, y le llegaran el 10% de las peticiones.

Con Istio, esta configuración, se implementaría creando un fichero llamado en esta caso, canary-release.yaml, con el siguiente contenido:

``` 
--- apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
    - reviews 
  http: 
  - route: 
    - destination: 
        host: reviews 
        subset: v1 
        weight: 90 
    - destination: 
        host: reviews 
        subset: v2 
        weight: 10
``` 

Con esta configuración, hemos establecido que el 90% vaya a v1 y el 10% a v2. Una vez creado el fichero, lo aplicaremos con un kubectl:

``` 
kubect apply -f canary-release.yaml
``` 

Una vez ejecutado, y al haber esperado un tiempo prudencial para que estos cambios hayan tomado efecto veremos que los porcentajes de las peticiones son los que hemos establecido.

La idea de este tipo de despliegues seria llegara hasta un punto medio de peso de solicitudes, es decir 50% para V1 y 50% para V2, si vemos que con el 10% no ha habido problemas de uso, podremos subir poco a poco hasta llegar a esta situación.

``` 
--- apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
    - reviews 
  http: 
  - route: 
    - destination: 
        host: reviews 
        subset: v1 
        weight: 50 
    - destination: 
        host: reviews 
        subset: v3 
        weight: 50
``` 

Con un 50-50, las peticiones se repartirán de manera equitativa, y si V2 sigue funcionando sin problemas, se podrá llegar al punto final de esta casuística, que el 100% de las peticiones sean para V2, dejando algo así:

``` 
--- apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
    - reviews 
  http: 
  - route: 
    - destination: 
        host: reviews 
        subset: v2
        weight: 100 
``` 

Como vemos, al no tener que mandarle tráfico a V1, podemos descartar este del fichero de configuración, llegando el 100% de peticiones a la nueva versión, y quedando algo parecido en funcionalidad al blue green deployment que vimos antes.

Por último, si queremos eliminar esta configuración, y volver la configuración por defecto, bastara con hacer un:

``` 
kubectl delete -f canary-release.yaml
``` 

### 5.3 Requests Timeouts

Por último, mostraremos como implementar requests timeouts, para demostrar esto haremos que entre la petición y la respuesta haya un tiempo de espera, para ello, primero haremos un blue green deployment para que todas las peticiones vayan a v1 exclusivamente, para ello, volveremos a ejecutar el fichero blue-green.yaml.
Ahora lo que haremos será establecer que use V2 de reviews:

```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
    - reviews 
  http: 
  - route: 
    - destination: 
      host: reviews 
      subset: v2 
EOF
``` 

Y estableceremos un tiempo de espera de 2 segundos a ratings:

```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: ratings
spec: 
  hosts: 
    - ratings
  http: 
  - fault:
    delay:
      percent: 100
      fixedDelay: 2s
    route: 
    - destination: 
      host: ratings 
      subset: v1
EOF
``` 

Ahora si abrimos el navegador, y entramos en nuestra página, veremos que tardara 2 segundos en cargarse.
Ahora agregaremos un timeout a reviews:

```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3 
kind: VirtualService 
metadata: 
  name: reviews 
spec: 
  hosts: 
    - reviews 
  http: 
  - route: 
    - destination: 
      host: reviews 
      subset: v2 
    timeout: 0.5s
EOF
``` 

Ahora cuando actualicemos la página, veremos que además de cargarse en medio segundo, este nos muestra un mensaje de error, diciéndonos que ha habido un error.

Esto se debe a que le hemos puesto un timeout de 0’5 segundos a “reviews”, pero cuando “reviews” le hace una llamada a “ratings”, esta tiene un tiempo de espera de 2 segundos, por lo que al no recibir respuesta en 0’5 segundos, “reviews” corta la conexión, y nos aparece este error, con el que demostramos el funcionamiento de un time out en Istio.




