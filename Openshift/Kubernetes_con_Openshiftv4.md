# Info del curso de Kubernetes con Openshift V4 de OW

### Instalación de la herramienta OC

Lo descargamos para el SO requerido y en la terminal nos aseguramos de contar con el archivo. Hecho esto, lo descomprimimos.

```
tar xvf oc.tar
```

Despues llevamos el archivo binario oc al path donde nos permitirá que pueda ser ejecutado:

```
sudo install oc /usr/local/bin
```

Para comprobarlo

```
oc version
```

### Configuración de oc para Developer Sandbox

Habremos de autenticarnos mediante el token proporcionado por Openshift. Lo hallamos en la parte superior a la derecha. Esto nos dará el token junto con el comando que permite su inserción:

```
oc login --token=xxxx --server=xxxxx
```

Para ver aspectos como el usuario, el servidor... en el cual estamos trabajando podemos verlo en .kube/config.
Y para ver los proyectos que tenemos activos:

```
oc get project
```

Para ver los namespaces que tenemos en nuestros proyectos

```
oc get ns
```

### CRC

**Instalación en local**

Instalamos los paquetes kvm. Estos comandos vienen en la docu de instalar CRC en ubuntu. 

```
sudo apt install qemu-kvm libvirt-daemon libvirt-daemon-system network-manager
```

Hemos de asegurarnos de que el usuario con el que estemos ejecutando estos pasos pertenezca al grupo libvirt:

```
sudo adduser usuario libvirt
```

A continuación tienes que bajarte la última versión de CRC, eligiendo la versión del sistema operativo que estés usando, desde la página oficial de descarga (para ello tendrás que hacer login con un cuenta de Red Hat): https://console.redhat.com/openshift/create/local

Además de bajarte la versión de CRC, tendrás que bajarte o copiar en el portapapeles un token que durante la instalación tendrás que introducir. (Download pull secret)

Una vez descargado el paquete lo descomprimimos y lo copiamos a un directorio del PATH para poder ejecutarlo:

```
tar -xf crc-linux-amd64.tar.xz
cd crc-linux-2.17.0-amd64
sudo install crc /usr/local/bin

crc version
CRC version: 2.17.0+44e15711
OpenShift version: 4.12.9
Podman version: 4.4.1
```

Para preparar el entorno, creando entre otras cosas la red para kvm.

```
crc setup
```

A continuación, aunque no es necesario, si necesitamos aumentar los recursos de la máquina virtual que vamos a crear podemos hacerlo de la siguiente manera:

```
crc config set cpus 8
crc config set memory 20480
```

Esto también es opcional, pero si queremos que durante la instalación se instalen los componentes de telemetría para mostrar las métricas de los recursos que se están utilizando, debes realizar la siguiente configuración:

```
crc config set enable-cluster-monitoring true
```

Finalmente creamos la máquina virtual, que ejecutará el clúster de OpenShift v4:

```
crc start
```

Durante el proceso nos pedirán que introduzcamos el token que hemos bajado o copiado:

? Please enter the pull secret  

Después de unos minutos, el clúster estará preparado y nos dará información para acceder.

No es necesario instalar la herramienta oc, durante el proceso de instalación se ha descargado, lo único que tenemos que hacer es configurar el PATH para que podamos acceder a ella, para ello ejecutamos:

```
eval $(crc oc-env)
```

**Algunos detalles de la instalación**

Como hemos dicho, todos los ficheros relacionados con CRC se guardan en el directorio ~/.crc. La configuración de acceso al clúster, al igual que en kubernetes se guarda en el fichero ~/.kube/config. En nuestro caso:

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tL..
    server: https://api.crc.testing:6443
  name: api-crc-testing:6443
contexts:
- context:
    cluster: api-crc-testing:6443
    namespace: default
    user: kubeadmin
  name: crc-admin
- context:
    cluster: api-crc-testing:6443
    namespace: default
    user: developer
  name: crc-developer
current-context: crc-admin
kind: Config
preferences: {}
users:
- name: developer
  user:
    token: sha256~gEE6O8LrHV444o44W6kryQSR8pDGKMxnNdkblvX1P9M
