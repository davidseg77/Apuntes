# Información del curso DO188 de Red Hat

### Trabajar con Podman

El siguiente comando muestra la versión que está usando.

```
[user@host ~]$ podman -v
```

**Extracción y visualización de imágenes**

Por ejemplo, el siguiente comando obtiene una versión contenerizada de Red Hat Enterprise Linux 7 del registro de Red Hat.

```
[user@host ~]$ podman pull registry.redhat.io/rhel7/rhel:7.9
```

Después de la ejecución del comando pull, la imagen se almacena localmente en su sistema. Puede enumerar las imágenes en su sistema con el comando podman images.

```
[user@host ~]$ podman images
```

**Ejecución y visualización de contenedores**

Con la imagen almacenada en su sistema local, puede usar el comando podman run para crear un nuevo contenedor que use la imagen. La imagen RHEL de ejemplos anteriores acepta comandos Bash como argumento. Estos comandos se proporcionan como un argumento y se ejecutan dentro de un contenedor RHEL.

```
[user@host ~]$ podman run registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

Cuando el contenedor finaliza la ejecución de echo, el contenedor se detiene porque ningún otro proceso lo mantiene en ejecución. Puede enumerar los contenedores en ejecución mediante el comando podman ps.

```
[user@host ~]$ podman ps
```

Puede enumerar todos los contenedores (en ejecución y detenidos) agregando el indicador --all al comando podman ps.

```
[user@host ~]$ podman ps --all
```

También puede eliminar automáticamente un contenedor cuando sale con la opción --rm al comando podman run.

```
[user@host ~]$ podman run --rm registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

Puede asignar un nombre a sus contenedores agregando el indicador --name al comando podman run.

```
[user@host ~]$ podman run --name podman_rhel7 \
registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

Si desea recuperar el ID de contenedor largo de UUID, puede agregar el indicador --format=json al comando podman ps --all.

```
[user@host ~]$ podman ps --all --format=json
```

El siguiente ejemplo crea un nuevo contenedor que ejecuta un servidor HTTP Apache al mapear el puerto 8080 en su máquina local al puerto 8080 dentro del contenedor.

```
[user@host ~]$ podman run -p 8080:8080 \
 registry.access.redhat.com/ubi8/httpd-24:latest
```

**Uso de variables del entorno**

Puede pasar variables de entorno a un contenedor con la opción -e. En el siguiente ejemplo, se pasa una variable de entorno denominada NAME con el valor Red Hat. Luego, la variable del entorno se imprime con el comando printenv dentro del contenedor.

```
[user@host ~]$ podman run -e NAME='Red Hat' \
 registry.redhat.io/rhel7/rhel:7.9 printenv NAME
