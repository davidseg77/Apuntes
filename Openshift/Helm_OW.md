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













