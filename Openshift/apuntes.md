# Comandos Red Hat, Quay.io y Openshift 

## Inicie sesión en Red Hat Container Catalog con su cuenta de Red Hat.
    podman login registry.redhat.io

## Verificación del estado de los nodos de OpenShift

### Los siguientes comandos presentan información sobre el estado de los nodos de un clúster de OpenShift, y del propio cluster:

**oc get nodes**
Presenta una columna con el estado de cada nodo. Si un nodo no tiene el estado Ready (Listo), no puede comunicarse con el plano de control de OpenShift y es como si no existiera para el clúster.

**oc adm top nodes**
Presenta el uso actual de la CPU y la memoria que hace cada nodo. Estos son números de uso reales, no son estimaciones de la capacidad disponible y usada del nodo hechas por el planificador de OpenShift a partir de las solicitudes de recursos.

**oc describe node my-node-name**
Presenta los recursos disponibles y usados desde el punto de vista del planificador, más otros datos. Busque los títulos "Capacity" (Capacidad), "Allocatable" (Asignable) y "Allocated resources" (Recursos asignados) en la salida. El título "Conditions" (Condiciones) indica si el nodo tiene poca memoria, tiene poco espacio en disco o sufre alguna otra condición que le impida iniciar nuevos contenedores.

**oc get clusterversion**

Para consultar la versión del clúster. En la salida, se enumeran la versión, incluidas las versiones secundarias, el tiempo de actividad del clúster para una versión determinada y el estado general del clúster.

**oc describe clusterversion**

 Para obtener información más detallada sobre el estado del clúster.

**oc get clusteroperators**

 Permite acceder a la descripción general de todos los operadores del clúster y a información detallada de cada uno.

 **oc get storageclass**

Para ver las clases de almacenamiento disponibles.


[user@host ~]$ oc set volumes deployment/example-application \
>   --add --name example-pv-storage --type pvc --claim-class nfs-storage \
>   --claim-mode rwo --claim-size 15Gi --mount-path /var/lib/example-app \
>   --claim-name example-pv-claim

Para agregar un volumen a una aplicación

**oc delete pvc/example-pvc-storage**

Para eliminar un volumen

**oc adm policy add-cluster-role-to-user** \
>    cluster-admin admin

Para asignar al usuario admin el rol cluster-admin.

**oc login -u davidseg -p davidseg**

Para iniciar sesion con davidseg.

**oc delete user manager**

Para eliminar un usuario

### Administración de RBAC mediante la CLI

Los administradores de clústeres pueden usar el comando oc adm policy para agregar y quitar roles de clústeres y roles de espacio de nombres.

Para agregar un rol de clúster a un usuario, use el subcomando add-cluster-role-to-user:

**​oc adm policy add-cluster-role-to-user cluster-role username**

Por ejemplo, para convertir un usuario regular en administrador de clúster, use el siguiente comando:

**oc adm policy add-cluster-role-to-user cluster-admin username**

Para quitar un rol de clúster a un usuario, use el subcomando remove-cluster-role-from-user:

**​oc adm policy remove-cluster-role-from-user cluster-role username**

Por ejemplo, para convertir un administrador de clúster en usuario regular, use el siguiente comando:

**​oc adm policy remove-cluster-role-from-user cluster-admin username**

Las reglas se definen con una acción y un recurso. Por ejemplo, la regla create user forma parte del rol cluster-admin.

Puede usar el comando oc adm policy who-can para determinar si un usuario puede ejecutar una acción en un recurso. Por ejemplo:

**oc adm policy who-can delete user**

-----------------------------------------------

Cree un grupo denominado dev-group.

**oc adm groups new dev-group**

Agregue el usuario developer a dev-group.

**oc adm groups add-users dev-group developer**

Revise todos los grupos de OpenShift existentes para verificar que tengan los miembros correctos.

**oc get groups**

### Secretos

**Creación de un secreto**

Si un pod requiere acceso a información confidencial, cree un secreto para la información antes de implementar el pod. Use uno de los siguientes comandos para crear un secreto:

Cree un secreto genérico que contenga pares de clave-valor de valores literales escritos en la línea de comandos:

```
[user@host ~]$ ​oc create secret generic secret_name \
>    ​--from-literal key1=secret1 \
>    ​--from-literal key2=secret2
```

Cree un secreto genérico con los nombres de clave especificados en la línea de comandos y los valores de los archivos:

```
[user@host ~]$ ​oc create secret generic ssh-keys \
>    ​--from-file id_rsa=/path-to/id_rsa \
>    ​--from-file id_rsa.pub=/path-to/id_rsa.pub
```
Cree un secreto TLS especificando un certificado y la clave asociada:

```
[user@host ~]$ ​oc create secret tls secret-tls \
>    ​--cert /path-to-certificate --key /path-to-key
```

**Exposición de secretos a los pods**

Para exponer un secreto a un pod, primero cree el secreto. Asigne cada parte de los datos confidenciales a una clave. Después de su creación, el secreto contiene pares de clave-valor. El siguiente comando crea un secreto genérico denominado demo-secret con dos claves: user con el valor demo-user y root_password con el valor zT1KTgk.

```
[user@host ~]$ ​oc create secret generic demo-secret \
>    --from-literal user=demo-user
>    --from-literal root_password=zT1KTgk
```

**Secretos como variables de entorno de pod**

Considere una aplicación de base de datos que lea la contraseña del administrador de la base de datos desde la variable de entorno MYSQL_ROOT_PASSWORD. Modifique la sección de variables de entorno de la configuración de implementación para usar los valores del secreto:

env:
  - name: MYSQL_ROOT_PASSWORD 1
    valueFrom:
      secretKeyRef: 2
        name: demo-secret 3
        key: root_password 4

1

El nombre de la variable de entorno del pod, que contiene datos de un secreto.

2

La clave secretKeyRef espera un secreto. Use la clave configMapKeyRef para asignaciones de configuración.

3

El nombre del secreto que contiene la información confidencial deseada.

4

El nombre de la clave contiene la información confidencial del secreto.

------------------------------------------------------------------------

### Config Maps

**Crear un config-map**

```
[user@host ~]$ ​oc create configmap my-config \
>    ​--from-literal key1=config1 --from-literal key2=config2
```

**Actualización de los secretos y mapas de configuración**

En ocasiones, los secretos y mapas de configuración requieren actualizaciones. Use el comando oc extract para asegurarse de contar con los datos más recientes. Guarde los datos en un directorio específico con la opción --to. Cada clave en el mapa de secretos o de configuración crea un archivo con el mismo nombre que la clave. El contenido de cada archivo es el valor de la clave asociada. Si ejecuta el comando oc extract más de una vez, use la opción --confirm para sobrescribir los archivos existentes.

```
[user@host ~]$ ​oc extract secret/htpasswd-ppklq -n openshift-config \
>    --to /tmp/ --confirm
```

Después de actualizar los archivos guardados localmente, use el comando oc set data para actualizar el secreto o el mapa de configuración. Para cada clave que requiere una actualización, especifique el nombre de una clave y el valor asociado. Si un archivo contiene un valor, use la opción --from-file.

En el ejemplo oc extract, el secreto htpasswd-ppklq contenía solo una clave denominada htpasswd. Mediante el comando oc set data, puede especificar explícitamente el nombre de la clave htpasswd usando --from-file htpasswd=/tmp/htpasswd. Si no se especifica el nombre de la clave, el nombre del archivo se usa como nombre de la clave.

```
[user@host ~]$ ​oc set data secret/htpasswd-ppklq -n openshift-config \
>    --from-file /tmp/htpasswd
```

--Para demostrar cómo un secreto se puede montar como volumen, monte el secreto mysql en el directorio /run/secrets/mysql dentro del pod. En este paso solo se muestra cómo montar un secreto como un volumen, no es necesario para corregir la implementación.

```
[student@workstation ~]$ oc set volume deployment/mysql --add --type secret \
>   --mount-path /run/secrets/mysql --secret-name mysql
info: Generated volume name: volume-nrh7r
deployment.apps/mysql volume updated
```

Enumere los puntos de montaje en el pod que contiene el patrón mysql

**df -h | grep mysql**

Acceda a la API REST status de la aplicación para probar la conexión con la base de datos.

```
[student@workstation ~]$ curl -s http://quotes.apps.ocp4.example.com/status
Database connection OK
```

Para probar si la aplicación funciona, acceda al punto de entrada de la API REST random.

```
[student@workstation ~]$ curl -s http://quotes.apps.ocp4.example.com/random
8: Those who can imagine anything, can create the impossible.
- Alan Turing
```

### SCC

 **Para verificar si el uso de una SCC diferente puede resolver el problema de permisos**

```
[student@workstation ~]$ ​oc get pod/gitlab-7d67db7875-gcsjl -o yaml | \
>    oc adm policy scc-subject-review -f -
RESOURCE                      ALLOWED BY
Pod/gitlab-7d67db7875-gcsjl   anyuid
```

