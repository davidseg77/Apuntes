# Apuntes sobre el curso DO370 de Red Hat

### Implementación de OpenShift Data Foundation desde la interfaz de la línea de comandos

Como se explica en otra parte de este curso, la instalación de OpenShift Data Foundation consiste en instalar el operador OCS y crear el objeto de clúster de almacenamiento. Además, si se conectan volúmenes locales a los nodos, se requiere la instalación del operador LSO.

El modelo de operador de Kubernetes permite a los administradores instalar un operador en RHOCP mediante la creación del grupo de operadores y la suscripción correspondientes.

Los siguientes son ejemplos del grupo de operadores y los objetos de suscripción, en formato YAML, para que los cree en un espacio de nombres openshift-local-storage preexistente:

```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-local-storage
  namespace: openshift-local-storage
spec:
  targetNamespaces:
    - openshift-local-storage
```

```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: local-storage-operator
  namespace: openshift-local-storage
spec:
  channel: "4.7"
  name: local-storage-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

**Instalación de Local Storage Operator**

Después de crear el grupo de operadores y los objetos de suscripción, debe crear los siguientes recursos personalizados para completar la instalación del LSO:

LocalVolumeDiscovery, para detectar los volúmenes conectados disponibles

LocalVolumeSet, para definir el modo de volumen (archivo o bloque)

A continuación, se muestran ejemplos de objetos de conjuntos de volúmenes locales y detección de volúmenes locales en formato YAML.

```
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeDiscovery
metadata:
  name: auto-discover-devices
  namespace: openshift-local-storage
spec
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: kubernetes.io/hostname
        operator: In
        values
        - worker01
        - worker02
        - worker03
```

Los objetos CR LocalVolumeDiscovery; LocalVolumeDiscoveryResults estarán disponibles en unos segundos. Puede recuperar los objetos LocalVolumeDiscoveryResults con el siguiente comando:

```
[user@demo ~]$ oc get localvolumediscoveryresults -n openshift-local-storage
NAME                        AGE
discovery-result-worker01   3s
discovery-result-worker02   2s
discovery-result-worker03   4s
```

Cuando finalice la detección, puede crear un LocalVolumeSet con los dispositivos detectados por LocalVolumeDiscovery.

```
apiVersion: local.storage.openshift.io/v1alpha1
kind: LocalVolumeSet
metadata:
  name: lso-volumeset
  namespace: openshift-local-storage
spec:
  deviceInclusionSpec:
    deviceTypes:
    - disk
    - part
    minSize: 1Gi
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: kubernetes.io/hostname
        operator: In
        values:
        - worker01
        - worker02
        - worker03
  storageClassName: lso-volumeset
  maxDeviceCount: 1
  volumeMode: Block
```

El ejemplo anterior usa solo un dispositivo de cada nodo trabajador para crear el conjunto de volúmenes locales.

**Instalación del operador Red Hat OpenShift Container Storage**

Para instalar el operador Red Hat OpenShift Container Storage, primero cree las etiquetas, el grupo de operadores y la suscripción correspondientes.

Suponiendo que el nombre del espacio de nombres es el predeterminado (openshift-storage), etiquételo para que use la solución de monitoreo de clúster disponible.

```
[user@demo ~]$ oc label namespace/openshift-storage \
  openshift.io/cluster-monitoring=
```

Como se mencionó anteriormente, primero debe crear el grupo de operadores y los objetos de suscripción, como se muestra en los siguientes ejemplos.

```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-storage
  namespace: openshift-storage
spec:
  targetNamespaces:
    - openshift-storage
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ocs-operator
  namespace: openshift-storage
spec:
  channel: "stable-4.7"
  name: ocs-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

Después de crear el grupo de operadores y los objetos de suscripción, cree el clúster de almacenamiento. Asumiendo una instalación en modo interno, el siguiente ejemplo de YAML crea el CR StorageCluster.

```
apiVersion: ocs.openshift.io/v1
kind: StorageCluster
metadata:
  name: ocs-storagecluster
  namespace: openshift-storage
spec:
  monDataDirHostPath: /var/lib/rook
  storageDeviceSets:
  - count: 1
    dataPVCTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: "1"
        storageClassName: lso-volumeset
        volumeMode: Block
    name: ocs-deviceset-lso-volumeset
    replica: 3
  version: 4.7.0
```

**Verificación de la instalación de Red Hat OpenShift Data Foundation**

Una vez finalizada la instalación, los administradores pueden verificar el estado de todos los objetos creados por los operadores LSO y OpenShift Data Foundation mediante la inspección de cada espacio de nombres.

El siguiente ejemplo es para el operador LSO y asume el espacio de nombres predeterminado openshift-local-storage.

```
[user@demo ~]$ oc get all -n openshift-local-storage
```

Verifique el estado del operador OpenShift Data Foundation, asumiendo que el espacio de nombres predeterminado es openshift-storage y tres nodos trabajadores que usan almacenamiento local.

