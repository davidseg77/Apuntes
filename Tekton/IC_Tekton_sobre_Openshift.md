# Integración continua con Tekton sobre OpenShift

## 1. Escenario necesario para la realización del proyecto

Como tal no tengo un escenario físico ya que como he comentado en varias ocasiones, mi versión
gratuita de Openshift está en la nube. 

Por otra parte me gustaría indicar la estructura que tengo en git para la realización de este proyecto

**Repositorio backend de mi app**: https://github.com/apalgar24/pipelines-vote-api
Aquí debemos de almacenar el código fuente del backend, el Dockerfile y el deployment, service y
route necesarios.

**Repositorio frontend de mi app**: https://github.com/apalgar24/pipelines-vote-api
Aquí debemos de almacenar el código fuente del frontend el Dockerfile y el deployment, service y
route necesarios.

**Repositorio donde tengo almacenado mis pipelines y triggers**:
https://github.com/apalgar24/pipelines-triggers/tree/master/01_pipeline


### 1.1 Instalaciones y configuraciones

#### 1.1.1 Comenzar en Openshift

Para poder conseguir esta edición de pruebas de Openshift, debemos de dirigirnos a la siguiente
URL https://www.redhat.com/es/technologies/cloud-computing/openshift/try-it

Debemos de tener una cuenta registrada en Red Hat y estar logueados para poder proceder con la obtención de la edición gratuita.
Clickamos en el botón “Comience Prueba”

Nos redirigirá a otra página de Red Hat
Luego clickamos en el botón donde dice “Start your sandbox for free”

Por último, ponemos un código de verificación que nos enviarán al correo de nuestra cuenta registrada en su plataforma web. Una vez introduzcamos el código de verificación, por fin tendremos Openshift listo para usar

#### 1.1.2 Backend

Antes de comenzar con lo bueno, debemos de preparar los dos repositorios de nuestra aplicación para su posterior despliegue.

Todo el contenido backend lo tendré en el siguiente repositorio:
https://github.com/apalgar24/pipelines-vote-api.git
Al tratarse de un backend, no necesitaremos un route para exponer la aplicación.

* Dockerfile

Lo primero es tener un Dockerfile en la raíz de nuestro repositorio.

``` 
FROM
image-registry.openshift-image-registry.svc:5000/openshift/golang:
latest as builder
WORKDIR /build
ADD . /build/

RUN mkdir /tmp/cache
RUN CGO_ENABLED=0 GOCACHE=/tmp/cache go build -mod=vendor -v -
o /tmp/api-server .

FROM scratch

WORKDIR /app
COPY --from=builder /tmp/api-server /app/api-server

CMD [ "/app/api-server" ]
``` 

* Deployment

Dentro de nuestra carpeta k8s almacenaremos nuestro deployment, service del backend.

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: pipelines-vote-api
  name: pipelines-vote-api
spec:
  replicas: 1
  selector:
  matchLabels:
    app: pipelines-vote-api
template:
metadata:
  labels:
    app: pipelines-vote-api
spec:
  containers:
    - image: quay.io/openshift-pipeline/vote-api:latest
      imagePullPolicy: Always
      name: pipelines-vote-api
      ports:
        - containerPort: 9000
          protocol: TCP
``` 

Hay que tener en cuenta que demos de poner el mismo nombre de la imagen que vamos a definir luego en la ejecución de nuestro pipeline, al igual que el nombre del deployment.

* Service

Ahora crearemos un service que servirá para que el backend se comunique con el frontend

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pipelines-vote-api
  name: pipelines-vote-api
spec:
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
  selector:
    app: pipelines-vote-api
``` 

#### 1.1.3 Frontend

El rpositorio del frontend de mi app será : https://github.com/apalgar24/pipelines-vote-ui

* Dockerfile

