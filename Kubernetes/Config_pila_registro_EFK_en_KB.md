# Cómo configurar una pila de registro de Elasticsearch, Fluentd y Kibana (EFK) en Kubernetes

## 1. Introducción

Al ejecutar múltiples servicios y aplicaciones en un clúster de Kubernetes, una pila de registro centralizada a nivel de clúster puede ayudarlo a clasificar y analizar rápidamente el gran volumen de datos de registro producidos por sus Pods. Una solución de registro centralizado popular es la pila Elasticsearch, Fluentd y Kibana (EFK).

**Elasticsearch** es un motor de búsqueda escalable, distribuido y en tiempo real que permite búsquedas estructuradas y de texto completo, así como análisis. Se utiliza habitualmente para indexar y buscar en grandes volúmenes de datos de registro, pero también se puede utilizar para buscar en muchos tipos diferentes de documentos.

Elasticsearch se implementa comúnmente junto con Kibana, una potente interfaz y panel de visualización de datos para Elasticsearch. **Kibana** le permite explorar sus datos de registro de Elasticsearch a través de una interfaz web y crear paneles y consultas para responder preguntas rápidamente y obtener información sobre sus aplicaciones Kubernetes.

En este tutorial usaremos Fluentd para recopilar, transformar y enviar datos de registro al backend de Elasticsearch. **Fluentd** es un popular recopilador de datos de código abierto que configuraremos en nuestros nodos de Kubernetes para seguir los archivos de registro del contenedor, filtrar y transformar los datos de registro y entregarlos al clúster de Elasticsearch, donde se indexarán y almacenarán.

Comenzaremos configurando y lanzando un clúster de Elasticsearch escalable y luego crearemos el servicio e implementación de Kibana Kubernetes. Para concluir, configuraremos Fluentd como DaemonSet para que se ejecute en todos los nodos trabajadores de Kubernetes.

Si está buscando un servicio de alojamiento de Kubernetes administrado, consulte nuestro servicio de Kubernetes administrado, simple y diseñado para crecer.


## 2. Requisitos previos

Antes de comenzar con esta guía, asegúrese de tener lo siguiente a su disposición:

* Un clúster de Kubernetes con control de acceso basado en roles (RBAC) habilitado

* Asegúrese de que su clúster tenga suficientes recursos disponibles para implementar la pila EFK y, si no, escale su clúster agregando nodos trabajadores. Implementaremos un clúster de Elasticsearch de 3 pods (puede reducirlo a 1 si es necesario), así como un solo pod de Kibana. Cada nodo trabajador también ejecutará un Fluentd Pod. El clúster de esta guía consta de tres nodos trabajadores y un plano de control administrado.
La kubectlherramienta de línea de comandos instalada en su máquina local, configurada para conectarse a su clúster. Puede leer más sobre la instalación kubectl en la documentación oficial.

Una vez que haya configurado estos componentes, estará listo para comenzar con esta guía.


## 3. Crear un espacio de nombres

Antes de implementar un clúster de Elasticsearch, primero crearemos un espacio de nombres en el que instalaremos toda nuestra instrumentación de registro. Kubernetes le permite separar los objetos que se ejecutan en su clúster mediante una abstracción de "clúster virtual" llamada espacios de nombres. En esta guía, crearemos un kube-logging espacio de nombres en el que instalaremos los componentes de la pila EFK. Este espacio de nombres también nos permitirá limpiar y eliminar rápidamente la pila de registros sin perder la función del clúster de Kubernetes.

Para comenzar, primero investigue los espacios de nombres existentes en su clúster usando kubectl:

``` 
kubectl get namespaces
``` 

Debería ver los siguientes tres espacios de nombres iniciales, que vienen preinstalados con su clúster de Kubernetes:

``` 
Output
NAME          STATUS    AGE
default       Active    5m
kube-system   Active    5m
kube-public   Active    5m
``` 

El default espacio de nombres alberga objetos que se crean sin especificar un espacio de nombres. El kube-system espacio de nombres contiene objetos creados y utilizados por el sistema Kubernetes, como kube-dns, kube-proxy y kubernetes-dashboard. Es una buena práctica mantener limpio este espacio de nombres y no contaminarlo con las cargas de trabajo de aplicaciones e instrumentación.

El kube-public espacio de nombres es otro espacio de nombres creado automáticamente que se puede utilizar para almacenar objetos que desea que sean legibles y accesibles en todo el clúster, incluso para usuarios no autenticados.

