# Cómo configurar el escalador automático de pods horizontales de Kubernetes usando Metrics Server

## 1. Introducción

Kubernetes tiene como objetivo proporcionar resiliencia y escalabilidad. Lo logra implementando múltiples pods con diferentes asignaciones de recursos, para brindar redundancia a sus aplicaciones. Aunque puede aumentar y reducir sus propias implementaciones manualmente según sus necesidades, Kubernetes brinda soporte de primera clase para escalar bajo demanda, utilizando una función llamada Horizontal Pod Autoscaling. Es un sistema de circuito cerrado que aumenta o reduce automáticamente los recursos (pods de aplicaciones) según sus necesidades actuales. Usted crea un recurso HorizontalPodAutoscaler(o HPA) para cada implementación de aplicación que necesita ajuste de escala automático y deja que se encargue del resto automáticamente.

En un nivel alto, HPA hace lo siguiente:

* Mantiene un ojo en las métricas de solicitudes de recursos provenientes de las cargas de trabajo de su aplicación (Pods), consultando el servidor de métricas.
* Compara el valor de umbral objetivo que estableció en la definición de HPA con la utilización promedio de recursos observada para las cargas de trabajo de sus aplicaciones (CPU y memoria).
* Si se alcanza el umbral objetivo, HPA ampliará la implementación de su aplicación para satisfacer demandas más altas. De lo contrario, si está por debajo del umbral, se reducirá la implementación. Para ver qué lógica utiliza HPA para escalar la implementación de su aplicación, puede revisar la página de detalles del algoritmo en la documentación oficial.
* Debajo del capó, HorizontalPodAutoscaler hay un CRD (Custom Resource Definition) que impulsa un bucle de control de Kubernetes implementado a través de un controlador dedicado dentro de Control Plane su clúster. Usted crea un HorizontalPodAutoscaler manifiesto YAML dirigido a su aplicación Deployment y luego lo utiliza kubectl para aplicar el recurso HPA en su clúster.

Para funcionar, HPA necesita un servidor de métricas disponible en su clúster para extraer las métricas requeridas, como la utilización de CPU y memoria. Una opción sencilla es Kubernetes Metrics Server. El servidor de métricas funciona recopilando métricas de recursos de Kubelets y exponiéndolas a través del Kubernetes API Server escalador automático de pod horizontal. También se puede acceder a la API de métricas kubectl top si es necesario.

En este tutorial, podrás:

* Implemente Metrics Server en su clúster de Kubernetes.
* Aprenda a crear escaladores automáticos de pods horizontales para sus aplicaciones.
* Pruebe cada configuración de HPA, utilizando dos escenarios: carga de aplicaciones constante y variable.


## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un clúster de Kubernetes con control de acceso basado en roles (RBAC) habilitado. 

* La kubectl herramienta de línea de comandos instalada en su entorno local y configurada para conectarse a su clúster. 

* La herramienta de control de versiones Git disponible en tu entorno de desarrollo. 

* El administrador de paquetes Kubernetes Helm también está disponible en su entorno de desarrollo. 


## 3. Instalar Metrics Server a través de Helm

Comenzará agregando el metrics-server repositorio a sus helm listas de paquetes. Puedes usar helm repo add:

``` 
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server
```

A continuación, utilice helm repo update para actualizar los paquetes disponibles:

``` 
helm repo update metrics-server
Output
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "metrics-server" chart repository
Update Complete. ⎈Happy Helming!⎈
``` 

Ahora que ha agregado el repositorio a helm, podrá agregarlo metrics-server a sus implementaciones de Kubernetes. Puede escribir su propia configuración de implementación aquí, pero este tutorial seguirá el kit de inicio de Kubernetes de DigitalOcean , que incluye una configuración para metrics-server.

Para hacerlo, clone el repositorio Git del kit de inicio de Kubernetes:

``` 
git clone https://github.com/digitalocean/Kubernetes-Starter-Kit-Developers.git
``` 

La metrics-serverconfiguración se encuentra en Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/metrics-server-values-v3.8.2.yaml. Puede verlo o editarlo usando nano su editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/metrics-server-values-v3.8.2.yaml
```

Contiene algunos parámetros de stock. Tenga en cuenta que replicases un valor fijo 2.

```
## Starter Kit metrics-server configuration
## Ref: https://github.com/kubernetes-sigs/metrics-server/blob/metrics-server-helm-chart-3.8.2/charts/metrics-server
##

