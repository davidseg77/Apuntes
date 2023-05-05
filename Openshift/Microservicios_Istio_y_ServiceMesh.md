# Construcción de Microservicios con Istio Y Service Mesh

### Instalación del operador de OpenShift ElasticSearch

La instalación del operador de OpenShift Elasticsearch implica los siguientes pasos:

Cree un archivo YAML de objeto de suscripción para suscribir el espacio de nombres openshift-operators al operador de OpenShift Elasticsearch, por ejemplo, elasticsearch.yaml.

```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: elasticsearch-operator
  namespace: openshift-operators1
spec:
  channel: "4.6" 2
  name: elasticsearch-operator 3
  source: redhat-operators 4
  sourceNamespace: openshift-marketplace
  installPlanApproval: Automatic
```

1

Espacio de nombres usado para instalar el operador.

2

Flujo de versiones del operador.

3

Nombre del operador a suscribir.

4

Fuente que proporciona el operador.

Cree el objeto de suscripción aplicando el archivo YAML.

```
[user@host ~]$ oc apply -f elasticsearch.yaml
subscription.operators.coreos.com/elasticsearch-operator created
```

Compruebe el estado de la instalación del operador.

```
[user@host ~]$ oc describe sub elasticsearch-operator \
 -n openshift-operators
Name:         elasticsearch-operator
Namespace:    openshift-operators
...output omitted...
Message:               all available catalogsources are healthy
...output omitted...
```

### Creación del plano de control de OpenShift Service Mesh

Red Hat recomienda implementar el plano de control en un proyecto separado.

A continuación se describe cómo implementar el plano de control usando la CLI.

Inicie sesión en Red Hat OpenShift con el usuario developer.

```
[user@host ~]$ oc login -u USER  -p PASSWORD  RHT_OCP4_API
```

Cree un proyecto, por ejemplo, istio-system.

```
[user@host ~]$ ​oc new-project istio-system
Now using project "istio-system" on server "https://api.ocp4.example.com:6443".
...output omitted...
```

Cree un archivo YAML de objeto ServiceMeshControlPlane, por ejemplo, istio-basic-installation.yaml.

```
apiVersion: maistra.io/v2
kind: ServiceMeshControlPlane
metadata:
  name: basic 1
  namespace: istio-system 2
spec:
  gateways: 3
    egress:
      enabled: true
      runtime:
        deployment:
          autoScaling:
            enabled: false
    ingress:
      enabled: true
      runtime:
        deployment:
          autoScaling:
            enabled: false

  tracing: 4
    sampling: 10000
    type: Jaeger

  telemetry:
    type: Istiod

  policy:
    type: Istiod

  addons:
    grafana: 5
      enabled: true
    jaeger: 6
      install:
        storage:
          type: Memory
    kiali: 7
      enabled: true
```

1

Nombre asignado al plano de control.

2

Espacio de nombres donde se implementa el plano de control.

3

Configuración de puertas de enlace de Istio. Desactiva el escalado automático en las puertas de enlace de entrada (ingress) y salida (egress).

4

Configuración de rastreo. Selecciona Jaeger y la frecuencia de muestreo.

5

Configuración de Grafana. Permite a Grafana analizar y monitorear la malla de servicio.

6

Configuración de Jaeger. Habilita el almacenamiento en la memoria. ElasticSearch debe usarse en un entorno de producción.

7

Configuración de Kiali. Permite a Kiali visualizar el tráfico en la malla de servicio.

La configuración completa del plano de control está disponible en GitHub. Consulte código fuente.

Implemente el plano de control.

```
[user@host ~]$ oc create -n istio-system \
 -f istio-basic-installation.yaml
servicemeshcontrolplane.maistra.io/basic created
```

Compruebe el estado de la instalación del plano de control.

```
[user@host ~]$ oc get smcp -n istio-system
NAME    READY
basic   True
```

Debe crear una nueva ServiceMeshMemberRoll (Lista de miembros de la malla de servicio) para cada nueva instalación del plano de control.

Para crear una nueva ServiceMeshMemberRoll (Lista de miembros de la malla de servicio):

Cree un archivo YAML de objeto ServiceMeshControlPlane, por ejemplo, service-mesh-member-roll.yaml.

```
apiVersion: maistra.io/v1
kind: ServiceMeshMemberRoll
metadata:
  name: default
  namespace: istio-system
spec:
  members:
    - a-project
```