```
# Using official python runtime base image
FROM
image-registry.openshift-image-registry.svc:5000/openshift/python:
latest

# Install our requirements.txt
ADD requirements.txt /opt/app-root/src/requirements.txt
RUN pip install -r requirements.txt

# Copy our code from the current folder to /app inside the
container
ADD . /opt/app-root/src

# Make port 80 available for links and/or publish
EXPOSE 8080

# Define our command to be run when launching the container
#CMD ["gunicorn", "app:app", "-b", "0.0.0.0:8080", "--log-file",
"-", "--access-logfile", "-", "--workers", "4", "--keep-alive",
"0"]
CMD ["python", "./app.py"]
``` 

* Deployment

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: pipelines-vote-ui
  name: pipelines-vote-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pipelines-vote-ui
  template:
    metadata:
      labels:
        app: pipelines-vote-ui
    spec:
      containers:
        - image: quay.io/openshift-pipeline/vote-ui:latest
          imagePullPolicy: Always
          name: pipelines-vote-ui
          ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 9090
              protocol: TCP
          env:
            - name: VOTING_API_SERVICE_HOST
              value: pipelines-vote-api
            - name: VOTING_API_SERVICE_PORT
              value: "9000"
``` 

Muy importante indicar el mismo puerto que usa el backend para que se comuniquen, en este caso 9000


* Route

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: pipelines-vote-ui
  name: pipelines-vote-ui
spec:
  port:
    targetPort: 8080-tcp
  to:
    kind: Service
    name: pipelines-vote-ui
    weight: 100
``` 

En este caso si necesitaremos de un route para mostrar la aplicación ya que se trata del frontend

#### 1.1.4 Tasks

Como ya sabemos, Tekton ya viene instalado sobre Openshift por lo cual no hace falta explicar la instalación de Tekton.

Hay dos maneras para trabajar con Tekton, desde la interfaz gráfica desde la terminal con el CLI de Tekton “tkn”. Personalmente prefiero trabajar con la terminal y monitorizar la ejecución del pipeline por interfaz gráfica.

Nuestro pipeline se compondrá de momento de 4 tareas (1º Git-clone 2º Construcción de imagen 3º aplicar despliegue 4º En caso de nueva versión, actualizar despliegue). Las dos últimas tareas tendremos que crearlas nosotros ya que las dos primeras son predeterminadas de Tekton

Comenzaremos creando dos tareas que servirán para hacer el despliegue de nuestra aplicación, es decir empezaremos creando las últimas dos tareas de nuestro pipeline.

``` 
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: apply-manifests
spec:
  workspaces:
  - name: source
  params:
    - name: manifest_dir
      description: The directory in source that contains yaml manifests
      type: string
      default: "k8s"
  steps:
    - name: apply
      image: image-registry.openshift-image-registry.svc:5000/openshift/cli:latest
      workingDir: /workspace/source
      command: ["/bin/bash", "-c"]
      args:
        - |-
          echo Applying manifests in $(inputs.params.manifest_dir) directory
          oc apply -f $(inputs.params.manifest_dir)
          echo -----------------------------------
``` 

En mi caso crearé la tarea desde mi terminal ya que tengo los ficheros subidos en un repositorio de
github.

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/mas
ter/01_pipeline/01_apply_manifest_task.yaml
``` 

Seguimos creando la segunda tarea

```
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: update-deployment
spec:
  params:
    - name: deployment
      description: The name of the deployment patch the image
      type: string
    - name: IMAGE
      description: Location of image to be patched with
      type: string
  steps:
    - name: patch
      image: image-registry.openshift-image-registry.svc:5000/openshift/cli:latest
      command: ["/bin/bash", "-c"]
      args:
        - |-
          oc patch deployment $(inputs.params.deployment) --patch='{"spec":{"template":{"spec":{
            "containers":[{
              "name": "$(inputs.params.deployment)",
              "image":"$(inputs.params.IMAGE)"
            }]
          }}}}'

          # issue: https://issues.redhat.com/browse/SRVKP-2387
          # images are deployed with tag. on rebuild of the image tags are not updated, hence redeploy is not happening
          # as a workaround update a label in template, which triggers redeploy pods
          # target label: "spec.template.metadata.labels.patched_at"
          # NOTE: this workaround works only if the pod spec has imagePullPolicy: Always
          patched_at_timestamp=`date +%s`
          oc patch deployment $(inputs.params.deployment) --patch='{"spec":{"template":{"metadata":{
            "labels":{
              "patched_at": '\"$patched_at_timestamp\"'
            }
          }}}}'
