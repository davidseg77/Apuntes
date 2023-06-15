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

### Replica Set

Un Replica Set es un mecanismo de abstracción sobre un conjunto de Pods que nos garantiza que un número concreto de instancias de un Pod esté siempre operativo. Debe aplicar sobre Pods stateless, es decir, que no tengan estado, no tengan sesiones, ni hagan escrituras a disco.

Así, por ejemplo, el Replica Set es el encargado de escalar arriba o abajo el número de instancias de un Pod según haya definido el usuario, o de detectar si un Pod está unhealty para reemplazarlo por uno nuevo.

Vamos a ver ahora un ejemplo con Replica Sets, para lo que nos vamos al directorio replica-sets desde el raíz del repo.

En el fichero replica-set.yaml tenemos un ejemplo muy sencillo de un Replica Set. Estos ficheros YAML son un poco complejos de crear desde cero, lo más sencillo es hacer un copy/paste de uno ya creado y editarlo con los valores que deseemos.

En replica-set.yaml definimos el tipo del objeto, el nombre del Replica Set, el número de réplicas en el campo replicas y un campo selector, que es un conjunto de matchLabels y matchExpressions de los pods que queremos controlar desde el Replica Set.

Para lanzar el Replica Set ejecutamos:

```
kubectl apply -f replica-set.yaml
```

Y si hacemos:

```
kubectl get all
```

veremos el Replica Set creado y 3 pods creándose.

Si eliminamos uno de los pods y volvemos a hacer kubectl get all veremos que el Replica Set ha creado un nuevo pod para suplantar el pod que acabamos de eliminar.

Por último, para eliminar el Replica Set y sus pods asociados ejecutamos:

```
kubectl delete -f replica-set.yaml
```

### Deployment

Los Deployments son una abstracción muy útil sobre el concepto de Replica Controllers, y es el objeto que se utiliza como norma general para desplegar nuestras aplicaciones.

Sobre la abstracción del Replica Controller ofrece rolling updates. De esta manera, si tenemos 4 instancias de un Pod y queremos desplegar una nueva versión de nuestra aplicación, el Deployment se encarga de ir sustituyendo uno a uno los Pods antiguos por los nuevos, de tal manera que no nos veamos afectados por un downtime. Además, el Deployment mantiene un histórico de versiones de los Pods que ha ejecutado, permitiendo hacer rollbacks a versiones pasadas si detectamos un problema en producción.

Vamos a ver ahora un ejemplo con Deployments, para lo que nos vamos al directorio deployments desde el raíz del repo.

En el fichero deployment.yaml tenemos un ejemplo muy sencillo de un Deployment. En deployment.yaml definimos el tipo del objeto, el nombre del Deployment, y la definición en Spec de un Replica Set.

Para lanzar el Deployment ejecutamos:

```
kubectl apply -f deployment.yaml
```

Y si hacemos:

```
kubectl get all
```

veremos el Deployment, el Replica Set creado y 3 pods creándose. El YAML de Deployment nos permite también definir el tipo de estrategia para upgrades en el campo strategy y cuántas versiones del Deployment vamos a almacenar de cara a hacer rollbacks.

Con el comando rollout gestionamos la historia de un deployment. Por ejemplo:

```
kubectl rollout history deployment.apps/deployment
```

Si cambiamos el manifest para cambiar la imagen de nginx y aplicamos los cambios:

```
kubectl apply -f deployment.yaml
```

Y hacemos un kubectl rollout status deployment.apps/deployment nos muestra el estado del despliegue. Si volvemos a hacer:

```
kubectl rollout history deployment.apps/deployment
```

vemos que ahora tenemos dos versiones en el historial. Si ahora necesitamos hacer un rollback a la versión anterior, podemos ejecutar:

```
kubectl rollout undo deployment.apps/deployment
```

que nos devuelve el deployment a la versión inicial.

### Service

Si algo hemos visto hasta este momento es que la vida de un Pod es muy dinámica. Los Pods se crean y destruyen constantemente en un clúster de Kubernetes, por lo que el acceso entre aplicaciones no se puede basar en la IP de los Pods. De alguna manera, es necesario tener un endpoint permanente que de acceso a un conjunto de Pods.

Esta es la función de un Service. Un Service se asocia a un conjunto de Pods. Cada Service tiene un endpoint de acceso fijo que se corresponde con su nombre (aunque también podríamos acceder a él vía un Load Balancer externo o un ingress). Por tanto, cada vez que accedamos por DNS a nombre de un servicio, el servicio redirigirá la petición a uno de los Pods a los que está asociado, ofreciendo un mecanismo eficaz de service discovery y de balanceo de carga.

Vamos a ver ahora un ejemplo con Services, para lo que nos vamos al directorio services desde el raíz del repo.

