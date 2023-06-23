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



