El plano de control gestiona proyectos listados como miembros.

Implemente ServiceMeshMemberRoll.

```
[user@host ~]$ oc create -n istio-system \
 -f service-mesh-member-roll.yaml
servicemeshmemberroll.maistra.io/default created
```

**Adición o eliminación de un proyecto del plano de control**

Solo los proyectos enumerados en ServiceMeshMemberRoll son administrados por la malla de servicio. Para agregar o eliminar un proyecto del plano de control:

Inicie sesión en Red Hat OpenShift Container.

Edite el recurso ServiceMeshMemberRoll.

```
[user@host ~]$ oc edit smmr -n istio-system
```

Modifique YAML para agregar o eliminar miembros del proyecto y guardar los cambios.


### Instalación de Red Hat OpenShift Service Mesh

OpenShift Service Mesh se instala mediante la consola web, o CLI, y un operador de Kubernetes. El proceso requiere primero la instalación de los operadores requeridos, luego la implementación del plano de control y, finalmente, la creación de una lista de miembros de la malla de servicio.

**Instalación del operador de OpenShift Service Mesh**

OpenShift Service Mesh se basa en los siguientes operadores:

- Jaeger
Proporciona funciones de rastreo para monitorear y solucionar problemas de la aplicación distribuida.

- ElasticSearch
Almacena los rastros (traces) y registros generados por Jaeger.

- Kiali
Proporciona observabilidad a la malla de servicio a través de una interfaz de usuario (IU) web.

Puede encontrar todos los operadores necesarios y el operador de Red Hat OpenShift Service Mesh en la página **OperatorHub**.

**Implementación del plano de control de OpenShift Service Mesh**

El plano de control gestiona la configuración y las políticas de la malla de servicio. La instalación de OpenShift Service Mesh Operator hace que el operador esté disponible en todos los espacios de nombres, por lo que puede instalar el plano de control en cualquier proyecto.

Para implementar un plano de control en un proyecto con la interfaz de usuario web, primero vaya a la página Installed Operators (Operadores instalados), luego, a la página Istio Service Mesh Control Plane (Plano de control de malla de servicio de Istio) y, finalmente, revise y configure los parámetros de implementación.

**Creación de una lista de miembros de la malla de servicio**

El recurso personalizado ServiceMeshMemberRoll define los proyectos que pertenecen a un plano de control.

Cualquier número de proyectos se puede agregar a ServiceMeshMemberRoll; sin embargo, un proyecto se puede agregar solo a un plano de control.

Para crear o editar una lista de miembros de la malla de servicio, primero vaya al proyecto donde Red Hat OpenShift Service Mesh está instalado, luego, vaya a la página Istio Service Mesh Member Roll (Lista de miembros de la malla de servicio de Istio) y, finalmente, revise y configure los parámetros de instalación.

### Visualización de rastros y tramos con la consola web de Jaeger

La consola web de Jaeger está instalada de forma predeterminada en Red Hat OpenShift Service Mesh y está estrechamente integrada con la consola web de OpenShift.

Para ver detalles sobre rastros (traces) y tramos (spans) en la consola de Jaeger, haga lo siguiente:

En la consola web de OpenShift, vaya a Networking (Red) → Routes (Rutas) y busque la ruta jaeger, que es la URL que aparece en la columna Location (Ubicación).

Use el mismo nombre de usuario y la misma contraseña que se usa para iniciar sesión en la consola web de OpenShift. Debería ver la página de inicio de la consola web de Jaeger.

En el panel izquierdo de la consola de Jaeger, en el menú Servicie (Servicio), seleccione su aplicación y haga clic en Find Traces (Buscar rastros) en la parte inferior del panel. Se muestra una lista de los rastros (traces) recopilados para la aplicación.

### Inyección automática del sidecar de Envoy

Para inyectar automáticamente el sidecar de Envoy en un servicio, debe especificar la notación sidecar.istio.io/inject con el valor establecido en "true" (verdadero) en el recurso Deployment (Implementación).

Ejemplo de inyección automática del sidecar:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: history
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: history
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
        - name: history
          image: quay.io/redhattraining/ossm-history:1.0
          ports:
            - containerPort: 8080
