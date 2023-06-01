# Desarrollo de microservicios nativos de Red Hat Cloud con Quarkus (DO378)

### Creación de proyectos

Hay dos opciones para crear la estructura de un nuevo proyecto de Quarkus:

**Complemento Maven**

El plug-in de Quarkus Maven proporciona un comando para crear una nueva aplicación. Este plug-in agrega configuraciones específicas y ejemplos para las extensiones proporcionadas (Codestarts).

```
[user@host ~]$ mvn com.redhat.quarkus.platform:
quarkus-maven-plugin:2.13.5.Final-redhat-00002:create \ 1
    -DprojectGroupId=com.redhat.training \
    -DprojectArtifactId=getting-started \ 2
    -Dextensions=resteasy 3
[INFO] Scanning for projects...
...output omitted...
applying codestarts... 4
...output omitted...
```

1

Crear comando

2

ID y directorio del proyecto

3

Extensión inicial

4

Extensiones aplicadas codestarts

**Web Starter**

Red Hat Build of Quarkus proporciona un Iniciador web , que puede utilizar para crear proyectos con una interfaz gráfica de usuario. La URL de este iniciador web es https://code.quarkus.redhat.com/. Esta aplicación web en línea permite una selección rápida de herramientas de compilación, versiones de Java y extensiones iniciales de Quarkus.

--------------------------------------------------------------
Abra el proyecto con VSCodium o cualquier otro editor de su elección.

```
[student@workstation tenther]$ codium .
```

Ejecute la aplicación en modo de desarrollo:

```
[student@workstation tenther]$ mvn quarkus:dev
```

------------------------------------------------------------------

### Creación local de imágenes de contenedores

**Creación de la imagen del contenedor**

Los archivos contenedores del directorio requieren los artefactos de aplicación compilados del directorio del proyecto. Por lo tanto, antes de crear la imagen, debe compilar la aplicación. Puede utilizar el siguiente comando:src/main/dockertarget

```
[user@host myapp]$ mvn package
```

Para crear la imagen de contenedor de su aplicación Quarkus, use el comando, como se indica a continuación:podman build

```
[user@host myapp]$ podman build \
-f src/main/docker/Dockerfile.jvm \ 1
-t myapp \ 2
. 3
```

1

Ruta de acceso al archivo contenedor. En este caso, el comando crea la imagen del contenedor JVM.

2

Nombre de la imagen que se va a asignar a la imagen generada.

3

El contexto de compilación. Normalmente, este contexto es el directorio raíz de su proyecto Quarkus.

**Ejecución local de la imagen del contenedor**

Puede probar la imagen del contenedor mediante el comando, por ejemplo:podman run

```
[user@host myapp]$ podman run \
--name my-application \ 1
--rm \ 2
-p 9090:8080 \ 3
myapp 4
```

1

Nombre del contenedor. Si no se proporciona, podman selecciona un nombre aleatorio.

2

Retire el recipiente después de que salga.

3

Enrute las solicitudes desde el puerto del equipo local al puerto del contenedor.90908080

4

Seleccione la imagen para el contenedor. Este contenedor utiliza el nombre de la imagen.localhost/myapp:latest

**Formato de nomenclatura del Registro de imágenes**

Por ejemplo, suponga que desea insertar la imagen local del ejemplo anterior en Quay.io. Debido a que no coincide con el formato anterior, debe etiquetar la imagen con el nombre que coincida con la dirección de la imagen en su cuenta de Quay, de la siguiente manera:myappmy_app

```
[user@host myapp]$ podman tag myapp \ 1
quay.io/your_username/myapp 2
```

1

Nombre actual de la imagen local.

2

La dirección de registro remoto de la imagen.

**Inserción de la imagen local en un registro de contenedores**

Para insertar la imagen, use el comando con la siguiente sintaxis:podman push

```
podman push IMAGE
```

Suponiendo que desea insertar la imagen del ejemplo anterior, el comando sería el siguiente:quay.io/your_username/myapp

```
[user@host myapp]$ podman push quay.io/your_username/myapp
```

**Generación de recursos RHOCP**

Puede usar la interfaz de línea de comandos (CLI) para interactuar con el clúster RHOCP y generar recursos RHOCP. Por ejemplo, utilice el comando para generar un archivo YAML:ococ createDeployment