Para crear el kube-logging espacio de nombres, primero abra y edite un archivo llamado kube-logging.yaml usando su editor favorito, como nano:

``` 
nano kube-logging.yaml
``` 

Dentro de su editor, pegue el siguiente objeto de espacio de nombres YAML:

```
kind: Namespace
apiVersion: v1
metadata:
  name: kube-logging
``` 

Luego, guarde y cierre el archivo.

Aquí, especificamos el objeto de Kubernetes kind como un Namespace objeto. Para obtener más información sobre Namespace los objetos, consulte el Tutorial de espacios de nombres en la documentación oficial de Kubernetes. También especificamos la versión de la API de Kubernetes utilizada para crear el objeto (v1) y le asignamos un name, kube-logging.

Una vez que haya creado el kube-logging.yaml archivo objeto del espacio de nombres, cree el espacio de nombres usando kubectl create la -f marca de nombre de archivo:

``` 
kubectl create -f kube-logging.yaml
``` 

Deberías ver el siguiente resultado:

``` 
Output
namespace/kube-logging created
``` 

Luego puede confirmar que el espacio de nombres se creó correctamente:

``` 
kubectl get namespaces
``` 

En este punto, deberías ver el nuevo kube-loggingespacio de nombres:

``` 
Output
NAME           STATUS    AGE
default        Active    23m
kube-logging   Active    1m
kube-public    Active    23m
kube-system    Active    23m
``` 

Ahora podemos implementar un clúster de Elasticsearch en este espacio de nombres de registro aislado.


## 4. Crear el StatefulSet de Elasticsearch

Ahora que hemos creado un espacio de nombres para albergar nuestra pila de registros, podemos comenzar a implementar sus diversos componentes. Primero comenzaremos implementando un clúster Elasticsearch de 3 nodos.

En esta guía, utilizamos 3 Elasticsearch Pods para evitar el problema del "cerebro dividido" que ocurre en clústeres de múltiples nodos de alta disponibilidad. En un nivel alto, el “cerebro dividido” es lo que surge cuando uno o más nodos no pueden comunicarse con los demás y se eligen varios maestros “divididos”. Con 3 nodos, si uno se desconecta temporalmente del clúster, los otros dos nodos pueden elegir un nuevo maestro y el clúster puede continuar funcionando mientras el último nodo intenta volver a unirse. 

### 4.1 Creando el servicio sin cabeza

Para comenzar, crearemos un servicio Kubernetes sin cabeza llamado elasticsearch que definirá un dominio DNS para los 3 Pods. Un servicio headless no realiza equilibrio de carga ni tiene una IP estática.

Abra un archivo llamado elasticsearch_svc.yaml usando su editor favorito:

``` 
nano elasticsearch_svc.yaml
``` 

Pegue el siguiente YAML del servicio de Kubernetes:

``` 
kind: Service
apiVersion: v1
metadata:
  name: elasticsearch
  namespace: kube-logging
  labels:
    app: elasticsearch
spec:
  selector:
    app: elasticsearch
  clusterIP: None
  ports:
    - port: 9200
      name: rest
    - port: 9300
      name: inter-node
``` 

Luego, guarde y cierre el archivo.

Definimos un Service llamado elasticsearch en el kube-logging espacio de nombres y le damos la app: elasticsearch etiqueta. Luego configuramos .spec.selectorpara app: elasticsearch que el Servicio seleccione Pods con la app: elasticsearch etiqueta. Cuando asociamos nuestro Elasticsearch StatefulSet con este Servicio, el Servicio devolverá registros DNS A que apuntan a Elasticsearch Pods con la app: elasticsearch etiqueta.

Luego configuramos clusterIP: None, lo que hace que el servicio sea sin cabeza. Finalmente, definimos los puertos 9200 y 9300, los cuáles se utilizan para interactuar con la API REST y para la comunicación entre nodos, respectivamente.

Cree el servicio usando kubectl:

``` 
kubectl create -f elasticsearch_svc.yaml
```

Deberías ver el siguiente resultado:

``` 
Output
service/elasticsearch created
``` 

Finalmente, verifique que el servicio se haya creado exitosamente usando kubectl get:

``` 
kubectl get services --namespace=kube-logging
```

Deberías ver lo siguiente:

``` 
Output
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None         <none>        9200/TCP,9300/TCP   26s
``` 

Ahora que hemos configurado nuestro servicio sin cabeza y un .elasticsearch.kube-logging.svc.cluster.local dominio estable para nuestros Pods, podemos seguir adelante y crear StatefulSet.

