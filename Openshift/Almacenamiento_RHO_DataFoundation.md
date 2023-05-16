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

### Análisis de Ceph con OpenShift

Ceph es gestionado por el operador Rook-Ceph. Local Storage Operator (LSO) crea volúmenes persistentes que se pasan al operador Rook-Ceph para crear el clúster de Ceph.

Cada elemento de Ceph se ejecuta dentro de un pod.

```
[user@demo ~]$ oc get pods -n openshift-storage
```

### Investigación de un clúster de Ceph

Hay muchas maneras de verificar el estado de un clúster de Ceph, por ejemplo, mediante la revisión de los registros del clúster, el uso de las herramientas de cliente de Ceph o de la herramienta must-gather. Estas son algunas de las formas comunes en que puede examinar el estado del clúster y obtener información de solución de problemas en un clúster Ceph administrado por OpenShift Data Foundation.

**Inspección de registros de componentes de Ceph**

Revisar los registros suele ser una forma rápida de determinar el origen de un problema. Por lo general, un problema se presenta en forma de mensajes de error. Con Ceph, muchos problemas son de recuperación automática, pero si el clúster no se recupera automáticamente, es probable que vea mensajes de error cerca del final de los registros.

Esta sección proporciona información sobre cómo acceder a cada registro de Ceph.

* Registros de OSD

```
[user@demo ~]$ oc logs \
  rook-ceph-osd-0-6cbf78cd77-t7dnr -n openshift-storage
```

* Registros de monitoreo

```
[user@demo ~]$ oc logs \
  rook-ceph-mon-a-585ffd45d5-7vvlv -n openshift-storage
```

* Registros del operador Rook

```
[user@demo ~]$ oc logs \
  rook-ceph-operator-548bcdc79f-xcgjb -n openshift-storage
```

* Registros de almacenamiento de archivos CephFS

```
[user@demo ~]$ oc logs \
  csi-cephfsplugin-722b2 -n openshift-storage -c csi-cephfsplugin
```

* Registros de almacenamiento en bloque del complemento (plug-in) RBD

```
[user@demo ~]$ oc logs \
  csi-rbdplugin-86dpc -n openshift-storage \
  -c csi-rbdplugin
```

### Uso de las herramientas integradas de Ceph para recopilar información del clúster

Ceph proporciona la herramienta integrada must-gather y la CLI de Ceph. Estas herramientas proporcionan una forma rápida de obtener información y registros para su revisión.

**Uso de la CLI de Ceph**

Para usar la CLI de Ceph, ingrese al operador Rook-Ceph y use el archivo de configuración /var/lib/rook/openshift-storage/openshift-storage.config para acceder al clúster:

```
[user@demo ~]$ oc exec -ti \
  pod/rook-ceph-operator-548bcdc79f-xcgjb -n openshift-storage \
  -c rook-ceph-operator -- /bin/bash
```

Puede obtener mucha información diferente a través de la CLI, como se muestra en los siguientes ejemplos.

* Estado del clúster

```
bash-4.4$ ceph -c /var/lib/rook/openshift-storage/openshift-storage.config health
```

* Estado (status) del clúster

```
[user@demo ~]$ oc exec -ti \
  pod/rook-ceph-operator-548bcdc79f-xcgjb \
  -n openshift-storage -c rook-ceph-operator -- /bin/bash
```

* Uso del almacenamiento

```
bash-4.4$ ceph -c /var/lib/rook/openshift-storage/openshift-storage.config df
```

* Uso de la herramienta must-gather

Use la herramienta must-gather para recopilar archivos de registro e información de diagnóstico sobre su clúster.

```
[user@demo ~]$ oc adm must-gather \
  --image=registry.redhat.io/ocs4/ocs-must-gather-rhel8:v4.7 \
  --dest-dir=must-gather
```

### Configuración de una reclamación de volumen persistente

La clase de almacenamiento de archivos de OpenShift Data Foundation que se usa en las cargas de trabajo de las aplicaciones se especifica en los requisitos de almacenamiento dentro de las definiciones de recursos personalizadas. Configure la clase de almacenamiento de archivos ocs-storagecluster-cephfs para permitir que varios nodos accedan al volumen de almacenamiento estableciendo el valor accessModes en ReadWriteMany.

**Clases de almacenamiento de archivos de OpenShift Data Foundation**

OpenShift Data Foundation proporciona la clase de almacenamiento de archivos ocs-storagecluster-cephfs a través de CSI para CephFS.

