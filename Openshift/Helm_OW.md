# Curso Helm

### Instalación de Helm

Si aún no tienes acceso de admin a un clister de Kubernetes, es el momento de instalarte uno en local con microK8S

Ejecuta estos comandos en tu ordenador Linux o Mac:

```
 sudo snap install microk8s --classic
 sudo micrko8s.start
 microk8s.config > /tmp/kubeconfig
 export KUBECONFIG=/tmp/kubeconfig
```

**Instalar Helm**

Se instala fácilmente con snap:

```
sudo snap install helm --classic
```

Se debe añadir un repo, por ejemplo:

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
```

Para añadir la aplicación de ejemplo para joomla:

```
helm install stable/joomla --generate-name
```

```
kubectl get pod
```

```
kubectl get services
```

Vamos al navegador y accedemos al servicio.

En mi caso, he tenido que instalar minikube en mi Ubuntu Desktop y después iniciarlo.

### Comandos básicos

Para actualizar el tiller a la última version de helm:

```
helm init --upgrade
```

Para actualizar los repositorios instalados en tu helm:

```
helm repo update
```

Para actualizar un chart

```
helm upgrade nombredelpod stable/joomla:2 (Ponemos la versión dos, por ejemplo)
```
Veamos aquí algunos de ellos en forma de listado:

* helm init           # Inicializa el entorno de Helm. Crea y gestiona el agente Tiller.
* helm repo update    # Actualiza la información de los repositorios
* helm install        # Instala un chart en tu clúster. Esto es, crea una nueva release a partir de un chart.
* helm upgrade        # Actualiza una release
* helm ls             # Lista todas las releases (instalaciones) que hay desplegadas en tu clúster
* helm get            # Información sobre una release existente
* helm get notes      # Información sobre las notas de una release existente, como usuario, contraseñas...
* helm get manifests  # Información sobre los yaml de una release existente

* helm history        # Muestra las versiones de una release
* helm rollback       # Permite volver a una versión anterior de la release
* helm delete         # Elimina una release del cluster, pero dejará los datos en el cluster. Se verifica con helm ls -a
* helm delete --purge   # Elimina una release del cluster al completo

### Repositorios de aplicaciones para Kubernetes

Son APIs (normalmente con una web para consultarlo desde el navegador) a las que accede el CLI de Helm para poder instalar los charts.

Kubeapps Hub es el estándar, viene configurado por defecto en la instalación de Helm (con el nombre stable). Es mantenido por la CNCF y solo contiene aplicaciones preparadas para producción.

Aparte del repositorio stable en KubeApps puedes encontrar muchos otros repositorios de distintos proveedores, ya mantenidos por cada creador. También encontrarás uno llamado incubator que en este caso también es oficial, y contiene aplicaciones que están siendo mejoradas para promocionar a stable

Puedes acceder a KubeApps Hub, bucear entre los charts que ofrece y los distintos proveedores y repositorios en https://hub.kubeapps.com/

Los nombres de los charts están compuestos por su repositorio y su nombre dentro del repositorio: /. Por ejemplo stable/wordpress se refiere al repositorio stable y al chart wordpress dentro de él. jfrog/artifactory se refiere al repositorio jfrog (que pertenece a la empresa con el mismo nombre) y al chart artifactory (un producto de esa empresa, que lo distribuye a través de Helm).

### Instalación de una aplicación, de un chart

helm install instala un chart en tu clúster. Lo que hace en realidad es crear una nueva release del chart que tú quieras.

Ejemplo:

```
helm install stable/mediawiki
```

o

```
helm install stable/mediawiki --name mywiki --namespace=default --set image.tag=1.32.1
```

o 

```
helm install mywiki stable/mediawiki
```

Crea una nueva release en tu clúster del chart mediawiki del repositorio stable.

Si ejecutas helm ls podrás ver esa nueva release.

Ejecuta helm install --help para ver todas las opciones de configuración. Las más importantes son --name para darle un nombre concreto a tu release. --namespace para desplegarla en un namespace concreto. --set para pasarle variables como veremos más adelante.

Visita la página de documentación sobre este comando: https://helm.sh/docs/helm/#helm-install

### Actualización de una aplicación, de un chart

```
helm upgrade <RELEASE NAME> stable/mediawiki
```

Con este comando se actualiza una release ya existente. Lo que hace en concreto es crear una nueva revisión de esa release (una versión nueva).

Pues especificar una versión distinta del chart, actualizar las variables (values), etc.

Las opciones son prácticamente las mismas que en helm install. Simplemente en helm upgrade debes especificar la release que quieres actualizar.

Visita la página de documentación sobre este comando: https://helm.sh/docs/helm/#helm-upgrade

**Rollback**

Puedes listar todas las revisiones de una release con el comando:

```
helm history <RELEASE NAME>
```
Y hacer rollback a cualquiera de esas versiones con:

```
helm rollback <RELEASE NAME> <REVISION NUMBRER>
```

Esto en realidad creará una revisión nueva con la misma configuración exacta que la revisión a la que querías volver.

Puedes ver el estado de nuevo con helm history <RELEASE NAME>

Las páginas para estos comandos:

https://helm.sh/docs/helm/#helm-history

https://helm.sh/docs/helm/#helm-rolback

### Despliegue de un repositorio privado

Después de instalar y actualizar un servidor mediawiki, vamos a instalar otra aplicación similar.

En este caso será un repositorio privado para charts de Helm. El proceso es exactamente igual que el que hemos visto hasta ahora.

Al finalizar este punto tendremos un repositorio privado al que subir nuestros propios charts con libertad y seguridad.

En concreto instalaremos esta aplicación: ChartMuseum. Está disponible en el canal stable de KubeApps. Las instrucciones para instalarlo las puedes encontrar aquí: https://hub.kubeapps.com/charts/stable/chartmuseum. Son las que seguiremos en el video.

Para nuestra instalación, vamos a especificar unas variables personalizadas. Esto lo hacemos con el parámetro --values, al que le pasamos un fichero con las variables que queramos en formato YAML. El nuestro será así:

```
env:
  open:
    STORAGE: local
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 8Gi
env:
  open:
    DISABLE_API: false
