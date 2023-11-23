# Cómo implementar el seguimiento distribuido con Jaeger en Kubernetes

# 1. Introducción

Kubernetes y las arquitecturas de microservicios que habilita pueden crear sistemas muy eficientes y escalables. Pero los problemas surgen cuando uno de estos microservicios desarrolla problemas de rendimiento. Normalmente, lo primero que notamos es que los tiempos de respuesta de nuestros servicios de atención al cliente son cada vez más largos. El problema podría estar en uno de los servicios backend, o quizás en una base de datos que está más allá de su capacidad óptima. Para descubrir la raíz de nuestro problema, necesitamos implementar el rastreo distribuido .

Jaeger es una solución de seguimiento distribuida y se graduó del Proyecto de Incubación de la Cloud Native Computing Foundation. Cuenta con una interfaz de usuario agradable para visualizar rastros, sidecars Jaeger para recopilar rastros y varios otros componentes. Los sistemas de seguimiento distribuido como Jaeger nos permiten rastrear el ciclo de vida de cada evento generado por el cliente y ver cómo cada servicio procesa ese evento.

En este tutorial, implementaremos una aplicación distribuida muy pequeña en un clúster de Kubernetes y simularemos un retraso en el rendimiento utilizando una función de suspensión en nuestro código. Para encontrar la causa raíz de este problema y rastrear cada evento, usaremos Jaeger. Con el rastreo habilitado, veremos qué tan efectivo es para observar el comportamiento de los Servicios e identificar problemas.


## 2. Requisitos previos

Antes de comenzar, necesitará las siguientes herramientas y cuentas:

* Un clúster de Kubernetes con su configuración de conexión establecida como kubectl predeterminada. 
* Docker instalado. 
* Una cuenta en Docker Hub para almacenar su imagen de Docker.
* La kubectl herramienta de línea de comandos instalada en su máquina local y configurada para conectarse a su clúster. 
* La curl utilidad de línea de comandos instalada en su máquina local. 


## 3. Creación de la aplicación de muestra

Para probar las capacidades de rastreo de Jaeger, crearemos e implementaremos una aplicación de muestra, sammy-jaeger que utiliza dos servicios: uno para el frontend y otro para el backend. Construiremos ambos usando Python y el microframework Flask.

Nuestra aplicación será un contador de visitas cuyo valor aumenta cada vez que llamamos al frontend. Para simular problemas de rendimiento, codificaremos una función de suspensión aleatoria que se ejecuta cada vez que el frontend envía una GET solicitud al backend. En este paso, crearemos e implementaremos esa aplicación. En los siguientes pasos, implementaremos la aplicación en Kubernetes, instalaremos Jaeger y luego la usaremos para rastrear nuestro problema de servicio.

Primero, creemos una estructura de directorios del proyecto y naveguemos dentro:

``` 
mkdir -p ./sammy-jaeger/frontend ./sammy-jaeger/backend && cd ./sammy-jaeger
``` 

Ahora tenemos un directorio raíz sammy-jaeger y dos subdirectorios:

```
output
.
├── backend
└── frontend
``` 

También cambiamos al directorio raíz, /sammy-jaeger. Ejecutaremos todos los comandos restantes desde aquí.

Comencemos a construir la aplicación frontend.

### 3.1 Construyendo la aplicación front-end

Usando su editor de texto preferido, cree y abra un nuevo archivo llamado frontend.py en ./frontend:

``` 
nano ./frontend/frontend.py
``` 

Agregue el siguiente código. Esto importará Flask, creará nuestras funciones de contador y definirá una ruta para las solicitudes HTTP:

``` 
import os
import requests
from flask import Flask
app = Flask(__name__)

def get_counter(counter_endpoint):
    counter_response = requests.get(counter_endpoint)
    return counter_response.text

def increase_counter(counter_endpoint):
    counter_response = requests.post(counter_endpoint)
    return counter_response.text

@app.route('/')
def hello_world():
    counter_service = os.environ.get('COUNTER_ENDPOINT', default="https://localhost:5000")
    counter_endpoint = f'{counter_service}/api/counter'
    counter = get_counter(counter_endpoint)

    increase_counter(counter_endpoint)

    return f"""Hello, World!

You're visitor number {counter} in here!\n\n"""
``` 