```

### Gestión de redes Podman

La gestión de la red Podman se realiza mediante el subcomando podman network. Estas operaciones de gestión incluyen:

* podman network create

Crea una nueva red Podman. Este comando acepta diversas opciones para configurar las propiedades de la red, incluida la dirección de la puerta de enlace, la máscara de subred y si se debe usar IPv4 o IPv6.

* podman network ls

Enumera las redes existentes y un breve resumen de cada una. Las opciones para este comando incluyen diversos filtros y un formato de salida para enumerar otros valores para cada red.

* podman network inspect

Genera un objeto JSON detallado que contiene datos de configuración para la red.

* podman network rm

Elimina una red.

* podman network prune

Elimina las redes que no están actualmente en uso por ningún contenedor en ejecución.

* podman network connect

Conecta un contenedor que ya se está ejecutando hacia o desde una red existente. Alternativamente, conecte contenedores a una red Podman en la creación de contenedores con la opción --net. El comando disconnect desconecta un contenedor de una red.

Por ejemplo, el siguiente comando crea una nueva red Podman llamada example-net:

```
[user@host ~]$ podman network create example-net
```

Para conectar un nuevo contenedor a esta red Podman, use la opción --net. El siguiente comando de ejemplo crea un nuevo contenedor llamado my-container, que está conectado a la red example-net.

```
[user@host ~]$ podman run -d --name my-container \
--net example-net container-image:latest
```

Cuando crea nuevos contenedores, puede conectarlos a varias redes especificando los nombres de las redes en una lista separada por comas. Por ejemplo, el siguiente comando crea un nuevo contenedor llamado double-connector que se conecta a ambas redes: postgres-net y redis-net.

```
[user@host ~]$ podman run -d --name double-connector \
--net postgres-net,redis-net \
container-image:latest
```

Alternativamente, si el contenedor my-container ya se está ejecutando, ejecute el siguiente comando para conectarlo a la red example-net:

```
[user@host ~]$ podman network connect example-net my-container
```

### Conexión de contenedores

Por ejemplo, tiene un contenedor en ejecución llamado nginx-host con un servidor HTTP vinculado al puerto 8080 y conectado a la red example-net. Dentro de otro contenedor conectado a la red example-net, el siguiente comando curl se resolvería en la raíz del servidor HTTP.

```
[user@host ~]$ curl http://nginx-host:8080
```

### Acceso a servicios de redes contenerizados

**Reenvío de puertos**

Por ejemplo, el siguiente comando asigna el puerto 80 dentro del contenedor al puerto 8075 en la máquina host.

```
[user@host ~]$ podman run -p 8075:80 my-app
```

Para publicar un contenedor en un host específico y limitar las redes desde las que puede accederse, use la siguiente forma.

```
[user@host ~]$ podman run -p 127.0.0.1:8075:80 my-app
```

Para enumerar los mapeos de puertos para un contenedor, use el comando podman port. Por ejemplo, el siguiente comando revela que el puerto 8010 de la máquina host está mapeado al puerto 8008 dentro del contenedor.

```
[user@host ~]$ podman port my-app
```

La opción --all muestra una lista de los mapeos de puertos de todos los contenedores.

```
[user@host ~]$ podman port --all
```

**Redes en contenedores**

A los contenedores conectados a las redes Podman se les asignan direcciones IP privadas para cada red. Otros contenedores de la red pueden realizar solicitudes a esta dirección IP.

Por ejemplo, un contenedor llamado my-app se conecta a la red apps. El siguiente comando recupera la dirección IP privada del contenedor dentro de la red apps.

```
[user@host ~]$ podman inspect my-app \
 -f '{{.NetworkSettings.Networks.apps.IPAddress}}'
10.89.0.2
```

### Acceso a los contenedores

**Iniciar procesos en contenedores**

Use el comando podman exec para iniciar un nuevo proceso en un contenedor en ejecución. El comando usa la siguiente sintaxis:

```
podman exec [options] container [command ...]
```

En la explicación de sintaxis anterior, las partes del comando entre corchetes, como [options], son opcionales. Por ejemplo, el siguiente comando imprime el archivo /etc/httpd/conf/httpd.conf usando la utilidad cat en un contenedor en ejecución denominado httpd:

```
podman exec httpd cat /etc/httpd/conf/httpd.conf
```

**Abrir una sesión interactiva en contenedores**

Para abrir una shell interactiva en un contenedor en ejecución, combine las opciones --interactive y --tty.

```
[user@host ~]$ podman exec -til /bin/bash
```

**Copiar archivos dentro y fuera de contenedores**

Use el comando podman cp para copiar archivos desde y hacia un contenedor en ejecución. El comando usa la siguiente sintaxis:

```
podman cp [options] [container]:source_path [container]:destination_path
```

Use el siguiente comando para copiar el archivo /tmp/logs de un contenedor con ID a3bd6c81092e al directorio actual:

```
podman cp a3bd6c81092e:/tmp/logs .
```

Use el siguiente comando para copiar el archivo nginx.conf al directorio /etc/nginx/ en un contenedor denominado nginx:

```
podman cp nginx.conf nginx:/etc/nginx
```

El comando anterior supone que el archivo nginx.conf existe en su directorio de trabajo.

Use el siguiente comando para copiar el archivo nginx.conf desde el contenedor nginx-test al contenedor nginx-proxy:

```
podman cp nginx-test:/etc/nginx/nginx.conf nginx-proxy:/etc/nginx
```

### Gestionar credenciales de registros con Podman

Algunos registros requieren que los usuarios se autentiquen. Por ejemplo, los contenedores de Red Hat basados en RHEL generalmente no permiten el acceso no autenticado:

```
[user@host ~]$ podman pull registry.redhat.io/rhel8/httpd-24
```

Puede elegir una imagen diferente que no requiera autenticación, como la imagen de UBI 8:

```
[user@host ~]$ podman pull registry.access.redhat.com/ubi8:latest
```

Como alternativa, debe ejecutar el comando podman-login para el registro antes de poder extraer la imagen de RHEL 8.

```
[user@host ~]$ podman login registry.redhat.io
Username: YOUR_USER
Password: YOUR_PASSWORD
Login Succeeded!
[user@host ~]$ podman pull registry.redhat.io/rhel8/httpd-24
```

Podman almacena las credenciales en el archivo ${XDG_RUNTIME_DIR}/containers/auth.json, donde ${XDG_RUNTIME_DIR} se refiere a un directorio específico del usuario actual. Las credenciales están codificadas en el formato base64 :

```
[user@host ~]$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
```

### Administración de imágenes

El uso de una etiqueta en Podman es opcional. Cuando no especifica una etiqueta en un comando de Podman, Podman usa la etiqueta latest de forma predeterminada.

```
[user@host ~]$ podman pull quay.io/argoproj/argocd
Trying to pull quay.io/argoproj/argocd:latest...
```

**Extracción de imágenes**

Para buscar imágenes en diferentes registros de imágenes, use un explorador web para navegar a la URL del registro y use la interfaz de usuario web.

Alternativamente, use el comando podman search para buscar imágenes en todos los registros presentes en la lista unqualified-search-registries en su archivo registries.conf. Esto le permite buscar diferentes registros de una sola vez.

```
[user@host ~]$ podman search nginx
```

**Creación de imágenes**

También puede crear una imagen a partir de un Containerfile, que describe los pasos que se usan para crear una imagen. Para ello, debe ejecutar podman build --file CONTAINERFILE --tag IMAGE_REFERENCE.

Por ejemplo, para crear una imagen que luego pueda enviar a Red Hat Quay.io, ejecute el siguiente comando:

```
[user@host ~]$ podman build --file Containerfile \
  --tag quay.io/YOUR_QUAY_USER/IMAGE_NAME:TAG