### 4.2 Creando el StatefulSet

Un Kubernetes StatefulSet le permite asignar una identidad estable a los Pods y otorgarles almacenamiento estable y persistente. Elasticsearch requiere un almacenamiento estable para conservar los datos durante la reprogramación y los reinicios del Pod. 

Abra un archivo llamado elasticsearch_statefulset.yaml en su editor favorito:

``` 
nano elasticsearch_statefulset.yaml
``` 

Nos desplazaremos por la definición del objeto StatefulSet sección por sección, pegando bloques en este archivo.

Comience pegando el siguiente bloque:

``` 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: es-cluster
  namespace: kube-logging
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
``` 

En este bloque, definimos un StatefulSet llamado es-cluster en el kube-logging espacio de nombres. Luego lo asociamos con nuestro elasticsearch Servicio creado previamente usando el service Name campo. Esto garantiza que se podrá acceder a cada Pod en StatefulSet utilizando la siguiente dirección DNS: es-cluster-[0,1,2].elasticsearch.kube-logging.svc.cluster.local, donde [0,1,2]corresponde al número entero ordinal asignado al Pod.

Especificamos 3 replicas(Pods) y configuramos el match Labels selector en app: elasticseach, que luego reflejamos en la .spec.template.metadata sección. Los campos .spec.selector.matchLabels y .spec.template.metadata.labels deben coincidir.

Ahora podemos pasar a la especificación del objeto. Pegue el siguiente bloque de YAML inmediatamente debajo del bloque anterior:

``` 
. . .
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
        resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
        ports:
        - containerPort: 9200
          name: rest
          protocol: TCP
        - containerPort: 9300
          name: inter-node
          protocol: TCP
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
        env:
          - name: cluster.name
            value: k8s-logs
          - name: node.name
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: discovery.seed_hosts
            value: "es-cluster-0.elasticsearch,es-cluster-1.elasticsearch,es-cluster-2.elasticsearch"
          - name: cluster.initial_master_nodes
            value: "es-cluster-0,es-cluster-1,es-cluster-2"
          - name: ES_JAVA_OPTS
            value: "-Xms512m -Xmx512m"
``` 

Aquí definimos los Pods en StatefulSet. Nombramos los contenedores elasticsearch y elegimos la docker.elastic.co/elasticsearch/elasticsearch:7.2.0 imagen de Docker. En este punto, puede modificar esta etiqueta de imagen para que corresponda a su propia imagen interna de Elasticsearch o a una versión diferente. Tenga en cuenta que, a los efectos de esta guía, solo 7.2.0 se ha probado Elasticsearch.

Luego usamos el resources campo para especificar que el contenedor necesita al menos 0,1 vCPU garantizados y puede explotar hasta 1 vCPU (lo que limita el uso de recursos del Pod cuando se realiza una ingesta inicial grande o se trata de un pico de carga). Debe modificar estos valores según la carga prevista y los recursos disponibles. 

Luego abrimos y nombramos puertos 9200 para 9300 API REST y comunicación entre nodos, respectivamente. Especificamos un volumeMount llamado data que montará el PersistentVolume nombrado data en el contenedor en la ruta /usr/share/elasticsearch/data. Definiremos VolumeClaims para este StatefulSet en un bloque YAML posterior.

Finalmente, configuramos algunas variables de entorno en el contenedor:

* **cluster.name:** el nombre del clúster de Elasticsearch, que en esta guía es k8s-logs.
* **node.name:** El nombre del nodo, que configuramos en el .metadata.name campo usando value From. Esto se resolverá en es-cluster-[0,1,2], según el ordinal asignado al nodo.
* **discovery.seed_hosts:** este campo establece una lista de nodos elegibles para maestros en el clúster que iniciarán el proceso de descubrimiento de nodos. En esta guía, gracias al servicio sin cabeza que configuramos anteriormente, nuestros Pods tienen dominios del formato es-cluster-[0,1,2].elasticsearch.kube-logging.svc.cluster.local, por lo que configuramos esta variable en consecuencia. Usando la resolución DNS de Kubernetes del espacio de nombres local, podemos acortarlo a es-cluster-[0,1,2].elasticsearch. Para obtener más información sobre el descubrimiento de Elasticsearch, consulte la documentación oficial de Elasticsearch.
* **cluster.initial_master_nodes:** este campo también especifica una lista de nodos elegibles para el maestro que participarán en el proceso de elección del maestro. Tenga en cuenta que para este campo debe identificar los nodos por su nombre de host node.name y no por su nombre de host.
* **ES_JAVA_OPTS:** Aquí configuramos esto para -Xms512m -Xmx512m indicarle a la JVM que use un tamaño de montón mínimo y máximo de 512 MB. Debe ajustar estos parámetros según la disponibilidad y las necesidades de recursos de su clúster. Para obtener más información, consulte Configuración del tamaño del montón.
  