# Number of metrics-server replicas to run
replicas: 2

apiService:
  # Specifies if the v1beta1.metrics.k8s.io API service should be created.
  #
  # You typically want this enabled! If you disable API service creation you have to
  # manage it outside of this chart for e.g horizontal pod autoscaling to
  # work with this release.
  create: true

hostNetwork:
  # Specifies if metrics-server should be started in hostNetwork mode.
  #
  # You would require this enabled if you use alternate overlay networking for pods and
  # API server unable to communicate with metrics-server. As an example, this is required
  # if you use Weave network on EKS
  enabled: false
``` 

Consulte la página del gráfico de Metrics Server para obtener una explicación de los parámetros disponibles para metrics-server.

Después de revisar el archivo y realizar cambios, puede continuar con la implementación metrics-serverproporcionando este archivo junto con el helm installcomando:

``` 
HELM_CHART_VERSION="3.8.2"

helm install metrics-server metrics-server/metrics-server --version "$HELM_CHART_VERSION" \
  --namespace metrics-server \
  --create-namespace \
  -f "Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/metrics-server-values-v${HELM_CHART_VERSION}.yaml"
```

Esto se implementará metrics-server en su clúster de Kubernetes configurado:

``` 
Output
NAME: metrics-server
LAST DEPLOYED: Wed May 25 11:54:43 2022
NAMESPACE: metrics-server
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
***********************************************************************
* Metrics Server                                                      *
***********************************************************************
  Chart version: 3.8.2
  App version:   0.6.1
  Image tag:     k8s.gcr.io/metrics-server/metrics-server:v0.6.1
***********************************************************************
``` 

Después de la implementación, puede usar helm ls para verificar que metrics-server se haya agregado a su implementación:

``` 
helm ls -n metrics-server
``` 

``` 
Output
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
metrics-server  metrics-server  1               2022-02-24 14:58:23.785875 +0200 EET    deployed        metrics-server-3.8.2    0.6.1
```

A continuación, puede comprobar el estado de todos los recursos de Kubernetes implementados en el metrics-server espacio de nombres:

``` 
kubectl get all -n metrics-server
``` 

Según la configuración que implementó, tanto los valores deployment.apps como replicaset.apps deben contar 2 instancias disponibles.

``` 
Output
NAME                                  READY   STATUS    RESTARTS   AGE
pod/metrics-server-694d47d564-9sp5h   1/1     Running   0          8m54s
pod/metrics-server-694d47d564-cc4m2   1/1     Running   0          8m54s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/metrics-server   ClusterIP   10.245.92.63   <none>        443/TCP   8m54s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/metrics-server   2/2     2            2           8m55s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/metrics-server-694d47d564   2         2         2       8m55s
``` 

Ya lo ha implementado metrics-server en su clúster de Kubernetes. En el siguiente paso, revisará algunos de los parámetros de una definición de recurso personalizada de HorizontalPodAutoscaler.

## 4. Conociendo las HPA

Hasta ahora, sus configuraciones han utilizado un valor fijo para la cantidad de ReplicaSet instancias a implementar. En este paso, aprenderá cómo definir un CRD HorizontalPodAutoscaler para que este valor pueda crecer o reducirse dinámicamente.

Un HorizontalPodAutoscaler CRD típico se ve así:

``` 
crd.yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
``` 

Los parámetros utilizados en esta configuración son los siguientes:

* **spec.scaleTargetRef:** una referencia con nombre al recurso que se está escalando.
* **spec.minReplicas:** el límite inferior para la cantidad de réplicas a las que el escalador automático puede reducirse.
* **spec.maxReplicas:** El límite superior.
* **spec.metrics.type:** la métrica que se utilizará para calcular el recuento de réplicas deseado. Este ejemplo utiliza el Resourcetipo, que le indica al HPA que escale la implementación en función de CPU la utilización promedio (o de la memoria). averageUtilization se establece en un valor umbral de 50.
Tiene dos opciones para crear un HPA para la implementación de su aplicación:

- Utilice el kubectl autoscale comando en una implementación existente.
- Cree un manifiesto HPA YAML y luego utilícelo kubectl para aplicar cambios a su clúster.
Primero probará la opción n.° 1, utilizando otra configuración del kit de inicio de DigitalOcean Kubernetes. Contiene una implementación llamada myapp-test.yaml que demostrará HPA en acción creando una carga de CPU arbitraria.

Puede revisar ese archivo usando nano su editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/myapp-test.yaml
```

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-test
spec:
  selector:
    matchLabels:
      run: myapp-test
  replicas: 1
  template:
    metadata:
      labels:
        run: myapp-test
    spec:
      containers:
        - name: busybox
          image: busybox
          resources:
            limits:
              cpu: 50m
            requests:
              cpu: 20m
          command: ["sh", "-c"]
          args:
            - while [ 1 ]; do
              echo "Test";
              sleep 0.01;
              done