```

**Inspección de imágenes**

El comando podman image inspect proporciona información útil sobre una imagen disponible localmente en su sistema.

El siguiente ejemplo de uso muestra información sobre una imagen mariadb.

```
[user@host ~]$ podman image inspect registry.redhat.io/rhel8/mariadb-103:1
```

### Crear imágenes con Containerfiles

Instrucciones de Containerfile
Containerfiles usan un pequeño lenguaje específico de dominio (DSL) que consta de instrucciones básicas para crear imágenes de contenedor. Las siguientes son las instrucciones más comunes.

* FROM
Toma el nombre de la imagen base como argumento.

* WORKDIR
Establece el directorio de trabajo actual dentro del contenedor. Las instrucciones posteriores se ejecutan dentro de este directorio.

* COPY
Copia archivos del host de compilación en el sistema de archivos de la imagen de contenedor resultante. Las rutas relativas respetan el directorio de trabajo actual del host, conocido como contexto de compilación y el directorio de trabajo dentro del contenedor definido por WORKDIR. No es posible copiar un archivo remoto mediante su URL con esta instrucción de Containerfile.

* ADD
Copia archivos o carpetas de una fuente local o remota y los agrega al sistema de archivos del contenedor. Si se usa para copiar archivos locales, estos deben estar en el directorio de trabajo. La instrucción ADD también descomprime los archivos de almacenamiento locales .tar en el directorio de la imagen de destino.

* RUN
Ejecuta un comando en el contenedor y confirma el estado resultante del contenedor en una nueva capa dentro de la imagen.

* ENTRYPOINT
Establece el ejecutable para que se ejecute cuando se inicia el contenedor.

* CMD
Ejecuta un comando cuando se inicia el contenedor. Este comando se pasa al ejecutable definido por ENTRYPOINT. Las imágenes base definen un ENTRYPOINT predeterminado, que suele ser un ejecutable de shell, como Bash.

* USER
Cambia el usuario activo dentro del contenedor. Las instrucciones posteriores se ejecutan como este usuario, incluida la instrucción CMD. La práctica adecuada es definir un usuario diferente a root por motivos de seguridad.

* LABEL
Agrega un par clave-valor a los metadatos de la imagen para la organización y la selección de imágenes.

* EXPOSE
Agrega un puerto a los metadatos para la imagen que indica que una aplicación dentro del contenedor se vinculará a este puerto. Tenga en cuenta que esto no vincula el puerto en el host y es para fines de documentación.

* ENV
Define las variables de entorno que están disponibles en el contenedor. Puede declarar varias instrucciones ENV dentro del Containerfile. Puede usar el comando env dentro del contenedor para ver cada una de las variables de entorno.

* ARG
Define variables de tiempo de compilación, generalmente para hacer una compilación de contenedor personalizable. Los desarrolladores suelen configurar las instrucciones ENV mediante la instrucción ARG. Esto es útil para conservar las variables de tiempo de compilación para el tiempo de ejecución.

* VOLUME
Define dónde almacenar los datos fuera del contenedor. El valor designa la ruta donde Podman monta el directorio dentro del contenedor. Puede definir más de una ruta para crear varios volúmenes.

Cada instrucción Containerfile se ejecuta en un contenedor independiente usando una imagen intermedia compilada de todos los comandos anteriores. Esto significa que cada instrucción es independiente de otras instrucciones en el Containerfile. A continuación, se incluye un ejemplo de Containerfile para compilar un contenedor de servidores web simple de Apache:

```
# This is a comment line 1
FROM        registry.redhat.io/ubi8/ubi:8.6 2
LABEL       description="This is a custom httpd container image" 3
RUN         yum install -y httpd 4
EXPOSE      80 5
ENV         LogLevel "info" 6
ADD         http://someserver.com/filename.pdf /var/www/html 7
COPY        ./src/   /var/www/html/ 8
USER        apache 9
ENTRYPOINT  ["/usr/sbin/httpd"] 10
CMD         ["-D", "FOREGROUND"] 11
```

1

Las líneas que comienzan con numeral (#) son comentarios.

2

La instrucción FROM declara que la nueva imagen de contenedor se extiende a la imagen base del contenedor registry.redhat.io/ubi8/ubi:8.6ubi8/ubi:8.6.

3

La instrucción LABEL es responsable de agregar metadatos genéricos a una imagen.

4

RUN ejecuta comandos en una nueva capa sobre la imagen actual. La shell que se usa para ejecutar comandos es /bin/sh.

5

EXPOSE indica que el contenedor escucha en el puerto de red especificado en tiempo de ejecución. La instrucción EXPOSE define metadatos únicamente; no permite que se pueda acceder a los puertos desde el host.

6

ENV es responsable de definir las variables de entorno que están disponibles en el contenedor.

7

La instrucción ADD copia archivos o carpetas de una fuente local o remota y los agrega al sistema de archivos del contenedor.

8

COPY copia archivos desde el directorio de trabajo y los agrega al sistema de archivos del contenedor.

9

USER especifica el nombre de usuario o el UID que se usará cuando se ejecute la imagen de contenedor para las instrucciones RUN, CMD y ENTRYPOINT

10

ENTRYPOINT especifica el comando predeterminado para ejecutarse cuando se ejecuta la imagen en un contenedor.

11

CMD proporciona los argumentos predeterminados para la instrucción ENTRYPOINT.

### Montaje de volúmenes

**Almacenamiento de datos con montajes de enlace**

Los montajes de enlace pueden existir en cualquier lugar del sistema host.

Por ejemplo, para montar el directorio /www de su máquina host en el directorio /var/www/html dentro del contenedor con la opción de solo lectura, use el siguiente comando podman-run:

```
[user@host ~] $ podman run -p 8080:8080 --volume  /www:/var/www/html:ro \
  registry.access.redhat.com/ubi8/httpd-24:latest
