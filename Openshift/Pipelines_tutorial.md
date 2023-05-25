# Tutorial de Openshift Pipelines

OpenShift Pipelines es una solución de integración y entrega continua (CI/CD) nativa de la nube para crear canalizaciones utilizando Tekton. Tekton es un marco de CI/CD flexible, nativo de Kubernetes y de código abierto que permite automatizar implementaciones en múltiples plataformas (Kubernetes, sin servidor, máquinas virtuales, etc.) al abstraer los detalles subyacentes.

Utilizará la CLI de Tekton () a lo largo de este tutorial. Descargue la CLI de Tekton siguiendo las instrucciones disponibles en el repositorio de GitHub de la CLI.tkn

### Para instalar Tekton en Linux:

Para instalar la herramienta tkn de Tekton en Ubuntu, puedes seguir los siguientes pasos:

Abre la terminal en Ubuntu.

Asegúrate de tener curl instalado. Si no lo tienes, puedes instalarlo ejecutando el siguiente comando:

```
sudo apt-get update
sudo apt-get install curl
```

Descarga el archivo binario de tkn utilizando curl con el siguiente comando:

```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.22.0/tkn_0.22.0_Linux_x86_64.tar.gz
```

Este comando descargará la versión 0.22.0 de tkn. Puedes verificar la última versión disponible en la página de GitHub de Tekton CD CLI (https://github.com/tektoncd/cli/releases) y reemplazar el enlace de descarga en el comando anterior si hay una versión más reciente.

Descomprime el archivo descargado utilizando el siguiente comando:

```
tar xvzf tkn_0.22.0_Linux_x86_64.tar.gz
```

Mueve el archivo binario tkn al directorio /usr/local/bin con el siguiente comando:

```
sudo mv tkn /usr/local/bin
``` 

Esto permitirá que tkn sea ejecutable desde cualquier ubicación en tu sistema.

Verifica que tkn se haya instalado correctamente ejecutando el siguiente comando:

```
tkn version
```

Deberías ver la información de la versión de tkn instalada en tu sistema.

Si recibes el mensaje de error "bash: tkn: command not found" después de seguir los pasos anteriores, puede ser porque la ubicación /usr/local/bin no está en tu variable de entorno PATH. Para solucionarlo, puedes intentar lo siguiente:

Abre la terminal y ejecuta el siguiente comando para editar el archivo .bashrc:

```
nano ~/.bashrc
```

Agrega la siguiente línea al final del archivo:

```
export PATH=$PATH:/usr/local/bin
```

Guarda los cambios en el archivo .bashrc y ciérralo.

Ejecuta el siguiente comando para cargar los cambios en el archivo .bashrc:

```
source ~/.bashrc
```

Ahora, intenta nuevamente ejecutar el comando tkn version y debería funcionar correctamente.

### ¿Qué es Tekton?

Tekton Pipelines es una extensión de Kubernetes que se instala y ejecuta en su clúster de Kubernetes. Define un conjunto de recursos personalizados de Kubernetes que actúan como bloques de creación a partir de los cuales puede ensamblar canalizaciones de CI/CD. Una vez instalado, Tekton Pipelines está disponible a través de la CLI de Kubernetes (kubectl) y a través de llamadas API, solo como pods y otros recursos. Tekton es de código abierto y parte de la Fundación CD, un proyecto de la Fundación Linux.

Tekton Pipelines define las siguientes entidades:

| Entidad |	Descripción |
|:---------:|:-------------:|
| Task | Define una serie de pasos que inician herramientas específicas de compilación o entrega que ingieren entradas específicas y producen salidas específicas. |
| TaskRun |	Crea instancias de una ejecución con entradas, salidas y parámetros de ejecución específicos. Se puede invocar por sí solo o como parte de un archivo .TaskPipeline |
| Pipeline | Define una serie de cosas que logran un objetivo específico de compilación o entrega. Puede desencadenarse mediante un evento o invocarse desde un archivo .TasksPipelineRun |
| PipelineRun |	Crea instancias de una ejecución con entradas, salidas y parámetros de ejecución específicos.Pipeline |
| PipelineResource (Deprecated) | Define las ubicaciones de las entradas ingeridas y las salidas producidas por los pasos de .Tasks |
| Run (alfa) | Crea instancias de una tarea personalizada para su ejecución cuando se introducen entradas específicas. |

Los disparadores extienden el Tekton arquitectura con los siguientes CRD:

* TriggerTemplate - Recursos de plantillas que se van a utilizar creados (por ejemplo, Crear PipelineResources y PipelineRun que los utilice)
* TriggerBinding - Valida eventos y extracciones Campos de carga útil
* Trigger: combina TriggerTemplate, TriggerBindings e interceptores.
* EventListener: proporciona un extremo direccionable (el receptor de eventos). se hace referencia dentro de la especificación EventListener. Utiliza los parámetros de evento extraídos de cada uno (y cualquier parámetro estático suministrado) para crear los recursos. especificado en el archivo . También permite opcionalmente un Servicio externo para preprocesar la carga útil del evento a través del campo.
TriggerTriggerBindingTriggerTemplateinterceptor
* ClusterTriggerBinding: un ámbito de clúster TriggerBinding
El uso junto con le permite cree fácilmente sistemas CI/CD completos donde la ejecución se defina completamente a través de los recursos de Kubernetes.tektoncd/triggerstektoncd/pipeline

### Instalar el operador Openshift Pipelines

A través de OperatorHub.

### Deploy Sample Application

Create a project for the sample application that you will be using in this tutorial:

```
$ oc new-project pipelines-tutorial
```

OpenShift Pipelines automatically adds and configures a ServiceAccount named pipeline that has sufficient permissions to build and push an image. This service account will be used later in the tutorial.

Run the following command to see the pipeline service account:

```
$ oc get serviceaccount pipeline
```

You will use the simple application during this tutorial, which has a frontend and backend

You can also deploy the same applications by applying the artifacts available in k8s directory of the respective repo

If you deploy the application directly, you should be able to see the deployment in the OpenShift Web Console by switching over to the Developer perspective of the OpenShift Web Console.


### Install Tasks

Tasks consist of a number of steps that are executed sequentially. Tasks are executed/run by creating TaskRuns. A TaskRun will schedule a Pod. Each step is executed in a separate container within the same pod. They can also have inputs and outputs in order to interact with other tasks in the pipeline.

Here is an example of a Maven task for building a Maven-based Java application:

```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: maven-build
spec:
  workspaces:
   -name: filedrop
  steps:
  - name: build
    image: maven:3.6.0-jdk-8-slim
    command:
    - /usr/bin/mvn
    args:
    - install
```

When a task starts running, it starts a pod and runs each step sequentially in a separate container on the same pod. This task happens to have a single step, but tasks can have multiple steps, and, since they run within the same pod, they have access to the same volumes in order to cache files, access configmaps, secrets, etc. You can specify volume using workspace. It is recommended that Tasks uses at most one writeable Workspace. Workspace can be secret, pvc, config or emptyDir.

Note that only the requirement for a git repository is declared on the task and not a specific git repository to be used. That allows tasks to be reusable for multiple pipelines and purposes.

Install the apply-manifests and update-deployment tasks from the repository using oc or kubectl, which you will need for creating a pipeline in the next section:

```
$ oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/01_apply_manifest_task.yaml
```

```
$ oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/02_update_deployment_task.yaml
```

You can take a look at the tasks you created using the Tekton CLI:

```
$ tkn task ls
```

We will be using buildah clusterTasks, which gets installed along with Operator. Operator installs few ClusterTask which you can see.

```
$ tkn clustertasks ls
```

### Create Pipeline

A pipeline defines a number of tasks that should be executed and how they interact with each other via their inputs and outputs.

In this tutorial, you will create a pipeline that takes the source code of the application from GitHub and then builds and deploys it on OpenShift.

Here is the YAML file that represents the pipeline:

```
apiVersion: tekton.dev/v1beta1
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

Once you deploy the pipelines, you should be able to visualize pipeline flow in the OpenShift Web Console by switching over to the Developer perspective of the OpenShift Web Console. select pipeline tab, select project as pipelines-tutorial and click on pipeline build-and-deploy.

This pipeline helps you to build and deploy backend/frontend, by configuring right resources to pipeline.

Pipeline Steps:

* Clones the source code of the application from a git repository by referring (git-url and git-revision param)
* Builds the container image of application using the buildah clustertask that uses Buildah to build the image
* The application image is pushed to an image registry by refering (image param)
* The new application image is deployed on OpenShift using the apply-manifests and update-deployment tasks.

You might have noticed that there are no references to the git repository or the image registry it will be pushed to in pipeline. That's because pipeline in Tekton are designed to be generic and re-usable across environments and stages through the application's lifecycle. Pipelines abstract away the specifics of the git source repository and image to be produced as PipelineResources or Params. When triggering a pipeline, you can provide different git repositories and image registries to be used during pipeline execution. Be patient! You will do that in a little bit in the next section.

The execution order of task is determined by dependencies that are defined between the tasks via inputs and outputs as well as explicit orders that are defined via runAfter.

workspaces field allow you to specify one or more volumes that each Task in the Pipeline requires during execution. You specify one or more Workspaces in the workspaces field.

Create the pipeline by running the following:

```
oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/04_pipeline.yaml
```

Upon creating the pipeline via the web console, you will be taken to a Pipeline Details page that gives an overview of the pipeline you created.

Check the list of pipelines you have created using the CLI:

```
$ tkn pipeline ls
```

### Trigger Pipeline

Now that the pipeline is created, you can trigger it to execute the tasks specified in the pipeline.

A PipelineRun is how you can start a pipeline and tie it to the persistentVolumeClaim and params that should be used for this specific invocation.

Lets start a pipeline to build and deploy backend application using tkn:

```
$ tkn pipeline start build-and-deploy \
    -w name=shared-workspace,volumeClaimTemplateFile=https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/03_persistent_volume_claim.yaml \
    -p deployment-name=pipelines-vote-api \
    -p git-url=https://github.com/openshift/pipelines-vote-api.git \
    -p IMAGE=image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/pipelines-vote-api \
    --use-param-defaults


Pipelinerun started: build-and-deploy-run-z2rz8

In order to track the pipelinerun progress run:
tkn pipelinerun logs build-and-deploy-run-z2rz8 -f -n pipelines-tutorial
```

Similarly, start a pipeline to build and deploy frontend application:

```
$ tkn pipeline start build-and-deploy \
    -w name=shared-workspace,volumeClaimTemplateFile=https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/01_pipeline/03_persistent_volume_claim.yaml \
    -p deployment-name=pipelines-vote-ui \
    -p git-url=https://github.com/openshift/pipelines-vote-ui.git \
    -p IMAGE=image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/pipelines-vote-ui \
    --use-param-defaults

Pipelinerun started: build-and-deploy-run-xy7rw

In order to track the pipelinerun progress run:
tkn pipelinerun logs build-and-deploy-run-xy7rw -f -n pipelines-tutorial
```

As soon as you start the build-and-deploy pipeline, a pipelinerun will be instantiated and pods will be created to execute the tasks that are defined in the pipeline.

```
$ tkn pipeline list
NAME               AGE             LAST RUN                     STARTED          DURATION   STATUS
build-and-deploy   6 minutes ago   build-and-deploy-run-xy7rw   36 seconds ago   ---        Running
```

Above we have started build-and-deploy pipeline, with relevant pipeline resources to deploy backend/frontend application using single pipeline

```
$ tkn pipelinerun ls
NAME                         STARTED         DURATION     STATUS
build-and-deploy-run-xy7rw   36 seconds ago   ---          Running
build-and-deploy-run-z2rz8   40 seconds ago   ---          Running
```

Check out the logs of the pipelinerun as it runs using the tkn pipeline logs command which interactively allows you to pick the pipelinerun of your interest and inspect the logs:

```
$ tkn pipeline logs -f
```

After a few minutes, the pipeline should finish successfully.

```
tkn pipelinerun list
```

You can get the route of the application by executing the following command and access the application

```
$ oc get route pipelines-vote-ui --template='http://{{.spec.host}}'
```

If you want to re-run the pipeline again, you can use the following short-hand command to rerun the last pipelinerun again that uses the same workspaces, params and service account used in the previous pipeline run:

```
$ tkn pipeline start build-and-deploy --last
```

Whenever there is any change to your repository we need to start pipeline explicity to see new changes to take effect

### Triggers

Triggers in conjuntion with pipelines enable us to hook our Pipelines to respond to external github events (push events, pull requests etc).

Adding Triggers to our Application:
Now let’s add a TriggerTemplate, TriggerBinding, and an EventListener to our project.

**Trigger Template**

A TriggerTemplate is a resource which have parameters that can be substituted anywhere within the resources of template.

The definition of our TriggerTemplate is given in 03_triggers/02-template.yaml.

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
        value: image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/$(tt.params.git-repo-name)
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

Run following command to apply Triggertemplate.

```
oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/03_triggers/02_template.yaml
```

**Trigger Binding**

TriggerBindings is a map enable you to capture fields from an event and store them as parameters, and replace them in triggerTemplate whenever an event occurs.

The definition of our TriggerBinding is given in 03_triggers/01_binding.yaml.

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

The exact paths (keys) of parameter we need can be found by examining the event payload (eg: GitHub events).

Run following command to apply TriggerBinding.

```
oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/03_triggers/01_binding.yaml
```

**Trigger**

Trigger combines TriggerTemplate, TriggerBindings and interceptors. They are used as ref inside the EventListener.

The definition of our Trigger is given in 03_triggers/03_trigger.yaml.

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
```

The secret is to verify events are coming from correct source code management

```
---
apiVersion: v1
kind: Secret
metadata:
  name: github-secret
type: Opaque
stringData:
  secretToken: "1234567"
```

Run following command to apply Trigger.

```
oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/03_triggers/03_trigger.yaml
```

**Event Listener**

This component sets up a Service and listens for events. It also connects a TriggerTemplate to a TriggerBinding, into an addressable endpoint (the event sink)

The definition for our EventListener can be found in 03_triggers/04_event_listener.yaml.

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

Run following command to create EventListener.

```
oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/03_triggers/04_event_listener.yaml
```

Run below command to expose eventlistener service as a route

```
oc expose svc el-vote-app
```

### Configuring GitHub WebHooks


Now we need to configure webhook-url on backend and frontend source code repositories with the Route we exposed in the previously.

Run below command to get webhook-url

```
echo "URL: $(oc  get route el-vote-app --template='http://{{.spec.host}}')"
```

**Configure webhook manually**

Open forked github repo (Go to Settings > Webhook) click on Add Webhook > Add

```
echo "$(oc  get route el-vote-app --template='http://{{.spec.host}}')"
```

to payload URL > Select Content type as application/json > Add secret eg: 1234567 > Click on Add Webhook

Now we should see a webhook configured on your forked source code repositories (on our GitHub Repo, go to Settings>Webhooks).

### Trigger pipeline Run

When we perform any push event on the backend the following should happen.

* The configured webhook in vote-api GitHub repository should push the event payload to our route (exposed EventListener Service).

* The Event-Listener will pass the event to the TriggerBinding and TriggerTemplate pair.

* TriggerBinding will extract parameters needed for rendering the TriggerTemplate. Successful rendering of TriggerTemplate should create 2 PipelineResources (source-repo-vote-api and image-source-vote-api) and a PipelineRun (build-deploy-vote-api)

We can test this by pushing a commit to vote-api repository from GitHub web ui or from terminal.

Let’s push an empty commit to vote-api repository.

Watch OpenShift WebConsole Developer perspective and a PipelineRun will be automatically created.










