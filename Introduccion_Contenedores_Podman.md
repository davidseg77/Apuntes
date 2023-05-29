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




