# Configuración de Helm Chart de instalación de ArgoCD y Manifiestos 

En este apartado veremos como configurar parámetros básicos en el values del Helm Chart de ArgoCD (Ej: contraseña de admin y repositorios) y como configurar, repositorios, proyectos y aplicaciones desde manifiestos aplicables desde la CLI.

## 1. Requisitos

* Tener instalado el binario de Helm https://helm.sh/docs/intro/install/

* Tener instalado Minikube https://minikube.sigs.k8s.io/docs/start/

* Tener instalado ArgoCD sigue los pasos en mi video anterior: https://github.com/TheAutomationRules/argocd/blob/main/Video_01/README.md


## 2. Configuración e instalación de ArgoCD desde values

Estando ubicados en el mismo path en el que se encuentra el directorio argo-cd con el chart de ArgoCD descomprimido y vemos los siguientes archivos:

``` 
argo-cd
argo-cd-5.8.2.tgz
``` 

Duplicamos el archivo values.yaml y creamos una copia con el nombre values-custom.yaml

``` 
cp argo-cd/values.yaml argo-cd/values-custom.yaml
```

Generamos la contraseña ARGOCDPASS todo en mayusculas con el siguiente comando y la usamos para editar el nuevo values-custom.yaml.

``` 
htpasswd -nbBC 10 "" ARGOCDPASS | tr -d ':\n' | sed 's/$2y/$2a/'
```

Copiamos el resultado de la contraseña generada sin incluir el caracter de porcentaje del final "%", editamos el values-custom.yaml, buscamos "argocdServerAdminPassword:" y definimos alli la contraseña generada.

``` 
argocdServerAdminPassword: "$2a$10$sTPh5sJJafZvxb1WG8D6h.seMOewbZKgE/GFTyHcTvgyHF5tboKWm"
``` 

Buscamos "credentialTemplates:" y aqui definimos el template de las credenciales que usaremos para conectarnos a nuestros repositorios de Git en este caso repositorios de GitHub.

``` 
github-user-public:
    url: https://github.com/TheAutomationRules
```

Buscamos "repositories: {}" y en esta seccion definimos los repositorios de git que queremos añadir en la instalación de ArgoCD.

```
theautomationrules-argocd:
    url: https://github.com/TheAutomationRules/argocd.git
    name: theautomationrules-argocd
```

Instalamos argocd seleccionando el values file que hemos customizado. Previamente, habrá que añadir el repo de Helm tal y como se hace en el archivo Introducción.

``` 
helm install argo-cd argo-cd/ \
  --namespace argocd \
  --create-namespace --wait \
  -f argo-cd/values-custom.yaml
``` 

Y hacemos un portforward para acceder desde el navegador:

``` 
kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443
```

## 3. Manifiestos y despliegue


Creamos un namespace de pruebas llamado testing

```
kubectl create ns argocd-demo
``` 

Ahora podemos crear o añadir el repositorio en el que se encuentra nuestra aplicación a nuestro ArgoCD para que pueda ser usado por el Project y la Application.

``` 
k apply -f repository.yaml
``` 

Vemos el contenido del yaml de repository:

``` 
apiVersion: v1
kind: Secret
metadata:
  name: argocd-demo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  name: argocd-demo
  url: https://github.com/The-Automation-Rules/argocd-demo.git
  password: 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 (token de Github)
```

Ahora creamos el Project aplicando el manifest donde hemos hecho la definición del mismo.

``` 
k apply -f project.yaml
``` 

Vemos el contenido del yaml de project:

``` 
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: argocd-demo
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  description: Demo Project

  sourceRepos:
    #- '*'
    - https://github.com/The-Automation-Rules/argocd-demo.git

  destinations:
    - namespace: '*'
      server: https://kubernetes.default.svc
      name: in-cluster

  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

Y finalmente creamos la aplicacion para que inicie la sincronización.

``` 
k apply -f application.yaml
``` 

Vemos el yaml de la aplicación:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd-demo
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    name: argocd-demo
spec:
  project: argocd-demo

  source:
    repoURL: https://github.com/The-Automation-Rules/argocd-demo.git
    targetRevision: main
    path: official/examples/helm-guestbook
    helm:
      valueFiles:
        - values-test.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd-demo

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - Validate=false
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true

  ignoreDifferences:
    - group: "*"
      kind: "*"
```

Ahora ya podemos ver nuestro repositorio, proyecto y aplicación instaladas, configuradas y sincronizadas!

Si queremos ver la aplicación funcionando hacemos un Port Forward del 8081 al 80 del service

``` 
kubectl port-forward svc/argocd-demo-helm-guestbook -n argocd-demo 8081:80
``` 

Accedemos desde el navegador a la dirección localhost:8081 y podemos ver la aplicacion funcionando!


## 4. Eliminación y desinstalación

Desinstalamos todo! para ello lo primero, es eliminar la aplicación desde el Dashboard de ArgoCD.

Y ahora desinstalamos el Chart de ArgoCD

``` 
helm uninstall argo-cd -n argocd
``` 

Ahora podemos eliminar los Namespaces que hemos creado

``` 
kubectl delete ns argocd argocd-demo
``` 

Ahora ya podemos detener nuestro Minikube

```
minikube stop
``` 