```

**Almacenamiento de datos con volúmenes**

Los volúmenes permiten que Podman gestione los montajes de datos. Puede gestionar volúmenes mediante el comando podman-volume.

Para crear un nuevo volumen llamado http-data, use el siguiente comando:

```
[user@host ~] $ podman volume create http-data
d721d941960a2552459637da86c3074bbba12600079f5d58e62a11caf6a591b5
```

Puede inspeccionar el volumen con el comando podman-volume-inspect:

```
[user@host ~] $ podman volume inspect http-data
```

Para los contenedores sin root, Podman almacena los datos del volumen local en el directorio $HOME/.local/share/containers/storage/volumes/.

Para montar el volumen en un contenedor, consulte el volumen por el nombre del volumen:

```
[user@host ~] $ podman run -p 8080:8080 --volume  http-data:/var/www/html \
  registry.access.redhat.com/ubi8/httpd-24:latest
```

**Importación de datos con volúmenes**

Puede importar datos de un archivo tar a un volumen Podman existente mediante el comando podman volume import VOLUME NAME TAR_ARCHIVE.

```
[user@host ~] $ podman volume import http_data web_data.tar.gz
```

También puede exportar datos de un volumen Podman existente y guardarlo como un archivo tar en la máquina local mediante el comando podman export VOLUME NAME --output ARCHIVE_NAME.

```
[user@host ~] $ podman volume export http_data --output web_data.tar.gz
```

**Almacenamiento de datos con un montaje tmpfs**

Algunas aplicaciones no pueden usar el sistema de archivos COW predeterminado en un directorio específico por razones de rendimiento, pero los desarrolladores no usan la persistencia o el uso compartido de datos para ese directorio.

Para tales casos, puede usar el tipo de montaje tmpfs, lo que significa que los datos en un montaje son efímeros pero no usan el sistema de archivos COW:

```
[user@host ~] $ podman run -e POSTGRESQL_ADMIN_PASSWORD=redhat --network lab-net \
  --mount  type=tmpfs,tmpfs-size=512M,destination=/var/lib/pgsql/data \
  registry.redhat.io/rhel9/postgresql-13:1
