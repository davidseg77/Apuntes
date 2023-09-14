# Helm sobre Openshift utilizando Tekton y Kubeseal

El objetivo a conseguir es la integración contínua y el despliegue contínuo de un chart de Helm sobre Openshift, con pods totalmente operativos y una URL externa con la que podamos visualizar la aplicación, esto sumado a un webhook que enlazaremos a nuestro repositorio de git. Es útil cuando necesitamos hacer cambios en nuestro chart y evitar despliegues y configuraciones repetitivas.


## 1. Escenario sobre el que vamos a trabajar

El escenario que utilizaremos será Openshift 4.10, es una plataforma montada sobre un clúster proporcionado por redhat el cual es un laboratorio que simula un entorno real que podría tener un cliente.
Para ello entramos en **Red Hat Cloudforms Management Engine**
Una vez dentro solicitamos el lab **Hands on Openshift 4.10**

Una vez tenemos este escenario, Openshift nos proporciona los recursos necesarios para poder integrar pipelines con Tekton a través del operador que debemos instalar.


## 2. Práctica

Comenzaremos configurando Openshift instalando el operador de Openshift Pipelines:
Iremos a Administrator -> Operators -> OperatorHub y en el buscador ingresaremos Red Hat Openshift Pipelines, el cual por dentro será Tekton el que esté proporcionando el servicio.

Ahora dentro de la terminal, vamos a crear un ClusterRole y se lo vamos asignar a la pipeline que podrá de esta manera hacer un deploy del sealedsecret que usaremos más adelante, de modo que si aplicamos estos .yaml, crearemos una regla que nos permita instalar sealedsecrets a través de la instalación del chart de helm:

**clusterrole-sealedsecrets.yaml**

``` 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: sealedsecrets-role
rules:
- apiGroups: ["bitnami.com"]
  resources: ["sealedsecrets"]
  verbs: ["get", "create", "update", "delete"]
``` 

**clusterrolebinding-sealedsecrets.yaml**

``` 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sealedsecrets-rolebinding
subjects:
- kind: ServiceAccount
  name: pipeline
  namespace: pipeline-helm
roleRef:
  kind: ClusterRole
  name: sealedsecrets-role
  apiGroup: rbac.authorization.k8s.io
``` 

Ahora vamos a proceder con la configuración del chart para que al ser procesado por Tekton no tengamos ningún tipo de problemas de seguridad en Openshift. Esto es debido a que Openshift viene con un parámetro de seguridad llamado PodSecurityContext, el cual es un grupo de seguridad que viene por defecto en Openshift y que evita que el pod se ejecute internamente a través de un usuario privilegiado. 

Esto es así para que un atacante que posea acceso al pod no pueda realizar cambios significativos en el clúster.

Entonces vamos a ir a nuestro repositorio de git:
https://github.com/Evanticks/charts-bitnami/tree/main/bitnami/wordpress

Y procederemos a modificar el archivo values.yaml. values.yaml establece las diferentes variables que va a poseer el chart de Helm, en el resto de configuraciones se hace referencias a los valores que se establecen en
este archivo, así que podríamos decir que es el núcleo del chart. Nos fijamos en lo siguiente, lo cual estableceremos a false:

``` 
podSecurityContext:
  enabled: false
  fsGroup: 1001
...
containerSecurityContext:
  enabled: false
  runAsUser: 1001
...
  primary:
    podSecurityContext:
    enabled: false
    containerSecurityContext:
    enabled: false
```

Una vez hecho esto podremos desplegar la aplicación sin que falle la pipeline debido a que estos parámetros ejecutan el pod en modo sin privilegios.

De modo alternativo podríamos desplegar Argocd, que es un operador que nos permite desplegar en Openshift imágenes que le proporcionemos, el cual no será nuestro caso, ya que Tekton tiene la posibilidad de desplegar el Chart por sí mismo.

Tras esto lo siguiente que debemos hacer será mencionar el secret que crearemos posteriormente. de tal manera que necesitaremos crear la última línea haciendo referencia al secret que crearemos posteriormente:

``` 
mariadb:
  architecture: standalone
  auth:
    rootPassword: ""
    database: bitnami_wordpress
    username: bn_wordpress
    password: ""
    existingSecret: "mysecret"
``` 

Gracias a esto, el chart esperará al secreto mysecret para obtener las credenciales de la base de datos.

Tras esto, nos conectaremos al bastión de nuestro clúster de Openshift e instalaremos helm para poder utilizarlo a través de línea de comandos y no solo a través de la consola web:

``` 
wget https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
tar zxvf helm-v3.11.0-linux-amd64.tar.gz
sudo install -m 755 helm /usr/local/bin/helm
```


### 2.1 Configurar e implantar secrets cifrados con Kubeseal

Kubeseal es una herramienta de Kubernetes que se utiliza para cifrar secretos y protegerlos de accesos no autorizados.

Kubeseal utiliza la criptografía asimétrica para cifrar los secretos y garantizar que solo los usuarios autorizados tengan acceso a ellos. En lugar de almacenar los secretos en texto plano en Kubernetes, Kubeseal cifra los secretos utilizando una clave pública y solo puede descifrarse utilizando la clave privada correspondiente.

