# Automatiza los despliegues con ArgoCD (Image Updater)

Vamos a mostrar como automatizar la actualizacion de versiones de las aplicaciones desplegadas en tu cluster kubernetes desde ArgoCD con ArgoCD Image Updater.

## 1. Instalación basado en el video anterior.

Para instalar ArgoCD seguiremos los mismos pasos que en nuestros videos anteriores, para ello usaremos el directorio de instalación del video anterior parados dentro de la raiz del directorio Video_03.

``` 
helm install argo-cd argo-cd/ \
      --namespace argocd \
      --create-namespace --wait \
      -f ../Video_02/argo-cd/values-custom.yaml
``` 

Creamos un proyecto de pruebas llamado argocd-demo

``` 
kubectl apply -f argocd-manifests/project-argocd-demo.yaml
``` 

Creamos un namespace de pruebas llamado argocd-demo

```
kubectl create ns argocd-demo
``` 

Creamos un secret con el Token de GitHub con permisos packages:read en el namespace argocd-demo con el formato username:token y en base64, es el token que usara ArgoCD Image Updater para conectar con el registry y hacer pull de la imagen.

``` 
kubectl apply -f k8s/git-secret.yaml
``` 

Ahora levantamos un port-forward para acceder a ArgoCD

``` 
kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443
``` 

Para instalar ArgoCD Image Updater añadimos el repo de Helm de ArgoCD Image Updater

``` 
helm repo add argo https://argoproj.github.io/argo-helm
``` 

Ahora hacemos un pull del Helm Chart de instalación de Image Updater, lo he ubicado en este mismo repo en official/argocd-image-updaterya me envio

``` 
helm pull argo/argocd-image-updater argo/argocd-image-updater --version 0.8.1
``` 

Duplicamos el archivo values.yaml y lo renombramos como values-custom.yaml, lo importante que tenemos que modificar es en la línea 101 (registries), necesitamos añadir nuestro registry, en mi caso voy a usar el GitHub Container Registry, ejemplo:

``` 
  registries: #[]
    - name: GitHub Container Registry
      prefix: ghcr.io
      api_url: https://ghcr.io
      ping: no
      insecure: true
      credentials: secret:argocd/git-credentials#creds
``` 

Instalamos ahora si el Image Updater pero lo hacemos desde una aplicacion de ArgoCD por supuesto:

```
kubectl apply -f argocd-manifests/image-updater.yaml
``` 

Ahora instalamos nuestra aplicacion DEMO que no es mas que un NGiNX customizado que hemos generado con el Dockerfile que tenemos en el directorio docker/nginx/Dockerfile, y obviamente tambien lo hacemos con una aplicacion de ArgoCD.

```
kubectl apply -f argocd-manifests/nginx-demo-arm.yaml
``` 

Levantamos un port-forward para que podamos acceder al NGiNX DEMO

``` 
kubectl port-forward service/nginx-demo-arm -n argocd-demo 8082:80
``` 

## 2. Simplificar instalando directamente desde un archivo Makefile

Para ello, se crea un archivo Makefile de la siguiente forma:

``` 
start: minikubestart installargo project createns gitsecret forward
installiu: installimageupdater installapp sleep forward-app

#----------------------------------------------------------------------------

minikubestart:
	minikube start

installargo:
	@echo "| > INSTALACION DE ARGOCD"
	helm install argo-cd ../Video_02/argo-cd/ \
      --namespace argocd \
      --create-namespace --wait \
      -f ../Video_02/argo-cd/values-custom.yaml

project:
	@echo "| > CREACION DE PROYECTO DEMO"
	kubectl apply -f argocd-manifests/project-argocd-demo.yaml

createns:
	kubectl create ns argocd-demo

gitsecret:
	@echo "| > INSTALACION DE GIT SECRET"
	kubectl apply -f k8s/git-secret.yaml

forward:
	@echo "| > FORWARD DE PUERTO DE ACCESO A ARGOCD"
	kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443

installimageupdater:
	@echo "| > INSTALACION DE IMAGE UPDATER"
	kubectl apply -f argocd-manifests/image-updater.yaml

installapp:
	@echo "| > INSTALACION DE APLICACION DEMO"
	kubectl apply -f argocd-manifests/nginx-demo-arm.yaml

sleep:
	@echo "DESPLEGANDO..."
	for i in `seq 60 -1 1` ; do echo -ne "segundos para finalizar despliegue! \r$i " ; sleep 1 ; done

forward-app:
	@echo "| > FORWARD DE PUERTO DE ACCESO A APP"
	kubectl port-forward service/nginx-demo-arm -n argocd-de
```