El siguiente bloque que pegaremos tiene el siguiente aspecto:

``` 
. . .
      initContainers:
      - name: fix-permissions
        image: busybox
        command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
        securityContext:
          privileged: true
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      - name: increase-fd-ulimit
        image: busybox
        command: ["sh", "-c", "ulimit -n 65536"]
        securityContext:
          privileged: true
``` 

En este bloque, definimos varios contenedores de inicio que se ejecutan antes del elasticsearch contenedor principal de la aplicación. Cada uno de estos contenedores de inicio se ejecuta hasta su finalización en el orden en que están definidos. 

El primero, denominado fix-permissions, ejecuta un chown comando para cambiar el propietario y el grupo del directorio de datos de Elasticsearch a 1000:1000, el UID del usuario de Elasticsearch. De forma predeterminada, Kubernetes monta el directorio de datos como root, lo que lo hace inaccesible para Elasticsearch. 

El segundo, llamado increase-vm-max-map, ejecuta un comando para aumentar los límites del sistema operativo en los recuentos de mmap, que de forma predeterminada pueden ser demasiado bajos, lo que genera errores de falta de memoria. 

El siguiente contenedor de inicio que se ejecutará es increase-fd-ulimit, que ejecuta el ulimit comando para aumentar el número máximo de descriptores de archivos abiertos. 

Ahora que hemos definido el contenedor de nuestra aplicación principal y los contenedores de inicio que se ejecutan antes para ajustar el sistema operativo del contenedor, podemos agregar la pieza final a nuestro archivo de definición de objetos StatefulSet: el archivo volumeClaimTemplates.

Pegue el siguiente volumeClaimTemplate bloque:

``` 
. . .
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: elasticsearch
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: do-block-storage
      resources:
        requests:
          storage: 100Gi
```

En este bloque, definimos el StatefulSet volumeClaimTemplates. Kubernetes usará esto para crear PersistentVolumes para los Pods. En el bloque anterior, le asignamos un nombre data (que es al que name nos referimos en los volumeMount definidos anteriormente) y le damos la misma app: elasticsearch etiqueta que nuestro StatefulSet.

Luego especificamos su modo de acceso como ReadWriteOnce, lo que significa que solo un nodo puede montarlo como lectura-escritura. Definimos la clase de almacenamiento como do-block-storage en esta guía, ya que utilizamos un clúster de DigitalOcean Kubernetes con fines de demostración. Debe cambiar este valor según dónde esté ejecutando su clúster de Kubernetes. Para obtener más información, consulte la documentación de Volumen persistente.

Finalmente, especificamos que nos gustaría que cada PersistentVolume tenga un tamaño de 100 GiB. Debe ajustar este valor dependiendo de sus necesidades de producción.

La especificación completa de StatefulSet debería verse así:

``` 
elasticsearch_statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: es-cluster
  namespace: kube-logging
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
        resources:
            limits:
              cpu: 1000m
            requests:
              cpu: 100m
        ports:
        - containerPort: 9200
          name: rest
          protocol: TCP
        - containerPort: 9300
          name: inter-node
          protocol: TCP
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
        env:
          - name: cluster.name
            value: k8s-logs
          - name: node.name
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: discovery.seed_hosts
            value: "es-cluster-0.elasticsearch,es-cluster-1.elasticsearch,es-cluster-2.elasticsearch"
          - name: cluster.initial_master_nodes
            value: "es-cluster-0,es-cluster-1,es-cluster-2"
          - name: ES_JAVA_OPTS
            value: "-Xms512m -Xmx512m"
      initContainers:
      - name: fix-permissions
        image: busybox
        command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
        securityContext:
          privileged: true
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      - name: increase-fd-ulimit
        image: busybox
        command: ["sh", "-c", "ulimit -n 65536"]
        securityContext:
          privileged: true
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: elasticsearch
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: do-block-storage
      resources:
        requests:
          storage: 100Gi
``` 

Una vez que esté satisfecho con su configuración de Elasticsearch, guarde y cierre el archivo.

