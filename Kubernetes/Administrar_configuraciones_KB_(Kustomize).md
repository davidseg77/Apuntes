# Cómo administrar sus configuraciones de Kubernetes con Kustomize

## 1. Introducción

La implementación de aplicaciones en Kubernetes a veces puede resultar engorrosa. Implementas algunos Pods, respaldados por un Deployment, con accesibilidad definida en un Service. Todos estos recursos requieren archivos YAML para una definición y configuración adecuadas.

Además de esto, es posible que su aplicación necesite comunicarse con una base de datos, administrar contenido web o establecer la detalle del registro. Además, es posible que estos parámetros deban diferir según el entorno en el que esté implementando. Todo esto puede resultar en una extensa base de código de definiciones YAML, cada una con cambios de una o dos líneas que son difíciles de identificar.

Kustomize es una herramienta de gestión de configuración de código abierto desarrollada para ayudar a abordar estas preocupaciones. Desde Kubernetes 1.14, kubectl es totalmente compatible con Kustomize y kustomization archivos.

En esta guía, creará una pequeña aplicación web y luego utilizará Kustomize para administrar su expansión de configuración. Implementarás tu aplicación en entornos de desarrollo y producción con diferentes configuraciones. También superpondrás estas configuraciones variables utilizando las bases y superposiciones de Kustomize para que tu código sea más fácil de leer y, por lo tanto, de mantener.


## 2. Requisitos previos

Para este tutorial, necesitarás:

* Un clúster de Kubernetes con su configuración de conexión establecida como kubectl predeterminada. 
* kubectlinstalado en su máquina local. 


## 3. Implementar su aplicación sin Kustomize

Antes de implementar su aplicación con Kustomize, primero deberá implementarla de manera más tradicional. En este caso, implementará una versión de desarrollo de sammy-app una aplicación web estática alojada en Nginx. Almacenará su contenido web como datos en un ConfigMap, que montará en un Pod en una implementación. Cada uno de estos requerirá un archivo YAML independiente, que ahora creará.

Primero, cree una carpeta para su aplicación y todos sus archivos de configuración. Aquí es donde ejecutará todos los comandos de este tutorial.

Cree una nueva carpeta en su directorio de inicio y navegue dentro:

``` 
mkdir ~/sammy-app && cd ~/sammy-app
``` 

Ahora use su editor de texto preferido para crear y abrir un archivo llamado configmap.yml:

``` 
nano configmap.yml
``` 

Añade el siguiente contenido:

``` 
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: sammy-app
  namespace: default
data:
  body: >
    <html>
      <style>
        body {
          background-color: #222;
        }
        p {
          font-family:"Courier New";
          font-size:xx-large;
          color:#f22;
          text-align:center;
        }
      </style>
      <body>
        <p>DEVELOPMENT</p>
      </body>
    </html>
``` 

Esta especificación crea un nuevo objeto ConfigMap. Le estás asignando un nombre sammy-app y guardando contenido web HTML en su interior data:.

Guarde y cierre el archivo.

Ahora cree y abra un segundo archivo llamado deployment.yml:

``` 
nano deployment.yml
``` 

Añade el siguiente contenido:

``` 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
  labels:
    app: sammy-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sammy-app
  template:
    metadata:
      labels:
        app: sammy-app
    spec:
      containers:
      - name: server
        image: nginx:1.17
        volumeMounts:
          - name: sammy-app
            mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            cpu: 100m
            memory: "128M"
          limits:
            cpu: 100m
            memory: "256M"
        env:
        - name: LOG_LEVEL
          value: "DEBUG"
      volumes:
      - name: sammy-app
        configMap:
          name: sammy-app
          items:
          - key: body
            path: index.html
``` 

Esta especificación crea un nuevo objeto de implementación. Está agregando el nombre y la etiqueta de sammy-app, configurando el número de réplicas en 1 y especificando el objeto para usar la imagen del contenedor Nginx versión 1.17. También está configurando el puerto del contenedor en 80, definiendo las solicitudes y limitaciones de la CPU y la memoria, y configurando su nivel de registro en DEBUG.

Guarde y cierre el archivo.

Ahora implemente estos dos archivos en su clúster de Kubernetes. Para crear múltiples objetos desde stdin, canalice el cat comando a kubectl:

``` 
cat configmap.yml deployment.yml | kubectl apply -f -
``` 

Espere unos momentos y luego use kubectl para verificar el estado de su solicitud:

``` 
kubectl get pods -l app=sammy-app
``` 

Eventualmente verás un Pod con tu aplicación ejecutándose y 1/1 contenedores en la READY columna:

```
Output
NAME                         READY   STATUS    RESTARTS   AGE
sammy-app-56bbd86cc9-chs75   1/1     Running   0          8s
``` 

Su Pod se está ejecutando y está respaldado por una implementación, pero aún no puede acceder a su aplicación. Primero, necesitas agregar un Servicio.

Cree y abra un tercer archivo YAML llamado service.yml:

``` 
nano service.yml
```

Añade el siguiente contenido:

``` 
---
apiVersion: v1
kind: Service
metadata:
  name: sammy-app
  labels:
    app: sammy-app
spec:
  type: LoadBalancer
  ports:
  - name: sammy-app-http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: sammy-app
``` 

Esta especificación crea un nuevo objeto de Servicio llamado sammy-app. Para la mayoría de los proveedores de nube, configurar spec.typeen LoadBalancer aprovisionará un balanceador de carga. 

Guarde y cierre el archivo.

Ahora implemente el Servicio en su clúster de Kubernetes:

``` 
kubectl apply -f service.yml
``` 

Espere unos momentos y luego use kubectl para verificar el estado de su solicitud:

``` 
kubectl get services -w
``` 

Finalmente, aparecerá una IP pública para su Servicio debajo de la EXTERNAL-IP columna. Aparecerá una IP única en lugar de your_external_ip:

``` 
Output
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP         PORT(S)        AGE
kubernetes   ClusterIP      10.245.0.1       <none>              443/TCP        7h26m
sammy-app    LoadBalancer   10.245.186.235   <pending>           80:30303/TCP   65s
sammy-app    LoadBalancer   10.245.186.235   your_external_ip   80:30303/TCP   2m29s
``` 

Copie la dirección IP que aparece e ingrésela en su navegador web. Verá la DEVELOPMENT versión de su aplicación. 

Desde tu terminal, escribe CTRL + C para dejar de ver tus Servicios.

En este paso, implementó una versión de desarrollo sammy-app en Kubernetes. En los pasos 2 y 3, utilizará Kustomize para volver a implementar una versión de desarrollo sammy-app y luego implementar una versión de producción con configuraciones ligeramente diferentes. Con este nuevo flujo de trabajo, verá qué tan bien Kustomize puede gestionar los cambios de configuración y simplificar su flujo de trabajo de desarrollo.


## 4. Implementar su aplicación con Kustomize

En este paso, implementará exactamente la misma aplicación, pero en la forma que espera Kustomize en lugar de la forma predeterminada de Kubernetes.

Su sistema de archivos actualmente se ve así:

``` 
sammy-app/
├── configmap.yml
├── deployment.yml
└── service.yml
``` 

Para que esta aplicación se pueda implementar con Kustomize, debe agregar un archivo, kustomization.yml. Hazlo ahora:

``` 
nano kustomization.yml
``` 

Como mínimo, este archivo debe especificar qué recursos administrar cuando se ejecuta kubectl con la -k opción, que dirigirá kubectl el procesamiento del kustomization archivo .

Añade el siguiente contenido:

``` 
---
resources:
- configmap.yml
- deployment.yml
- service.yml
``` 

Guarde y cierre el archivo.

Ahora, antes de implementar nuevamente, elimine sus recursos de Kubernetes existentes del Paso 1:

``` 
kubectl delete deployment/sammy-app service/sammy-app configmap/sammy-app
``` 

Y impleméntelos nuevamente, pero esta vez con Kustomize:

``` 
kubectl apply -k .
``` 

En lugar de proporcionar la -f opción de kubectl indicarle a Kubernetes que cree recursos a partir de un archivo, usted proporciona -k un directorio (en este caso, . indica el directorio actual). Esto indica kubectl usar Kustomize e inspeccionar el archivo kustomization.yml.

Esto crea los tres recursos: ConfigMap, Deployment y Service. Utilice el get pods comando para comprobar su implementación:

``` 
kubectl get pods -l app=sammy-app
```