```
[user@demo ~]$ oc get storageclasses.storage.k8s.io
```

Use el comando oc describe para ver los detalles de la clase de almacenamiento ocs-storagecluster-cephfs.

```
[user@demo ~]$ oc describe storageclass ocs-storagecluster-cephfs
```

### Ejemplo de reclamación de volúmenes persistentes para ceph

El siguiente archivo muestra un ejemplo de una definición de recurso personalizada para production-application con un volumen de 10Gi provisto por la clase de almacenamiento de archivos ocs-storagecluster-cephfs de OpenShift Data Foundation.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: production-application
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: ocs-storagecluster-cephfs
  resources:
    requests:
      storage: 10Gi
```

Observe que accessModes está configurado en ReadWriteMany para admitir el escenario común de compartir archivos centralizados con numerosas instancias de aplicaciones. Por ejemplo, esta configuración permite que varios servidores web compartan un conjunto de datos de almacenamiento de archivos común.

### Ejemplo de reclamación de volúmenes persistentes para almacenamiento en bloque

En el siguiente archivo se muestra un ejemplo de una definición de recurso personalizada para production-application con un volumen de 10Gi provisto por la clase de almacenamiento en bloque ocs-storagecluster-ceph-rbd de OpenShift Data Foundation.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: production-application
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ocs-storagecluster-ceph-rbd
  resources:
    requests:
      storage: 10Gi
```

Tenga en cuenta que accessModes está configurado en ReadWriteOnce para proporcionar seguridad e integridad de datos al permitir el acceso a un solo nodo. Por ejemplo, esta configuración es apropiada para el volumen de datos de una base de datos SQL que usa el almacenamiento en bloque.

### Administración de RBAC mediante la CLI

Los administradores de clústeres pueden usar el comando oc adm policy para agregar y quitar roles de clústeres y roles de espacio de nombres.

Para otorgar a un usuario un rol de clúster, use el subcomando add-cluster-role-to-user:

```
[user@host ~]$ oc adm policy add-cluster-role-to-user cluster-role username
```

Un usuario con el rol cluster-admin puede administrar completamente todos los conjuntos (pools) de almacenamiento, clases de almacenamiento y reclamaciones de volúmenes persistentes (PVC) en todos los proyectos.

```
[user@host ~]$ oc adm policy add-cluster-role-to-user cluster-admin username
```

**Roles predeterminados**

OpenShift proporciona un conjunto de roles de clúster predeterminados que se pueden asignar localmente o a todo el clúster. Puede modificar estos roles para controlar de forma detallada el acceso a los recursos de OpenShift, pero se requieren pasos adicionales que no se cubren este curso.

| Roles predeterminados |	Descripción |
|:--------:|:------------:|
| admin	| Los usuarios con este rol pueden administrar todos los recursos del proyecto, incluso otorgar acceso al proyecto a otros usuarios. |
| basic-user	| Los usuarios con este rol tienen acceso de lectura al proyecto. |
| cluster-admin |	Los usuarios con este rol tienen acceso de superusuario a los recursos del clúster. Estos usuarios pueden realizar cualquier acción en el clúster y tienen control total de todos los proyectos. |
| cluster-status	| Los usuarios con este rol pueden obtener información sobre el estado del clúster. |
| edit	| Los usuarios con este rol pueden crear, modificar y eliminar recursos de aplicaciones comunes del proyecto, como servicios e implementaciones. No pueden actuar sobre recursos de administración, como cuotas y rangos límite, y no pueden administrar permisos de acceso al proyecto. |
| self-provisioner |	Los usuarios con este rol pueden crear nuevos proyectos. Este es un rol de clúster, no un rol de proyecto. |
| view	| Los usuarios con este rol pueden ver los recursos del proyecto, pero no pueden modificar esos recursos. |

Agregue el rol a un usuario con el subcomando add-role-to-user.

```
[user@host ~]$ oc policy add-role-to-user role-name username -n project
```

Por ejemplo, agregue el usuario developer al rol admin en el proyecto custom-app, use el siguiente comando:

```
[user@host ~]$ oc policy add-role-to-user admin developer -n custom-app
```

Use el comando oc adm policy who-can para ver y verificar si un usuario puede crear ahora el recurso persistentvolumeclaims.

```
[user@host ~]$ oc adm policy who-can create persistentvolumeclaims -n custom-app
```