```

Tenga en cuenta las últimas líneas de este archivo. Contienen cierta sintaxis de shell para imprimir repetidamente "Prueba" cien veces por segundo, para simular la carga. Una vez que haya terminado de revisar el archivo, puede implementarlo en su clúster usando kubectl:

``` 
kubectl apply -f Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/myapp-test.yaml
```

A continuación, utilice kubectl autoscale para crear un HorizontalPodAutoscaler objetivo para la myapp-test implementación:

```
kubectl autoscale deployment myapp-test --cpu-percent=50 --min=1 --max=3
``` 

Tenga en cuenta los argumentos pasados ​​a este comando; esto significa que su implementación se escalará entre 1 réplicas 3 cada vez que la utilización de la CPU alcance 50 el porcentaje.

Puede comprobar si el recurso HPA se creó ejecutando kubectl get hpa:

``` 
kubectl get hpa
``` 

La TARGETS columna de salida eventualmente mostrará una cifra de current usage%/target usage%.

``` 
Output
NAME         REFERENCE                  TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
myapp-test   Deployment/myapp-test      240%/50%   1         3         3          52s
``` 

También puede observar los eventos registrados que genera un HPA usando kubectl describe:

``` 
kubectl describe hpa myapp-test
Output
Name:                                                  myapp-test
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 28 May 2022 10:10:50 -0800
Reference:                                             Deployment/myapp-test
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  240% (48m) / 50%
Min replicas:                                          1
Max replicas:                                          3
Deployment pods:                                       3 current / 3 desired
...
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  17s   horizontal-pod-autoscaler  New size: 2; reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  37s   horizontal-pod-autoscaler  New size: 3; reason: cpu resource uti
  ``` 

Este es el kubectl autoscale método. En un escenario de producción, normalmente debería utilizar un manifiesto YAML dedicado para definir cada HPA. De esta manera, puede realizar un seguimiento de los cambios enviando el manifiesto a un repositorio Git y modificarlo según sea necesario.

Verá un ejemplo de esto en el último paso de este tutorial. Antes de continuar, elimine la myapp-test implementación y el recurso HPA correspondiente:

``` 
kubectl delete hpa myapp-test
kubectl delete deployment myapp-test
``` 

## 5. Escalar aplicaciones automáticamente a través de Metrics Server

En este último paso, experimentará con dos formas diferentes de generar carga y escalado del servidor a través de un manifiesto YAML:

* Una implementación de aplicaciones que crea una carga constante al realizar algunos cálculos intensivos en la CPU.
* Un script de shell simula esa carga externa realizando llamadas HTTP sucesivas y rápidas para una aplicación web.
  
### 5.1 Prueba de carga constante

En este escenario, creará una aplicación de muestra implementada con Python, que realiza algunos cálculos intensivos de CPU. De manera similar al script de shell del último paso, este código Python se incluye en uno de los manifiestos de ejemplo del kit de inicio. Puedes abrir el constant-load-deployment-test.yaml usando nano o tu editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/constant-load-deployment-test.yaml
```

``` 
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: python-test-code-configmap
data:
  entrypoint.sh: |-
    #!/usr/bin/env python

    import math

    while True:
      x = 0.0001
      for i in range(1000000):
        x = x + math.sqrt(x)
        print(x)
      print("OK!")

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: constant-load-deployment-test
spec:
  selector:
    matchLabels:
      run: python-constant-load-test
  replicas: 1
  template:
    metadata:
      labels:
        run: python-constant-load-test
    spec:
      containers:
        - name: python-runtime
          image: python:alpine3.15
          resources:
            limits:
              cpu: 50m
            requests:
              cpu: 20m
          command:
            - /bin/entrypoint.sh
          volumeMounts:
            - name: python-test-code-volume
              mountPath: /bin/entrypoint.sh
              readOnly: true
              subPath: entrypoint.sh
      volumes:
        - name: python-test-code-volume
          configMap:
            defaultMode: 0700
            name: python-test-code-configmap
``` 

