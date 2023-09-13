# Despliegue de EKS usando CrossPlane + ArgoCD


## 1. Escenario que se ha realizado

El escenario consiste en lo siguiente:

● Clúster local de kubernetes, en el que se han instalado las siguientes
aplicaciones:

    ○ ArgoCD
    ○ Crossplane, con los proveedores de AWS y Kubernetes.

● Clúster de EKS creado y gestionado por Crossplane, en el que hay 3 nodos.

### 1.1 Instalaciones

Para la realización del proyecto, se necesita instalar los siguientes recursos:

**Clúster local de k8s**

Para desplegar un clúster local de k8s, se usará kind, ya que el provider de aws de crossplane consume muchos recursos, y kind es más liviano.. Para su instalación se seguirá la documentación oficial: Instalación kind. Para gestionar el clúster también es necesario kubectl, y la documentación oficial de instalación es la siguiente: instalación kubectl. Es muy importante partir de un clúster nuevo y sin configuraciones previas.

**ArgoCD**

Para instalar ArgoCD se ejecutan los siguientes comandos:

``` 
kubectl create namespace argocd
kubectl apply -n argocd -f
https://raw.githubusercontent.com/argoproj/argo-cd/stable/mani
fests/install.yaml
``` 

Para acceder a la página de administración, necesito la contraseña inicial. Se obtiene con el siguiente comando; el usuario es admin:

``` 
kubectl -n argocd get secret argocd-initial-admin-secret -o
jsonpath="{.data.password}" | base64 -d;echo
``` 

Y para acceder a la página, se realiza el siguiente port-forward:

``` 
kubectl port-forward svc/argocd-server -n argocd 8080:80
--address=0.0.0.0
``` 

Tras eso ya está configurado ArgoCD.

**Crossplane**

Para instalar crossplane utilizamos helm

``` 
helm repo add \
crossplane-stable https://charts.crossplane.io/stable
helm repo update
``` 

```
helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace
``` 

### 1.2 Configuraciones

**AWS**

Es necesaria una cuenta con los permisos de AWS para crear y gestionar los siguientes recursos:

● instancias EC2
● clúster de EKS
● redes VPC
● Tokens de acceso a la propia cuenta

Una vez se disponga de dicha cuenta, creamos el token en el siguiente apartado: credenciales de seguridad > crear clave de acceso > seleccionamos el cuadro y crear clave de acceso > descargar archivo.csv

**Proveedores de Crossplane**

- AWS

Ahora instalamos la última versión del provider de AWS de crossplane.

``` 
cat <<EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
name: provider-aws
spec:
package:
xpkg.upbound.io/crossplane-contrib/provider-aws:v0.39.0
EOF
``` 

Tras eso, creamos el fichero aws-credentials.txt con las claves generadas en AWS con el siguiente formato:

``` 
[default]
aws_access_key_id = <aws_access_key>
aws_secret_access_key = <aws_secret_key>
``` 

Creamos un secret de kubernetes con las credenciales

``` 
kubectl create secret \
generic aws-creds \
-n crossplane-system \
--from-file=creds=./aws-credentials.txt
```

Finalmente, se crea un providerconfig con el secret que hemos creado

``` 
cat <<EOF | kubectl apply -f -
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
name: default
spec:
credentials:
source: Secret
secretRef:
namespace: crossplane-system
name: aws-creds
key: creds
EOF
``` 

- Kubernetes
  
Instalamos el proveedor con el siguiente comando:

``` 
cat <<EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
name: crossplane-provider-kubernetes
spec:
package: crossplane/provider-kubernetes:main
EOF
``` 

**Recursos gestionados**

Para desplegar un clúster de aws de forma sencilla, se van a utilizar dos recursos gestionados:

● eks.yaml, de tipo composition que contiene parámetros para la creación del clúster, como lo son las redes que se crearán, en qué servidores, qué tipo de máquinas pueden crearse, etc..
● definition.yaml, de tipo compositeResourceDefinition, contiene los parámetros que se le van a pasar con el manifiesto final, así como los recursos a los que corresponden en composition. También contiene valores
por defecto.

Ambos ficheros se encuentran en el repositorio del proyecto en la carpeta crossplane:
https://github.com/robertorodriguez98/proyecto-integrado/tree/main/crossplane
Para añadirlos, se utiliza kubectl:

``` 
kubectl apply -f eks.yaml && kubectl apply -f definition.yaml
``` 

**Ngrok**

Para que ArgoCD esté atento a los commits de github y siga así la metodología GitOps, al tenerlo instalado en un clúster local, es necesario instalar ngrok, exponiendo el puerto 8080:

``` 
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo
tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && echo "deb
https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee
/etc/apt/sources.list.d/ngrok.list && sudo apt update && sudo
apt install ngrok
``` 

Añadimos el token:

```
ngrok config add-authtoken [TOKEN]
``` 

y exponemos el puerto:

``` 
ngrok http https://localhost:8080
``` 

**Webhook**

Para que GitHub notifique a ArgoCD en el evento de un push al repositorio, tenemos que añadirlo en la configuración del mismo. Para hacerlo, en el repositorio accedemos al apartado Settings; en la barra lateral seleccionamos Webhooks y Add webhook.

Dentro de la página, tenemos que rellenar los siguientes campos:

● Payload URL: la url generada por Ngrok, con la cadena /api/webhook
añadida al final.
● Content type: application/json.
● Event: Cuando se produce un push.

**Preparación previa a la demo**

Para preparar la demo, iniciaremos la aplicación de argocd app.yaml que se encuentra en el repositorio, ésta despliega el siguiente manifiesto:

