# Construcción de Microservicios con Istio Y Service Mesh

## 1. Instalación del operador de OpenShift ElasticSearch

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

## 2. Creación del plano de control de OpenShift Service Mesh

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

### 2.1 Adición o eliminación de un proyecto del plano de control

Solo los proyectos enumerados en ServiceMeshMemberRoll son administrados por la malla de servicio. Para agregar o eliminar un proyecto del plano de control:

Inicie sesión en Red Hat OpenShift Container.

Edite el recurso ServiceMeshMemberRoll.

```
[user@host ~]$ oc edit smmr -n istio-system
```

Modifique YAML para agregar o eliminar miembros del proyecto y guardar los cambios.


## 3. Instalación de Red Hat OpenShift Service Mesh

OpenShift Service Mesh se instala mediante la consola web, o CLI, y un operador de Kubernetes. El proceso requiere primero la instalación de los operadores requeridos, luego la implementación del plano de control y, finalmente, la creación de una lista de miembros de la malla de servicio.

### 3.1 Instalación del operador de OpenShift Service Mesh

OpenShift Service Mesh se basa en los siguientes operadores:

- Jaeger
Proporciona funciones de rastreo para monitorear y solucionar problemas de la aplicación distribuida.

- ElasticSearch
Almacena los rastros (traces) y registros generados por Jaeger.

- Kiali
Proporciona observabilidad a la malla de servicio a través de una interfaz de usuario (IU) web.

Puede encontrar todos los operadores necesarios y el operador de Red Hat OpenShift Service Mesh en la página **OperatorHub**.

### 3.2 Implementación del plano de control de OpenShift Service Mesh

El plano de control gestiona la configuración y las políticas de la malla de servicio. La instalación de OpenShift Service Mesh Operator hace que el operador esté disponible en todos los espacios de nombres, por lo que puede instalar el plano de control en cualquier proyecto.

Para implementar un plano de control en un proyecto con la interfaz de usuario web, primero vaya a la página Installed Operators (Operadores instalados), luego, a la página Istio Service Mesh Control Plane (Plano de control de malla de servicio de Istio) y, finalmente, revise y configure los parámetros de implementación.

### 3.3 Creación de una lista de miembros de la malla de servicio

El recurso personalizado ServiceMeshMemberRoll define los proyectos que pertenecen a un plano de control.

Cualquier número de proyectos se puede agregar a ServiceMeshMemberRoll; sin embargo, un proyecto se puede agregar solo a un plano de control.

Para crear o editar una lista de miembros de la malla de servicio, primero vaya al proyecto donde Red Hat OpenShift Service Mesh está instalado, luego, vaya a la página Istio Service Mesh Member Roll (Lista de miembros de la malla de servicio de Istio) y, finalmente, revise y configure los parámetros de instalación.

## 4. Visualización de rastros y tramos con la consola web de Jaeger

La consola web de Jaeger está instalada de forma predeterminada en Red Hat OpenShift Service Mesh y está estrechamente integrada con la consola web de OpenShift.

Para ver detalles sobre rastros (traces) y tramos (spans) en la consola de Jaeger, haga lo siguiente:

En la consola web de OpenShift, vaya a Networking (Red) → Routes (Rutas) y busque la ruta jaeger, que es la URL que aparece en la columna Location (Ubicación).

Use el mismo nombre de usuario y la misma contraseña que se usa para iniciar sesión en la consola web de OpenShift. Debería ver la página de inicio de la consola web de Jaeger.

En el panel izquierdo de la consola de Jaeger, en el menú Servicie (Servicio), seleccione su aplicación y haga clic en Find Traces (Buscar rastros) en la parte inferior del panel. Se muestra una lista de los rastros (traces) recopilados para la aplicación.

## 5. Inyección automática del sidecar de Envoy

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

## 6. Agregar el proyecto hello a la lista de miembros en el recurso ServiceMeshMemberRoll.

ServiceMeshMemberRoll está disponible en el proyecto istio-system.

Use el script add-project-to-smmr.sh para agregar el proyecto hello a la lista de miembros en el recurso ServiceMeshMemberRoll.

```
[student@workstation traffic-deploy]$ sh add-project-to-smmr.sh
servicemeshmemberroll.maistra.io/default patched
```

## 7. Cree una puerta de enlace de entrada (ingress) para permitir el tráfico de entrada a la malla.

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

## 8. Cree un servicio virtual para redirigir el tráfico de entrada (ingress) a la aplicación Vert.x.

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

### 8.1 Exporte la URL de la puerta de enlace de entrada (ingress) a una variable de entorno llamada GATEWAY_URL.

```
[student@workstation traffic-deploy]$ GATEWAY_URL=$(sh get-ingress-gateway-url.sh)
```

### 8.2 Ejecute el comando curl en combinación con la variable GATEWAY_URL para confirmar el acceso a la aplicación desde el terminal.

```
[student@workstation traffic-deploy]$ curl ${GATEWAY_URL}
Hello World!
```
### 8.3 Visualice el tráfico de entrada (ingress) con Kiali