Y, ahora sí, instalamos ArgoCD

``` 
make start
``` 

Instalamos ArgoCD Image Updater

``` 
make installiu
``` 

Accedemos a la App DEMO

``` 
make forward-app
``` 


## 3. Para que Image Updater tire de la última versión disponible

En primer lugar, dentro del directorio de kubernetes (k8s), creamos un yaml git-secret:

``` 
apiVersion: v1
kind: Secret
metadata:
  name: git-credentials
  namespace: argocd
type: Opaque
data:
  # Format: username:password (base64 encoded)
  # Eg: echo -n "username:password" | base64
  # Token permissions: read:packages
  creds: dGhlYXV0b21hdGlvbnJ1bGVzOmdocF91azgyUnZXcTRldGd1UnRnVnpLN1NVVDM0NjYyMG4wYUhVUXcK
```

Eso por una parte, por otra, en el manifiesto de Image-Updater de argo cd añadimos el value-custom.yaml:

* image-updater.yaml

``` 
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: image-updater
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    name: image-updater
spec:
  project: argocd-demo

  source:
    repoURL: https://github.com/TheAutomationRules/argocd.git
    targetRevision: main
    path: official/argocd-image-updater
    helm:
      valueFiles:
        - values-custom.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

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

Y dentro del values-custom.yaml, que va dentro del directorio argocd-image-updater, añadimos las siguientes líneas. Primero vemos el archivo en su extensión:

``` 
# -- Replica count for the deployment. It is not advised to run more than one replica.
replicaCount: 1
image:
  # -- Default image repository
  repository: quay.io/argoprojlabs/argocd-image-updater
  # -- Default image pull policy
  pullPolicy: Always
  # -- Overrides the image tag whose default is the chart appVersion
  tag: ""

# -- The deployment strategy to use to replace existing pods with new ones
updateStrategy:
  type: Recreate
# -- ImagePullSecrets for the image updater deployment
imagePullSecrets: []
# -- Global name (argocd-image-updater.name in _helpers.tpl) override
nameOverride: ""
# -- Global fullname (argocd-image-updater.fullname in _helpers.tpl) override
fullnameOverride: ""

# -- Extra arguments for argocd-image-updater not defined in `config.argocd`.
# If a flag contains both key and value, they need to be split to a new entry
extraArgs: []
  # - --disable-kubernetes
  # - --dry-run
  # - --health-port
  # - 8080
  # - --interval
  # - 2m
  # - --kubeconfig
  # - ~/.kube/config
  # - --match-application-name
  # - staging-*
  # - --max-concurrency
  # - 5
  # - --once
  # - --registries-conf-path
# - /app/config/registries.conf

# -- Extra environment variables for argocd-image-updater
extraEnv: []
  # - name: AWS_REGION
#   value: "us-west-1"

# -- Init containers to add to the image updater pod
initContainers: []
  #  - name: download-tools
  #    image: alpine:3.8
  #    command: [sh, -c]
  #    args:
  #      - wget -qO- https://get.helm.sh/helm-v2.16.1-linux-amd64.tar.gz | tar -xvzf - &&
  #        mv linux-amd64/helm /custom-tools/
  #    volumeMounts:
  #      - mountPath: /custom-tools
#        name: custom-tools

# -- Additional volumeMounts to the image updater main container
volumeMounts: []

# -- Additional volumes to the image updater pod
volumes: []
  ## Use init containers to configure custom tooling
  ## https://argo-cd.readthedocs.io/en/stable/operator-manual/custom_tools/
  ## When using the volumes & volumeMounts section bellow, please comment out those above.
  #  - name: custom-tools