El código Python, que genera repetidamente raíces cuadradas arbitrarias, se destaca arriba. La implementación buscará una imagen de la ventana acoplable que aloja el tiempo de ejecución de Python requerido y luego la adjuntará ConfigMapa la aplicación Pod que aloja el script de Python de muestra que se mostró anteriormente.

Primero, cree un espacio de nombres separado para esta implementación (para una mejor observación), luego impleméntelo a través de kubectl:

``` 
kubectl create ns hpa-constant-load

kubectl apply -f Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/constant-load-deployment-test.yaml -n hpa-constant-load
```

``` 
Output
configmap/python-test-code-configmap created
deployment.apps/constant-load-deployment-test created
```

Verifique que la implementación se haya creado correctamente y que esté en funcionamiento:

```
kubectl get deployments -n hpa-constant-load
Output
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
constant-load-deployment-test   1/1     1            1           8s
``` 

A continuación, deberá implementar otro HPA en este clúster. Hay un ejemplo que coincide con este escenario en constant-load-hpa-test.yaml, que puede abrir con nanosu editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/constant-load-hpa-test.yaml -n hpa-constant-load
```

``` 
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: constant-load-test
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: constant-load-deployment-test
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
``` 

Implementarlo a través de kubectl:

``` 
kubectl apply -f Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/constant-load-hpa-test.yaml -n hpa-constant-load
```

Esto creará un HPA recurso dirigido a la implementación de Python de muestra. Puede comprobar el constant-load-test estado de HPA a través de kubectl get hpa:

``` 
kubectl get hpa constant-load-test -n hpa-constant-load
``` 

Tenga en cuenta la REFERENCE columna targeting constant-load-deployment-test, así como la TARGETS columna que muestra las solicitudes de recursos de CPU actuales versus el valor de umbral, como en el último ejemplo.

``` 
Output
NAME                 REFERENCE                                  TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
constant-load-test   Deployment/constant-load-deployment-test   255%/50%   1         3         3          49s
```

También puede observar que el REPLICAS valor de la columna aumentó de 1 a 3 para la implementación de la aplicación de muestra, como se indica en la especificación HPA CRD. Esto sucedió muy rápidamente porque la aplicación utilizada en este ejemplo genera una carga de CPU muy rápidamente. Como en el ejemplo anterior, también puedes inspeccionar los eventos HPA registrados usando kubectl describe hpa -n hpa-constant-load.

### 5.2 Prueba de carga externa

Un escenario más interesante y realista es observar dónde se crea la carga externa. Para este ejemplo final, utilizará un espacio de nombres y un conjunto de manifiestos diferentes para evitar reutilizar datos de la prueba anterior.

Este ejemplo utilizará la cita del servidor de muestra del momento. Cada vez que se realiza una solicitud HTTP a este servidor, envía una cotización diferente como respuesta. Creará carga en su clúster enviando solicitudes HTTP cada 1 ms. Esta implementación está incluida en quote_deployment.yaml. Revise este archivo usando nanosu editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/quote_deployment.yaml
```

``` 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: quote
  template:
    metadata:
      labels:
        app: quote
    spec:
      containers:
        - name: quote
          image: docker.io/datawire/quote:0.4.1
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 50Mi
            limits:
              cpu: 200m
              memory: 100Mi

---
apiVersion: v1
kind: Service
metadata:
  name: quote
spec:
  ports:
    - name: http
      port: 80
      targetPort: 8080
  selector:
    app: quote
``` 

Tenga en cuenta que esta vez el script de consulta HTTP real no está contenido en el manifiesto; este manifiesto solo proporciona una aplicación para ejecutar las consultas por ahora. Cuando haya terminado de revisar el archivo, cree el espacio de nombres de cotización y la implementación usando kubectl:

``` 
kubectl create ns hpa-external-load

kubectl apply -f Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/quote_deployment.yaml -n hpa-external-load
```

Verifique que la quote implementación de la aplicación y los servicios estén en funcionamiento:

``` 
kubectl get all -n hpa-external-load
Output
NAME                             READY   STATUS    RESTARTS   AGE
pod/quote-dffd65947-s56c9        1/1     Running   0          3m5s

NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/quote   ClusterIP   10.245.170.194   <none>        80/TCP    3m5s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/quote       1/1     1            1           3m5s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/quote-6c8f564ff        1         1         1       3m5s
``` 

