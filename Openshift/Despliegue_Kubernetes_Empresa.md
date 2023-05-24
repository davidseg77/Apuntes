# Info del curso DO380 de Red Hat

### Consulta de recursos de OpenShift

**Extracción de información de recursos**

El siguiente comando de ejemplo muestra cómo enumerar los recursos de un lugar específico ubicado dentro de un mediante el comando:typenamespaceoc get type -n namespace -o json

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator -o json
```

El comando siguiente muestra cómo obtener información sobre el "esquema" de un objeto o sus propiedades:

```
[user@host ~]$ oc explain deployment.status.replicas
```

**Extracción de información de un único recurso**

Utilice el comando para recuperar un único objeto en el espacio de nombres especificado. Además, utilice plantillas JSONPath para extraer y dar formato al contenido de la información consultada.oc get type name -n namespace

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.status.availableReplicas}'
```

Iterar sobre las listas del recurso mediante un índice comodín:[*]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.status.conditions[*].type}'
```

Obtener un elemento específico en una lista utilizando la notación de indexación:[index]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.spec.template.spec.containers[0].name}'
```

Filtrar elementos de una lista mediante una expresión de filtro:[?(condition)]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator \
  -o jsonpath='{.status.conditions[?(@.type=="Available")].status}'
```

**Extraer una sola propiedad de varios recursos**

El comando devuelve un objeto JSON que contiene un objeto, que es una lista de todos los objetos consultados.oc getitems

A continuación se muestra un ejemplo de cómo enumerar una sola propiedad de muchos objetos:

```
[user@host ~]$ oc get route -n openshift-monitoring \
  -o jsonpath='{.items[*].spec.host}'
```

Se utiliza para imprimir propiedades específicas en formato tabular:oc get -o=custom-columns

```
[user@host ~]$ oc get pod --all-namespaces \
  -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

**Extracción de una sola propiedad con varios anidamientos de recursos**

El comando devuelve una lista de pods. Suministro de iteraciones sobre cada pod Cada pod contiene un objeto anidado, que es una lista de los contenedores del pod. El uso itera sobre todos los contenedores de todos los pods.oc get pods.items[*]spec.containers[*]

```
[user@host ~]$ oc get pods -A \
  -o jsonpath='{.items[*].spec.containers[*].image}'
```

**Extracción de varias propiedades en diferentes niveles de anidamiento de recursos**

Utilice la instrucción para realizar iteraciones más complejas. La instrucción se ejecuta para cada elemento en hasta alcanzar el .{range}{range x}…​template…​{end}templatexend

Utilícelo como alternativa a .{range}.items[*].property

```
[user@host ~]$ oc get pods -A \
  -o jsonpath='{range .items[*]}' \
  '{.metadata.namespace} {.metadata.creationTimestamp}{"\n"}'
```

**Uso del filtrado de etiquetas**

Utilice esta opción para ver etiquetas con .--show-labelsoc get

```
[user@host ~]$ oc get deployment -n openshift-cluster-storage-operator \
  --show-labels
```

Utilice el argumento para filtrar por etiqueta:-l label

```
[user@host ~]$ oc get deployment -n openshift-cluster-storage-operator \
  -l app=csi-snapshot-controller-operator -o name
```

### Programación de OpenShift CronJobs

CronJobs crea trabajos basados en un horario determinado. Utilice CronJobs para ejecutar automatización, como copias de seguridad o informes, que deben ejecutarse en un intervalo regular.

Cuando se utiliza CronJobs, el cronograma de ejecución se especifica en el formato cron estándar.

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: backup-cron
  namespace: database
spec:
  schedule: "1 0 * * *" 1
  jobTemplate: 2
    spec:
      activeDeadlineSeconds: 600
      parallelism: 2
      template:
        metadata:
          name: backup
        spec:
          serviceAccountName: backup_sa
          containers:
          - name: backup
            image: example/backup_maker:v1.2.3
          restartPolicy: OnFailure