Examine el script get-kiali-url.sh, que usa el comando oc para recopilar la URL de Kiali.

Exporte la URL de Kiali a una variable de entorno llamada KIALI_URL.

```
[student@workstation traffic-deploy]$ KIALI_URL=$(sh get-kiali-url.sh)
```

Abra la URL KIALI_URL en un navegador para acceder a Kiali.

```
[student@workstation traffic-deploy]$ firefox ${KIALI_URL} &
```

## 9. Recupere la URL de la puerta de enlace de entrada (ingress) de Istio ejecutando el siguiente comando:

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

## 10. Lanzamiento de canarios

Genere el lanzamiento canario. Implemente la versión 2 de la aplicación vertx-greet creando una nueva implementación. Una vez implementada la nueva versión, dirija el 20 % del tráfico a la nueva versión.

Haga una copia del archivo deployment-v1.yaml y nómbrela deployment-v2.yaml.

```
[student@workstation release-canary]$ cp deployment-v1.yaml deployment-v2.yaml
```

Modifique el archivo deployment-v2.yaml para introducir los cambios para la versión 2. Cambie el valor de metadata.name a vertx-greet-v2 y de spec.template.metadata.labels.version a v2. Por último, agregue la variable de entorno GREETING con el valor Hello Red Hat!.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vertx-greet-v2
spec:
  selector:
    matchLabels:
      app: vertx-greet
  replicas: 1
  template:
    metadata:
      labels:
        app: vertx-greet
        version: v2
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
        - name: vertx-greet
          image: quay.io/redhattraining/ossm-vertx-greet:1.0
          ports:
            - containerPort: 8080
          env:
          - name: GREETING
            value: "Hello Red Hat!"
```

Puede ver la implementación completa de la versión 2 en ~/DO328/solutions/release-canary/deployment-v2.yaml.

Implemente la versión 2 creando la nueva implementación.

```
[student@workstation release-canary]$ oc create -f deployment-v2.yaml
deployment.apps/vertx-greet-v2 created
```

Revise el archivo destination-rule.yaml proporcionado.

```
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: vertx-greet
spec:
  host: vertx-greet
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

El archivo define dos subconjuntos, uno para cada versión. Cada subconjunto representa una parte del tráfico e incluye un nombre y una etiqueta que asocia el subconjunto con la versión de implementación. La versión es la especificada por la propiedad spec.template.metadata.labels.version en cada recurso Deployment.

Cree una regla de destino con el archivo destination-rule.yaml.

```
[student@workstation release-canary]$ oc create -f destination-rule.yaml
destinationrule.networking.istio.io/vertx-greet created
```

Modifique el recurso VirtualService con el comando oc edit para enviar el 80 % del tráfico a v1 y el 20 % restante a v2. Asocie el destino existente con el subconjunto v1 y agregue una ponderación de 80. A continuación, agregue otro destino para el subconjunto v2 con una ponderación de 20.

```
[student@workstation release-canary]$ oc edit virtualservice vertx-greet
```

Aplique los cambios en el editor de texto.

```
...output omitted...
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  ...output omitted...
  name: vertx-greet
  ...output omitted...
spec:
  hosts:
  - "*"
  gateways:
  - vertx-greet-gateway
  http:
  - route:
    - destination:
        host: vertx-greet
        subset: v1
        port:
          number: 8080
      weight: 80
    - destination:
        host: vertx-greet
        subset: v2
        port:
          number: 8080
      weight: 20
...output omitted...
```

Guarde los cambios en el recurso y cierre el editor de texto.

Puede ver el servicio virtual completo en ~/DO328/solutions/release-canary/virtual-service-v1.yaml. Aplique la configuración con el comando oc apply.

Ejecute test_canary.py para comprobar que la porción de respuestas esperada para cada servicio corresponde a las ponderaciones configuradas en los pasos anteriores. Este script envía una secuencia de 50 solicitudes a la URL especificada y muestra el resultado. Desde un paso anterior, debe tener la URL de la ruta de la puerta de enlace almacenada en la variable de entorno GATEWAY_URL. Ejecute el script pasando esta variable como primer parámetro.

```
[student@workstation release-canary]$ ./test_canary.py $GATEWAY_URL
```

Espere hasta que termine el script.

```
...output omitted...
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello World!
Hello Red Hat!
...output omitted...
Total requests: 50
* 'Hello World!' responses: 42 (84.0%)
* 'Hello Red Hat!' responses: 8 (16.0%)
* Errors: 0 (0.0%)
```

## 11. Duplicación en OpenShift Service Mesh

OpenShift Service Mesh usa los recursos DestinationRule para definir subconjuntos (generalmente versiones de servicio) y la entrada de destino en los recursos VirtualService para enrutar las solicitudes entre subconjuntos. OpenShift Service Mesh proporciona la duplicación de tráfico mediante el uso de los mismos recursos DestinationRule y la introducción de una entrada de duplicación en la ruta VirtualService:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my_virtual_service
spec:
  hosts:
    - target_host
  http:
  - route:
    - destination:
        host: old_service_name
        subset: old_subset
    mirror: 1
      host: new_service_name 2
      subset: new_subset 3