```

Con estas variables le decimos que queremos que el almacenamiento de los charts sea en un volumen persistente local de 8Gi, y que active su API para que podamos conectarnos con Helm.

La documentación de estas variables la puedes encontar en la info del propio chart en KubeApps: https://hub.kubeapps.com/charts/stable/chartmuseum

Añadimos el repositorio privado a Helm

```
helm repo add myRepo http://localhost:<PORT>
```

helm repo add configura un nuevo repositorio en helm. En este caso le pasamos el nombre (myRepo) y la URL que debe utilizar Helm para conectarse a la API del repositorio que acabamos de desplegar, que será de la forma http://localhost:PORT. El número del puerto lo encontramos como siempre con kubectl get services.

A continuación actualizamos la información de los repositorios para que Helm se descargue la información actualizada:

```
helm repo update
```

Otros comandos para gestionar repositorios son:

```
helm repo list # Listar todos los repos configurados

helm repo remove # Elimina un repositorio

helm search # Busca charts en todos los repositorios configurados
```

### Instalar un kubeadds privado

Vamos a desplegar otra aplicación más antes de terminar la sección.

En este caso será un servidor privado de KubeApps. Exactamente igual que el que vemos en https://hub.kubeapps.com/ pero privado en nuestro clúster y con interesantes nuevas funciones, como poder ver las releases que tenemos desplegadas, añadir nuestro repositorio privado que desplegamos en el punto anterior, y desplegar nuevas releases desde el navegador.

Las instrucciones para desplegarlo las puedes encontrar aquí:

https://github.com/kubeapps/kubeapps/blob/master/docs/user/getting-started.md

El token se copia dentro de la web de KubeApps.

### ¿Qué son los charts?

Los charts (paquetes) de Helm contienen archivos YAML de Kubernetes (Deployments, services, statefulsets, ingresses, etc.). Estos se pueden personalizar o templatizar con el leguaje de templates de Go.

**Templates**

Todos los templates se almacenan en una carpeta llamada templates dentro de la carpeta raiz del chart.

En esta sección vamos a ver cómo crear esos templates y cómo hacer un chart de Helm a partir de ellos.

Puedes insertar partes dinámicas en un YAML de Kubernetes poniendo el contenido entre llaves:

{{ CÓDIGO PARA TEMPLATIZAR }}
Por ejemplo este deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: {{ .Values.image }}
        ports:
        - containerPort: 80
```

Con {{ .Values.image }} le estamos diciendo que inserte en esa ubicación el valor del value image que especifiquemos a la hora de instalar el chart.

Recordemos que los values son variables que podemos personalizar al instalar o actualizar un chart.

Por ejemplo con helm install stable/joomla --set image.tag=1.2.3 Le estamos diciendo que instale el chart joomla del repositorio stable, y que la variable image.tag tenga valor 1.2.3

De esta manera es como especificamos dentro del template dónde queremos que esa variable sea utilizada.

Aparte de los templates tenemos varios archivos importantes:

**Chart.yaml**

```
apiVersion: The chart API version, always "v1" (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
...
...
```

Contiene la definición del chart (name, version, description, maintainers, etc)

**values.yaml**

Contiene los values (variables) por defecto del chart. Estos values pueden ser personalizados al instalar o actualizar el chart, pero si no se especifican, se tomarán los que se encuentren en este archivo.