```

### Registro de contenedores y solución de problemas

**Solucionar problemas de inicio de contenedores**

Si el contenedor está en el estado Exited, entonces el problema podría estar en el proceso de inicio. Muchas aplicaciones muestran información de error cuando encuentran un problema durante el inicio. Para acceder a esta información, puede usar el comando podman logs.

```
[user@host ~]$ podman logs CONTAINER
```

Puede usar el comando podman port CONTAINER para enumerar el mapeo de puertos del contenedor actual.

```
[user@host ~]$ podman port CONTAINER
```

Para verificar los puertos de la aplicación en uso, enumere los puertos de red abiertos en el contenedor en ejecución. Use comandos de Linux como el comando de estadísticas de socket (ss) para enumerar los puertos abiertos. Un socket es la combinación de un puerto y una dirección IP. El comando ss enumera los sockets abiertos en un sistema. Puede proporcionar opciones al comando ss para filtrar y producir la salida deseada:

-p: mostrar el proceso usando el socket

-a: mostrar la escucha y las conexiones establecidas

-n: mostrar las direcciones IP

-t: mostrar sockets TCP

```
[user@host ~]$ podman exec -it CONTAINER ss -pant
```

Para obtener el PID del contenedor, puede usar el siguiente comando podman inspect:

```
[user@host ~]$ podman inspect CONTAINER --format '{{.State.Pid}}'
```

Tenga en cuenta que debe anteponer el comando nsenter con el comando sudo para otorgarle los permisos necesarios.

```
[user@host ~]$ sudo nsenter -n -t CONTAINER_PID ss -pant
```

**Problemas de conectividad de la red de contenedores**

Puede usar el comando podman inspect para verificar que cada contenedor esté usando una red específica.

```
[user@host ~]$ podman inspect CONTAINER --format='{{.NetworkSettings.Networks}}'
```

**Problemas de resolución de nombre**

Para asegurarse de que el DNS esté habilitado para una red Podman, use el comando podman nework inspect.

```
[user@host ~]$ podman network inspect NETWORK
```

**Solucionar problemas de montajes de enlace**

Al usar montajes de enlace, debe configurar los permisos de archivo y el acceso a SELinux manualmente. SELinux es un mecanismo de seguridad adicional usado por Red Hat Enterprise Linux (RHEL) y otras distribuciones de Linux.

Considere el siguiente ejemplo de montaje de enlace:

```
[user@host ~]$ podman run -p 8080:8080 --volume /www:/var/www/html \
  registry.access.redhat.com/ubi8/httpd-24:latest
