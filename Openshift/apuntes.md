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












 