Estamos importando tres módulos. El os módulo se comunicará con nuestro sistema operativo. El requests módulo es una biblioteca para enviar solicitudes HTTP. Flask es un microframework que alojará nuestra aplicación.

Luego estamos definiendo nuestras funciones get_counter() y increase_counter(), que aceptan el parámetro counter_endpoint. get_counter()llamará al backend usando el GET método para encontrar el estado actual del contador. increase_counter()llamará al backend con el POST método para incrementar el contador.

Luego definimos nuestra ruta /, que llamará a otra función llamada hello_world(). Esta función recuperará una URL y un puerto para nuestro módulo de backend, lo asignará a una variable y luego pasará esa variable a nuestras dos primeras funciones get_counter() y increase_counter(), que enviarán las solicitudes GET y POST al backend. Luego, el backend se detendrá durante un período de tiempo aleatorio (nuestro retraso simulado) antes de incrementar el número del contador actual y luego devolver ese número. Por último, hello_world()tomará este valor e imprimirá un "¡Hola mundo!" cadena a nuestra consola que incluye nuestro nuevo recuento de visitantes.

Es posible que hayas notado que no creamos un entorno Python ni lo instalamos pip en nuestra máquina local. Completaremos estos pasos cuando incluyamos nuestra aplicación en contenedores usando Docker.

Guardar y cerrar frontend.py.

Ahora crearemos un Dockerfile para la aplicación frontend. Este Dockerfile incluirá todos los comandos necesarios para construir nuestro entorno en contenedores.

Crea y abre una nueva Dockerfile en ./frontend:

``` 
nano ./frontend/Dockerfile
``` 

Añade el siguiente contenido:

``` 
FROM alpine:3.8

RUN apk add --no-cache py3-pip python3 && \
    pip3 install flask requests

COPY . /usr/src/frontend

ENV FLASK_APP frontend.py

WORKDIR /usr/src/frontend

CMD flask run --host=0.0.0.0 --port=8000
``` 

En esto Dockerfile, le indicamos a nuestra imagen que se construya a partir de la imagen base de Alpine Linux. Luego instalamos Python3, pip y varias dependencias adicionales. A continuación, copiamos el código fuente de la aplicación, configuramos una variable de entorno que apunta al código principal de la aplicación, configuramos el directorio de trabajo y escribimos un comando para ejecutar Flask cada vez que creamos un contenedor a partir de la imagen.

Guarde y cierre el archivo.

Ahora vamos a crear la imagen de Docker para nuestra aplicación frontend y enviarla a un repositorio en Docker Hub.

Primero, verifique que haya iniciado sesión en Docker Hub:

``` 
docker login --username=your_username --password=your_password
``` 

Construye la imagen:

``` 
docker build -t your_username/do-visit-counter-frontend:v1 ./frontend
``` 

Ahora envíe la imagen a Docker Hub:

```
docker push your_username/do-visit-counter-frontend:v1
``` 

Nuestra aplicación frontend ahora está construida y disponible en Docker Hub. Sin embargo, antes de implementarlo en Kubernetes, codifiquemos y construyamos nuestra aplicación backend.

### 3.2 Construyendo la aplicación backend

La aplicación backend requiere los mismos pasos que requirió la interfaz.

Primero, cree y abra un archivo llamado backend.py en ./backend:

``` 
nano ./backend/backend.py
``` 

Añade el siguiente contenido, que definirá dos funciones y otra ruta:

``` 
from random import randint
from time import sleep

from flask import request
from flask import Flask
app = Flask(__name__)

counter_value = 1

def get_counter():
    return str(counter_value)

def increase_counter():
    global counter_value
    int(counter_value)
    sleep(randint(1,10))
    counter_value += 1
    return str(counter_value)

@app.route('/api/counter', methods=['GET', 'POST'])
def counter():
    if request.method == 'GET':
        return get_counter()
    elif request.method == 'POST':
        return increase_counter()
``` 