``` 

```
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/mas
ter/01_pipeline/02_update_deployment_task.yaml
```

Ahora podremos listar las tareas que tenemos definidas en nuestro Tekton

``` 
$ tkn task list
```

Como podemos ver ahora tenemos creada las dos tareas nuevas. La tarea s21 y git-clone son predeterminadas de Tekton.


#### 1.1.5 Pipeline

Tendremos dos repositorios para la app (Backend y Frontend) Cuando haya un push en uno de los dos repositorios, github se comunicará con la api de Openshift a través de un EventListener, el EventListener avisará a los disparadores para que estos ejecuten nuestro pipeline. El pipeline clonará de nuevo el repositorio actualizado, construirá la imagen y desplegará la aplicación actualizada.

``` 
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  workspaces:
  - name: shared-workspace
  params:
  - name: deployment-name
    type: string
    description: name of the deployment to be patched
  - name: git-url
    type: string
    description: url of the git repo for the code of deployment
  - name: git-revision
    type: string
    description: revision to be used from repo of the code for deployment
    default: master
  - name: IMAGE
    type: string
    description: image to be build from the code
  tasks:
  - name: fetch-repository
    taskRef:
      name: git-clone
      kind: ClusterTask
    workspaces:
    - name: output
      workspace: shared-workspace
    params:
    - name: url
      value: $(params.git-url)
    - name: subdirectory
      value: ""
    - name: deleteExisting
      value: "true"
    - name: revision
      value: $(params.git-revision)
  - name: build-image
    taskRef:
      name: buildah
      kind: ClusterTask
    params:
    - name: IMAGE
      value: $(params.IMAGE)
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - fetch-repository
  - name: apply-manifests
    taskRef:
      name: apply-manifests
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - build-image
  - name: update-deployment
    taskRef:
      name: update-deployment
    params:
    - name: deployment
      value: $(params.deployment-name)
    - name: IMAGE
      value: $(params.IMAGE)
    runAfter:
    - apply-manifests
```

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/mas
ter/01_pipeline/04_pipeline.yaml
``` 

En el interior del pipeline no hay nada indicado, sino que todo está construido con variables de entornos (Parámetros). Por tanto a la hora de ejecutar el pipeline por primera vez debemos de indicarle el contenido de cada variable de entorno. Ahora ejecutaremos el pipeline 2 veces, una para el backend y otra para el frontend.

* BACKEND

``` 
tkn pipeline start build-and-deploy \
-w
name=shared-workspace,volumeClaimTemplateFile=https://raw.githubus
ercontent.com/apalgar24/pipelines-triggers/master/01_pipeline/
03_persistent_volume_claim.yaml \
-p deployment-name=pipelines-vote-api \
-p git-url=https://github.com/apalgar24/pipelines-vote-api.git \
-p
IMAGE=image-registry.openshift-image-registry.svc:5000/apalgar24-
dev/pipelines-vote-api \
--use-param-defaults
``` 

* FRONTEND

``` 
tkn pipeline start build-and-deploy \
-w
name=shared-workspace,volumeClaimTemplateFile=https://raw.githubus
ercontent.com/apalgar24/pipelines-triggers/master/01_pipeline/
03_persistent_volume_claim.yaml \
-p deployment-name=pipelines-vote-ui \
-p git-url=https://github.com/apalgar24/pipelines-vote-ui.git \
-p
IMAGE=image-registry.openshift-image-registry.svc:5000/apalgar24-
dev/pipelines-vote-ui \
--use-param-defaults
``` 