```

1

La programación de ejemplo ejecuta el trabajo un minuto después de la medianoche.

2

Especifique un trabajo normal en el campo CronJob.jobTemplate

### Ejercicio guiado: Navegar por la API REST de OpenShift

**Intente una solicitud GET al servidor API de OpenShift sin autenticación.**

```
[student@workstation ~]$ curl -k https://api.ocp4.example.com:6443/api
```

**Autentíquese con el servidor OAuth de OpenShift para recuperar un token de acceso.**

Inicie sesión en OpenShift como usuario.admin

```
[student@workstation ~]$ oc login -u admin -p redhat \
  https://api.ocp4.example.com:6443
```

Busque la ruta al servidor OAuth de OpenShift mediante el comando y, a continuación, asígnela a una variable.oc

```
[student@workstation ~]$ oc get route -n openshift-authentication
NAME              HOST/PORT                               PATH   SERVICES          PORT   TERMINATION            WILDCARD
oauth-openshift   oauth-openshift.apps.ocp4.example.com          oauth-openshift   6443   passthrough/Redirect   None

[student@workstation ~]$ OAUTH_HOST=$(oc get route oauth-openshift \
  -n openshift-authentication -o jsonpath='{.spec.host}')

[student@workstation ~]$ echo $OAUTH_HOST
oauth-openshift.apps.ocp4.example.com
```

Autentíquese con el servidor OAuth mediante el comando. OpenShift responde con un código de respuesta HTTP 302 Redirect, pero no hay necesidad de seguirlo. El token se puede encontrar como parámetro en la URL.curlaccess_tokenLocation

```
[student@workstation ~]$ curl -u admin -kv "https://$OAUTH_HOST/oauth/\
  authorize?client_id=openshift-challenging-client&response_type=token"
```

Guarde el token en una variable.

```
[student@workstation ~]$ TOKEN=sha256~xvZ8SsTiA3jRIiEX9QMUOLdaZRUPqubLy2AiQbQGDb0
```

**Intente una solicitud GET al servidor API de OpenShift utilizando el token de acceso.**

```
[student@workstation ~]$ HEADER="Authorization: Bearer $TOKEN"
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/
```

**Recorra el grupo de API mediante solicitudes.v1curl**

Obtenga la lista de recursos de API disponibles en el grupo de API.v1

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/
```

A continuación, obtenga una lista de uno de los recursos, como .pods

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/v1/pods
```

Filtre los resultados para enumerar sólo los nombres que incluyen .jq

```
[student@workstation ~]$ curl -sk --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/v1/pods/ \
  | jq ".items[].metadata.name"
```

**Busque el punto de enlace de la API para los recursos mediante el comando y, a continuación, obtenga una lista de todos los hosts que usan el comando.Routeoccurl**

Utilice el comando para detectar el recurso y .oc explainKindAPI Version

```
[student@workstation ~]$ oc explain routes
```

Enumere las API disponibles para el grupo de versiones de API.route.openshift.io/v1

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/apis/route.openshift.io/v1
```

Enumere los hosts para todas las rutas utilizando los comandos and.curljq

```
[student@workstation ~]$ curl -sk --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/apis/route.openshift.io/v1/routes \
  | jq '.items[].spec.host'
```

### Operadores de instalación

**Instalación de un operador desde OLM mediante la CLI**

Consulte la lista de operadores disponibles.

```
[user@host ~]$ oc get packagemanifests
```

Inspeccione a un operador. Cada operador agrega nuevos tipos de recursos mediante definiciones de recursos personalizadas (CRD).

```
[user@host ~]$ oc describe packagemanifests file-integrity-operator
```

En la CLI, no hay una forma directa de obtener los canales del operador, por lo que necesita una consulta JSONPath.

```
[user@host ~]$ oc get packagemanifests file-integrity-operator \
  --output='jsonpath={range .status.channels[*]}{.name}{"\n"}{end}'
```

**Verificación del estado del operador**

Hay varias maneras de comprobar el estado del operador. El resto de esta sección proporciona información adicional acerca de la inspección de registros de eventos, objetos de suscripción e información de espacio de nombres para comprobar el estado de un operador.

Compruebe los registros del pod OLM para verificar la instalación del operador.

```
[user@host ~]$ oc logs pod/olm-operator-c5599dfd7-nknfx \
  -n openshift-operator-lifecycle-manager
```

Inspeccione el objeto de suscripción para comprobar el estado del operador.

```
[user@host ~]$ oc describe sub <subscription-name> -n <namespace>
...output omitted...
```