Estamos importando varios módulos, incluidos random y sleep. Luego establecemos nuestro valor de contador 1 y definimos dos funciones. El primero, get_counter devuelve el valor del contador actual, que se almacena como counter_value. La segunda función, increase_counter realiza dos acciones. Incrementa nuestro valor de contador 1 y utiliza el sleep módulo para retrasar la finalización de la función en un período de tiempo aleatorio.

El backend también tiene una ruta ( /api/counter) que acepta dos métodos: POST y GET. Cuando llamamos a esta ruta usando el GET método, llama get_counter() y devuelve nuestro valor de contador. Cuando llamamos a esta ruta usando el POST método, llama increase_counter() y aumenta el valor del contador mientras espera una cantidad de tiempo aleatoria.

Guarde y cierre el archivo.

Nuestra aplicación backend también requerirá su propia versión Dockerfile, que es casi idéntica a la versión frontend.

Crea y abre un segundo Dockerfile en ./backend:

``` 
nano ./backend/Dockerfile
``` 

Agregue el siguiente contenido. La principal diferencia aquí, además de las rutas de archivo, será el puerto:

``` 
FROM alpine:3.8

RUN apk add --no-cache py3-pip python3 && \
    pip3 install flask

COPY . /usr/src/backend

ENV FLASK_APP backend.py

WORKDIR /usr/src/backend

CMD flask run --host=0.0.0.0 --port=5000
```

Guarde y cierre el archivo.

Ahora construye la imagen:

``` 
docker build -t your_username/do-visit-counter-backend:v1 ./backend
``` 

Empújelo a Docker Hub:

``` 
docker push your_username/do-visit-counter-backend:v1
``` 

Con nuestra aplicación disponible en Docker Hub, ahora estamos listos para implementarla en nuestro clúster y probarla en el siguiente paso.


## 4. Implementar y probar la aplicación

Escribir nuestro código y publicar nuestros contenedores fue nuestro primer gran paso. Ahora necesitamos implementar en Kubernetes y probar la aplicación básica. Después de eso, podemos agregar Jaeger y explorar el potencial del rastreo distribuido.

Comencemos con la implementación y las pruebas.

En este punto, nuestro árbol de directorios se ve así:

``` 
.
├── backend
│   ├── Dockerfile
│   └── backend.py
└── frontend
    ├── Dockerfile
    └── frontend.py
``` 

Para implementar esta aplicación en nuestro clúster, también necesitaremos dos manifiestos de Kubernetes; uno por cada mitad de la solicitud.

Cree y abra un nuevo archivo de manifiesto en ./frontend:

``` 
nano ./frontend/deploy_frontend.yaml
``` 

Agregue el siguiente contenido. Este manifiesto especificará cómo Kubernetes construye nuestra implementación (recuerde reemplazar la sección resaltada con su nombre de usuario de Docker Hub):

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: do-visit-counter-frontend
  labels:
    name: do-visit-counter-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: do-visit-counter-frontend
  template:
    metadata:
      labels:
        app: do-visit-counter-frontend
    spec:
      containers:
        - name: do-visit-counter-frontend
          image: your_dockerhub_username/do-visit-counter-frontend:v1
          imagePullPolicy: Always
          env:
            - name: COUNTER_ENDPOINT
              value: "http://do-visit-counter-backend.default.svc.cluster.local:5000"
          ports:
            - name: frontend-port
              containerPort: 8000
              protocol: TCP
``` 

Hemos especificado Kubernetes para crear una implementación, asignarle un nombre do-visit-counter-frontend e implementar una réplica utilizando nuestra imagen de interfaz en Docker Hub. También hemos configurado una variable de entorno llamada COUNTER_ENDPOINT para vincular las dos mitades de nuestra aplicación.

Guarde y cierre el archivo.

Ahora cree el manifiesto para nuestra aplicación backend en ./backend:

``` 
nano ./backend/deploy_backend.yaml
``` 

Agregue el siguiente contenido, reemplazando nuevamente la sección resaltada con su nombre de usuario de Docker Hub:

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: do-visit-counter-backend
  labels:
    name: do-visit-counter-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: do-visit-counter-backend
  template:
    metadata:
      labels:
        app: do-visit-counter-backend
    spec:
      containers:
        - name: do-visit-counter-backend
          image: your_dockerhub_username/do-visit-counter-backend:v1
          imagePullPolicy: Always
          ports:
            - name: backend-port
              containerPort: 5000
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
    name: do-visit-counter-backend
spec:
    selector:
        app: do-visit-counter-backend
    ports:
        - protocol: TCP
          port: 5000
          targetPort: 5000
``` 