El secreto cifrado puede entonces ser almacenado de manera segura en Kubernetes o un repositorio Git o similar.

Cuando Kubernetes necesita hacer uso de ese secreto utiliza la clave privada de Kubeseal para descifrarlo.

Primero vamos a proceder a instalar Kubeseal en nuestro cúster, de moto que si ya anteriormente instalamos helm, ahora necesitaremos instalar el chart de Kubeseal, así que procedemos a ello:

``` 
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update
helm install sealed-secrets-controller sealed-secrets/sealed-secrets -n kube-system
``` 

Y su respectivo binario:

``` 
wget https://github.com/bitnami-labs/sealed-secrets/
releases/download/v0.20.5/kubeseal-0.20.5-linux-amd64.tar.gz
tar -xvzf kubeseal-0.20.5-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
``` 

Ahora procedemos a crear un secreto en Kubernetes, como vamos a utilizarlo para la base de datos de mariadb, lo crearemos con sus propias credenciales:

``` 
kubectl create secret generic mysecret \
--from-literal=mariadb-root-password=hola123 \
--from-literal=mariadb-replication-password=hola987 \
--from-literal=mariadb-password=hola933 \
--dry-run=client -o yaml > secretonuevo.yaml
``` 

Una vez realizada esta acción, se nos generará el archivo secretonuevo.yaml, aquí es donde entrará kubeseal:

``` 
kubeseal < secretonuevo.yaml > mysealedsecret.yaml
``` 

Una vez hecho esto, se generará el cifrado de cada valor del secreto.

Ya con esto solo nos quedará aplicarlo al clúster

``` 
oc create -f mysealedsecret.yaml
``` 

O bien podremos subirlo a cualquier repositorio sin problema, ya que será sólo nuestro clúster el que sea capaz de descifrarlo a través de las claves asimétricas.


## 2.2 Establecer un ciclo CI/CD con Tekton

Este apartado es el más denso, ya que tekton se encarga de las configuraciones, las integraciones y procesos de despliegue, así que vamos a proceder a explicar qué es Tekton

Tekton es una plataforma de orquestación de pipelines de CI/CD (Integración Continua/Entrega Continua) que se ejecuta en un clúster de Kubernetes. Permite a los equipos de desarrollo automatizar el proceso de construcción, prueba y entrega de aplicaciones.

Tekton utiliza una arquitectura basada en contenedores para ejecutar los pasos del pipeline. Los pipelines se definen mediante archivos YAML que describen los pasos del pipeline, como la construcción de la aplicación, la ejecución de pruebas y la implementación en un entorno de producción.

Tekton proporciona varios recursos para la creación de pipelines, como tareas, recursos y pipelines. Las tareas representan un trabajo específico que se realiza en el pipeline, mientras que los recursos son elementos externos utilizados por las tareas, como el código fuente o las imágenes de Docker. Los pipelines combinan tareas y recursos para crear un flujo de trabajo completo de CI/CD.

Tekton también incluye una biblioteca de componentes reutilizables de tareas y pipelines, lo que hace que sea fácil para los equipos de desarrollo construir y personalizar sus pipelines de CI/CD.

Para poder realizar un CI/CD con Tekton necesitamos de varios tipos de pipelines, que relacionados entre sí encajan como si piezas de un puzzle se tratase, estos archivos son:

● Pipeline.yaml: se encarga de nombrar las task que se van a utilizar, ya que es necesarios porque podemos almacenar archivos .yaml de task de cosas totalmente diferentes, así que la función del propio pipeline es importante.

● Tasks.yaml: se encarga de realizar algún tipo de acción sobre la que vamos a realizar, las tasks son consideradas las más importantes de Tekton y posee una comunidad detrás que implementa plantillas de tasks para que los demás usuarios podamos utilizarlos o modificarlos a nuestro antojo.
https://hub.tekton.dev/

● PipelineRun.yaml: es el archivo iniciador del pipeline, aquí se indica el pipeline que se va a utilizar y lo va a poner en producción, además es el que nos indica el estado del pipeline ejecutado, cada vez que queramos ejecutar el pipeline, debemos ejecutar PipelineRun.

● EventListener.yaml: como su nombre indica es un servicio que establece un puerto de escucha el cual si lo asociamos a github, cada vez que haya un cambio en el repositorio lanza el PipelineRun.

Para poder crear una pipeline vamos a empezar a crear la estructura por pasos, y el primer paso será crear el volumen que vamos a solicitar para que en el espacio compartido donde se ejecute la pipeline las tareas puedan enlazarse:

**PersistentVolumeClaim.yaml**

``` 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pipelinerun-vol
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
    storage: 1Gi
```

Ahora procederemos a instalar cada tarea, las cuales serán dos, una que realiza ungit clone y otra que he modificado a mano para que haga un despliegue de la aplicación:

task-git-clone.yaml se encuentra en mi repositorio git al ser un texto demasiado extenso, pero de task-install-chart he modificado a manos unas líneas de código para que me permita realizar el despliegue haciendo CI/CD:

**task-install-chart.yaml**