Volverá a ver un Pod con su aplicación ejecutándose y 1/1 contenedores en la READY columna.

Ahora vuelva a ejecutar el get services comando. También verá su Servicio con un acceso público EXTERNAL-IP:

``` 
kubectl get services -l app=sammy-app
``` 

Ahora está utilizando Kustomize con éxito para administrar sus configuraciones de Kubernetes. En el siguiente paso, implementará sammy-app en producción con una configuración ligeramente diferente. También utilizará Kustomize para gestionar estas variaciones.


## 5. Gestionar la variación de las aplicaciones con Kustomize

Los archivos de configuración para los recursos de Kubernetes realmente pueden comenzar a expandirse una vez que comienza a tratar con múltiples tipos de recursos, especialmente cuando existen pequeñas diferencias entre entornos (como desarrollo versus producción, por ejemplo). Es posible que tengas un deployment-development.yml y deployment-production.yml en lugar de solo un deployment.yml. La situación también podría ser similar para todos los demás recursos.

Imagínese lo que podría suceder cuando se publique una nueva versión de la imagen de Nginx Docker y desee comenzar a usarla. Quizás pruebes la nueva versión deployment-development.yml y quieras continuar, pero luego te olvides de actualizar deployment-production.yml con la nueva versión. De repente, estás ejecutando una versión de Nginx diferente en desarrollo que en producción. Pequeños errores de configuración como este pueden dañar rápidamente su aplicación.

Kustomize puede simplificar enormemente estos problemas de gestión. Recuerde que ahora tiene un sistema de archivos con sus archivos de configuración de Kubernetes y un kustomization.yml:

``` 
sammy-app/
├── configmap.yml
├── deployment.yml
├── kustomization.yml
└── service.yml
``` 

Imagine que ahora está listo para implementarlo sammy-app en producción. También decidió que la versión de producción de su aplicación diferirá de su versión de desarrollo en las siguientes formas:

* Replicas aumentarán de 1 a 3.
* El recurso del contenedor requests aumentará de 100m CPU y 128M memoria a 250m CPU y 256M memoria.
* El recurso del contenedor limits aumentará de 100m CPU y 256M memoria a 1 CPU y 1G memoria.
* La LOG_LEVEL variable de entorno cambiará de DEBUG a INFO.
* Los datos de ConfigMap cambiarán para mostrar contenido web ligeramente diferente.

Para comenzar, cree algunos directorios nuevos para organizar las cosas de una manera más específica de Kustomize:

``` 
mkdir base
``` 

Esto mantendrá su configuración “predeterminada”: su base . En su ejemplo, esta es la versión de desarrollo de sammy-app.

Ahora mueva su configuración actual sammy-app/ a este directorio:

``` 
mv configmap.yml deployment.yml service.yml kustomization.yml base/
``` 

Luego cree un nuevo directorio para su configuración de producción. Kustomize llama a esto superposición. Piense en las superposiciones como capas encima de la base; siempre requieren una base para funcionar:

``` 
mkdir -p overlays/production
``` 
Cree otro kustomization.yml archivo para definir su superposición de producción:

```
nano overlays/production/kustomization.yml
``` 

Añade el siguiente contenido:

``` 
---
bases:
- ../../base
patchesStrategicMerge:
- configmap.yml
- deployment.yml
``` 

Este archivo especificará un parámetro base para la superposición y qué estrategia utilizará Kubernetes para parchear los recursos. En este ejemplo, especificará un parche de estilo de fusión estratégica para actualizar ConfigMap y los recursos de implementación.

Guarde y cierre el archivo.

Y finalmente, agregue archivos nuevos deployment.yml y configmap.yml al overlays/production/ directorio.

Primero cree el nuevo deployment.yml archivo:

``` 
nano overlays/production/deployment.yml
``` 

Agregue lo siguiente a su archivo. Las secciones resaltadas indican cambios en su configuración de desarrollo:

``` 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: server
        resources:
          requests:
            cpu: 250m
            memory: "256M"
          limits:
            cpu: 1
            memory: "1G"
        env:
        - name: LOG_LEVEL
          value: "INFO"
``` 

Observe el contenido de esta noticia deployment.yml. Contiene solo los TypeMeta campos utilizados para identificar el recurso que cambió (en este caso, la implementación de su aplicación), y solo los campos restantes suficientes para ingresar a la estructura anidada y especificar un nuevo valor de campo, por ejemplo, las solicitudes y límites de recursos del contenedor. 