Ahora, implemente StatefulSet usando kubectl:

``` 
kubectl create -f elasticsearch_statefulset.yaml
``` 

Deberías ver el siguiente resultado:

``` 
Output
statefulset.apps/es-cluster created
``` 

Puede monitorear StatefulSet a medida que se implementa usando kubectl rollout status:

``` 
kubectl rollout status sts/es-cluster --namespace=kube-logging
``` 

Debería ver el siguiente resultado a medida que se implementa el clúster:

``` 
Output
Waiting for 3 pods to be ready...
Waiting for 2 pods to be ready...
Waiting for 1 pods to be ready...
partitioned roll out complete: 3 new pods have been updated...
``` 

Una vez que se hayan implementado todos los Pods, puede verificar que su clúster de Elasticsearch esté funcionando correctamente realizando una solicitud en la API REST.

Para hacerlo, primero reenvíe el puerto local 9200 al puerto 9200 en uno de los nodos de Elasticsearch ( es-cluster-0) usando kubectl port-forward:

``` 
kubectl port-forward es-cluster-0 9200:9200 --namespace=kube-logging
``` 

Luego, en una ventana de terminal separada, realice una curl solicitud a la API REST:

``` 
curl http://localhost:9200/_cluster/state?pretty
``` 

Deberías ver el siguiente resultado:

```
Output
{
  "cluster_name" : "k8s-logs",
  "compressed_size_in_bytes" : 348,
  "cluster_uuid" : "QD06dK7CQgids-GQZooNVw",
  "version" : 3,
  "state_uuid" : "mjNIWXAzQVuxNNOQ7xR-qg",
  "master_node" : "IdM5B7cUQWqFgIHXBp0JDg",
  "blocks" : { },
  "nodes" : {
    "u7DoTpMmSCixOoictzHItA" : {
      "name" : "es-cluster-1",
      "ephemeral_id" : "ZlBflnXKRMC4RvEACHIVdg",
      "transport_address" : "10.244.8.2:9300",
      "attributes" : { }
    },
    "IdM5B7cUQWqFgIHXBp0JDg" : {
      "name" : "es-cluster-0",
      "ephemeral_id" : "JTk1FDdFQuWbSFAtBxdxAQ",
      "transport_address" : "10.244.44.3:9300",
      "attributes" : { }
    },
    "R8E7xcSUSbGbgrhAdyAKmQ" : {
      "name" : "es-cluster-2",
      "ephemeral_id" : "9wv6ke71Qqy9vk2LgJTqaA",
      "transport_address" : "10.244.40.4:9300",
      "attributes" : { }
    }
  },
...
``` 

Esto indica que nuestro clúster de Elasticsearch k8s-logs se creó correctamente con 3 nodos: es-cluster-0, es-cluster-1 y es-cluster-2. El nodo maestro actual es es-cluster-0.

Ahora que su clúster de Elasticsearch está en funcionamiento, puede continuar y configurar una interfaz Kibana para él.


## 5. Creación de la implementación y el servicio de Kibana

Para iniciar Kibana en Kubernetes, crearemos un servicio llamado kibana y una implementación que constará de una réplica de Pod. Puede escalar la cantidad de réplicas según sus necesidades de producción y, opcionalmente, especificar un LoadBalancer tipo para que el Servicio equilibre la carga de las solicitudes entre los pods de implementación.

Esta vez, crearemos el Servicio y la Implementación en el mismo archivo. Abra un archivo llamado kibana.yaml en su editor favorito:

``` 
nano kibana.yaml
``` 

Pegue la siguiente especificación de servicio:

``` 
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: kube-logging
  labels:
    app: kibana
spec:
  ports:
  - port: 5601
  selector:
    app: kibana
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: kube-logging
  labels:
    app: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.2.0
        resources:
          limits:
            cpu: 1000m
          requests:
            cpu: 100m
        env:
          - name: ELASTICSEARCH_URL
            value: http://elasticsearch:9200
        ports:
        - containerPort: 5601
``` 

Luego, guarde y cierre el archivo.

En esta especificación definimos un servicio llamado kibana en el kube-logging espacio de nombres y le asignamos la app: kibana etiqueta.

También especificamos que debe ser accesible en el puerto 5601 y usar la app: kibana etiqueta para seleccionar los Pods de destino del Servicio.

En la Deployment especificación, definimos una implementación llamada kibana y especificamos que nos gustaría 1 réplica de Pod.