Indicamos

• Volumen persistente va a utilizar, en mi caso la defiición del volúmen la tengo en un repositorio git,
• Nombre del deployment, esta variable nos servirá a la hora de ejecutar la tarea de despliegues
• Url del repositorio git que contiene el código fuente y el Dockerfile
• Registro donde guardaremos la imagen construida por el pipeline

He de recalcar que en mi caso guardo las imágenes en local, también podremos guardarlas en una registradora online como docherhub.

Si hacemos un listado los runs de nuestro pipeline podemos ver que se ha ejecutado correctamente.

``` 
bash-4.4 ~ $ tkn pipelinerun ls
``` 

Desde la interfaz gráfica podemos ver el estado de las tareas (en mi caso todas exitosas) y también
podemos ver el log de cada una de las tareas del pipeline.

En la demo de este proyecto mostraré mejor como desplazarnos por el interfaz gráfico, en esta memoria me centraré en los pasos necesarios para hacer el despliegue.

Como podemos ver la aplicación ha sido desplegad con éxito


#### 1.1.6 Triggers

Como hemos comentado antes, los triggers son disparadores que nos ayudarán a ejecutar el pipeline en cuanto la api de Openshift detecte algún cambio en nuestro repositorio git

* Trigger template

``` 
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: vote-app
spec:
  params:
  - name: git-repo-url
    description: The git repository url
  - name: git-revision
    description: The git revision
    default: master
  - name: git-repo-name
    description: The name of the deployment to be created / patched

  resourcetemplates:
  - apiVersion: tekton.dev/v1beta1
    kind: PipelineRun
    metadata:
      generateName: build-deploy-$(tt.params.git-repo-name)-
    spec:
      serviceAccountName: pipeline
      pipelineRef:
        name: build-and-deploy
      params:
      - name: deployment-name
        value: $(tt.params.git-repo-name)
      - name: git-url
        value: $(tt.params.git-repo-url)
      - name: git-revision
        value: $(tt.params.git-revision)
      - name: IMAGE
        value: image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/$(tt.params.git-repo-name)
      workspaces:
      - name: shared-workspace
        volumeClaimTemplate:
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 500Mi
``` 

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/master/03_triggers/02_template.yaml
``` 

Como su nombre indica es una plantilla que usaremos para definir los parámetros y el registro donde se guardarán nuestras imágenes.

* Trigger binding

``` 
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: vote-app
spec:
  params:
  - name: git-repo-url
    value: $(body.repository.url)
  - name: git-repo-name
    value: $(body.repository.name)
  - name: git-revision
    value: $(body.head_commit.id)
```

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/master/03_triggers/01_binding.yaml
``` 

Este rellena los parámetros del trigger template recogiendo la información a partir del evento que ocurra (push a nuestro rep git) .

* Trigger interceptor

```
apiVersion: triggers.tekton.dev/v1beta1
kind: Trigger
metadata:
  name: vote-trigger
spec:
  serviceAccountName: pipeline
  interceptors:
    - ref:
        name: "github"
      params:
        - name: "secretRef"
          value:
            secretName: github-secret
            secretKey: secretToken
        - name: "eventTypes"
          value: ["push"]
  bindings:
    - ref: vote-app
  template:
    ref: vote-app
---
apiVersion: v1
kind: Secret
metadata:
  name: github-secret
type: Opaque
stringData:
  secretToken: "1234567"
```

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/master/03_triggers/03_trigger.yaml
``` 

Este trigger lo usa el EventListener como referencia ya que indica que estamos utilizando github, el secreto que se va a utilizar para añadir el webhook.

* EventListener

``` 
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: vote-app
spec:
  serviceAccountName: pipeline
  triggers:
    - triggerRef: vote-trigger