- name: kubeadmin
  user:
    token: sha256~qdvyZgGGYo32tdpGh1adh8eu_-NaP5ESgoJTmD2LA1Y
```

**Algunos comando útiles de crc**

Cada vez que empecemos a utilizar CRC iniciamos la máquina virtual con:

```
crc start
```

Cuando terminemos de trabajar, paramos la máquina con:

```
crc stop
```

Si necesitamos hacer una nueva instalación y eliminar la máquina virtual, ejecutamos:

```
crc delete
```

Podemos acceder a la consola web usando la URL https://console-openshift-console.apps-crc.testing o ejecutar el comando:

```
crc console
```

Si queremos la información para loguearnos con el comando oc podemos ejecutar:

```
crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p xxxxxxxxxxxxxxx https://api.crc.testing:6443'
```

**Configuración de oc para CRC**

Por defecto la instalación de OpenShift en local no tiene ningún Operador instalado. Los Operadores nos permiten instalar componentes internos de OpenShift que añaden funcionalidades extras a nuestro clúster.

Para poder conectarnos a un terminal desde la consola web y tener a nuestra disposición la herramienta oc tenemos que instalar el operador WebTerminal (por dependencias se instará también el operador DevWorkspace Operator). Nos tenemos que conectar con el usuario administrador kubeadmin y en la vista Administrator accedemos a la opción Operators->OperatorHub y filtramos con el nombre del operador "WebTerminal"

Nos aparece una ventana con información del operador y pulsamos sobre el botón Install para comenzar la instalación, dejamos los valores por defecto, realizamos la instalación y comprobamos los operadores que hemos instalados en la opción Operators->Installed Operators.

### Proyectos en Openshift

Un proyecto en OpenShift v4 es una agrupamiento lógico de recursos. En realidad es muy parecido al recurso namespace de Kubernetes pero puede guardar información adicional. Si eliminamos un proyecto se eliminarán todos los recursos que hemos creado en él.

Vamos a seguir trabajando con el usuario developer, que hemos usado para acceder al clúster de OpenShift:

```
oc login -u developer -p developer https://api.crc.testing:6443
```

Por lo tanto lo primero que vamos a hacer es crear un nuevo proyecto que utilizará este usuario:

```
oc new-project developer
```

Vemos que ahora está usando el proyecto developer. Para obtener la lista de proyectos disponibles:

```
oc get projects
```

Y para ver los detalles del proyecto, ejecutamos:

```
oc describe project developer
```

Si creamos un nuevo proyecto:

```
oc new-project developer2
```

Nos podemos posicionar en uno de ellos, ejecutando:

```
oc project developer
```

**Consola web CRC**

Para acceder a la consola web, usamos la URL: https://console-openshift-console.apps-crc.testing y nos pide que hagamos login.

Usamos el usuario developer o el usuario kubeadmin para acceder con un usuario normal o un usuario administrador. Las opciones serán las mismas, pero como hemos visto el usuario developer no tendrá acceso a algunos recursos.

También se puede acceder mediante:

```
crc console
```

**Proyectos y namespaces**

Como hemos comentado un objeto project es en realidad un namespace que puede guardar más información. De hecho, cada vez que creamos un proyecto se creará un namespace. Sin embargo el usuario developer no tiene permiso para acceder a los namespaces:

```
oc get ns
Error from server (Forbidden): namespaces is forbidden: User "developer" cannot list resource "namespaces" in API group "" at the cluster scope
```

Vamos a conectarnos como administrador para poder acceder a los namespaces:

```
oc login -u kubeadmin https://api.crc.testing:6443
```

El administrador tiene acceso a 68 proyectos, por defecto, está usando el último que se ha creado. Podemos ver todos los proyectos, ejecutando:

```
oc get projects
```

Ahora si podemos acceder a los namespaces, ejecutando:

```
oc get ns
```

Finalmente para borrar un proyecto, ejecutamos:

```
oc delete project developer2
```