Guarde y cierre el archivo.

Ahora crea una nueva configmap.yml para tu superposición de producción:

``` 
nano /overlays/production/configmap.yml
```

Añade el siguiente contenido:

``` 
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: sammy-app
  namespace: default
data:
  body: >
    <html>
      <style>
        body {
          background-color: #222;
        }
        p {
          font-family:"Courier New";
          font-size:xx-large;
          color:#22f;
          text-align:center;
        }
      </style>
      <body>
        <p>PRODUCTION</p>
      </body>
    </html>
```

Aquí ha cambiado el texto que se mostrará PRODUCTION en lugar de DEVELOPMENT. Tenga en cuenta que también cambió el color del texto de un tono rojo #f22 a un tono azul #22f. Considere lo difícil que podría ser localizar y rastrear cambios tan pequeños si no estuviera utilizando una herramienta de gestión de configuración como Kustomize.

Su estructura de directorio ahora se ve así:

```
sammy-app/
├── base
│   ├── configmap.yml
│   ├── deployment.yml
│   ├── kustomization.yml
│   └── service.yml
└── overlays
    └── production
        ├── configmap.yml
        ├── deployment.yml
        └── kustomization.yml
``` 

Está listo para implementar usando su configuración base. Primero, elimine los recursos existentes:

``` 
kubectl delete deployment/sammy-app service/sammy-app configmap/sammy-app
``` 

Implemente su configuración base en Kubernetes:

``` 
kubectl apply -k base/
``` 

Inspeccione su implementación:

``` 
kubectl get pods,services -l app=sammy-app
``` 

Verá la configuración base esperada, con la versión de desarrollo visible en la página EXTERNAL-IP del Servicio:

``` 
Output
NAME                             READY   STATUS    RESTARTS   AGE
pod/sammy-app-5668b6dc75-rwbtq   1/1     Running   0          21s

NAME                TYPE           CLUSTER-IP       EXTERNAL-IP            PORT(S)        AGE
service/sammy-app   LoadBalancer   10.245.110.172   your_external_ip   80:31764/TCP   7m43s
``` 

Ahora implemente su configuración de producción:

``` 
kubectl apply -k overlays/production/
``` 

Inspeccione su implementación nuevamente:

``` 
kubectl get pods,services -l app=sammy-app
``` 

Verá la production configuración esperada, con la versión de producción visible en la página EXTERNAL-IP del Servicio:

``` 
Output
NAME                             READY   STATUS    RESTARTS   AGE
pod/sammy-app-86759677b4-h5ndw   1/1     Running   0          15s
pod/sammy-app-86759677b4-t2dml   1/1     Running   0          17s
pod/sammy-app-86759677b4-z56f8   1/1     Running   0          13s

NAME                TYPE           CLUSTER-IP       EXTERNAL-IP            PORT(S)        AGE
service/sammy-app   LoadBalancer   10.245.110.172   your_external_ip   80:31764/TCP   8m59s
``` 

Observe en la configuración de producción que hay 3 Pods en total en lugar de 1. Puede ver el recurso de Implementación para confirmar que los cambios menos aparentes también han tenido efecto:

``` 
kubectl get deployments -l app=sammy-app -o yaml
``` 

Visite your_external_ip en un navegador para ver la versión de producción de su sitio.

Ahora está utilizando Kustomize para gestionar la variación de la aplicación. Pensando en uno de sus problemas originales, si ahora quisiera cambiar la versión de la imagen de Nginx, solo necesitaría modificar deployment.yml en la base, y sus superposiciones que usan esa base también recibirán ese cambio a través de Kustomize. Esto simplifica enormemente su flujo de trabajo de desarrollo, mejora la legibilidad y reduce la probabilidad de errores.

**Conclusión**

En este tutorial, creó una pequeña aplicación web y la implementó en Kubernetes. Luego utilizó Kustomize para simplificar la administración de la configuración de su aplicación para diferentes entornos. Reorganizó un conjunto de archivos YAML casi duplicados en un modelo en capas. Esto reducirá los errores, reducirá la configuración manual y hará que su trabajo sea más reconocible y fácil de mantener.