**Cree una cuenta de servicio denominada gitlab-sa**

```
[student@workstation ~]$ oc create sa gitlab-sa
serviceaccount/gitlab-sa created
```

**Asigne la SCC anyuid a la cuenta de servicio gitlab-sa**

```
[student@workstation ~]$ oc adm policy add-scc-to-user anyuid -z gitlab-sa
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "gitlab-sa"
```

**Asigne la cuenta de servicio gitlab-sa a la implementación de gitlab**

```
[student@workstation ~]$ oc set serviceaccount deployment/gitlab gitlab-sa
deployment.apps/gitlab serviceaccount updated
```

**Otorgue la SCC anyuid a la cuenta de servicio wordpress-sa**

 ```
[student@workstation ~]$ oc adm policy add-scc-to-user anyuid -z wordpress-sa
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "wordpress-sa"
```

### Servicios 

En la siguiente definición de YAML se muestra cómo se crea un servicio. Esto define el servicio application-frontend, que crea una IP virtual que expone el puerto TCP 443. La aplicación front-end escucha en el puerto no privilegiado 8843.

```
kind: Service
apiVersion: v1
metadata:
  name: application-frontend 1
  labels:
    app: frontend-svc 2
spec:
  ports: 3
    - name: HTTP 4
      protocol: TCP
      port: 443 5
      targetPort: 8443 6
  selector: 7
    app: shopping-cart
    name: frontend
  type: ClusterIP 8

```

1

El nombre del servicio. Este identificador le permite administrar el servicio después de su creación.

2

Una etiqueta que puede usar como selector. Esto le permite agrupar lógicamente sus servicios.

3

Una serie de objetos que describe los puertos de red que se exponen.

4

Cada entrada define el nombre para la asignación de puertos. Este valor es genérico y se usa únicamente para la identificación.

5

Este es el puerto que el servicio expone. Use este puerto para conectarse con la aplicación que el servicio expone.

6

Este es el puerto en el que escucha la aplicación. El servicio crea una regla de reenvío desde el puerto del servicio hasta el puerto de destino del servicio.

7

El selector define los pods que se encuentran en el conjunto (pool) del servicio. Los servicios usan este selector para determinar dónde enrutar el tráfico. En este ejemplo, el servicio apunta a todos los pods con las etiquetas app: shopping-cart y name: frontend.

8

Esta es la forma en que se expone el servicio. ClusterIP expone el servicio mediante el uso de una dirección IP interna al clúster y es el valor predeterminado. Otros tipos de servicio se describirán en otra parte del curso.

### SDN

**Creación de rutas**

La manera más fácil y preferida de crear una ruta (segura o insegura) es usar el comando oc expose service service, donde service corresponde a un servicio. Use la opción --hostname para proporcionar un nombre de host personalizado para la ruta.

```
[user@host ~]$ oc expose service api-frontend \
>    --hostname api.apps.acme.com
```

--En la siguiente lista, se muestra una definición mínima para una ruta:

```
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: a-simple-route 1
  labels: 2
    app: API
    name: api-frontend
spec:
  host: api.apps.acme.com 3
  to:
    kind: Service
    name: api-frontend 4
  port: 5
    targetPort: 8443
```

1

El nombre de la ruta. El nombre debe ser único.

2

Un conjunto de etiquetas que puede usar como selectores.

3

El nombre de host de la ruta. Este nombre de host debe ser un subdominio de su dominio comodín, dado que OpenShift enruta el dominio comodín a los enrutadores.

4

El servicio al que se redirigirá el tráfico. Si bien usa un nombre de servicio, la ruta solo usa esta información para determinar la lista de pods que reciben el tráfico.

5

El puerto de la aplicación. Dado que las rutas omiten los servicios, esto debe coincidir con el puerto de la aplicación y no con el puerto del servicio.

-----------------------------------------------------------------------------------

Antes de crear una ruta segura, debe generar un certificado de TLS. El siguiente comando muestra cómo crear una ruta perimetral (edge) segura con un certificado TLS:

 ```
[user@host ~]$ oc create route edge \
>    --service api-frontend --hostname api.apps.acme.com \
>    *--key api.key --cert api.crt  *1 2
```

1

La opción --key requiere la clave privada del certificado.

2

La opción --cert requiere el certificado que ha sido firmado con esa clave.

**Genere la clave privada para el certificado firmado por la CA**

```
[student@workstation certs]$ openssl genrsa -out training.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
...output omitted...
e is 65537 (0x010001)
```

--------------------------------------------------------------------------------