```

Para solucionar problemas de permisos de archivos, use el comando podman unshare para ejecutar el comando ls -l. El comando podman unshare ejecuta los comandos de Linux proporcionados en un nuevo espacio de nombres, como el que crea Podman para el contenedor. Esto mapea los ID de usuario a medida que se mapean en un nuevo contenedor, lo cual es útil para solucionar problemas de permisos de usuario.

```
[user@host ~]$ podman unshare ls -l /www/
```

Para solucionar problemas de permisos de SELinux, inspeccione la configuración de SELinux del directorio /www ejecutando el comando ls con la opción -Z. Use la opción -d para imprimir solo la información del directorio.

```
[user@host ~]$ ls -Zd /www
```

Para corregir la configuración de SELinux, agregue la opción :z o :Z al montaje de enlace:

La z minúscula permite que diferentes contenedores compartan el acceso a un montaje de enlace.

La Z mayúsculas proporciona al contenedor acceso exclusivo al montaje de enlace.

```
[user@host ~]$ podman run -p 8080:8080 --volume /www:/var/www/html:Z \
  registry.access.redhat.com/ubi8/httpd-24:latest
```

Después de agregar la opción correspondiente, ejecute el comando ls -Zd y observe el tipo de SELinux correcto.

```
[user@host ~]$ ls -Zd /www
```

### Descripción general y casos de uso de Compose

**Orqueste contenedores con Podman Compose**

Podman Compose es una herramienta de código abierto que puede usar para ejecutar archivos Compose. Un archivo Compose es un archivo YAML que especifica los contenedores que se deben gestionar, así como las dependencias entre ellos.

Por ejemplo, considere el siguiente archivo Compose:

```
version: '3'
services:
  orders: 1
    image: quay.io/user/python-app 2
    ports:
    - 3030:8080 3
    environment:
      ACCOUNTS_SERVICE: http://accounts 4
```

1

Declare el contenedor orders.

2

Use la imagen de contenedor python-app.

3

Vincule el puerto 3030 en su máquina host al puerto 8080 en el contenedor.

4

Pase la variable de entorno ACCOUNTS_SERVICE a la aplicación.

--------------------------------------------------------------------------------------------------------------------
**Importante**

No se recomienda usar Podman Compose en un entorno de producción. Podman Compose no soporta funciones avanzadas que pueda necesitar en un entorno de producción, como el balanceo de carga, la distribución de contenedores a varios nodos, la gestión de contenedores en diferentes nodos y otros.

Si necesita una solución de orquestación de contenedores de producción, Kubernetes o Red Hat OpenShift es una mejor opción. Red Hat OpenShift puede ejecutar aplicaciones de varios contenedores en diferentes máquinas host o nodos.
--------------------------------------------------------------------------------------------------------------------

El siguiente archivo Compose define los servicios backend y db. Los términos servicio y contenedor se usan indistintamente. También anula el comando predeterminado para la imagen de back end al especificar la propiedad command.

```
version: "3.9"
services:
  backend:
    image: quay.io/user/backend
    ports:
      - "8081:8080"
    command: sh -c "COMMAND"
  db:
    image: registry.redhat.io/rhel8/postgresql-13
    environment:
      POSTGRESQL_ADMIN_PASSWORD: redhat
```

**Iniciar y detener contenedores con Podman Compose**

Puede ejecutar un archivo Compose mediante el comando podman-compose up, que crea objetos que se definen en el archivo Compose e inicia los contenedores.

```
[user@host ~]$ podman-compose up
```

Puede enumerar los objetos que están definidos en el archivo Compose usando el comando podman, como podman network ls, para enumerar las redes Podman.

```
[user@host ~]$ podman network ls
```

El comando podman-compose stop detiene la ejecución de contenedores definidos como servicios. Ejecute podman-compose down para detener y eliminar contenedores que se definen como servicios:

```
[user@host ~]$ podman-compose down
```

**Redes**

Use la palabra clave networks en el mismo nivel de sangría que la palabra clave services para crear y usar redes Podman. Si la palabra clave networks no está definida en el archivo Podman Compose, Podman Compose crea una red predeterminada que tiene el DNS habilitado.

Considere el siguiente archivo Compose que declara tres contenedores: una aplicación front end, una aplicación back end y una base de datos.

```
version: "3.9"
services:
  frontend:
    image: quay.io/user/frontend
    networks: 1
      - app-net
    ports:
      - "8082:8080"
  backend:
    image: quay.io/user/backend
    networks: 2
      - app-net
      - db-net
  db:
    image: registry.redhat.io/rhel8/postgresql-13
    environment:
      POSTGRESQL_ADMIN_PASSWORD: redhat
    networks: 3
      - db-net