Usamos la docker.elastic.co/kibana/kibana:7.2.0 imagen. En este punto, puede sustituir su propia imagen de Kibana pública o privada.

Especificamos que nos gustaría garantizar al menos 0,1 vCPU al Pod, ampliando hasta un límite de 1 vCPU. Puede cambiar estos parámetros según la carga prevista y los recursos disponibles.

A continuación, usamos la ELASTICSEARCH_URL variable de entorno para configurar el punto final y el puerto para el clúster de Elasticsearch. Al utilizar Kubernetes DNS, este punto final corresponde a su nombre de servicio elasticsearch. Este dominio se resolverá en una lista de direcciones IP para los 3 pods de Elasticsearch. 

Finalmente, configuramos el puerto de contenedores de Kibana en 5601, al que el kibana Servicio reenviará las solicitudes.

Una vez que esté satisfecho con su configuración de Kibana, puede implementar el servicio y la implementación usando kubectl:

```
kubectl create -f kibana.yaml
``` 

Deberías ver el siguiente resultado:

``` 
Output
service/kibana created
deployment.apps/kibana created
``` 

Puede comprobar que la implementación se realizó correctamente ejecutando el siguiente comando:

``` 
kubectl rollout status deployment/kibana --namespace=kube-logging
``` 

Deberías ver el siguiente resultado:

``` 
Output
deployment "kibana" successfully rolled out
``` 

Para acceder a la interfaz de Kibana, una vez más reenviaremos un puerto local al nodo de Kubernetes que ejecuta Kibana. Obtenga los detalles de Kibana Pod usando kubectl get:

``` 
kubectl get pods --namespace=kube-logging
Output
NAME                      READY     STATUS    RESTARTS   AGE
es-cluster-0              1/1       Running   0          55m
es-cluster-1              1/1       Running   0          54m
es-cluster-2              1/1       Running   0          54m
kibana-6c9fb4b5b7-plbg2   1/1       Running   0          4m27s
``` 

Aquí observamos que nuestro Kibana Pod se llama kibana-6c9fb4b5b7-plbg2.

Reenvíe el puerto local 5601 al puerto 5601 en este Pod:

``` 
kubectl port-forward kibana-6c9fb4b5b7-plbg2 5601:5601 --namespace=kube-logging
``` 

Deberías ver el siguiente resultado:

``` 
Output
Forwarding from 127.0.0.1:5601 -> 5601
Forwarding from [::1]:5601 -> 5601
``` 

Ahora, en su navegador web, visite la siguiente URL:

``` 
http://localhost:5601
```

Si ve la siguiente página de bienvenida de Kibana, ha implementado Kibana correctamente en su clúster de Kubernetes.

Ahora puede pasar a implementar el componente final de la pila EFK: el recopilador de registros, Fluentd.


## 6. Crear el DaemonSet de Fluentd

En esta guía, configuraremos Fluentd como DaemonSet, que es un tipo de carga de trabajo de Kubernetes que ejecuta una copia de un Pod determinado en cada nodo del clúster de Kubernetes. Usando este controlador DaemonSet, implementaremos un Pod del agente de registro Fluentd en cada nodo de nuestro clúster. 

En Kubernetes, aplicaciones en contenedores que inician sesión stdout y stderr cuyos flujos de registro se capturan y redirigen a archivos JSON en los nodos. Fluentd Pod seguirá estos archivos de registro, filtrará los eventos de registro, transformará los datos de registro y los enviará al backend de registro de Elasticsearch que implementamos en el Paso 2.

Además de los registros de contenedores, el agente Fluentd seguirá los registros de componentes del sistema Kubernetes, como kubelet, kube-proxy y Docker. Para ver una lista completa de fuentes seguidas por el agente de registro Fluentd, consulte el kubernetes.conf archivo utilizado para configurar el agente de registro. 

Comience abriendo un archivo llamado fluentd.yaml en su editor de texto favorito:

``` 
nano fluentd.yaml
``` 

Una vez más, pegaremos las definiciones de objetos de Kubernetes bloque por bloque, proporcionando contexto a medida que avanzamos. En esta guía, utilizamos la especificación Fluentd DaemonSet proporcionada por los mantenedores de Fluentd. Otro recurso útil proporcionado por los mantenedores de Fluentd es Kuberentes Fluentd.

Primero, pegue la siguiente definición de ServiceAccount:

``` 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-logging
  labels:
    app: fluentd
``` 