```
[user@demo ~]$ oc get all -n openshift-storage
```

### Clases de almacenamiento provistas por Red Hat OpenShift Data Foundation
Red Hat OpenShift Data Foundation configura las siguientes clases de almacenamiento para proporcionar servicios de datos que soportan diferentes tecnologías de almacenamiento.

* ocs-storagecluster-ceph-rbd. Instalada por el operador Rook-Ceph, esta clase soporta dispositivos de almacenamiento en bloques, principalmente para cargas de trabajo de bases de datos. Los ejemplos incluyen la pila (stack) de registro y monitoreo de Red Hat OpenShift Container Platform (OCP) y PostgreSQL.

* ocs-storagecluster-cephfs. Instalada por el operador Rook-Ceph, esta clase proporciona servicios de datos de sistema de archivos compartidos y distribuidos, usados principalmente para el desarrollo de software, la mensajería y las cargas de trabajo de agregación de datos. Los ejemplos incluyen fuentes y artefactos de compilación de Jenkins, contenido cargado de WordPress, el registro de RHOCP y la mensajería con JBoss AMQ.

* openshift-storage.noobaa.io. Instalada por el operador NooBaa, esta clase proporciona el servicio Multicloud Object Gateway (MCG). En última instancia, el servicio MCG proporciona almacenamiento de objetos en múltiples nubes como un extremo (endpoint) de la API S3 que puede abstraer el almacenamiento y la recuperación de datos de múltiples almacenes de objetos en la nube.

* ocs-storagecluster-ceph-rgw. Instalada por el operador Rook-Ceph, esta clase proporciona almacenamiento de objetos en las instalaciones La interfaz de almacenamiento de objetos Ceph RADOS Gateway (Ceph RGW) es un extremo (endpoint) sólido de API S3 que se escala a decenas de petabytes y miles de millones de objetos, principalmente dirigidas a aplicaciones de uso intensivo de datos. Los ejemplos incluyen el almacenamiento y acceso de datos de filas, columnas y semiestructurados con aplicaciones como Spark, Presto, Red  Hat AMQ Streams (Kafka) y marcos (frameworks) de aprendizaje automático como TensorFlow y Pytorch.

### Necesidades de almacenamiento de los servicios de clúster de OpenShift

Además de gestionar el almacenamiento para las cargas de trabajo de las aplicaciones, OpenShift Data Foundation también satisface las tres necesidades principales de almacenamiento de los clústeres de producción OpenShift y Kubernetes.

Estas necesidades son las siguientes:

Persistencia de datos de la pila (stack) de monitoreo

Conservación de datos de la pila de registro (logging stack)

Provisión de registro interno de imágenes de OpenShift

La persistencia de los datos de la pila (stack) de monitoreo sirve para el análisis de causa raíz, big data, gestión de costos o estadísticas de carga. OpenShift Data Foundation proporciona servicios de almacenamiento en bloque mediante el almacenamiento de dispositivos de bloque Ceph RADOS (Ceph RBD). La conservación de los datos de la pila de registro (logging stack) sirve para la trazabilidad, la resolución de problemas y el análisis de la causa raíz. La provisión de un registro interno permite el acceso a imágenes y flujos de trabajo de imágenes usando un registro de Red Hat Quay, u otro registro de contenedores, a través de la puerta de enlace de almacenamiento de objetos Ceph (Ceph RGW).

OpenShift Data Foundation proporciona servicios de almacenamiento de archivos mediante el almacenamiento de sistemas de archivo Ceph (Ceph FS).

### Configuración del almacenamiento persistente para el registro interno RHOCP

Para configurar el almacenamiento para el registro, el recurso configs.imageregistry.operator.openshift.io ofrece el parámetro storage. Por ejemplo, use los siguientes parámetros para habilitar el almacenamiento S3:

```
storage:
  s3:
    bucket: <bucket-name>
    region: <region-name>
    regionEndpoint: <region-endpoint-name>
```

Además, el secreto image-registry-private-configuration-user está disponible en el openshift-image-registry namespace. Este secreto se usa para configurar el acceso y la gestión del almacenamiento. Las credenciales de este secreto anulan las credenciales predeterminadas usadas por el operador del registro de imágenes, si están configuradas.

Para configurar las credenciales de acceso al almacenamiento S3, el secreto image-registry-private-configuration-user debe contener dos claves:

* REGISTRY_STORAGE_S3_ACCESSKEY

* REGISTRY_STORAGE_S3_SECRETKEY

El siguiente comando crea el secreto image-registry-private-configuration-user:

```
[user@demo ~]$ oc create secret generic \
  image-registry-private-configuration-user \
  --from-literal=REGISTRY_STORAGE_S3_ACCESSKEY=myaccesskey \
  --from-literal=REGISTRY_STORAGE_S3_SECRETKEY=mysecretkey \
  --namespace openshift-image-registry
```