En este manifiesto, usted define una Implementación y un Servicio para nuestro backend. La implementación describe cómo y qué se ejecutará el contenedor. Tenga en cuenta que nuestro puerto ha cambiado 8000 del frontend al 5000 backend. El Servicio permite conexiones entre clústeres desde el frontend hasta el backend.

Guarde y cierre el archivo.

Ahora implementemos nuestro contador en el clúster usando kubectl. Comience con la interfaz:

``` 
kubectl apply -f ./frontend/deploy_frontend.yaml
``` 

Y luego implemente el backend:

``` 
kubectl apply -f ./backend/deploy_backend.yaml
``` 

Para verificar que todo esté funcionando, llame a kubectl get pods:

``` 
kubectl get pods
```

Verá un resultado como este:

``` 
Output
NAME                                         READY   STATUS    RESTARTS   AGE
do-visit-counter-backend-79f6964-prqpb       1/1     Running   0          3m
do-visit-counter-frontend-6985bdc8fd-92clz   1/1     Running   0          3m
``` 

Queremos todas las cápsulas del READY estado. Si aún no están listos, espere unos minutos y vuelva a ejecutar el comando anterior.

Finalmente, queremos usar nuestra aplicación. Para hacer eso, reenviaremos puertos desde el clúster y luego nos comunicaremos con la interfaz usando el curl comando. Asegúrese de abrir una segunda ventana de terminal porque los puertos de reenvío bloquearán una ventana.

Utilice kubectl para reenviar el puerto:

``` 
kubectl port-forward $(kubectl get pods -l=app="do-visit-counter-frontend" -o name) 8000:8000
```

Ahora, en una segunda ventana de terminal, envíe tres solicitudes a su aplicación frontend:

```
for i in 1 2 3; do curl localhost:8000; done
``` 

Cada curl llamada incrementará el número de visita. Verá un resultado como este:

``` 
Output
Hello, World!

You're visitor number 1 in here!

Hello, World!

You're visitor number 2 in here!

Hello, World!

You're visitor number 3 in here!
``` 

Nuestro contador de visitantes funciona correctamente, pero probablemente haya notado un retraso entre cada respuesta. Este es el resultado de nuestra función de sueño, que simula un retraso en el rendimiento.

Con nuestra aplicación distribuida lista, es hora de instalar Jaeger y rastrear estos eventos.


## 5. Implementación de Jaeger

Recopilar huellas y visualizarlas es la especialidad de Jaeger. En este paso, implementaremos Jaeger en nuestro clúster para que pueda encontrar nuestros retrasos de rendimiento.

La documentación oficial de Jaeger incluye comandos para instalar Jaeger Operador. También incluye cuatro manifiestos adicionales que debes implementar para que la herramienta funcione. Hagamos eso ahora:

* Primero, cree la definición de recurso personalizada requerida por el operador Jaeger. Usaremos las plantillas recomendadas disponibles en la documentación oficial de Jaeger:

``` 
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
``` 

* A continuación, cree una cuenta de servicio , un rol y un enlace de roles para el control de acceso basado en roles :

``` 
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
```

* Finalmente, implemente el Operador Jaeger:

``` 
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
```

El Operador en sí no significa que tengamos a Jaeger trabajando. Aquí es donde entran en juego las definiciones de recursos personalizados. Necesitamos crear un recurso que describa la instancia de Jaeger que queremos que administre el Operador. Una vez más, seguiremos los pasos enumerados en la documentación oficial de Jaeger:

Utilice un heredoc para crear este recurso desde la línea de comando:

``` 
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
EOF
``` 

Presione ENTER para crear el recurso.

Ahora revisa tus implementaciones nuevamente:

``` 
kubectl get pods
``` 

Verá un resultado con su operador Jaeger y la simplest implementación:

``` 
Output
NAME                                                   READY   STATUS    RESTARTS   AGE
do-visit-counter-backend-79f6964-prqpb                 1/1     Running   0          3m
do-visit-counter-frontend-6985bdc8fd-92clz             1/1     Running   0          3m
jaeger-operator-547567dddb-rxsd2                       1/1     Running   0          73s
simplest-759cb7d586-q6x28                              1/1     Running   0          42s
``` 

Para validar que Jaeger esté funcionando correctamente, reenviemos su puerto y veamos si podemos acceder a la interfaz de usuario:

``` 
kubectl port-forward $(kubectl get pods -l=app="jaeger" -o name) 16686:16686
``` 

Abra un navegador y navegue hasta http://localhost:16686. Se cargará la interfaz de usuario de Jaeger.

Tanto nuestra aplicación como Jaeger están funcionando. En el siguiente paso, agregaremos instrumentación para permitir que Jaeger recopile datos y encuentre nuestro retraso en el rendimiento.


## 6. Agregar instrumentación

Aunque Jaeger automatiza muchas tareas cuando se usa con Kubernetes, aún necesitamos agregar instrumentación manualmente a nuestra aplicación. Afortunadamente, tenemos el módulo Flask-OpenTracing para realizar esa tarea.

OpenTracing es uno de los estándares de seguimiento distribuido. Ha sido propuesto por los autores de Jaeger con el objetivo de admitir también otras herramientas de rastreo. Es independiente del proveedor y admite muchos lenguajes de programación y marcos populares diferentes.

Como es el caso con todas las implementaciones de OpenTracing, necesitamos modificar nuestras aplicaciones originales agregando la configuración de Jaeger y agregando los decoradores de seguimiento a los puntos finales que queremos rastrear.

Agreguemos Flask-OpenTracing a nuestro código de interfaz.

Reabrir .frontend.py:

``` 
nano ./frontend/frontend.py
``` 

Ahora agregue el siguiente código resaltado, que incrustará OpenTracing:

``` 
import os
import requests
from flask import Flask
from jaeger_client import Config
from flask_opentracing import FlaskTracing

app = Flask(__name__)
config = Config(
    config={
        'sampler':
        {'type': 'const',
         'param': 1},
                        'logging': True,
                        'reporter_batch_size': 1,}, 
                        service_name="service")
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

def get_counter(counter_endpoint):
    counter_response = requests.get(counter_endpoint)
    return counter_response.text

def increase_counter(counter_endpoint):
    counter_response = requests.post(counter_endpoint)
    return counter_response.text

@app.route('/')
def hello_world():
    counter_service = os.environ.get('COUNTER_ENDPOINT', default="https://localhost:5000")
    counter_endpoint = f'{counter_service}/api/counter'
    counter = get_counter(counter_endpoint)

    increase_counter(counter_endpoint)

    return f"""Hello, World!

You're visitor number {counter} in here!\n\n"""
``` 

Guarde y cierre el archivo. 
Ahora abra el código de su aplicación backend:

``` 
nano ./backend/backend.py
``` 

Agregue el código resaltado. Este es el mismo código que colocamos en frontend.py:

``` 
from random import randint
from time import sleep

from flask import Flask
from flask import request
from jaeger_client import Config
from flask_opentracing import FlaskTracing


app = Flask(__name__)
config = Config(
    config={
        'sampler':
        {'type': 'const',
         'param': 1},
                        'logging': True,
                        'reporter_batch_size': 1,}, 
                        service_name="service")
jaeger_tracer = config.initialize_tracer()
tracing = FlaskTracing(jaeger_tracer, True, app)

counter_value = 1

def get_counter():
    return str(counter_value)

def increase_counter():
    global counter_value
    int(counter_value)
    sleep(randint(1,10))
    counter_value += 1
    return str(counter_value)

@app.route('/api/counter', methods=['GET', 'POST'])
def counter():
    if request.method == 'GET':
        return get_counter()
    elif request.method == 'POST':
        return increase_counter()
```

Guarde y cierre el archivo.

Dado que estamos agregando bibliotecas adicionales, también tenemos que modificar la nuestra Dockerfiles para ambos servicios.

Abra el Dockerfile para la interfaz:

``` 
nano ./frontend/Dockerfile
``` 

Agregue el código resaltado:

``` 
FROM alpine:3.8

RUN apk add --no-cache py3-pip python3 && \
    pip3 install flask requests Flask-Opentracing jaeger-client

COPY . /usr/src/frontend

ENV FLASK_APP frontend.py

WORKDIR /usr/src/frontend

CMD flask run --host=0.0.0.0 --port=8000
``` 

Guarde y cierre el archivo.

Ahora abre el backend Dockerfile:

``` 
nano ./backend/Dockerfile
``` 

Agregue el código resaltado:

``` 
FROM alpine:3.8

RUN apk add --no-cache py3-pip python3 && \
    pip3 install flask Flask-Opentracing jaeger-client

COPY . /usr/src/backend

ENV FLASK_APP backend.py

WORKDIR /usr/src/backend

CMD flask run --host=0.0.0.0 --port=5000
``` 

Con estos cambios, queremos reconstruir e impulsar las nuevas versiones de nuestros contenedores.

Construya e impulse la aplicación frontend. Tenga en cuenta la v2 etiqueta al final:

``` 
docker build -t your_username/do-visit-counter-frontend:v2 ./frontend
docker push your_username/do-visit-counter-frontend:v2
``` 

Ahora cree y envíe la aplicación backend:

``` 
docker build -t your_username/do-visit-counter-backend:v2 ./backend
docker push your_username/do-visit-counter-backend:v2
``` 

Nuestro sistema de seguimiento distribuido requiere una pieza final: queremos inyectar sidecars Jaeger en nuestros pods de aplicaciones para escuchar los rastros del pod y reenviarlos al servidor Jaeger. Para eso, necesitamos agregar una anotación a nuestros manifiestos.

Abra el manifiesto para la interfaz:

``` 
nano ./frontend/deploy_frontend.yaml
``` 

Agregue el código resaltado. Tenga en cuenta que también estamos reemplazando nuestra imagen con la v2 versión. Asegúrese de revisar esa línea y agregue su nombre de usuario de Docker Hub:

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: do-visit-counter-frontend
  labels:
    name: do-visit-counter-frontend
  annotations:
    "sidecar.jaegertracing.io/inject": "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: do-visit-counter-frontend
  template:
    metadata:
      labels:
        app: do-visit-counter-frontend
    spec:
      containers:
        - name: do-visit-counter-frontend
             image: your_dockerhub_username/do-visit-counter-frontend:v2
          imagePullPolicy: Always
          env:
            - name: COUNTER_ENDPOINT
              value: "http://do-visit-counter-backend.default.svc.cluster.local:5000"
          ports:
            - name: frontend-port
              containerPort: 8000
              protocol: TCP
``` 

Esta anotación inyectará un sidecar Jaeger en nuestra cápsula.

Guarde y cierre el archivo.

Ahora abra el manifiesto para el backend:

``` 
nano ./backend/deploy_backend.yaml
``` 

Repita el proceso, agregando las líneas resaltadas para inyectar el sidecar Jaeger y actualizando su etiqueta de imagen:

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: do-visit-counter-backend
  labels:
    name: do-visit-counter-backend
  annotations:
    "sidecar.jaegertracing.io/inject": "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: do-visit-counter-backend
  template:
    metadata:
      labels:
        app: do-visit-counter-backend
    spec:
      containers:
        - name: do-visit-counter-backend
             image: your_dockerhub_username/do-visit-counter-backend:v2
          imagePullPolicy: Always
          ports:
            - name: backend-port
              containerPort: 5000
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
    name: do-visit-counter-backend
spec:
    selector:
        app: do-visit-counter-backend
    ports:
        - protocol: TCP
          port: 5000
          targetPort: 5000
``` 

Con nuestros nuevos manifiestos implementados, debemos aplicarlos al clúster y esperar a que se creen los pods.

Eliminemos nuestros recursos antiguos:

```
kubectl delete -f ./frontend/deploy_frontend.yaml
kubectl delete -f ./backend/deploy_backend.yaml
``` 

Y luego reemplácelos:

``` 
kubectl apply -f ./frontend/deploy_frontend.yaml
kubectl apply -f ./backend/deploy_backend.yaml
``` 

Esta vez, las cápsulas para nuestras aplicaciones consistirán en dos contenedores: uno para la aplicación y otro para el sidecar Jaeger.

Utilice kubectl para comprobar esto:

``` 
kubectl get pods
``` 

Nuestros grupos de aplicaciones ahora aparecen 2/2 en la READY columna:

``` 
Output
NAME                                                   READY   STATUS    RESTARTS   AGE
jaeger-operator-547567dddb-rxsd2                       1/1     Running   0          23m
simplest-759cb7d586-q6x28                              1/1     Running   0          22m
do-visit-counter-backend-694c7db576-jcsmv              2/2     Running   0          73s
do-visit-counter-frontend-6d7d47f955-lwdnf             2/2     Running   0          42s
``` 

Con nuestros sidecars e instrumentación instalados, ahora podemos volver a ejecutar nuestro programa e investigar los rastros en la interfaz de usuario de Jaeger.


## 7. Investigar rastros en Jaeger

Ahora podemos aprovechar los beneficios del rastreo. El objetivo aquí es ver qué llamada podría ser un problema de rendimiento observando la interfaz de usuario de Jaeger. Por supuesto, si queremos ver algunos rastros en la interfaz de usuario, primero debemos generar algunos datos utilizando nuestra aplicación.

Configuremos esto abriendo una segunda y tercera ventana de terminal. Usaremos dos ventanas para reenviar Jaeger y nuestra aplicación y la tercera para enviar solicitudes HTTP al frontend desde nuestra máquina a través de curl.

En la primera ventana, reenvíe el puerto para el servicio frontend:

``` 
kubectl port-forward $(kubectl get pods -l=app="do-visit-counter-frontend" -o name) 8000:8000
```

En la segunda ventana, reenvíe el puerto de Jaeger:

``` 
kubectl port-forward $(kubectl get pods -l=app="jaeger" -o name) 16686:16686
``` 

En la tercera ventana, utilícela curl en un bucle para generar 10 solicitudes HTTP:

``` 
for i in 0 1 2 3 4 5 6 7 8 9; do curl localhost:8000; done
``` 

Recibirá un resultado como antes:

``` 
Output
Hello, World!

You're visitor number 1 in here!

Hello, World!

You're visitor number 2 in here!

. . .

Hello, World!

You're visitor number 10 in here!
``` 

Esto nos dará suficientes puntos de datos diferentes para compararlos en la visualización.

Abra un navegador y navegue hasta http://localhost:16686. Configure el menú desplegable Servicio en servicio y cambie los resultados del límite a 30. Presione Buscar rastros.

Las trazas de nuestra aplicación aparecerán en el gráfico.

Aquí vemos que diferentes llamadas al servicio tienen diferentes tiempos de ejecución. Jaeger ha rastreado cuánto tardan nuestras aplicaciones en procesar información y qué funciones aportan más tiempo. Observe cómo, como resultado de nuestra función de suspensión, el tiempo que tarda nuestra hello_world() función en completarse es muy variable. Esto es muy sospechoso y nos da un lugar donde centrar nuestra investigación. Jaeger ha visualizado eficazmente la pérdida de rendimiento dentro de nuestra aplicación distribuida.

Al implementar el rastreo y utilizar la interfaz de usuario de Jaeger, pudimos encontrar la causa de nuestro tiempo de respuesta irregular.


## 8. Conclusión

En este artículo, configuramos un sistema de seguimiento distribuido utilizando Jaeger y agregamos instrumentación a una pequeña aplicación. Ahora podemos implementar otras cargas de trabajo en el clúster, inyectar sidecars de Jaeger y ver cómo interactúan nuestros diversos servicios y qué operaciones toman más tiempo.

Encontrar cuellos de botella en el rendimiento en aplicaciones que utilizan múltiples servicios es mucho más rápido con el seguimiento distribuido. Sin embargo, este ejemplo demuestra sólo una fracción del potencial de Jaeger. En un entorno de producción más complejo, puede utilizar Jaeger para comparar diferentes seguimientos y profundizar realmente en las fugas de rendimiento. 