```
[user@host ~]$ oc create deployment myapp \ 1
--image quay.io/example/myapp \ 2
--dry-run=client \ 3
-o yaml 4
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
spec:
...output omitted...
```

1

Cree un recurso llamado .Deploymentmyapp

2

Utilice la imagen.quay.io/example/myapp

3

No cree el recurso en el clúster RHOCP.

4

Imprima el recurso en formato YAML.

También puede usar el comando para crear recursos sin archivos YAML, por ejemplo:oc create

```
[user@host ~]$ oc create configmap my-config \ 1
--from-literal=key1=value1 \ 2
--from-literal=key2=value2 3
configmap/my-config created
```

1

Cree un recurso llamado .ConfigMapmy-config

2 3

Cree los pares clave-valor.key1: value1key2: value2

**Implementación del contenedor en RHOCP**

Después de insertar la imagen del contenedor en un registro, puede implementar el contenedor en RHOCP. Puede implementar la aplicación mediante un recurso similar al que se muestra en el ejemplo siguiente:Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp 1
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp 2
  template: 3
    metadata:
      labels:
        app: myapp 4
    spec:
      containers:
      - name: myapp 5
        image: registry/username/myapp 6
        ports:
        - containerPort: 8080 7
```

1

El campo es el nombre del recurso.metadata.nameDeployment

2

El campo define cómo la implementación selecciona los pods que se van a administrar. En este caso, la implementación administra pods etiquetados como .spec.selectorapp: myapp

3

El campo define la plantilla para crear los pods de esta implementación.spec.template

4

El campo define las etiquetas asignadas a los pods de esta implementación. Este campo debe ser coherente con , para que la implementación pueda encontrar sus propios pods.spec.template.metadata.labelsspec.selector

5

El campo define el nombre del contenedor que se va a crear.spec.template.spec.containers[0].name

6

El campo define la imagen utilizada para crear el contenedor.spec.template.spec.containers[0].image

7

El campo define los pods que expone el contenedor.spec.template.spec.containers[0].ports

Para que el pod sea accesible dentro y fuera del clúster, debe crear un servicio y una ruta. Un ejemplo de un servicio para la implementación anterior es el siguiente:

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp
  name: myapp
spec:
  ports:
  - port: 8070 1
    targetPort: 8080 2
  selector:
    app: myapp 3
```

1

El servicio expone el puerto .8070

2

El servicio apunta al puerto de cualquier pod que cumpla los criterios.8080selector

3

El servicio se dirige a cualquier pod con la etiqueta.app: myapp

Un ejemplo de una ruta para el servicio anterior es el siguiente:

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  to:
    kind: Service
    name: myapp
  port:
    targetPort: 8070
```

**Configuración de inserción**

En muchos casos, las aplicaciones requieren parámetros de configuración en tiempo de ejecución, que puede pasar como variables de entorno o volúmenes montados en un contenedor. En RHOCP, puede insertar valores de configuración pasando y recursos en una aplicación existente. Los recursos y son objetos de Kubernetes utilizados para almacenar valores de configuración. Si los valores son sensibles, entonces se prefiere.ConfigMapSecretConfigMapSecretSecret

En el ejemplo siguiente se muestra un recurso llamado que contiene una propiedad.ConfigMapmyapp-configmy_var

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: myapp-config
data:
  my_var: hi
```

Para inyectar o recursos, especifíquelos en los parámetros de contenedor del recurso. También puede definir variables de entorno individuales específicas de una implementación.ConfigMapSecretDeployment

Por ejemplo, para insertar una única variable de entorno, específica de la aplicación, puede utilizar el objeto en el recurso:spec.template.containers[].envDeployment

```
kind: Deployment
apiVersion: apps/v1
spec:
  ...definition omitted...
  template:
    ...definition omitted...
    spec:
      containers:
        - name: myapp
          env:
          - name: LOGLEVEL
            value: '1'
          - name: TRACING
            value: '0'
```

Del mismo modo, puede insertar variables de entorno de un recurso.ConfigMap

```
kind: Deployment
apiVersion: apps/v1
spec:
  ...definition omitted...
  template:
    ...definition omitted...
    spec:
      containers:
        - name: myapp
          env:
          - name: LOGLEVEL
            valueFrom:
              configMapKeyRef:
                name: myapp-config
                key: my_var
```

