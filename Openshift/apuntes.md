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









 