Aquí, creamos una Cuenta de Servicio llamada fluentd que los Fluentd Pods usarán para acceder a la API de Kubernetes. Lo creamos en el kube-logging Namespace y una vez más le damos la etiqueta app: fluentd. 

A continuación, pegue el siguiente ClusterRole bloque:

``` 
. . .
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
  labels:
    app: fluentd
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch
``` 

Aquí definimos un ClusterRole llamado fluentd al que otorgamos permisos get, list y watch sobre los objetos pods y . namespaces ClusterRoles le permite otorgar acceso a recursos de Kubernetes con ámbito de clúster, como Nodes. 

Ahora, pega el siguiente ClusterRoleBinding bloque:

``` 
. . .
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-logging
``` 

En este bloque, definimos una ClusterRoleBinding llamada fluentd que vincula fluentd ClusterRole a la fluentd cuenta de servicio. Esto otorga a fluentdServiceAccount los permisos enumerados en la fluentd función del clúster.

En este punto podemos comenzar a pegar la especificación real de DaemonSet:

``` 
. . .
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-logging
  labels:
    app: fluentd
``` 

Aquí, definimos un DaemonSet llamado fluentd en el kube-logging espacio de nombres y le damos la app: fluentd etiqueta.

A continuación, pegue en la siguiente sección:

```
. . .
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch.kube-logging.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENTD_SYSTEMD_CONF
            value: disable
``` 

Aquí, hacemos coincidir la app: fluentd etiqueta definida en .metadata.labels y luego asignamos al DaemonSet la fluentd cuenta de servicio. También seleccionamos app: fluentd como Pods administrados por este DaemonSet.

A continuación, definimos una NoSchedule tolerancia para que coincida con la contaminación equivalente en los nodos maestros de Kubernetes. Esto garantizará que DaemonSet también se implemente en los maestros de Kubernetes. Si no deseas ejecutar un Fluentd Pod en tus nodos maestros, elimina esta tolerancia. 

A continuación, comenzamos a definir el contenedor Pod, al que llamamos fluentd.

Usamos la imagen oficial de Debian v1.4.2 proporcionada por los mantenedores de Fluentd. Si deseas utilizar tu propia imagen privada o pública de Fluentd, o usar una versión de imagen diferente, modifica la image etiqueta en la especificación del contenedor. El Dockerfile y el contenido de esta imagen están disponibles en el repositorio de Github fluentd-kubernetes-daemonset de Fluentd.

A continuación, configuramos Fluentd usando algunas variables de entorno:

* **FLUENT_ELASTICSEARCH_HOST:** Configuramos esto en la dirección del Servicio sin cabeza de Elasticsearch definida anteriormente: elasticsearch.kube-logging.svc.cluster.local. Esto se resolverá en una lista de direcciones IP para los 3 pods de Elasticsearch. Lo más probable es que el host real de Elasticsearch sea la primera dirección IP devuelta en esta lista. Para distribuir registros en el clúster, deberá modificar la configuración del complemento Elasticsearch Output de Fluentd. 
* **FLUENT_ELASTICSEARCH_PORT:** Lo configuramos en el puerto Elasticsearch que configuramos anteriormente 9200.
* **FLUENT_ELASTICSEARCH_SCHEME:** Lo configuramos en http.
* **FLUENTD_SYSTEMD_CONF:** Configuramos esto para disable suprimir la salida relacionada con systemd no estar configurada en el contenedor.
  
Finalmente, pegue en la siguiente sección:

``` 
. . .
        resources:
          limits:
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
``` 

Aquí especificamos un límite de memoria de 512 MiB en el FluentD Pod y le garantizamos 0,1 vCPU y 200 MiB de memoria. Puede ajustar estos límites de recursos y solicitudes según el volumen de registro previsto y los recursos disponibles.

A continuación, montamos las rutas del host /var/logy /var/lib/docker/containers en el contenedor usando varlog y varlibdockercontainers volumeMounts. Estos volumes se definen al final del bloque.

El parámetro final que definimos en este bloque es termination GracePeriodSeconds, que le da a Fluentd 30 segundos para apagarse correctamente al recibir una SIGTERM señal. Después de 30 segundos, los contenedores reciben una SIGKILL señal. El valor predeterminado terminationGracePeriodSecondses 30 segundos, por lo que en la mayoría de los casos este parámetro se puede omitir. 

La especificación completa de Fluentd debería verse así:

``` 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-logging
  labels:
    app: fluentd
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
  labels:
    app: fluentd
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-logging
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-logging
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch.kube-logging.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENTD_SYSTEMD_CONF
            value: disable
        resources:
          limits:
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
``` 

Una vez que hayas terminado de configurar Fluentd DaemonSet, guarda y cierra el archivo.

Ahora, implemente DaemonSet usando kubectl:

``` 
kubectl create -f fluentd.yaml
``` 

Deberías ver el siguiente resultado:

``` 
Output
serviceaccount/fluentd created
clusterrole.rbac.authorization.k8s.io/fluentd created
clusterrolebinding.rbac.authorization.k8s.io/fluentd created
daemonset.extensions/fluentd created
``` 

Verifique que su DaemonSet se haya implementado correctamente usando kubectl:

``` 
kubectl get ds --namespace=kube-logging
``` 

Debería ver el siguiente resultado de estado:

``` 
Output
NAME      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd   3         3         3         3            3           <none>          58s
``` 

Esto indica que hay 3 fluentdPods en ejecución, lo que corresponde a la cantidad de nodos en nuestro clúster de Kubernetes.

Ahora podemos verificar Kibana para verificar que los datos de registro se recopilen y envíen correctamente a Elasticsearch.

Con el archivo kubectl port-forward aún abierto, navegue hasta http://localhost:5601.

Haga clic en Descubrir en el menú de navegación de la izquierda.

Esto le permite definir los índices de Elasticsearch que le gustaría explorar en Kibana. Para obtener más información, consulte Definición de patrones de índice en los documentos oficiales de Kibana. Por ahora, solo usaremos el logstash-* patrón comodín para capturar todos los datos de registro en nuestro clúster de Elasticsearch. Ingresa logstash-* en el cuadro de texto y haz clic en Siguiente paso.

Luego serás llevado a la siguiente página.

Esto le permite configurar qué campo utilizará Kibana para filtrar los datos de registro por tiempo. En el menú desplegable, seleccione el campo @timestamp y presione Crear patrón de índice.

Ahora, presione Descubrir en el menú de navegación de la izquierda.

Deberías ver un gráfico de histograma y algunas entradas de registro recientes.

En este punto, ha configurado e implementado correctamente la pila EFK en su clúster de Kubernetes. Para aprender a utilizar Kibana para analizar sus datos de registro, consulte la Guía del usuario de Kibana .

En la siguiente sección opcional, implementaremos un Pod contador simple que imprime números en la salida estándar y busca sus registros en Kibana.


## 7. (opcional): Prueba del registro del contenedor

Para demostrar un caso de uso básico de Kibana para explorar los registros más recientes para un Pod determinado, implementaremos un Pod contador mínimo que imprime números secuenciales en la salida estándar.

Comencemos creando el Pod. Abra un archivo llamado counter.yaml en su editor favorito:

``` 
nano counter.yaml
``` 

Luego, pegue la siguiente especificación de Pod:

``` 
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
``` 

Guarde y cierre el archivo.

Este es un Pod mínimo llamado counter que ejecuta un while bucle e imprime números secuencialmente.

Implemente el counter Pod usando kubectl:

``` 
kubectl create -f counter.yaml
``` 

Una vez que el Pod se haya creado y se esté ejecutando, regrese a su panel de Kibana.

Desde la página Descubrir, en la barra de búsqueda ingresa kubernetes.pod_name:counter. Esto filtra los datos de registro para los pods denominados counter.

Luego deberías ver una lista de entradas de registro para el counterPod.

Puede hacer clic en cualquiera de las entradas del registro para ver metadatos adicionales como el nombre del contenedor, el nodo de Kubernetes, el espacio de nombres y más.


## 8. Conclusión

En esta guía, hemos demostrado cómo instalar y configurar Elasticsearch, Fluentd y Kibana en un clúster de Kubernetes. Hemos utilizado una arquitectura de registro mínima que consta de un único Pod de agente de registro que se ejecuta en cada nodo trabajador de Kubernetes.

Antes de implementar esta pila de registro en su clúster de Kubernetes de producción, es mejor ajustar los requisitos y límites de recursos como se indica a lo largo de esta guía. Es posible que también desee configurar X-Pack para habilitar funciones integradas de monitoreo y seguridad.

La arquitectura de registro que hemos utilizado aquí consta de 3 Elasticsearch Pods, un único Kibana Pod (sin equilibrio de carga) y un conjunto de Fluentd Pods implementados como DaemonSet. Es posible que desee ampliar esta configuración según su caso de uso de producción.
