# Tutorial de Openshift Pipelines

OpenShift Pipelines es una solución de integración y entrega continua (CI/CD) nativa de la nube para crear canalizaciones utilizando Tekton. Tekton es un marco de CI/CD flexible, nativo de Kubernetes y de código abierto que permite automatizar implementaciones en múltiples plataformas (Kubernetes, sin servidor, máquinas virtuales, etc.) al abstraer los detalles subyacentes.

Utilizará la CLI de Tekton () a lo largo de este tutorial. Descargue la CLI de Tekton siguiendo las instrucciones disponibles en el repositorio de GitHub de la CLI.tkn

Para instalar Tekton en Linux:

**Get the tar.xz**

```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.31.0/tkn_0.31.0_Linux_x86_64.tar.gz
```

**Extract tkn to your PATH (e.g. /usr/local/bin)**

```
sudo tar xvzf tkn_0.31.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
```

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

