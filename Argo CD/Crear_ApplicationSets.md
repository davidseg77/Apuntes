# Como crear ApplicationSets 📦en ArgoCD 🐙

Aqui se muestra como crear un set de aplicaciones o ApplicationSet en ArgoCD consiguiendo así estandarizar el despliegue de varias aplicaciones o servicios.


## 1. Instalación basado en el video 02 instalando directamente desde el Makefile

Instalamos ArgoCD, y creamos el siguiente archivo Makefile:

``` 
start: minikubestart installargo project createns gitsecret forward
installiu: installimageupdater installapp sleep forward-app
applicationset1: namespaces1 projectappset1 appset1
applicationset2: namespaces2 projectappset2 repohelm appset2

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
	kubectl port-forward service/nginx-demo-arm -n argocd-demo 8082:80

namespaces1:
	@echo "| > CREACION DE NAMESPACES 1"
	kubectl apply -f k8s/namespaces1.yaml

namespaces2:
	@echo "| > CREACION DE NAMESPACES 2"
	kubectl apply -f k8s/namespaces2.yaml

projectappset1:
	@echo "| > CREACION DE PROJECT APPLICATION SET 1s"
	kubectl apply -f argocd-manifests/project-appset1.yaml

projectappset2:
	@echo "| > CREACION DE PROJECT APPLICATION SET 2"
	kubectl apply -f argocd-manifests/project-appset2.yaml

appset1:
	@echo "| > CREACION DE APPLICATION SET 1"
	kubectl apply -f argocd-manifests/applicationset1.yaml

repohelm:
	@echo "| > CREACION DE REPOSITORY DE HELM"
	kubectl apply -f argocd-manifests/repository.yaml

appset2:
	@echo "| > CREACION DE APPLICATION SET 2"
	kubectl apply -f argocd-manifests/applicationset2.yaml

forward-guestbook:
	@echo "| > FORWARD DE PUERTO DE ACCESO A GUESTBOOK DEV"
	kubectl port-forward service/guestbook-test1-dev-helm-guestbook -n guestbook-dev 8081:80

forward-consul:
	@echo "| > FORWARD DE PUERTO DE ACCESO A CONSUL"
	kubectl port-forward service/stack-consul-dev-consul-server -n stack-dev 8500:8500

forward-prometheus:
	@echo "| > FORWARD DE PUERTO DE ACCESO A VAULT"
	kubectl port-forward service/stack-prometheus-test-server -n stack-test 8082:80

deleteall:
	@echo "| > DELETE ALL APPLICATION SETS"
	kubectl delete applicationset guestbook -n argocd && k
```

Y lo iniciamos:

``` 
make start
``` 

## 2. Application Set 1 (Una aplicación dentro de un cluster con tres entornos diferentes)

Aquí hacemos un inciso y mostramos los archivos yaml de cada application set:

* applicationset1.yaml

``` 
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - env: dev
            test: test1
          - env: test
            test: test1
          - env: pro
            test: test1
  template:
    metadata:
      name: 'guestbook-{{test}}-{{env}}'
      namespace: argocd
      finalizers:
        - resources-finalizer.argocd.argoproj.io
    spec:
      project: appset1
      source:
        repoURL: https://github.com/TheAutomationRules/argocd.git
        targetRevision: HEAD
        path: official/examples/helm-guestbook
        helm:
          valueFiles:
            - values-{{env}}.yaml
      destination:
        server: https://kubernetes.default.svc
        namespace: guestbook-{{env}}
```

Como vemos, se crea dentro de un cluster una aplicación con tres entornos diferentes. En valueFiles se toma como referencia un yaml de helm relacionado con cada entorno. Serían estos:

* values-dev.yaml

``` 
apiVersion: v1
kind: Service
metadata:
  name: guestbook-test1-dev-helm-guestbook
spec:
  selector:
    app: helm-guestbook
    release: guestbook-test1-dev
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30001
      protocol: TCP
```

* values-pro.yaml

``` 
apiVersion: v1
kind: Service
metadata:
  name: guestbook-test1-pro-helm-guestbook
spec:
  selector:
    app: helm-guestbook
    release: guestbook-test1-pro
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30003
      protocol: TCP
```

* values-test.yaml

``` 
apiVersion: v1
kind: Service
metadata:
  name: guestbook-test1-test-helm-guestbook
spec:
  selector:
    app: helm-guestbook
    release: guestbook-test1-test
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30002
      protocol: TCP
```

Desplegamos los namespaces, project y application set del primer ejemplo

``` 
make applicationset1
``` 

Con esto comprobamos en ArgoCD como se despliegan las aplicaciones definidas en el ApplicationSet 1.

Ahora podemos por ver el Guest Book en funcionamiento, levantamos un forward al de Dev

```
make forward-guestbook
```


## 3. Application Set 2 (Dos aplicaciones distintas en un mismo cluster con dos entornos diferentes)


* applicationset2.yaml

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: stack
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - app: consul
            version: 1.1.1
            env: dev
          - app: prometheus
            version: 22.6.1
            env: test
  template:
    metadata:
      name: 'stack-{{app}}-{{env}}'
      namespace: argocd
      finalizers:
        - resources-finalizer.argocd.argoproj.io
    spec:
      project: appset2
      source:
        repoURL: https://github.com/TheAutomationRules/helm.git
        targetRevision: HEAD
        path: charts/{{app}}/{{version}}
        helm:
          valueFiles:
            - values-{{env}}.yaml
      destination:
        server: https://kubernetes.default.svc
        namespace: stack-{{env}}
``` 

Aquí podemos ver la aplicacion funcionando y podemos hacer uso de ella.

``` 
make applicationset2
``` 

Con esto comprobamos en ArgoCD como se despliegan las aplicaciones definidas en el ApplicationSet 2.

Ahora podemos ver las aplicaciones funcionando, levantamos un forward para ver Consul funcionando.

``` 
make forward-consul
``` 

Levantamos un forward para ver Prometheus funcionando.

``` 
make forward-prometheus
``` 

Si queremos eliminar todos los application sets

```
make deleteall
``` 

