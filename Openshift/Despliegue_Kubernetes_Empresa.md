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