**Listar todos los operadores**

Enumere los operadores instalados comprobando las versiones del servicio de clúster. Para enumerar todos los operadores instalados, obtenga las versiones del servicio de clúster en todos los espacios de nombres.

```
[user@host ~]$ oc get csv -A
```

Los operadores se pueden suscribir a un espacio de nombres o a todos los espacios de nombres. Para enumerar los operadores administrados por OLM, enumere las suscripciones activas.

```
[user@host ~]$ oc get subs -A
```

Para ver el estado y los eventos de los recursos personalizados relacionados con un operador determinado, describa la implementación del operador.

```
[user@host ~]$ oc describe deployment.apps/file-integrity-operator | \
  grep -i kind
```

Compruebe los elementos del operador obteniendo todos los objetos del espacio de nombres del operador. Si el operador está instalado en todos los espacios de nombres, realice la consulta en el espacio de nombres y busque el nombre del operador para descubrir los elementos.openshift-operators

```
[user@host ~]$ oc get all -n openshift-file-integrity
```

Utilice el comando para ver eventos y registros de un operador.oc logs

```
[user@host ~]$ oc logs deployment.apps/file-integrity-operator
```

**Actualización de un operador desde OLM mediante la CLI**

Modifique el archivo YAML del operador y aplique los cambios deseados al objeto de suscripción.

```
[user@host ~]$ oc apply -f file-integrity-operator-subscription.yaml
```

**Eliminación de operadores**

Para quitar un operador del clúster, elimine los objetos de suscripción y versión del servicio de clúster.

Compruebe la versión actual del operador suscrito en el campo.currentCSV

```
[user@host ~]$ oc get sub <subscription-name> -o yaml | grep currentCSV
currentCSV: ...output omitted...
```

Elimine el objeto de suscripción. Utilice el valor obtenido del comando anterior para eliminar el objeto de versión del servicio de Cluster Server.

```
[user@host ~]$ oc delete sub <subscription-name>
[user@host ~]$ oc delete csv <currentCSV>
```

### Implementación de Jenkins en OpenShift mediante plantillas estándar

Las plantillas estándar definen parámetros y valores predeterminados para el tamaño de volumen persistente y los requisitos de memoria, entre otros. Las instancias pequeñas de Jenkins no requieren cambios en estos valores de parámetros predeterminados y se pueden implementar creando un proyecto y, a continuación, ejecutando:

```
[user@host ~]$ oc new-app --template jenkins-persistent
```

**Generación de tokens de API de Jenkins**

Para usar la API de Jenkins o la CLI de Jenkins, necesita un token de API. Ese token reemplaza la contraseña en un encabezado de autenticación HTTP BASIC, por ejemplo:

```
[user@host ~] curl --user 'jenkins-user:token' \
  https://jenkins-host/resource-path
```

### Configuración del proveedor de identidades LDAP

Para configurar un servidor LDAP como proveedor de identidades:

Cree un secreto con la contraseña de usuario administrador de IdM. Este secreto se utilizará para el proveedor de identidad LDAP:

```
[user@host ~]$ oc create secret generic ldap-secret \
  --from-literal=bindPassword=${LDAP_ADMIN_PASSWORD} \
  -n openshift-config
```

La comunicación TLS necesita validación de la autoridad de certificación, en este caso, la CA se obtiene del IdM, que se configura con el aula.

Cree un mapa de configuración que contenga el certificado raíz de la autoridad de certificación IdM para que OpenShift confíe en los certificados IdM:

```
[user@host ~]$ oc create configmap ca-config-map \
  --from-file=\
  ca.crt=<(curl http://idm.ocp-${GUID}.example.com/ipa/config/ca.crt) \
  -n openshift-config
```

Cree un archivo para modificar la configuración de OAuth de OpenShift agregando el proveedor de identidad LDAP y los demás elementos necesarios para una configuración adecuada.

```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ldapidp
    mappingMethod: claim
    type: LDAP
    ldap:
      attributes:
        id:
        - dn
        email:
        - mail
        name:
        - cn
        preferredUsername:
        - uid
      bindDN: "uid=admin,cn=users,cn=accounts,dc=ocp4,dc=example,dc=com"
      bindPassword:
        name: ldap-secret
      ca:
        name: ca-config-map
      insecure: false
      url: "ldaps://idm.ocp4.example.com/cn=users,cn=accounts,dc=ocp4,dc=example,dc=com?uid"
```