En el fichero service.yaml tenemos un ejemplo muy sencillo de un Service. En él definimos el tipo del objeto, el nombre del Service, un campo selector para selecionar los pods a los que aplica el servicio, y el mapeo de puertos entre en servicio y los pods. En este caso usamos un servicio NodePort que nos balancea a el puerto 80 de los pods desde el puerto 30080 de la máquina de Minikube.

Para lanzar el Service ejecutamos:

```
kubectl apply -f service.yaml
```

Y si hacemos:

```
kubectl get all
```

veremos el Service ya creado. Desde este momento podemos acceder a los pods usando la ip de Minikube:

```
minikube ip
```

y accediendo desde el navegador:

**192.168.99.100:30080**

Además, si accedemos con kubectl exec a uno de los pods, vemos que podemos acceder a los pods a través de la ip del servicio, y usando directamente service, que es el nombre del servicio, lo que resulta muy útil para service discovery.

Por último, para eliminar el Service ejecutamos:

```
kubectl delete -f service.yaml
```

Y para eliminar el Deployment, su replica set y pods asociados ejecutamos:

```
kubectl delete deployment.apps/deployment
```

### Secrets

Un Secret es la manera que tenemos en Kubernetes de insertar secretos, como pueda ser un password, en el entorno de ejecución de un Pod de la manera más segura posible. La realidad es que a día de hoy, los secretos de Kubernetes no son del todo seguros, por lo que muchas empresas utilizan servicios externos como Vault.

Existen dos maneras de pasar un secreto a un Pod. Una opción es pasarlos como variables de entorno. Otra opción más seguro es pasarlos como un volumen que se monta en un path concreto dentro del contenedor, de tal manera que podemos eliminar el fichero con los secretos una vez que han sido leídos por la aplicación principal.

Vamos a ver una demo del uso de Secrets, para lo que nos vamos al directorio secrets desde el raíz del repo.

En este caso vamos a crear el Secret desde un fichero. Creamos un fichero con el comando:

```
echo XXXXX > password
```

Ahora creamos el Secret desde ese fichero con el comando:

