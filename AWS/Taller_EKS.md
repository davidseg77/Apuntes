# Taller EKS OW

### EKS

Ofrece un Control plane as a service, garantizando la disponiblidad. Nos da una URL para interactuar con nuestro clúster. Los contenedores se ejecutan bien en AWS Fargate, que es el servicio de AWS encargado de levantar contenedores sin necesidad de VM, o en instancias de EC2.

**Añadir nodos**

Managed node groups. Son grupos de autoescalado de EC2 cuyas instancias se conectan automáticamente con nuestro clúster.

AWs dispone de una plantilla AMI para nodos de EKS que mantiene actualizada y con las mejores configuraciones para AWS.

### Creación de un clúster EKS

Vamos al servicio EKS y lo creamos con el nombre deseado. 

Primero, vamos a IAM y creamos un rol. En IAM seleccionamos EKS y tenemos algunas políticas predefinidas. Vamos a seleccionar el rol EKS, nos quedamos con los permisos que nos da y le damos nombre y creamos rol. 

Segundo, vamos a EKS, create cluster y creamos una VPC con mínimo dos zonas de disponibilidad y cuatro subnets, dos públicas y dos privadas. El security group por defecto de AWS nos vale.

Si quisieramos conectarnos a nuestro clúster desde una VPN, activamos el acceso privado. Podemos también habilitar los logs deseados para que Cloudwatch los monitorice. Y creamos.

Hecho esto, nos vamos a la terminal y comprobamos la versión de AWS

```
aws --version
```

Lo configuramos con nuestras credenciales y la región donde hayamos creado el clúster.
También actualizamos kubectl para que esté en la misma versión que el clúster.

```
aws eks --region x update-kubeconfig --name nombrecluster 
```

Ahora ya podemos utilizar kubectl para conectarnos con nuestro clúster. Para corroborarlo:

```
kubectl version
```

También:

```
kubectl cluster-info
```

Para ver los servicios y asegurarnos de tener el del clúster:

kubectl get services

### Desplegar contenedores en nuestro clúster

Lo haremos en Fargate, pero antes crearemos un perfil en Fargate. Add Fargate Profile dentro de EKS, Cluster. Le damos nombre y necesitamos un rol para ejecutar los pods. Para ello, volvemos a IAM y en EKS, tenemos el rol EKS - Fargate Pod. Lo seleccionamos, damos nombre y asignamos rol. 

En subnets asignamos solo las privadas, pues Fargate solo trabaja con ellas. Después, seleccionamos los selectores. En el primer namespace ponemos default, en el segundo kube-system, esto último para que los dos pods se levanten en AWS Fargate por defecto y el DNS de AWS trabaje con ellos. Y next y creamos nuestro perfil de Fargate.

Vamos a levantar un contenedor con nginx facilmente testeable.

```
kubectl run nginx --image nginx
```

Nos crea el deployment de nginx. Comprobamos:

```
kubectl get pod
```

Vemos los nodos:

```
kubectl get nodes
```

El DNS está corriendo en el namespace kube-system. Inspeccionamos que tenemos en este namespace

```
kubectl -n kube-system get pod
```

Nos mostrará los pods asociados. Si no están asociados con Fargate, hacemos lo siguiente:

```
kubectl -n kube-system edit deploy coredns
```

Nos llevará a un archivo. En él, bajamos hasta anotations y modificamos ec2 por fargate. 

Guardamos y volvemos a listar los pods para que los muestre de forma actualizada.

```
kubectl -n kube-system get pod -w
```

Verificamos que el cambio se ha producido

```
kubectl get nodes
```

### Node Groups (Grupos de nodos gestionados)

Vamos a crear nuestro grupo de nodos gestionados. Dentro de EKS, del cluster, add node groups. De ahí vamos a IAM, y seleccionamos EC2. Hay tres políticas básicas que tienen que teer estos nodos: AmazonEKSWorkerNodePolicy, AmazonEKS_CNI_Policy y AmazonEC2ContainerRegistryReadOnly. Escogidas las tres, damos nombre al rol y lo creamos.

Volvemos al node group y añadimos el rol recien creado. Dejamos solo las subnets públicas y deshabilitamos el acceso ssh. 

Seleccionamos el tipo de instancia que deseamos y el número de nodos. Creamos.

Podemos comprobar que todo funciona en la terminal con kubectl get nodes y, posteriormente, yendo a las instancias de EC2 y revisando las ips dadas que coincidan con las vistas con el comando.

Si vamos a Autoscaling group, veremos como automáticamente se ha creado un grupo de autoescalado para Kubernetes con las dos subnets indicadas, incluso modificar para añadir algún nodo más.

Vamos a crear un nuevo namespace, diferente al kube-system que se encarga de gestionar los pods para Fargate. 

```
kubectl create namespace demo
```

Y desplegamos la imagen nginx en base a este namespace creado.

```
kubectl -n demo run nginx-demo --image nginx
```

Verificamos:

```
kubectl -n demo get pod
```

A continuación, vamos a ver como exponer a Internet nuestro nginx de una forma sencilla.

Primero, eliminamos el deployment recien creado

```
kubectl delete deployment nginx-demo -n demo
```

Y volvemos a ejecutar el run asignando los puertos

```
kubectl -n demo run nginx-demo --image nginx --port 80
```

Exponemos el servicio

```
kubectl -n demo expose deployment nginx-demo --type=LoadBalancer --name=nginx-demo
```

Se nos ha creado un servicio de nginx que sale a Internet con balanceo de carga.

Verificamos:

```
kubectl get services -n demo
```

Nos mostrará la URL pública del balanceador. De hecho, en EC2, en LoadBalancers ya vemos como está activo nuestro balanceador de carga. En este apartado, en Port Configuration vemos como se redirecciona del 80 al 32206. Esto sucede porque Kubernetes asigna los puertos de forma aleatoria en este caso.

También ha creado un nuevo Security Group. 

### Consejos

Es importante desactivar el acceso público cuando estamos en producción. Solo el privado.