Aplique el recurso personalizado y espere hasta que se reinicien los pods del espacio de nombres para que el proveedor de identidades esté activo:openshift-authentication

```
[user@host ~]$ oc apply -f tmp/ldap-cr.yml
```

Compruebe que puede iniciar sesión con los usuarios de IdM. Si se agregan más usuarios al IdM, estarán disponibles inmediatamente y los pods de OAuth de OpenShift no se reiniciarán, a diferencia del caso de uso de HTPasswd.

```
[user@host ~]$ oc login -u admin -p ${LDAP_ADMIN_PASSWORD}
[user@host ~]$ oc whoami
```

Para comprobar si hay algún problema con la integración de IdM, compruebe el estado del pod y los registros de la implementación en el espacio de nombres.deployment.apps/oauth-openshiftopenshift-authentication

```
[user@host ~]$ oc get pods -n openshift-authentication
[user@host ~]$ oc logs deployment.apps/oauth-openshift
```

Cuando los usuarios inician sesión utilizando el servidor LDAP como proveedor de identidades, OpenShift crea nuevas entradas.useridentity

```
[user@host ~]$ oc get user
[user@host ~]$ oc get identity
```

En caso de que sea necesario, agregue el rol a los usuarios del IdM que necesitan privilegios de administración de clústeres.cluster-admin

```
[user@host ~]$ oc adm policy add-cluster-role-to-user cluster-admin admin
```

### Creación de configuraciones de máquina

Las configuraciones de la máquina se especifican con una etiqueta. Enumere las configuraciones de máquina para un rol específico mediante el argumento.machineconfiguration.openshift.io/role--selector

```
[user@host ~]$ oc get machineconfig \
  --selector=machineconfiguration.openshift.io/role=worker
```

**Escribir archivos personalizados**

Especifique el contenido de archivo personalizado, como la configuración de Systemd o los certificados TLS, en un formato de escape siguiendo el estándar de esquema de URL de datos. El contenido del archivo suele estar codificado en Base64 en archivos de encendido. Otro contenido, como las unidades Systemd o las claves ssh, no están codificadas en Base64. En un terminal, utilice los comandos y para codificar archivos o entradas estándar.base64base64 -d

**Ejemplo de MachineConfig de Journald**

```
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker 1
  name: 60-journald 2
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,VGVzdGl...E8gKItAo= 3
        filesystem: root
        mode: 0644
        path: /etc/systemd/journald.conf
``` 

1

Etiquete los recursos de configuración de la máquina por rol.

2

Prefijo el nombre con un número de dos dígitos que especifica cuándo aplicar la configuración, en relación con las configuraciones de máquina que pertenecen al mismo grupo de configuraciones de máquina.
 
3

Utilice el formato de URL de datos para incrustar contenido de archivo de escape (evacuado). La codificación Base64 es común para los archivos Ignition.

### Creación de grupos de configuración de máquina

En la siguiente especificación se muestra la creación de un grupo independiente para los nodos de aprendizaje automático (ml).MachineConfigPool

```
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
    name: ml
spec:
    machineConfigSelector:
        matchExpressions:
            - key: machineconfiguration.openshift.io/role
              operator: In
              values: [worker, ml] 1
    nodeSelector:
        matchLabels:
        node-role.kubernetes.io/ml: "" 2
```

1

Incluya ambas configuraciones y configuraciones de máquina.workerml

2

Aplicar a los nodos con la etiqueta.node-role.kubernetes.io/ml

### Administración de volúmenes persistentes de archivos de Azure

OpenShift tiene un aprovisionador integrado para volúmenes persistentes de archivos de Azure. Para usar volúmenes de archivos de Azure en OpenShift, primero cree un secreto que contenga las credenciales de su cuenta de Azure Storage. En segundo lugar, cree una clase de almacenamiento para cada nivel de Azure File que necesite para el clúster de OpenShift.