```
kubectl create secret generic secret --from-file=password`
```

que nos crea un Secret con el nombre secret de tipo genérico. Podemos ver que lo ha creado con el comando:

```
kubectl get secrets
```

he incluso podemos ver su valor asociado (en base64) con el comando:

```
kubectl get secret secret -oyaml
```

En el fichero secret.yaml tenemos un ejemplo muy sencillo de un Deployment que consume un Secret. En este fichero hemos defnido un campo env dentro del contenedor nginx. Y en él tenemos una variable de entorno llamada USERNAME que toma el valor de pablo, y una variable de entorno llamada SECRET que toma su valor del secreto que acabamos de crear.

Para lanzar el Deployment ejecutamos:

```
kubectl apply -f secret.yaml
```

Y si hacemos:

```
kubectl get all
```

veremos el Deployment creado y 3 pods creándose. Si hacemos kubectl exec a uno de los pods veremos que las variables de entorno han sido correctamente creadas, pero en YAML del Deployment no tenemos acceso al valor del secreto.

Por último, para eliminar el Deployment y sus pods asociados ejecutamos:

```
kubectl delete -f secret.yaml
```

y el secreto lo eliminamos con:

```
kubectl delete secret/secret
```

### Imágenes privadas

Si estás acostumbrado a usar imágenes privadas con Docker, sabrás que el acceso a imágenes privadas se configura con el comando docker login, y a partir de ahí, los comandos docker que ejecutas utilizarán esas credenciales para acceder al Registro de Docker.

En Kubernetes la gestión de imágenes privadas es distinta. Deberemos añadir un secreto al clúster con nuestras credenciales del Registro de Docker. Y después montar ese secreto en el Pod que queremos que use nuestra imagen privada.

Vamos a ver ahora un ejemplo, para lo que nos vamos al directorio images desde el raíz del repo.

En el fichero images.yaml tenemos un ejemplo de cómo utilizar una imagen privada en un Deployment. En este fichero vemos que hemos añadido un campo imagePullSecrets que referencia a un secreto. Para hacer este ejemplo tendremos que crear una imagen privada con vuestras credenciales. Y el contenido de secreto se crearía con el comando:

```
kubectl create secret docker-registry regpablo --docker-server=https/index.docker.io/v1/ --username=pchico83 --password=XXXXX --email=pchico83@gmail.com
```

### Labels

Las Labels son un mecanismo de consulta y filtrado muy potente que nos ofrece Kubernetes, pero con el que hay que tener especial atención porque es una fuente común de errores en la configuración de mis aplicaciones.

Los Labels son pares clave/valor que podemos asociar a cualquier objeto de Kubernetes. Por ejemplo, podemos añadir la clave environment en todos mis objetos, y darle el valor dev, staging o prod según el entorno en el que estemos ejecutando. Esto nos va a permitir hacer consultas filtrando por el valor de estos labels, y así refinar los resultados que recibo.

Pero muy importante, las Labels sirven para muchas más cosas. Ya hemos visto como usarlas para mapear los Pods a los que afecta un Replica Controller, y también para decidir entre qué Pods hace balanceo de carga un Service. Otro uso común es para seleccionar los nodos en los que un Pod puede ser ejecutado.

### Healthchecks

Los Healthchecks son un mecanismo fundamental para cargas productivas. Es el principal mecanismo por el cual Kubernetes va a saber si nuestros Pods están funcionando correctamente o no.

Hay dos tipos de healchecks. Podemos configurar la ejecución de comando periódico que se ejecuta en contexto de uno de los contenedores de mi pod, o podemos ejecutar llamadas HTTP aun endpoint que nos responda si nuestro servicio está funcionando correctamente.

Por último, Kubernetes distingue entre dos tipos de healthchecks: ReadinessProbe y LivenessProbe. LivenessProbe indica si el contenedor está funcionando incorrectamente y tiene que ser recreado. ReadinessProbe indica si está listo para recibir tráfico. Nótese que no significan lo mismo, un contenedor podría estar pasando el LivenessProbe, pero no pasar el ReadinessProbe porque, por ejemplo, necesita acceder a una base de datos que en este momento no está disponible.

### Namespaces

Los Namespaces son un concepto muy útil para crear particiones lógicas dentro de mi clúster, o lo que también podemos llamar clúster virtuales dentro de mi clúster físico de Kubernetes.

Es importante recordar que para cada tipo de objeto de Kubernetes, por ejemplo un Service o un Deployment, el nombre del objeto debe ser único. Pero dentro de un clúster podemos crear distintos Namespaces, y esta restricción de unicidad aplica solo dentro de cada Namespace. Así, por ejemplo, puedo tener un Namespace staging y otro production, y seré capaz de crear un Deployment web tanto en uno como en otro Namespace.

Otro uso muy útil de los Namespaces es para gestionar equipos, ya que cada desarrollador podría tener acceso a un Namespace distinto, y de esta manera compartir los recursos del clúster sin que el trabajo de un desarrollador afecte al de otro miembro del equipo porque tienen aislamiento a nivel de Namespace. Por defecto, este aislamiento es solo a nivel de los nombres de los objetos que vamos a crear (y por ejemplo se comparte la misma red), pero podemos hacer uso de Pod Policies y de Network Policies para limitar estos accesos.

Por último, podemos limitar los recursos máximos que se pueden consumir dentro de un Namespace. De esta forma podemos asegurarnos de que ningún desarrollador consume más de una cierta CPU o memoria en su Namespace de desarrollo.

### Ingress

Cuando tenemos que exponer nuestros servicios a tráfico externo hemos visto que podemos crear un servicio de tipo Load Balancer, pero esta solución puede resultar demasiado rígida y costosa. La alternativa es utilizar un Ingress.

Un Ingress nos permite definir rustas de entrada a nuestro servicio de una manera programática. Existen numerosos Ingress Controllers, como el de Nginx, HAProxy o Istio, cada uno de ellos permitiendo distintas configuraciones.

Por ejemplo, con el Nginx Ingress Controller, podemos definir a dónde redirigir una petición en base al dominio solicitado, o al path solicitado dentro de esa petición.

### Otros recursos

Hemos visto los conceptos principales que se necesitan para hacer desarrollo de aplicaciones que corren en Kubernetes. Sin embargo, la lista de objetos que gestiona Kubernetes es bastante más amplia.

Vamos a nombrar algunos de estos objetos para que os suenen en el caso de que los necesitéis:

* ConfigMaps: funcionan de una manera muy parecida a los Secrets, pero están pensados para definir ficheros de configuración que no necesitan estar encriptados.

* Volumes: aunque hemos visto un ejemplo de un volumen compartido entre contenedores en un mismo Pod, hay escenarios más avanzados con Volumes que nos permiten tener persistencia de datos entre la ejecución de distintas instancias de un Pod.

* Jobs: son un tipo especial de Pod para tareas que tienen principio y final, como una tarea asíncrona o la ejecución de una tarea de integración continua.

* DaemonSet: son un tipo especial de Pod que se caracteriza por ejecutar una instancia del Pod en cada nodo del clúster. Son interesantes para correr herramientas de monitorización o de control del clúster.

* StatefulSet: están pensados para aplicaciones con estado, y que suelen hacer uso de Volumes. Kubernetes trata de recrear lo mínimo posible Pods pertenecientes a un StatefulSet.

* Pod Presets: es una manera muy útil de hacer modificaciones (por ejemplo añadir Labels) a cada Pod que se cree en el clúster.

* Affinity: nos permite definir reglas del tipo a que dos tipos de Pods deben desplegarse siempre en el mismo nodo, o que siempre tienen que desplegarse en nodos distintos. Por ejemplo, las instancias de un Pod de base de datos deberían de desplegarse en nodos distintos para aumentar su tolerancia a un fallo de infraestructura.