``` 
apiVersion: prodready.cluster/v1alpha1
kind: ClusterClaim
metadata:
name: proyecto-eks
labels:
cluster-owner: robertorm
spec:
id: proyecto-eks
compositionSelector:
matchLabels:
provider: aws
cluster: eks
parameters:
nodeSize: medium
minNodeCount: 3
``` 

Donde podemos modificar los siguientes parámetros:

● name: Nombre del clúster.
● nodesize: tipo de máquina que se van a utilizar en los nodos. En el fichero eks.yaml se establecen los siguientes tamaños:

    ○ small: t2:micro
    ○ medium: t3.micro
    ○ large: t3.medium

Los dos primeros tamaños, entran dentro de la capa gratuita de AWS.

● minNodeCount: mínimo de nodos que tienen que crearse en el clúster. Tras aproximadamente 15 minutos, el clúster estará creado junto a sus nodos. Si observamos el panel de ArgoCD de la aplicación, podemos ver todos los recursos de AWS que se han desplegado:


## 2. Demostraciones


### 2.1 Modificación del número de nodos

El escenario final de la demo es el siguiente:

* Clúster local de k8s (Con Argo CD y Crossplane)
* Proveedor AWS 
* Infraestructura del clúster EKS (Master, Nodo1 y Nodo2 EC2)

Y el flujo de trabajo en el que consiste la primera demo, con el clúster ya desplegado es el siguiente:

1. Se realiza un commit a github con un cambio en el manifiesto aws-eks.yaml, concretamente se cambia el número de nodos que va a tener el clúster de EKS.

2. ArgoCD se da cuenta de que se han realizado cambios en el repositorio, y, aplicando la metodología gitops, hace que en los recursos que gestiona se vean reflejados dichos cambios. Compruebo que en el panel de la aplicación se ve reflejado el cambio en el recurso nodegroup.
   
3. Crossplane, haciendo uso del provider de AWS, se comunica con la API referente a EKS, e indica que el número de nodos ha cambiado.
   
4. Finalmente, en AWS se hacen efectivos los cambios, por lo que podemos comprobarlo accediendo a la consola de EKS y viendo el nuevo nodo.

Tras ello, para demostrar el funcionamiento de Crossplane, y como este asegura que se siga el marco de trabajo GitOps, se añade un nodo desde la consola de AWS, mostrando como Crossplane lo detecta y vuelve a dejarlo como está definido en los recursos.

### 2.2 Despliegue en el clúster

Una vez con el clúster, se va a desplegar una aplicación sobre él, utilizando también crossplane. El escenario es el siguiente:

* Clúster local de k8s (Con Argo CD y Crossplane)
* Proveedor AWS 
* Infraestructura del clúster EKS (Master, Nodo1 y Nodo2 EC2)
* Proveedor Kubernetes
* Clúster de EKS (Pod1, Pod2 y Pod3)

#### 2.2.1 Configuración del proveedor de kubernetes

Para configurar el proveedor de kubernetes para que tenga acceso al clúster que acabamos de crear, se ejecutan los siguientes comandos; Primero obtenemos el kubeconfig y lo guardamos en un fichero:

``` 
kubectl --namespace crossplane-system \
get secret proyecto-eks-cluster \
--output jsonpath="{.data.kubeconfig}" \
| base64 -d >kubeconfig.yaml
```

Enviamos el contenido del fichero a la variable KUBECONFIG:

``` 
KUBECONFIG=$(<kubeconfig.yaml)
``` 

Usando la variable, creamos un secret que pueda usar el proveedor:

``` 
kubectl -n crossplane-system create secret generic
cluster-config --from-literal=kubeconfig="${KUBECONFIG}"
``` 

Ahora, añadimos la configuración del proveedor usando el secret (el fichero está en la raiz del repositorio). El archivo es: config-kubernetes.yaml:

``` 
#apiVersion: v1
#kind: Secret
#metadata:
#  namespace: crossplane-system
#  name: example-provider-secret
#type: Opaque
#data:
  # credentials: BASE64ENCODED_PROVIDER_CREDS
---
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: kubernetes-provider
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: cluster-config
      key: kubeconfig
# identity:
#   type: GoogleApplicationCredentials
#   source: Secret
#   secretRef:
#     name: gcp-credentials
#     namespace: crossplane-system
#     key: credentials.json
```

``` 
kubectl apply -f config-kubernetes.yaml
``` 

#### 2.2.2 Script

Para facilitar la ejecución de la demo, se ha creado el script configurar-k8s.sh que
aúna los pasos anteriores.

``` 
#!/bin/bash

KUBECONFIG=$(kubectl --namespace crossplane-system get secret proyecto-eks-cluster --output jsonpath="{.data.kubeconfig}" | base64 -d)
kubectl -n crossplane-system create secret generic cluster-config --from-literal=kubeconfig="${KUBECONFIG}"
#echo "$KUBECONFIG"
kubectl apply -f ../config-kubernetes.yaml
```

#### 2.2.3 Aplicación de ArgoCD

Una vez realizados los pasos anteriores, solo queda desplegar la aplicación. Para ello se va a volver a utilizar ArgoCD; en este caso se va a utilizar la aplicación despliegue-remoto.yaml. 

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: despliegue-remoto
  namespace: argocd
spec:
  destination:
    namespace: proyecto
    server: https://kubernetes.default.svc
  project: default
  source:
    path: kubernetes
    repoURL: https://github.com/robertorodriguez98/proyecto-integrado.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: false
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

Para obtener la dirección en la que está la aplicación desplegada (gracias al loadbalancer) se utiliza el siguiente comando:

``` 
k describe object loadbalancer-aplicacion-remoto
``` 

O si queremos la dirección directamente:

```
k describe object loadbalancer-aplicacion-remoto | egrep
"Hostname"
``` 

Finalmente, si accedemos a la dirección podemos visualizar la aplicación desplegada.