Si la configuración contiene datos confidenciales, como contraseñas o claves de API, es posible que desee almacenar estos valores en un recurso y, a continuación, inyectar este recurso en el contenedor. Inyectar un valor de un recurso es similar a inyectar desde un recurso. En lugar de utilizar la propiedad, utilice , de la siguiente manera:SecretSecretConfigMapconfigMapKeyRefsecretKeyRef

```
env:
- name: API_KEY
  valueFrom:
    secretKeyRef:
      name: myapp-secrets
      key: api_key
```
### Extensión Quarkus OpenShift

Red Hat Build of Quarkus proporciona extensiones para implementar sus aplicaciones en Kubernetes y Red Hat OpenShift Container Platform (RHOCP). Estas extensiones automatizan tareas, como crear, insertar e implementar una imagen de contenedor. Esto es útil para los desarrolladores que deben probar su aplicación en RHOCP pero no están familiarizados con los objetos RHOCP.

Puede usar la extensión para implementar su aplicación en RHOCP con una configuración mínima.io.quarkus:quarkus-openshift

Agregue la extensión mediante el comando:mvn

```
[user@host ~]$ mvn quarkus:add-extension -Dextensions=openshift
```

El comando anterior crea la siguiente dependencia en el archivo:pom.xml

```
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-openshift</artifactId>
</dependency>
```

**Implementar aplicaciones en RHOCP**

La extensión requiere credenciales RHOCP válidas para implementar la aplicación en el clúster RHOCP. La extensión puede usar el archivo para autenticar sus solicitudes RHOCP.quarkus-openshift~/.kube/config

Utilice el comando para generar una entrada de credenciales basada en token en el archivo:oc login~/.kube/config

```
[user@host ~]$ oc login -u USER -p PASSWORD http://CLUSTER_API
```

Puede utilizar las siguientes propiedades para proporcionar las credenciales RHOCP a la extensión:quarkus-openshift

quarkus.kubernetes-client.master-url: establece la API RHOCP.

quarkus.kubernetes-client.token: establece el token RHOCP.

Utilice el comando para obtener un token RHOCP válido para un usuario que ha iniciado sesión:oc whoami

```
[user@host ~]$ oc whoami -t
sha256~JIEP....wQqM
```

En el ejemplo siguiente se muestra la implementación del proyecto en RHOCP mediante un token RHOCP:

```
[user@host ~]$ mvn clean package \ 1
 -Dquarkus.kubernetes.deploy=true \ 2
 -Dquarkus.kubernetes-client.master-url=URL \ 3
 -Dquarkus.kubernetes-client.token=TOKEN 4
```

1

El comando genera recursos RHOCP para la aplicación.package

2

Utilice los recursos generados para implementar la aplicación en RHOCP.

3

El servidor API de RHOCP.

4

El token utilizado para autenticar las solicitudes a la API de RHOCP.

Si el proyecto RHOCP contiene recursos ConfigMap o Secret, puede insertar los valores especificados o todos los valores mediante las entradas y:quarkus.openshift.env.configmapsquarkus.openshift.env.secrets

```
quarkus.openshift.env.configmaps=aConfigMap,anotherConfigMap 1
quarkus.openshift.env.secrets=aSecret,anotherSecret 2
quarkus.openshift.env.mapping.DB_USER.from-configmap=datasource 3
quarkus.openshift.env.mapping.DB_USER.with-key=username 4
quarkus.openshift.env.mapping.DB_PW.from-secret=datasource 5
quarkus.openshift.env.mapping.DB_PW.with-key=password 6
```

1

Inyecta los pares clave-valor de los recursos y ConfigMap como variables de entorno.aConfigMapanotherConfigMap

2

Inyecta los pares clave-valor de los recursos Secret y y como variables de entorno.aSecretanotherSecret

3 4

Crea la variable de entorno con el valor de la clave del recurso ConfigMap.DB_USERusernamedatasource

5 6

Crea la variable de entorno con el valor de la clave del recurso Secret.DB_PWpassworddatasource

### Conexión con servicios externos

**El recurso personalizadoServiceBinding**