```

``` 
oc create -f
https://raw.githubusercontent.com/apalgar24/pipelines-triggers/master/03_triggers/04_event_listener.yaml
```

Este componente configura un servicio y escucha eventos. También conecta un TriggerTemplate a un TriggerBinding.

Importante exponer el trigger ya que la url proporcionada del route, será el webhook que añadamos en github

``` 
oc expose svc el-vote-app
``` 

#### 1.1.7 Git Webhook

Para sacar la url de nuestro webhook debemos de teclear lo siguiente en nuestra línea de comandos

``` 
echo "$(oc get route el-vote-app --template='http://{{.spec.host}}')"
``` 

También podemos hacerlo desde la interfaz gráfica mirando el route de nuestro trigger interceptor. Una vez tengamos la URL, debemos de dirigirnos a nuestros dos repositorios con el código fuente de nuestra app (backend y frontend)

Settings –> Webhook –> add webhook

- Payload URL
http://el-vote-app-apalgar24-dev.apps.sandbox-m3.1530.p1.openshiftapps.com
- Content type
application/json
- Secret
1234567


## 2. Comprobación despliegue continuo con webhook

Como hemos visto en la captura anterior, nuestra app estaba en la versión 1.3, cambiaremos algo en el index.html del frontend, haremos un commit y posteriormente un push y veremos cómo comienza la construcción del nuevo frontend y el despliegue de la nueva versión.

``` 
git commit -am "versión 2.3"
git push
```

Una vez hacemos el push, automáticamente se ejecuta el pipeline y empieza a construir y desplegar la nueva versión de la aplicación. Una vez finalice todo, veremos el cambio en nuestra web.

Ya tenemos la versión 2.3 disponible en nuestra aplicación.


## 3. Tolerancia a fallos, Escalabilidad y Balanceo de carga

Como ya sabemos, estas características nos la ofrece Openshift de manera automática, simplemente teniendo varios pods en nuestro despliegue ya es más que suficiente para tener tolerancia a fallos y un balanceador de carga.

Aquí vamos a hacer unas pequeñas pruebas para verificar el funcionamiento de estas carácteristicas que ofrece Openshift. Empezaremos con la escalabilidad.

```
oc scale deployment pipelines-vote-ui –replicas=2
``` 

De esta manera nuestra aplicación estará respaldada por dos pods.

``` 
$ oc get pods
```

Como podemos ver, en el despliegue pipelines-vote-ui (frontend) tengo dos pods en ejecución
Si nosotros intentamos borrar por ejemplo el pod pipelines-vote-ui-b79b97d94-z46qs, se generará
automáticamente otro pod sustituyendo el que hemos borrado, a esto se le llama Tolerancia a fallos.

``` 
oc delete pod pipelines-vote-ui-b79b97d94-z46qs
```

Comprobamos si se ha generado otro diferente

```
bash-4.4 ~ $ oc delete pod pipelines-vote-ui-b79b97d94-z46qs
pod "pipelines-vote-ui-b79b97d94-z46qs" deleted
bash-4.4 ~ $ oc get pods
``` 

Como podemos ver el pod se ha borrado correctamente y se ha generado un nuevo pod llamado pipelines-vote-ui-b79b97d94-f8264.

Por último demostraremos el balanceo de carga haciendo unos cuantos curl a nuestra aplicación.

Como podemos ver en las capturas, hay veces que accede a un pod y otras veces accede a otro pod
 

## 4. Autoescalado

El autoescalado puede llegar a ser muy útil si estudiamos la carga que tiene nuestro despligue, es decir si veo que mi aplicación suele estar por debajo del 50% de uso de CPU pero de vez en cuando supera los 75%, podemos incrementar el número de pods cuando sobrepase ese porcentaje y disminuir el número de pod cuando vuelva a la normalidad.

En este caso haré un autoescalado de mi trigger que dispara el pipeline. Para ello ejecutamos los
siguiente

``` 
oc autoscale deployment el-vote-app --min=1 --max=3 --cpupercent=15
``` 

Como podemos ver en la interfaz gráfica, el deployment ha sido autoescalado, ahora cuando el despliegue sobrepase el 15% de la CPU, se autoescalará a 3 pods.


## 5. Rollback

Unas de las características mas llamativas y útiles, es el rollback, ya que gracias a este recurso que nos ofrece Openshift, podemos volver a una versión anterior de nuestra aplicación en caso de que la última versión desplegada dé problemas en producción.

Para poder hacer un rollback, primero debemos de mapear el historial de versiones de nuestro despliegue, por tanto debemos de ejecutar lo siguiente en nuestra terminal.

``` 
oc rollout history deployment pipelines-vote-ui
```

Como podemos ver tenemos un montón de versiones a las que volver atrás.
Para hacer el rollback a la versión seleccionada debemos de ejuctar lo siguiente

``` 
oc rollout undo deployment/pipelines-vote-ui --to-revision=19
``` 

El rollback ha sido ejecutado con éxito, ahora accederemos a la web. Efectivamente ha retrocedido a una versión anterior, ahora volveremos a la actual de nuevo, para ello miramos en nuestro historial que versión es la más reciente y ejecutamos el rollback

``` 
$ oc rollout undo deployment/pipelines-vote-ui --to-revision=22
``` 

Comprobamos la web. Como podemos ver, ha vuelto a la versión mas reciente.


## 6. SonarQube

SonarQube es una plataforma de análisis estático de código (Static Code Analysis) diseñada para evaluar la calidad y seguridad del código fuente. Se utiliza como una herramienta de análisis automatizado que ayuda a los equipos de desarrollo a identificar y corregir problemas en el código, así como a mantener altos estándares de calidad.

Para desplegar SonarQube, debemos de descargarnos desde la terminal, el deployment

```
curl -o sonarqube-h2-db-template.yml
https://raw.githubusercontent.com/edwin/sonarqube-on-openshift-4/m
aster/sonarqube-h2-db-template.yml
``` 

Una vez descargado, aplicamos el despliegue en nuestro Openshift

``` 
oc create -f sonarqube-h2-db-template.yml
``` 

Por último ejecutamos el despliegue con el siguiente comando

``` 
oc new-app --template sonarqube-h2-db
``` 

Como podemos ver se nos ha creado todo los recursos necesarios para levantar la aplicación, ahora solo queda esperar a que se despliegue todo.

Ahora debemos de loguearnos, por defecto el usuario es “admin” y la contraseña “admin”. Nos obligará, como es obvio, a cambiar la contraseña de administrador.

Lo siguiente que haremos será sacar un token de acceso para luego implementarlo en nuestro pipeline para que pueda acceder y realizar el testeo de la app. Para ello:

Inicia sesión en SonarQube con una cuenta que tenga privilegios de administrador.
1. En la parte superior derecha de la interfaz de SonarQube, haz clic en tu nombre de usuario y selecciona "My Account" (Mi cuenta) en el menú desplegable.
2. En la página de tu cuenta, selecciona la pestaña "Security" (Seguridad) en la parte superior.
3. En la sección "Tokens", haz clic en el botón "Generate Tokens" (Generar tokens).
4. En la ventana emergente, ingresa un nombre descriptivo para el token en el campo "Token Name" (Nombre del token).
5. Si deseas asignar permisos específicos al token, puedes seleccionar los roles correspondientes en la lista desplegable "Token Permissions" (Permisos del token). Esto determinará qué acciones puede realizar el token en SonarQube.
6. Haz clic en el botón "Generate" (Generar) para crear el token.

Ahí tendríamos generado nuestro token

Ahora debemos de crear un secreto con el token y la rl de nuestro SonarQube.

```
kind: Secret
apiVersion: v1
metadata:
name: sonarqube-access
data:
SONARQUBE_TOKEN: sqa_4d00eed37cdf6d5b59594171ada96bad10162b8c
SONARQUBE_URL: https://sonar-apalgar24-dev.apps.sandboxm3.1530.
p1.openshiftapps.com/
type: Opaque
``` 

Una vez creado el secreto, debemos de instalarnos la tarea sonarqube-scanner del catálogo de Tekton

``` 
tkn hub search sonarqube-scanner
tkn hub install task sonarqube-scanner
``` 

Y aquí es donde lamentablemente, debo de decir que la prueba gratuita de Openshift no me deja instalar tareas del catálogo de Tekton, solo me deja jugar con las tareas ya instaladas por defecto, por tanto no puedo hacer una demostración, pero puedo explicar los pasos posteriores ya que son muy sencillos, de hecho es lo mas sencillo.

Solo quedaría agregar la tarea sonarqube-scanner a nuestro pipeline que creamos anteriormente. Osea que el pipeline quedaría tal que así:

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  creationTimestamp: '2023-05-17T06:43:43Z'
  generation: 1
  managedFields:
    - apiVersion: tekton.dev/v1beta1
      fieldsType: FieldsV1
      fieldsV1:
        'f:spec':
          .: {}
          'f:params': {}
          'f:tasks': {}
          'f:workspaces': {}
      manager: kubectl-create
      operation: Update
      time: '2023-05-17T06:43:43Z'
  name: build-and-deploy
  namespace: apalgar24-dev
  resourceVersion: '738725163'
  uid: 2252ddfc-1d9d-417f-87b5-afb8b78311cf
spec:
  params:
    - description: name of the deployment to be patched
      name: deployment-name
      type: string
    - description: url of the git repo for the code of deployment
      name: git-url
      type: string
    - default: master
      description: revision to be used from repo of the code for deployment
      name: git-revision
      type: string
    - description: image to be build from the code
      name: IMAGE
      type: string
  tasks:
    - name: fetch-repository
      params:
        - name: url
          value: $(params.git-url)
        - name: subdirectory
          value: ''
        - name: deleteExisting
          value: 'true'
        - name: revision
          value: $(params.git-revision)
      taskRef:
        kind: ClusterTask
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-workspace
- name: code-analysis
    params:
      - name: SONAR_HOST_URL
        value: >-
          https://sonar-apalgar24-dev.apps.sandboxm3.1530.p1.openshiftapps.com
    runAfter:
      - fetch-repository
    taskRef:
      kind: Task
      name: sonarqube-scanner
    workspaces:
      - name: source-dir
        workspace: shared-workspace
      - name: sonar-settings
        workspace: sonar-settings
  - name: build-image
    params:
      - name: IMAGE
        value: $(params.IMAGE)
    runAfter:
      - code-analysis
    taskRef:
      kind: ClusterTask
      name: buildah
    workspaces:
      - name: source
        workspace: shared-workspace
  - name: code-analysis
    params:
      - name: SONAR_HOST_URL
        value: >-
          https://sonar-apalgar24-dev.apps.sandboxm3.1530.p1.openshiftapps.com
    runAfter:
      - fetch-repository
    taskRef:
      kind: Task
      name: sonarqube-scanner
    workspaces:
      - name: source-dir
        workspace: shared-workspace
      - name: sonar-settings
        workspace: sonar-settings
  - name: apply-manifests
    runAfter:
      - build-image
    taskRef:
      kind: Task
      name: apply-manifests
    workspaces:
      - name: source
        workspace: shared-workspace
  - name: update-deployment
    params:
      - name: deployment
        value: $(params.deployment-name)
      - name: IMAGE
        value: $(params.IMAGE)
    runAfter:
      - apply-manifests
    taskRef:
      kind: Task
      name: update-deployment
    workspaces:
      - name: shared-workspace
      - name: sonar-settings
```

Los cambios respecto al pipeline original es que hemos añadido la tarea code-analysis (sonarqubescanner)
entre medio de las tarea de clonado y de construcción.

Es decir que el orden sería el siguiente (Clonas el repositorio → Testeas la app → Compilas la
imagen → Despliegas la aplicación)


