**REAMDE.md**

README con instrucciones para instalar y utilizar la aplicación, junto con una explicación de los values disponibles.

**templates/NOTES.txt**

Este archivo también es un template. Pero en vez de ser un archivo YAML para desplegar el Kubernetes, es un archivo .txt que se muestra cuando se instala o actualiza la aplicación con éxito. Suele contener instrucciones para comenzar a utilizarla.

El árbol de directorios sería:

.
├── templates
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── NOTES.txt
├── Chart.yaml
└── values.yaml
└── README.md

### Creación de un chart: Deployment y personalización

Para crear un chart solo necesitas definir los archivos que hemos comentado en el video anterior.

También puedes crear un nuevo chart fácilmente con helm create <NOMBRE DEL CHART>. Esto creará una carpeta nueva con todos los archivos necesarios, incluyendo templates con los objetos más comunes en Kubernetes, ya templatizados adecuadamente, como un deploment.yaml, service.yaml o ingress.yaml.

Nosotros vamos a utilizar helm create pero vamos a eliminar esos templates por defecto, para ir creando los nuestros, paso a paso, y conociendo los detalles de la sintaxis de Helm para los templates.

Después vamos a personalizar el chart.yaml con los datos de nuestro chart. Vamos a vaciar también el archivo values.yaml para poder llenarlo más adelante con los values que utilicemos.

Vamos a añadir un único template, que será deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.name }}-nginx
  namespace: {{ .Release.namespace }}
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Release.name }}-nginx
  template:
    metadata:
      labels:
        app: {{ .Release.name }}-nginx
    spec:
      containers:
      - name: nginx
        image: {{ .Values.image }}
        ports:
        - containerPort: 80
```

Y especificamos los values por defecto en values.yaml:

```
replicas: 1
image: nginx:latest
```

.Release es un objeto que siempre está presente y que contiene información sobre la release en concreto. En este caso hemos utilizado .Release.name para llamar y etiquetar nuestro deployment con el mismo nombre que la release, añadiéndole -nginx al final. Y .Release.namespace para desplegar el deployment en el namespace que se especifique al instalar el chart.

**Debuggear los templates**

Siempre que creamos código es bueno saber cómo podemos verlo en acción. En este caso queremos ver si nuestro deployment.yaml está bien construido y nos daría el YAML que queremos después de renderizarlo (esto es, después de sustituir todas las partes dinámicas). Lo podemos hacer con:

```
helm install . --dry-run --debug
```

Esto simulará una instalación del chart pero sin crear absolutamente nada (—dry-run). Y con —debug nos imprimirá en pantalla todos los YAML para que podamos observarlos antes de continuar.

### Creación del chart.yaml y del values.yaml

En el **chart.yaml** es donde definimos el chart en si. 

```
apiVersion: v1
appVersion: "1.0.1"
description: A Helm chart for Kubernetes
name: app-example
version: 1.0.1
```

En el **values.yaml** contenemos los valores por defecto. Es un buen consejo que siempre contenga algún valor por defecto.

```
replicas: 2
# image:
# podAnnotations:
#   annotation1: value1
# customVar: test
```

No es imprescindible, pero es bueno tener el **notes.txt**. Es el fichero donde se muestra cuando escalas o actualizas una aplicación.

```
{{ .Release.Name }} have been deployed successfully in namespace {{ .Release.Namespace}} !
```

### Creación de un chart: Despliegue y actualización

Para desplegarlo:

```
helm install . --name myrelease
```

Para actualizarlo:

```
helm upgrade myrelease . --set replicas=3
```

Para volver a una version anterior:

```
helm rollback myrelease 1 (El 1 hace a la versión inicial, que puede comprobarse con helm history myrelease)
```

### Subida del chart a nuestro repositorio privado

Primero hay que empaquetar nuestro chart. Esto es, crear un archivo comprimido tar.gz que podamos subir fácilmente a nuestro repositorio. Helm tiene un comando para ello:

```
helm package .
```

A continuación lo subimos al repositorio. La manera manual de hacerlo es simplemente haciendo una petición HTTP POST a la API de nuestro repositorio.

En realidad la manera más sencilla de hacerlo es utilizar el plugin helm push que realiza directamente el empaquetado y lo sube al repositorio todo en un paso:

**Instala el plugin**

```
helm plugin install https://github.com/chartmuseum/helm-push
```

**Empaqueta y sube el chart**

```
helm push .
```

Hay muchísimas más opciones para gestionar los repositorios de Helm. Si estuviésemos utilizando un bucket de S3 como almacenamiento podríamos subirlo directamente ahí. También puede ser un repositorio de GitHub. En este curso no vamos a profundizar más en este tema. Te dejo documentación a continuación para que puedas conocer más sobre la gestión de repositorios.

En concreto, Chart Museum (nuestro repositorio privado), automatiza la gestión del índice del repositorio, pieza que necesitarás controlar tú en caso de no usar Chart Museum y su API o el plugin helm push.

**Versionado de paquetes**

La versión de un chart está definida en el archivo chart.yaml

Helm utiliza semantic versioning para numerar las versiones de un chart.

Puedes elegir la versión al instalar un chart nuevo con el argumento --version de helm install y helm upgrade

### Sintaxis de Helm y condicionales

**Condicionales**

Documentación: https://helm.sh/docs/chart_template_guide/#if-else

Ejemplo:

```
{{- if VARIABLE }}
  # Do something
{{- else if OTHER_VARIABLE }}
  # Do something else
{{- else }}
  # Default case
{{- end }}
 