#    emptyDir: {}

config:
  # -- API kind that is used to manage Argo CD applications (`kubernetes` or `argocd`)
  applicationsAPIKind: ""

  # Described in detail here https://argocd-image-updater.readthedocs.io/en/stable/install/running/#flags
  argocd:
    # -- Use the gRPC-web protocol to connect to the Argo CD API
    grpcWeb: true
    # -- Connect to the Argo CD API server at server address
    serverAddress: ""
    # -- If specified, the certificate of the Argo CD API server is not verified.
    insecure: true
    # -- If specified, use an unencrypted HTTP connection to the ArgoCD API instead of TLS.
    plaintext: false
    # -- If specified, the secret with ArgoCD API key will be created.
    token: ""

  # -- Disable kubernetes events
  disableKubeEvents: false

  # -- Username to use for Git commits
  gitCommitUser: "rockemsockem"

  # -- E-Mail address to use for Git commits
  gitCommitMail: "theautomationrules@gmail.com"

  # -- Changing the Git commit message
  gitCommitTemplate: "Commit from ArgoCD Imagen Updater"

  # -- ArgoCD Image Update log level
  logLevel: "info"

  # -- ArgoCD Image Updater registries list configuration. More information [here](https://argocd-image-updater.readthedocs.io/en/stable/configuration/registries/)
  registries: #[]
    - name: GitHub Container Registry
      prefix: ghcr.io
      api_url: https://ghcr.io
      ping: no
      insecure: true
      credentials: secret:argocd/git-credentials#creds
    # - name: Docker Hub
    #   api_url: https://registry-1.docker.io
    #   ping: yes
    #   credentials: secret:foo/bar#creds
    #   defaultns: library
    # - name: Google Container Registry
    #   api_url: https://gcr.io
    #   prefix: gcr.io
    #   ping: no
    #   credentials: pullsecret:foo/bar
    # - name: RedHat Quay
    #   api_url: https://quay.io
    #   ping: no
    #   prefix: quay.io
    #   credentials: env:REGISTRY_SECRET
    # - name: ECR
    #   api_url: https://123456789.dkr.ecr.eu-west-1.amazonaws.com
    #   prefix: 123456789.dkr.ecr.eu-west-1.amazonaws.com
    #   ping: yes
    #   insecure: no
    #   credentials: ext:/scripts/auth1.sh
    #   credsexpire: 10h

  # -- ArgoCD Image Updater ssh client parameter configuration.
  sshConfig:
    {}
    #  config: |
    #    Host *
    #          PubkeyAcceptedAlgorithms +ssh-rsa
  #          HostkeyAlgorithms +ssh-rsa

# whether to mount authentication scripts, if enabled, the authentication scripts will be mounted on /scripts that can be used to authenticate with registries (ECR)
# refer to https://argocd-image-updater.readthedocs.io/en/stable/configuration/registries/#specifying-credentials-for-accessing-container-registries for more info
authScripts:
  # -- Whether to mount the defined scripts that can be used to authenticate with a registry, the scripts will be mounted at `/scripts`
  enabled: false
  # -- Map of key-value pairs where the key consists of the name of the script and the value the contents
  scripts: {}
    # auth1.sh: |
    #   #!/bin/sh
    #   echo "auth script 1 here"
    # auth2.sh: |
    #   #!/bin/sh
  #   echo "auth script 2 here"

serviceAccount:
  # -- Specifies whether a service account should be created
  create: true
  # -- Annotations to add to the service account
  annotations: {}
  # -- The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

# -- Pod Annotations for the deployment
podAnnotations: {}

# -- Pod security context settings for the deployment
podSecurityContext: {}
# fsGroup: 2000

# -- Security context settings for the deployment
securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
# runAsUser: 1000

rbac:
  # -- Enable RBAC creation
  enabled: true

# -- Pod memory and cpu resource settings for the deployment
resources: {}

# -- Kubernetes nodeSelector settings for the deployment
nodeSelector: {}

# -- Kubernetes toleration settings for the deployment
tolerations: []