Cada definición de clase de almacenamiento contiene una referencia al secreto de credenciales de almacenamiento de Azure. OpenShift usa el aprovisionador de archivos de Azure para administrar todos los volúmenes persistentes de Azure File en el clúster. A continuación se muestra un ejemplo de definición de clase de almacenamiento de archivos de Azure:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-lrs 1
provisioner: kubernetes.io/azure-file 2
parameters:
  skuName: Standard_LRS 3
  storageAccount: azure_storage_account_name 4
  secretNamespace: my-secret-namespace 5
  secretName: azure-storage-credentials 6
```

1

Nombre de la clase de almacenamiento. Para solicitar este tipo de almacenamiento, una notificación de volumen persistente utiliza un valor de .StorageClassNameazurefile-lrs

2

Un valor de indica a OpenShift que use el aprovisionador de Azure Files para crear volúmenes persistentes. El aprovisionador de archivos de Azure usa los valores de la sección para crear volúmenes persistentes con características específicas.provisionerkubernetes.io/azure-fileparameters

3

El parámetro especifica un nivel de almacenamiento de archivos de Azure. El valor corresponde al almacenamiento estándar con redundancia local (LRS).skuNameStandard_LRS

4

El aprovisionador requiere un nombre de cuenta para autenticarse en la API de Azure Files.

5 6

El aprovisionador requiere credenciales confidenciales para autenticarse en la API de Azure Files. El aprovisionador recupera credenciales confidenciales de un recurso secreto y, a continuación, las usa para autenticarse en la API de Azure Files.

### Establecer la clase de almacenamiento como la clase de almacenamiento predeterminada.

Inicie sesión como usuario.admin

```
[student@workstation storage-file]$ oc login -u admin -p redhat
Login successful.

...output omitted...
```

Revise la definición de recursos de clase de almacenamiento. Compruebe que la clase de almacenamiento no es la clase de almacenamiento predeterminada.nfs-storagenfs-storage

```
[student@workstation storage-file]$ oc get storageclass nfs-storage -o yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
  creationTimestamp: "2020-05-28T19:33:31Z"
...output omitted...
```

La anotación se establece en un valor de .storageclass.kubernetes.io/is-default-classfalse

Mostrar y revisar el contenido del archivo de revisión.set-default-storageclass.yml

```
[student@workstation storage-file]$ cat set-default-storageclass.yml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

Utilice el contenido del archivo de revisión para establecer la clase de almacenamiento como la clase de almacenamiento predeterminada.set-default-storageclass.ymlnfs-storage

```
[student@workstation storage-file]$ oc patch storageclass nfs-storage \
  -p "$(cat set-default-storageclass.yml)"
storageclass.storage.k8s.io/nfs-storage patched
```

Compruebe que la clase de almacenamiento es la clase de almacenamiento predeterminada para el clúster. Compruebe también que solo una clase de almacenamiento está etiquetada como clase de almacenamiento predeterminada.nfs-storage

```
[student@workstation storage-file]$ oc get storageclasses
NAME                    PROVISIONER               RECLAIMPOLICY   ...
nfs-storage (default)   nfs-storage-provisioner   Delete          ...
```

La clase de almacenamiento es ahora la clase de almacenamiento predeterminada para el clúster.nfs-storage

### Definición de las definiciones de recursos personalizadas de la pila de supervisión de clústeres

El operador de supervisión del clúster implementa y gestiona el ciclo de vida de una pila basada en Prometheus para supervisar el clúster. El operador crea las siguientes cinco definiciones de recursos personalizados (CRD):

Prometheus para implementar instancias de Prometheus. Muestre todas las instancias de recursos mediante el siguiente comando:

```
[user@host ~]$ oc get prometheuses -A
```

ServiceMonitor Para administrar archivos de configuración que describen cómo y dónde extraer los datos de los servicios de aplicación. Muestre todas las instancias de recursos mediante el siguiente comando:

```
[user@host ~]$ oc get servicemonitors -A
```

PodMonitor Para administrar archivos de configuración para extraer datos de pods. Muestre todas las instancias de recursos mediante el siguiente comando:

```
[user@host ~]$ oc get podmonitors -A
```

Alertmanager para implementar y administrar instancias de Alert Manager. Muestre todas las instancias de recursos mediante el siguiente comando:

```
[user@host ~]$ oc get alertmanagers -A
```

PrometheusRule para administrar reglas. Las reglas corresponden a un conjunto de consultas, un filtro y cualquier otro campo. Muestre todas las instancias de recursos mediante el siguiente comando:

```
[user@host ~]$ oc get prometheusrules -A
```

El mapa de configuración en el espacio de nombres proporciona una vista combinada de todas las reglas de Prometheus.prometheus-k8s-rulefiles-0openshift-monitoring

**Asignación de roles de supervisión de clúster**

Asigne el rol de clúster a un usuario para permitir la visualización de alertas y métricas de clúster. Este rol permite a un usuario ver la sección de la consola web de OpenShift.cluster-monitoring-viewMonitoring

```
[user@host ~]$ oc adm policy add-cluster-role-to-user \
  cluster-monitoring-view USER
```

### Instalación del operador de Elasticsearch

El operador de Elasticsearch maneja la creación y las actualizaciones del clúster de Elasticsearch definido en el recurso personalizado de registro de clúster. Cada nodo del clúster de Elasticsearch se implementa con un PVC nombrado y administrado por el operador de Elasticsearch. Se crea un objeto único para cada nodo de Elasticsearch para garantizar que cada nodo de Elasticsearch tenga un volumen de almacenamiento propio.Deployment

El operador de Elasticsearch debe instalarse en un espacio de nombres que no sea para evitar posibles conflictos con métricas de otros operadores de la comunidad. Cree y use el espacio de nombres para el operador de Elasticsearch.openshift-operatorsopenshift-operators-redhat

```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
```

Cree un objeto para instalar el operador en todos los espacios de nombres y un objeto para suscribir el espacio de nombres al operador.OperatorGroupSubscriptionopenshift-operators-redhat

```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: "elasticsearch-operator"
  namespace: "openshift-operators-redhat"
spec:
  channel: "4.6"
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
```

Cree los objetos RBAC para conceder permiso a Prometheus para acceder al espacio de nombres.openshift-operators-redhat

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - pods
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
  - kind: ServiceAccount
    name: prometheus-k8s
    namespace: openshift-operators-redhat
---
```

Compruebe que el operador está disponible en cada espacio de nombres.

```
[user@host ~]$ oc get csv -A
NAMESPACE        NAME                                        ...  PHASE
default          elasticsearch-operator.4.6.0-202108311008   ...  Succeeded
kube-node-lease  elasticsearch-operator.4.6.0-202108311008   ...  Succeeded
kube-public      elasticsearch-operator.4.6.0-202108311008   ...  Succeeded
kube-system      elasticsearch-operator.4.6.0-202108311008   ...  Succeeded
...output omitted...
```

### Instalación del operador de registro de clústeres

El operador de registro de clústeres crea componentes detallados en el recurso personalizado de registro de clúster y actualiza la implementación en función de cualquier cambio en el recurso personalizado. Similar al operador de Elasticsearch, el operador de registro de clúster requiere: espacio de nombres, grupo de operadores y objetos de suscripción.

```
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
    - openshift-logging
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "4.6"
  name: cluster-logging
  source: redhat-operators
  sourceNamespace: openshift-marketplace
---
```

Confirme la instalación.

```
[user@host ~]$ oc get csv -n openshift-logging
NAME                                        ...  PHASE
clusterlogging.4.6.0-202108311008           ...  Succeeded
elasticsearch-operator.4.6.0-202108311008   ...  Succeeded
```

### Examinar los estados de los nodos de trabajo en buen estado

Recuperar estado del nodo:

```
[user@host ~]$ oc get nodes <NODE>
```

Obtenga información para un nodo a través de la salida para los procesos que se están ejecutando actualmente:top

```
[user@host ~]$ oc adm top node <NODE>
```

Compruebe si hay manchas aplicadas para un nodo:

```
[user@host ~]$ oc describe node <NODE> | grep -i taint
```

Además, inspeccione el nodo de trabajo a través de la consola web y los comandos de la CLI para familiarizarse con los estados operativos correctos y dónde encontrar información valiosa, como en la página Supervisión. Puede encontrar un enlace a la consola web de OpenShift para su clúster mediante el comando:

```
[user@host ~]$ oc whoami --show-console
```