A continuación, creará el archivo HPA para la quote implementación. Esto está configurado en quote-deployment-hpa-test.yaml. Revise el archivo en nanosu editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/quote-deployment-hpa-test.yaml
```

``` 
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: external-load-test
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: quote
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 20
``` 

Tenga en cuenta que en este caso hay un valor de umbral diferente establecido para la métrica de recursos de utilización de CPU (20%). También hay un comportamiento de escala diferente. Esta configuración altera el scaleDown.stabilizationWindowSeconds comportamiento y lo establece en un valor inferior de 60 segundos. Esto no siempre es necesario en la práctica, pero en este caso es posible que desee acelerar las cosas para ver más rápidamente cómo el escalador automático realiza la acción de reducción. De forma predeterminada, HorizontalPodAutoscaler tiene un período de enfriamiento de 5 minutos. Esto es suficiente en la mayoría de los casos y debería evitar fluctuaciones cuando se escalan las réplicas.

Cuando esté listo, impleméntelo usando kubectl:

``` 
kubectl apply -f Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/manifests/hpa/metrics-server/quote-deployment-hpa-test.yaml -n hpa-external-load
```

Ahora, verifique si el recurso HPA está en su lugar y activo:

``` 
kubectl get hpa external-load-test -n hpa-external-load
Output
NAME                 REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
external-load-test   Deployment/quote   1%/20%    1         3         1          108s
``` 

Finalmente, ejecutará las consultas HTTP reales, utilizando el script de shell quote_service_load_test.sh. La razón por la que este script de shell no se incorporó anteriormente en el manifiesto es para que pueda observarlo ejecutándose en su clúster mientras inicia sesión directamente en su terminal. Revise el script usando nano su editor de texto favorito:

``` 
nano Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/scripts/quote_service_load_test.sh
quote_service_load_test.sh
#!/usr/bin/env sh

echo
echo "[INFO] Starting load testing in 10s..."
sleep 10
echo "[INFO] Working (press Ctrl+C to stop)..."
kubectl run -i --tty load-generator \
    --rm \
    --image=busybox \
    --restart=Never \
    -n hpa-external-load \
    -- /bin/sh -c "while sleep 0.001; do wget -q -O- http://quote; done" > /dev/null 2>&1
echo "[INFO] Load testing finished."
```

Para esta demostración, abra dos ventanas de terminal independientes. En el primero, ejecute el quote_service_load_test.sh script de shell:

``` 
Kubernetes-Starter-Kit-Developers/09-scaling-application-workloads/assets/scripts/quote_service_load_test.sh
```

A continuación, en la segunda ventana, ejecute un kubectl comando de vigilancia usando la -w bandera en el recurso HPA:

``` 
kubectl get hpa -n hpa-external-load -w
``` 

Deberías ver la carga marcar hacia arriba y escalar automáticamente:

``` 
Output
NAME                 REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
external-load-test   Deployment/quote   1%/20%    1         3         1          2m49s
external-load-test   Deployment/quote   29%/20%   1         3         1          3m1s
external-load-test   Deployment/quote   67%/20%   1         3         2          3m16s
``` 

Puede observar cómo el escalador automático se activa cuando aumenta la carga e incrementa la quote réplica de implementación del servidor configurada a un valor más alto. Tan pronto como se detiene el script del generador de carga, hay un período de enfriamiento y, después de aproximadamente 1 minuto, el conjunto de réplicas se reduce al valor inicial de 1. Puede presionar Ctrl+C para finalizar el script en ejecución después de regresar a la primera ventana del terminal.


## 6. Conclusión

En este tutorial, implementó y observó el comportamiento del escalado automático de pod horizontal (HPA) utilizando Kubernetes Metrics Server en varios escenarios diferentes. HPA es un componente esencial de Kubernetes que ayuda a su infraestructura a manejar más tráfico según sea necesario.

Metrics Server tiene una limitación importante: no puede proporcionar ninguna métrica más allá del uso de CPU o memoria. Puede revisar más a fondo la documentación de Metrics Server para comprender cómo trabajar dentro de sus casos de uso. Si necesita escalar utilizando otras métricas (como el uso del disco o la carga de la red), puede usar Prometheus a través de un adaptador especial, llamado prometheus-adapter.