networks: 4
  app-net: {}
  db-net: {}
```

1

El servicio frontend es parte de la red app-net.

2

El servicio backend es parte de las redes app-net y db-net.

3

El servicio db es parte de db-net.

4

Definición de las redes.

**Volúmenes**

Puede declarar volúmenes con la palabra clave volumes. Luego, monte los volúmenes en contenedores con la sintaxis VOLUME_NAME:_CONTAINER_DIRECTORY_:_OPTIONS_.

```
version: "3.9"
services:
  db:
    image: registry.redhat.io/rhel8/postgresql-13
    environment:
      POSTGRESQL_ADMIN_PASSWORD: redhat
    ports:
      - "5432:5432"
    volumes: 1
      - db-vol:/var/lib/postgresql/data

volumes: 2
  db-vol: {}
```

1

El servicio db asigna el volumen db-vol al directorio /var/lib/postgresql/data dentro del contenedor.

2

Declaración de volúmenes.

### Implementar aplicaciones en OpenShift

**Crear pods de forma declarativa**

El siguiente objeto YAML demuestra los campos importantes de un objeto de pod:

```
kind: Pod 1
apiVersion: v1
metadata:
  name: example-pod 2
  namespace: example-project 3
spec: 4
  containers: 5
  - name: example-container 6
    image: quay.io/example/awesome-container 7
    ports: 8
    - containerPort: 8080
    env: 9
    - name: GREETING
      value: "Hello from the awesome container"
```

1

Este objeto YAML define un pod.

2

El campo .metadata.name define el nombre del pod.

3

El proyecto en el que crea el pod. Si este proyecto no existe, falla la creación del pod. Si no especifica un proyecto, RHOCP usa el proyecto configurado actualmente.

4

El campo .spec contiene la configuración del objeto del pod.

5

El campo .spec.containers define los contenedores en un pod. En este ejemplo, example-pod contiene un contenedor.

6

Nombre del contenedor dentro de un pod. Los nombres de los contenedores son importantes para los comandos oc cuando un pod contiene varios contenedores.

7

La imagen del contenedor.

8

Los metadatos del puerto especifican qué puertos usa el contenedor. Esta propiedad es similar a la propiedad de Containerfile EXPOSE.

9

La propiedad env define variables de entorno.

**Crear pods de forma imperativa**

Para fines de prueba, RHOCP también proporciona el enfoque imperativo para crear objetos RHOCP. El enfoque imperativo usa el comando oc run para crear un pod sin una definición.

El siguiente comando crea el pod example-pod :

```
[user@host ~]$ oc run example-pod \ 1
  --image=quay.io/example/awesome-container \ 2
  --env GREETING='Hello from the awesome container' \ 3
  --port=8080 4
pod/example-pod created
```

1

La definición de pod .metadata.name.

2

La imagen usada para el contenedor único en este pod.

3

La variable de entorno para el contenedor único en este pod.

4

La definición de metadatos del puerto.

El siguiente comando es un ejemplo de cómo generar la definición YAML para el pod example-pod.

```
[user@host ~]$ oc run example-pod \
  --image=quay.io/example/awesome-container \
  --env GREETING='Hello from the awesome container' \
  --port=8080 \
  --dry-run=client -o yaml
```

**Crear servicios de forma declarativa**

El siguiente objeto YAML muestra un objeto de servicio:

```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  ports:
  - port: 8080 1
    protocol: TCP
    targetPort: 8080 2
  selector: 3
    app: backend-app
```

1

Puerto de servicio. Este es el puerto en el que escucha el servicio.

2

Puerto de destino. Este es el puerto del pod al que el servicio enruta las solicitudes. Este puerto corresponde al valor containerPort en la definición de pod.

3

El selector configura a qué pods apuntar. En este caso, el servicio se enruta a cualquier pod que contenga la etiqueta app=backend-app.

**Crear servicios de forma imperativa**

De manera similar a los pods, puede crear servicios de manera imperativa mediante el comando oc expose. El siguiente ejemplo crea un servicio para el pod backend-app :

```
[user@host ~]$ oc expose pod backend-app \
  --port=8080 \ 1
  --targetPort=8080 \ 2
  --name=backend-app 3