```
helm dependency update bitnami/wordpress

helm list --namespace "$(params.release_namespace)"
    echo current installed helm releases
helm list --namespace "$(params.release_namespace)"

if helm ls | grep "helm-release"
then
    echo "WordPress de Bitnami está instalado. Desinstalando..."
    helm uninstall helm-release
    echo "WordPress de Bitnami ha sido desinstalado exitosamente."
fi

helm upgrade --install --values
"$(params.charts_dir)/$(params.values_file)" --namespace
"$(params.release_namespace)" --version "$(params.release_version)"
"$(params.release_name)" "$(params.charts_dir)" --debug --set
"$(params.overwrite_values)" $(params.upgrade_extra_params) --set
service.type=ClusterIP
```

Con todos estos parámetros, la task detectará un chart del helm del cual vamos a realizar la demo, si está instalada la desinstala, tras esto procederá a instalarla con un servicio ClusterIP.

Tenemos el volumen, luego las tareas, ahora vamos a definir el pipeline el cual volveremos a elegir un segmento a analizar, lo cual sintetizamos el funcionamiento del mismo.

En Tekton, el pipeline se utiliza para definir las tareas y el orden con el que estas se van a ejecutar, aunque existe la posibilidad de que esto no sea así, pudiendo ejecutar tareas a la par, en nuestro caso una tarea depende de la otra así que quedaría la pipeline de la siguiente manera:

**pipeline-helm.yaml**

``` 
...
  tasks:
    - name: task-install-chart
    runAfter:
    - git-clone
    taskRef:
    kind: Task
    name: task-install-chart
...
    - name: git-clone
    taskRef:
    kind: Task
    name: git-clone
    params:
    - name: url
    value: 'https://github.com/Evanticks/charts-bitnami.git'
    - name: revision
    value: main
...
```

De esta manera tendremos definido tanto la tarea en la pipeline como parámetros los cuales luego el task puede utilizar para recibir la información como el propio github que ingreso.

Una vez hecho esto vamos a ejecutar un pipelineRun, el cual se encargará de coger el pipeline de helm y ejecutarlo, vamos a analizar la parte más importante del código:

``` 
pipelineRef:
    name: "pipeline-helm"
serviceAccountName: pipeline
workspaces:
    - name: compartido
    persistentVolumeClaim:
    claimName: pipelinerun-vol
``` 

Podemos ver el pipelineRef, el cual será al que hace la referencia para ejecutarlo, el serviceAccountName es la identidad del pod generado, ya que cuando se ejecuta una Run realmente se crea un pod que lo ejecuta y luego muere tras finalizar su función.

El workspace es el nombre del espacio en el que se compartirá el recurso de la pipeline, por tanto debe tener el mismo nombre en los diferentes .yaml

Por último hacemos referencia al recurso del persistentVolumeClaim para que el clúser cree ese recurso que habíamos definido en el primer paso.


## 3. Demostración final

Para la demostración final una vez hechas las configuraciones entraremos en bastión dentro de la terminal de Openshift, bajaremos el repositorio donde se encuentra la pipeline de tekton.

``` 
git clone https://github.com/Evanticks/wordpress-helm-tekton.git
cd wordpress-helm-tekton
oc create -f .
``` 

Una vez creados los recursos, se estará ejecutando la pipelineRun, por tanto la pipeline en openshift está corriendo, podremos verlo entrando en la consola web.

Una vez desplegado el pod con frontend y su respectivo backend persistente, veremos que no posee una ruta a la cual acceder, por tanto la añadiremos en nuestro entorno local y haremos un push:

``` 
cp wordpress-ruta.yaml charts-bitnami/bitnami/wordpress/templates/
git commit -am "añado ruta"
git push
``` 

A través de un event listener, Github notificará a Openshift para que se vuelva a ejecutar el pipelineRun, por tanto se volverá a desplegar el chart de Helm esta vez utilizando la ruta.

Finalmente mostraré el interior del sealedsecret que ingresé dentro del chart de helm para mostrar la fiabilidad de la operación.


## 4. Dificultades encontradas

● En primer lugar encontré el problema de los privilegios de los pods, Openshift no corre pods privilegiados, entonces se deben ejecutar desde el usuario 1000, bien especificándolo en el Dockerfile o bien en el values.yaml del chart de Bitnami.

● En segundo lugar me encontré los problemas con las pipelines. Tekton como tal tiene una curva de aprendizaje más elevada que Jenkins, ya que debemos aprender para qué se utiliza cada yaml.

● En tercer lugar el volumen persistente, ya que cada vez que se va a desplegar la aplicación genera una nueva contraseña, y si posee ese volumen persistente las contraseñas serán las del primer despliegue, entonces al hacer CI/CD el pod de Mariadb no funcionaba, por eso opté por la opción de Kubeseal ya que me facilita mucho los despliegues.

● En cuarto lugar aprender sobre los roles del clúster de Openshift para otorgar el rol de que pueda desplegarse el chart de bitnami con un SealedSecret dentro de la template del chart a través de la pipeline.

● Por último, tuve problemas con el LoadBalancer que me otorga AWS que es sobre donde está montado Openshift










