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

 