```

1

La entrada duplicada define el servicio al que Istio está enviando copias de solicitud.

2

El nombre del servicio que recibe el tráfico duplicado.

3

El subconjunto de hosts que reciben el tráfico duplicado, como se define en DestinationRule.

## 12. Duplicación de un porcentaje del tráfico

Hay situaciones en las que no es necesario o deseable reflejar todo el tráfico en el nuevo servicio. Por ejemplo, cuando no se requiere mantener el estado más reciente del servicio o cuando reducir el tráfico entre servicios es más importante que probar todas las solicitudes. En esas situaciones, Istio y OpenShift Service Mesh permiten definir un porcentaje del tráfico duplicado:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my_virtual_service
spec:
  hosts:
    - target_host
  http:
  - route:
    - destination:
        host: old_service_name
        subset: old_subset
    mirror:
      host: new_service_name
      subset: new_subset
    mirror_percent: 10
```

## 13. Lanzamiento de errores HTTP

Por ejemplo, si desea descartar el 20 % de las conexiones a example-svc de otros servicios y devolver el error Bad Request (Mala solicitud), puede usar un código como este en el servicio virtual:

```
apiVersion: networking.istio.io/v1beta1
  kind: VirtualService
  metadata:
    name: example-vs
  spec:
    hosts:
    - example-svc
    http:
    - route:
      - destination:
          host: example-svc
          subset: v1
      fault: 1
        abort: 2
          percentage: 3
            value: 20.0
          httpStatus: 400 4
```

1

Objeto de configuración HTTPFaultInjection responsable de todas las fallas inyectadas en el servicio.

2

HTTPFaultInjection.Objeto de configuración Abort (Cancelar) responsable de la configuración de la inyección de error.

3

Porcentaje de las conexiones que se cancelarán.

4

El código de estado HTTP que se devolverá al cancelar.

## 14. Creación de demoras en los servicios

Por ejemplo, para agregar un retraso de 400 milisegundos al 10 % de las conexiones al servicio example-svc, puede usar lo siguiente:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: example-vs
spec:
  hosts:
  - example-svc
  http:
  - route:
    - destination:
        host: example-svc
        subset: v1
    fault: 1
      delay: 2
        percentage: 3
          value: 10.0
        fixedDelay: 400ms 4
```

1

Objeto de configuración HTTPFaultInjection responsable de las fallas inyectadas en el servicio.

2

El objeto de configuración HTTPFaultInjection.Delay responsable de la configuración de la inyección de demoras.

3

Porcentaje de las conexiones que se retrasarán.

4

Cantidad de tiempo para retrasar la conexión.

## 15. Configuración de tiempos de espera usando servicios virtuales

Los servicios virtuales le permiten configurar tiempos de espera para todo el tráfico enrutado a un servicio. Puede aplicar una configuración de tiempo de espera usando el campo timeout (tiempo de espera) en las reglas de ruta y asignando un valor medido en segundos.

En el siguiente ejemplo se muestra una configuración de tiempo de espera en un servicio virtual:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: a-service-vs
spec:
  hosts:
    - example-svc
  http:
    - route:
        - destination:
          host: preference
      timeout: 1s
```

En el ejemplo anterior, Envoy espera hasta 1 segundo en cualquier llamada al servicio example-svc antes de devolver un error de tiempo de espera.

## 16. Configuración de reintentos usando servicios virtuales

Puede configurar el patrón de reintento en el recurso del servicio virtual, por ejemplo:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: example-vs
spec:
  hosts:
  - example-svc
  http:
  - route:
    - destination:
        host: example-svc
        subset: v1
    retries: 1
      attempts: 3 2
      perTryTimeout: 2s 3
      retryOn: 5xx,retriable-4xx 4
```

1

El objeto HTTPRetry responsable de configurar los reintentos.

2

El número de veces que se reenvía una solicitud.

3

Un valor de tiempo de espera para cada solicitud de reintento. Los valores válidos están en milisegundos (ms), segundos (s), minutos (m) u horas (h).

4

Una política que especifica las condiciones que provocan que se vuelvan a intentar las solicitudes con errores. El valor es una lista de valores separados por comas.

## 17. Configuración de interruptores en OpenShift Service Mesh

Para habilitar un interruptor, incluya una entrada outlierDetection en el recurso DestinationRule relacionado con el servicio:

```
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myDestinationRule
spec:
  host: myService 1
  trafficPolicy: 2
    outlierDetection:
      consecutive5xxErrors: 1 3
      interval: 1s 4
      baseEjectionTime: 3m 5
      maxEjectionPercent: 100 6
```

1

host no se refiere al host físico, sino al nombre del servicio. Consulte las referencias para obtener más detalles.

2

La entrada outlierDetection pertenece al objeto trafficPolicy.

3

Define cuántos errores 5xx se permiten antes de desalojar el host.

4

El intervalo de tiempo entre la comprobación de los recuentos de errores.

5

La cantidad mínima de tiempo de expulsión del host.

6

El porcentaje máximo de hosts desalojados que pertenece a un servicio en cualquier momento.