Para administrar la comunicación de red entre dos espacios de nombres, asigne una etiqueta al espacio de nombres que necesita acceso a otro espacio de nombres. El siguiente comando asigna la etiqueta name=network-1 al espacio de nombres network

 ```
[user@host ~]$ oc label namespace network-1 name=network-1
```

**Para ver las políticas de red en un espacio de nombres**

```
[student@workstation network-policy]$ oc get networkpolicies -n network-policy
NAME             POD-SELECTOR       AGE
allow-specific   deployment=hello   11s
deny-all         <none>             5m6s
```

 **Para asignar al espacio de nombres network-test la etiqueta name=network-test**

```
 [student@workstation network-policy]$ oc label namespace network-test \
>    name=network-test
namespace/network-test labeled
```

### Etiquetado de nodos

Use el comando oc label como administrador de clústeres para agregar, actualizar o quitar inmediatamente una etiqueta de nodo. Por ejemplo, use el siguiente comando para etiquetar un nodo con env=dev:

```
[user@host ~]$ ​oc label node master01 env=dev
```

Use la opción --overwrite para cambiar una etiqueta existente:

```
[user@host ~]$ ​oc label node master01 env=prod --overwrite
```

Quite una etiqueta especificando el nombre de la etiqueta seguido de un guión, como env-:

```
[user@host ~]$ ​oc label node master01 env-
```

### Límites de recursos

Se usan para evitar que un pod agote todos los recursos de cómputo de un nodo. El nodo que ejecuta un pod configura la característica cgroups (grupos de control) de kernel de Linux para aplicar los límites de recursos del pod.

Las solicitudes de recursos y los límites de recursos se deben definir para cada contenedor en un recurso de implementación o de configuración de implementación. Si no se han definido las solicitudes ni los límites, encontrará una línea resources: {} para cada contenedor.

Modifique la línea resources: {} para especificar las solicitudes o los límites que desee. Por ejemplo:

```
...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/hello-world-nginx:v1.0
        name: hello-world-nginx
        resources:
          requests:
            cpu: "10m"
            memory: 20Mi
          limits:
            cpu: "80m"
            memory: 100Mi
status: {}
```

El siguiente comando establece las mismas solicitudes y límites que el ejemplo anterior:

```
[user@host ~]$ oc set resources deployment hello-world-nginx \
>    --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
```

### Actualizar cluster

En los siguientes pasos, se describe el procedimiento para actualizar un clúster como administrador de clústeres mediante la interfaz de línea de comandos:

Asegúrese de actualizar todos los operadores instalados a través del gestor del ciclo de vida del operador (OLM) a la versión más reciente antes de actualizar el clúster de OpenShift.

Recupere la versión del clúster, revise la información del canal de actualización actual y confirme el canal. Si está ejecutando el clúster en producción, asegúrese de que el canal indique stable (estable).

```
[user@host ~]$ oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.10.3    True        False         43d     Cluster version is 4.10.3
```

```
[user@host ~]$ oc get clusterversion -o jsonpath='{.items[0].spec.channel}{"\n"}'
stable-4.10
```

Vea las actualizaciones disponibles y observe el número de versión de la actualización que desea aplicar.

```
[user@host ~]$ oc adm upgrade
Cluster version is 4.10.3

Updates:

VERSION IMAGE
4.10.4   quay.io/openshift-release-dev/ocp-release@sha256:...
...output omitted...
```

Aplique la última actualización de su clúster o actualice a una versión específica:

Ejecute el siguiente comando para instalar la última actualización disponible para su clúster.

```
[user@host ~]$ oc adm upgrade --to-latest=true
```

Ejecute el siguiente comando para instalar una versión específica. VERSION corresponde a una de las versiones disponibles que devuelve el comando oc adm upgrade.

```
[user@host ~]$ ​oc adm upgrade --to=VERSION
```

El comando anterior inicializa el proceso de actualización. Ejecute el siguiente comando para revisar el estado del operador de versión de clúster (CVO) y los operadores instalados del clúster.

```
[user@host ~]$ oc get clusterversion
NAME     VERSION  AVAILABLE  PROGRESSING  SINCE  STATUS
version  4.10.3   True       True         30m    Working towards 4.10.4 ...
```

```
[user@host ~]$ oc get clusteroperators
NAME                  VERSION   AVAILABLE   PROGRESSING   DEGRADED
authentication        4.10.3    True        False         False
cloud-credential      4.10.4    False       True          False
openshift-apiserver   4.10.4    True        False         True
...output omitted...
```







 