{{- if .Values.customVar }}
- name: CUSTOM_VAR
  value: {{ .Values.customVar}}
{{- end }}
```

En este ejemplo, si existe el value customVar, inserta esa pieza de código. Si no, esa parte queda vacía.

El guion después de la llave {{- sirve para que Helm quite la línea en blanco que dejaría ese comando.

**Bucles**

Documentación: https://helm.sh/docs/chart_template_guide/#looping-with-the-range-action

Ejemplo:

```
{{- range ARRAY_VARIABLE}}
- {{ . | quote }}
{{- end }}


{{- range $key, $value :=DICT_VARIABLE }}
  {{ $key }}: {{ $value | quote }}
{{- end }}
 
{{- if .Values.podAnnotations }}
annotations:
  {{- range $key, $value := .Values.podAnnotations }}
  {{ $key | quote }}: {{ $value | quote }}
  {{- end }}
{{- end }}
```

Este ejemplo haría un bucle sobre la variable podAnnotations (si esta existe) y transformaría todo ese código en:

```
annotations:
  key1: value1
  key2: value2
  ...
```

### Sintaxis avanzada: Variables, pipelines y funciones

**Variables**

Documentación: https://helm.sh/docs/chart_template_guide/#variables

Ejemplo:

```
 {{ $newVar := .Values.myVar - }}
 

 {{ $deployName := printf "%s-%s" .Release.Name "server" - }}
```

En este caso se almacena el resultado en la variable $deployName, que puedes utilizar en cualquier parte posterior del código.

La función printf devuelve una cadena formateada. En este caso, queremos que la cadena sean dos variables también de tipo cadena (%s) unidas por un guion.

deployName sería -server

Es muy útil para no repetir fragmentos de código complicados y simplificar la gestión de los templates.

**Pipelines y funciones**

Documentación: https://helm.sh/docs/chart_template_guide/#template-functions-and-pipelines

Los pipelines sirven para modificar cualquier variable u objeto. Siempre van encadenados con una barra vertical |

Por ejemplo  {{ .Values.myVar | quote }} devolvería el valor del value myVar entre comillas: "<MYVAR VALUE>"

Las funciones son de muchos tipos. Por ejemplo el printf del apartado anterior es una función. La diferencia con los pipelines es que no modifican el elemento anterior de la cadena, sino que solo les puedes pasar parámetros. Por ejemplo quote también puede ser una función de esta forma:

```
 {{ quote .Values.myVar }}
```

Puedes encontrar las más útiles en la documentación anterior

**Temas más avanzados**

* Lectura de ficheros externos

* Named templates

* Hooks

### Dependencias

Puedes incluir cualquier otro chart(s) como dependencias de tu aplicación. Por ejemplo, bases de datos.

Lo más sencillo y mejor práctica es hacerlo con el archivo requirements.yaml. Este sería un ejemplo de ese archivo que añade un mongodb:

```
dependencies:
- name: mongodb
  version: 5.19.3
  repository: https://kubernetes-charts.storage.googleapis.com/
  tags:
    - mongodb
```

A continuación, siempre antes de empaquetar y pushear tu chart, debes ejecutar helm dependecy update. Este comando resolverá las dependencias que has incluido en requirements.yaml y descargará esos charts en la carpeta ./charts

Puedes controlar qué dependencias se instalan con tu aplicación y cuáles no mediante los campos tags y condition.

En este caso hemos usado tags ya que es más directo. Tal como está escrito nuestro requirements.yaml anterior, solo se instalará nuestro mongodb si existe un value llamado tags que contiene mongodb. Es decir en values.yaml deberíamos incluir:

```
tags:
  - mongodb
```

Como cualquier otro value, este puede ser modificado a la hora de instalar el chart.

Hay muchas más posibilidades para gestionar nuestras dependencias, te invito a que leas detenidamente la documentación ya que son conceptos más avanzados:

https://helm.sh/docs/charts/#chart-dependencies