service/backend-app exposed
```

1

El puerto en el que escucha el servicio.

2

Puerto de contenedores de destino.

3

Nombre del servicio

También puede usar las opciones --dry-run=client y -o para generar una definición de servicio, por ejemplo:

```
[user@host ~]$ oc expose pod backend-app \
  --port=8080 \
  --targetPort=8080 \
  --name=backend-app \
  --dry-run=client -o yaml
```

### Aplicaciones de varios pods

**Crear implementaciones de forma declarativa**

El siguiente objeto YAML demuestra una implementación de Kubernetes:

```
apiVersion: apps/v1
kind: Deployment
metadata: 1
  labels:
    app: deployment-label
  name: example-deployment
spec:
  replicas: 3 2
  selector: 3
    matchLabels:
      app: example-deployment
  strategy: RollingUpdate 4
  template: 5
    metadata:
      labels:
        app: example-deployment
    spec: 6
      containers:
      - image: quay.io/example/awesome-container
        name: awesome-pod
```

1

Metadatos del objeto de implementación. Esta implementación usa la etiqueta app=deployment-label.

2

Cantidad de réplicas del pod. Esta implementación mantiene 3 contenedores idénticos en 3 pods.

3

Selector de pods que gestiona la implementación. Esta implementación gestiona pods que usan la etiqueta app=example-deployment.

4

Estrategia de actualización de pods. La estrategia RollingUpdate predeterminada garantiza la implementación gradual del pod sin tiempo de inactividad cuando modifica la imagen del contenedor.

5

La configuración de la plantilla para los pods que crea y gestiona la implementación.

6

El campo .spec.template.spec corresponde al campo .spec del objeto Pod.

**Crear implementaciones de forma imperativa**

Puede usar el comando oc create deployment para crear una implementación.

```
[user@host ~]$ oc create deployment example-deployment \ 1
  --image=quay.io/example/awesome-container \ 2
  --replicas=3 3
deployment/example-deployment created
```

1

Establezca el nombre en example-deployment.

2

Defina la imagen.

3

Cree y mantenga 3 pods de réplica.

También puede usar las opciones --dry-run=client y -o para generar una definición de implementación, por ejemplo:

```
[user@host ~]$ oc create deployment example-deployment \
  --image=quay.io/example/awesome-container \
  --replicas=3 \
  --dry-run=client -o yaml
```

**Selección de pod de implementación**

Los controladores crean y administran el ciclo de vida de los pods que se especifican en la configuración del controlador. En consecuencia, un controlador debe tener una relación de propiedad entre los pods y un pod puede ser propiedad de un controlador como máximo.

El controlador de implementación usa etiquetas para apuntar a los pods dependientes. Por ejemplo, considere la siguiente configuración de implementación:

```
apiVersion: apps/v1
kind: Deployment
...configuration omitted...
spec:
  selector:
    matchLabels: 1
      app: example-deployment
  template:
    metadata:
      labels: 2
        app: example-deployment
    spec:
...configuration omitted...
```

1

El campo .spec.selector.matchLabels.

2

El campo .spec.template.metadata.labels.

El campo .spec.template.metadata.labels determina el conjunto de etiquetas aplicadas a los pods creados o administrados por esta implementación. En consecuencia, el campo .spec.selector.matchLabels debe ser un subconjunto de etiquetas del campo .spec.template.metadata.labels.

**Crear rutas de forma declarativa**

El siguiente objeto YAML demuestra una ruta de RHOP:

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: app-ui
  name: app-ui
  namespace: awesome-app
spec:
  port:
    targetPort: 8080 1
  host: ""
  to: 2
    kind: "Service"
    name: "app-ui"
```

1

El puerto de destino en los pods seleccionados por el servicio al que apunta esta ruta. Si usa una cadena, la ruta usa un puerto con nombre en la lista de puertos de los extremos (endpoints) de destino.

2

El destino de la ruta. Actualmente, solo se permite el destino Service.

**Crear rutas de forma imperativa**

Puede usar el comando oc expose service para crear una ruta:

```
[user@host ~]$ oc expose service app-ui
```

También puede usar las opciones --dry-run=client y -o para generar una definición de ruta, por ejemplo:

```
[user@host ~]$ oc expose service app-ui \
  --dry-run=client -o yaml
```