# -- Kubernetes affinity settings for the deployment
affinity: {}

# Metrics configuration
metrics:
  # -- Deploy metrics service
  enabled: false
  service:
    # -- Metrics service annotations
    annotations: {}
    # -- Metrics service labels
    labels: {}
    # -- Metrics service port
    servicePort: 8081
  serviceMonitor:
    # -- Enable a prometheus ServiceMonitor
    enabled: false
    # -- Prometheus ServiceMonitor interval
    interval: 30s
    # -- Prometheus [RelabelConfigs] to apply to samples before scraping
    relabelings: []
    # -- Prometheus [MetricRelabelConfigs] to apply to samples before ingestion
    metricRelabelings: []
    # -- Prometheus ServiceMonitor selector
    selector: {}
    # promtheus: kube-prometheus

    # -- Prometheus ServiceMonitor namespace
    namespace: ""
    # -- Prometheus ServiceMonitor labels
    additionalLabels: {}
```

Dentro de este archivo yaml, añadimos los registries requeridos, donde añadimos dentro de credentials el secret encontrado en el namespace argocd llamado git-credentials. Con el númeral (#), pasamos el campo de tipo secreto del namespace argocd llamado creds. Es justo lo que hemos definido previamente en git-secret.yaml.

``` 
  # -- ArgoCD Image Update log level
  logLevel: "info"

  # -- ArgoCD Image Updater registries list configuration. More information [here](https://argocd-image-updater.readthedocs.io/en/stable/configuration/registries/)
  registries: #[]
    - name: GitHub Container Registry
      prefix: ghcr.io
      api_url: https://ghcr.io
      ping: no
      insecure: true
      credentials: secret:argocd/git-credentials#creds
    # - name: Docker Hub
    #   api_url: https://registry-1.docker.io
    #   ping: yes
    #   credentials: secret:foo/bar#creds
    #   defaultns: library
    # - name: Google Container Registry
    #   api_url: https://gcr.io
    #   prefix: gcr.io
    #   ping: no
    #   credentials: pullsecret:foo/bar
    # - name: RedHat Quay
    #   api_url: https://quay.io
    #   ping: no
    #   prefix: quay.io
    #   credentials: env:REGISTRY_SECRET
    # - name: ECR
    #   api_url: https://123456789.dkr.ecr.eu-west-1.amazonaws.com
    #   prefix: 123456789.dkr.ecr.eu-west-1.amazonaws.com
    #   ping: yes
    #   insecure: no
    #   credentials: ext:/scripts/auth1.sh
    #   credsexpire: 10h
```

Todo esto es lo único que tiene configurado ArgoCD con Image Updater. 

Dentro del directorio argocd-manifests, encontramos el archivo nginx-demo.arm.yaml. Dentro de este archivo contamos con dos líneas, dos annotations, que van a posibilitar el actualizar directamente la versión de la app de forma automática:

``` 
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo-arm
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  annotations:
    argocd-image-updater.argoproj.io/image-list: ghcr.io/theautomationrules/nginx-demo-arm:~0.1
    argocd-image-updater.argoproj.io/git-branch: main

spec:
  project: argocd-demo
  source:
    repoURL: https://github.com/TheAutomationRules/argocd.git
    targetRevision: HEAD
    path: Video_03/nginx-demo/arm/kustomize/overlays/testing
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd-demo
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
      allowEmpty: true
```

* argocd-image-updater.argoproj.io/image-list: ghcr.io/theautomationrules/nginx-demo-arm:~0.1: Dentro de la ruta especificada, indicamos que el eje sobre el que va a tomar forma la actualización automática va a ser a partir de la versión 0.1. En este momento, yo tengo la 0.1.0, pero en cuanto salga la 0.1.1 nuestra app pasará a dicha versión a través de Image Updater para ArgoCD.
    
* argocd-image-updater.argoproj.io/git-branch: main: Indica la rama de nuestro repo en el cual se está trabajando la app.


Hecho esto, en cuanto haya una nueva versión, la app pasará a ella. Para comprobarlo, volvemos a efectuar un port-forward y vamos a la app en el navegador.


























