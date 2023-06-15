# Curso de Kubernetes para desarrolladores (OW)

### Minikube

Necesitaremos de esta herramienta para trabajar con Kubernetes. Cuando se instale, lo iniciamos con minikube start.

Para ver la ip de minikube:

```
minikube ip
```

### Kubectl 

Como en cualquier comando, podemos hacer:

```
kubectl --help
```

para tener una vista general de las cosas que hacer con kubectl.

Para ver lo que hay corriendo en nuestro clúster, podemos ejecutar:

```
kubectl get all
```

donde veremos los servicios, deployments y pods que estamos ejecutando.

Para cualquiera de los objetos creados podemos recibir más información con el comando:

```
kubecetl describe deployment.apps/vote
```

kubectl describe nos muestra los eventos asociados al objeto, que es muy útil para buscar errores en el despliegue. Sin embargo, si quiero ver los logs de uun contenedor, tengo que ejecutar el comando:

```
kubectl logs -f pod-xxxxx-yyy
```

Si queremos acceder a los puertos de un pod podemos usar el comando kubectl port-forward y exponerlos en la interfaz de localhost, y si queremos entrar dentro de un contenedor tenemos el comando kubectl exec.

Finalmente, para eliminar un objeto, ejecutamos el comando kubectl delete. Por ejemplo, para eliminar la aplicación entera ejecutamos:

```
kubectl delete -f .
```

### Pods

Vamos a hacer algunos ejemplos con Pods, para lo que nos vamos al directorio pods del repo.

En el fichero pod.yaml tenemos un ejemplo muy sencillo de un Pod. En él definimos el tipo del objeto, el nombre del Pod, y los contenedores que contiene. En este caso es solo un contenedor nginx que expone el puerto 80.

Para lanzar el pod ejecutamos:

```
kubectl apply -f pod.yaml
```

Y si hacemos:

```
kubectl get all
```

veremos el Pod creándose. Con kubectl describe pod/pod vemos los eventos asociados a este pod.

Ahora, podemos acceder a este Pod con el comando:

```
kubectl port-forward 8081:80
```

y lo tendríamos accesible en localhost:8081.

Para ver los logs del Pod ejecutamos:

```
kubectl logs pod/pod
```

Ahora vamos a eliminar este Pod con el comando:

```
kubectl delete -f pod.yaml
```

Pasemos aver ahora en el fichero pod-volumes.yaml. Este ejemplo muestra cómo compartir volúmenes entre contenedores dentro del mismo pod. Para ello definimos una sección de volumes en la Spec del pod, y lo montamos con volumeMounts en cada uno de los contenedores.