El componente principal para integrar cargas de trabajo y servicios de respaldo es el CRD. Con este CRD, también puede definir cómo insertar los datos de enlace a la carga de trabajo. Puede inyectar los datos mediante variables de entorno o volúmenes montados (valor predeterminado).ServiceBinding

En el ejemplo siguiente se enlaza una aplicación Quarkus con un clúster de PostgreSQL.

```
apiVersion: binding.operators.coreos.com/v1beta1 1
kind: ServiceBinding 2
metadata:
 name: my-service-binding
 namespace: my-namespace
spec:
 application: 3
   name: my-quarkus-application
   group: apps.openshift.io
   kind: DeploymentConfig
   version: v1
 bindAsFiles: true 4
 services: 5
 - name: db-demo
   group: postgres-operator.crunchydata.com
   kind: PostgresCluster
   version: v1beta1
```

1

Versión de API de recursos personalizada

2

Tipo de recurso personalizado

3

Definición de la carga de trabajo

4

Marcar para usar archivos para la proyección de carga de trabajo

5

Definición del servicio de respaldo

**Enlace de servicio con Quarkus**

En las aplicaciones desplegadas en Red Hat OpenShift, de forma predeterminada, el operador de enlace de servicio monta los datos de enlace en el directorio. Puede utilizar la variable de entorno para definir la ubicación de los datos de enlace. En el ejemplo siguiente se muestra la proyección de carga de trabajo montada en una carga de trabajo./bindingsSERVICE_BINDING_ROOT

```
sh-4.4$ tree /bindings
/bindings 1
└── k8s-sb 2
    ├── a-db-binding 3
    │    ├── database
    │    ├── host
    │    ├── password
    │    ├── port
    │    ├── type 4
    │    └── username
    └── another-binding 5
        ├── type 6
        ├── connection-count
        ├── uri
        ├── certificates
        └── private-key
```

1

Ubicación de montaje para la proyección de la carga de trabajo. El operador de enlace de servicio monta la proyección de carga de trabajo en el directorio./bindings

2

Directorio raíz de enlace de servicio.

3 5

Directorio de datos de enlace.

4 6

Archivo necesario para identificar el tipo de datos de enlace.


**Configuración de enlace de servicio**

El proceso de enlace de servicio comienza definiendo los servicios de respaldo que desea enlazar a una aplicación. Debe definir los enlaces en un CR. En lugar de crear y aplicar manualmente el CR, puede usar Quarkus para automatizar el proceso.ServiceBinding

Si la aplicación utiliza el , o las extensiones, Quarkus puede crear automáticamente el CR utilizando las propiedades de configuración. Quarkus utiliza esos valores de configuración para crear un enlace de servicio para la aplicación.quarkus-kubernetesquarkus-openshiftServiceBinding

En el ejemplo siguiente se definen las opciones de configuración del archivo para declarar un enlace con un clúster de PostgreSQL.application.properties

```
quarkus.kubernetes-service-binding.services.expenses-db.api-version = postgres-operator.crunchydata.com/v1beta1 1
quarkus.kubernetes-service-binding.services.expenses-db.kind = PostgresCluster 2
quarkus.kubernetes-service-binding.services.expenses-db.name = pgcluster 3
```

1

Define el grupo y la versión del servicio de respaldo.

2

Define el tipo de servicio de respaldo.

3

Define el nombre del servicio de copia de seguridad específico que se va a utilizar.

Quarkus utiliza los ajustes de configuración anteriores para crear automáticamente el siguiente CR:ServiceBinding

```
apiVersion: binding.operators.coreos.com/v1alpha1
kind: ServiceBinding
metadata:
  name: expenses-service-expenses-db
  namespace: my-namespace
spec:
  application:
    group: apps.openshift.io
    kind: DeploymentConfig
    name: expenses-service
    version: v1
  bindAsFiles: true
  services:
  - group: postgres-operator.crunchydata.com
    kind: PostgresCluster
    name: pgcluster
    version: v1beta1
```

### Implementación de tolerancia a errores en microservicios

**Uso de MicroProfile Fault Tolerance**

Para empezar a utilizar las políticas de tolerancia a fallos de MicroProfile en Red Hat Build de Quarkus, añada la extensión al proyecto.quarkus-smallrye-fault-tolerance

```
[user@host myapp]$ mvn quarkus:add-extension \
-Dextensions=smallrye-fault-tolerance
```