```

### Agregar el proyecto hello a la lista de miembros en el recurso ServiceMeshMemberRoll.

ServiceMeshMemberRoll está disponible en el proyecto istio-system.

Use el script add-project-to-smmr.sh para agregar el proyecto hello a la lista de miembros en el recurso ServiceMeshMemberRoll.

```
[student@workstation traffic-deploy]$ sh add-project-to-smmr.sh
servicemeshmemberroll.maistra.io/default patched
```

### Cree una puerta de enlace de entrada (ingress) para permitir el tráfico de entrada a la malla.

Examine el archivo gateway.yaml, en el que se describe el tráfico permitido para entrar en la malla.

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: hello-gateway1
spec:
  selector:
    istio: ingressgateway 2
  servers:
    - port: 3
        number: 80
        name: http
        protocol: HTTP
      hosts: 4
        - "*"
```

1

Nombre asignado a la configuración de la puerta de enlace.

2

Indica a qué implementaciones de puerta de enlace del proxy se aplican las reglas. En este caso, es el proxy de Envoy de la puerta de enlace de entrada (ingress).

3

Puerto y protocolo donde la puerta de enlace está escuchando las conexiones entrantes.

4

Los hosts expuestos por esta puerta de enlace; el "*" significa que este campo no se usa para filtrar el tráfico entrante.

Use el comando oc create para crear la configuración de la puerta de enlace de entrada (ingress).

```
[student@workstation traffic-deploy]$ oc create -f gateway.yaml
gateway.networking.istio.io/hello-gateway created
```

### Cree un servicio virtual para redirigir el tráfico de entrada (ingress) a la aplicación Vert.x.

Examine el archivo virtual-service.yaml, que enruta el tráfico de entrada (ingress) con la aplicación Vert.X.

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: hello-vs 1
spec:
  hosts:
  - "*"
  gateways:
  - hello-gateway 2
  http: 3
  - route: 4
    - destination:
        host: hello 5
        port: 6
          name: http-8080
          number: 8080
```

1

Nombre asignado a la configuración del servicio virtual.

2

Lista de puertas de enlace que deben aplicar las rutas. Este servicio virtual se aplica al tráfico configurado por la puerta de enlace hello-gateway.

3

Lista de reglas de ruta para el tráfico HTTP.

4

Ruta predeterminada, ya que no se definen condiciones de coincidencia. Esta ruta redirige todo el tráfico al destino especificado.

5

Regla de destino que envía el tráfico al servicio hello.

6

Regla de destino que envía el tráfico al puerto http-8080 del servicio hello.

Use el comando oc create para crear un servicio virtual.

```
[student@workstation traffic-deploy]$ oc create -f virtual-service.yaml
virtualservice.networking.istio.io/hello-vs created
```

**Exporte la URL de la puerta de enlace de entrada (ingress) a una variable de entorno llamada GATEWAY_URL.**

```
[student@workstation traffic-deploy]$ GATEWAY_URL=$(sh get-ingress-gateway-url.sh)
```

**Ejecute el comando curl en combinación con la variable GATEWAY_URL para confirmar el acceso a la aplicación desde el terminal.**

```
[student@workstation traffic-deploy]$ curl ${GATEWAY_URL}
Hello World!
```
**Visualice el tráfico de entrada (ingress) con Kiali**

Examine el script get-kiali-url.sh, que usa el comando oc para recopilar la URL de Kiali.

Exporte la URL de Kiali a una variable de entorno llamada KIALI_URL.

```
[student@workstation traffic-deploy]$ KIALI_URL=$(sh get-kiali-url.sh)
```

Abra la URL KIALI_URL en un navegador para acceder a Kiali.

```
[student@workstation traffic-deploy]$ firefox ${KIALI_URL} &
```

### Recupere la URL de la puerta de enlace de entrada (ingress) de Istio ejecutando el siguiente comando:

```
[student@workstation ~]$ ISTIO_GW=$(oc get route istio-ingressgateway \
 -n istio-system -o jsonpath="{.spec.host}{.spec.path}")
[student@workstation ~]$ echo $ISTIO_GW
istio-ingressgateway-istio-system.apps.ocp4.example.com
```

Abra un navegador web para comprobar que la aplicación funciona correctamente. Se accede a la aplicación Financial a través de la URL de la puerta de enlace de entrada (ingress) que se acaba de recuperar con la ruta /frontend adjunta. Puede usar su navegador favorito para abrir esa URL o el siguiente comando para abrir la URL en Firefox:

```
[student@workstation ~]$ firefox $ISTIO_GW/frontend &
```






