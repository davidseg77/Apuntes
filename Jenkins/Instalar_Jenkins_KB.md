# Cómo instalar Jenkins en Kubernetes

## 1. Introducción

Las canalizaciones de integración continua/implementación continua (CI/CD) son uno de los componentes centrales del entorno DevOps. Ayudan a optimizar el flujo de trabajo entre varios equipos y aumentar la productividad. Jenkins es un servidor de automatización de código abierto ampliamente utilizado que puede configurar canalizaciones de CI/CD.

En este tutorial, instalará Jenkins en Kubernetes. Luego accederá a la interfaz de usuario de Jenkins y ejecutará una canalización de muestra.


## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un clúster de Kubernetes en funcionamiento y kubectlconfigurado en su estación de trabajo. 


## 3. instalar Jenkins en Kubernetes

Kubernetes tiene una API declarativa y puede transmitir el estado deseado mediante un archivo YAML o JSON. Para este tutorial, utilizará un archivo YAML para implementar Jenkins. Asegúrese de tener el kubectl comando configurado para el clúster.

Primero, utilícelo kubectl para crear el espacio de nombres de Jenkins:

``` 
kubectl create namespace jenkins
``` 

A continuación, cree el archivo YAML que implementará Jenkins.

Cree y abra un nuevo archivo llamado jenkins.yaml usando nanoo su editor preferido:

``` 
nano jenkins.yaml
``` 

Ahora agregue el siguiente código para definir la imagen de Jenkins, su puerto y varias configuraciones más:

``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
          - name: http-port
            containerPort: 8080
          - name: jnlp-port
            containerPort: 50000
        volumeMounts:
          - name: jenkins-vol
            mountPath: /var/jenkins_vol
      volumes:
        - name: jenkins-vol
          emptyDir: {}
``` 

Este archivo YAML crea una implementación utilizando la imagen Jenkins LTS y también abre el puerto 8080 y 50000. Utilice estos puertos para acceder a Jenkins y aceptar conexiones de los trabajadores de Jenkins, respectivamente.

Ahora cree esta implementación en el jenkins espacio de nombres:

``` 
kubectl create -f jenkins.yaml --namespace jenkins
``` 

Déle al clúster unos minutos para extraer la imagen de Jenkins y ejecutar el pod de Jenkins.

Úselo kubectl para verificar el estado del pod:

``` 
kubectl get pods -n jenkins
``` 

Recibirá un resultado como este:

``` 
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-6fb994cfc5-twnvn   1/1     Running   0          95s
``` 

Tenga en cuenta que el nombre del pod será diferente en su entorno.

Una vez que el pod se está ejecutando, debe exponerlo mediante un Servicio. Utilizará el tipo de servicio NodePort para este tutorial. Además, creará un servicio tipo ClusterIP para que los trabajadores se conecten a Jenkins.

Crea y abre un nuevo archivo llamado jenkins-service.yaml:

``` 
nano jenkins-service.yaml
```

Agregue el siguiente código para definir el servicio NodePort:

``` 
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30000
  selector:
    app: jenkins

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins
``` 

En el archivo YAML anterior, usted define su servicio NodePort y luego expone el puerto 8080 del pod de Jenkins al puerto 30000.

Ahora cree el Servicio en el mismo espacio de nombres:

``` 
kubectl create -f jenkins-service.yaml --namespace jenkins
```

Compruebe que el Servicio se esté ejecutando:

```
kubectl get services --namespace jenkins
``` 

Recibirá un resultado como este:

```
Output
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins   NodePort   your_cluster_ip   <none>        8080:30000/TCP   15d
``` 

Con NodePort y Jenkins operativos, está listo para acceder a la interfaz de usuario de Jenkins y comenzar a explorarla.


## 4. acceder a la interfaz de usuario de Jenkins

En este paso, accederá y explorará la interfaz de usuario de Jenkins. Se puede acceder a su servicio NodePort en el puerto 30000 a través de los nodos del clúster. Debe recuperar la IP de un nodo para acceder a la interfaz de usuario de Jenkins.

Úselo kubectl para recuperar las IP de su nodo:

``` 
kubectl get nodes -o wide
``` 

kubectl producirá una salida con sus IP externas:

``` 
Output
NAME        STATUS   ROLES    AGE   VERSION    INTERNAL-IP        EXTERNAL-IP        OS-IMAGE                       KERNEL-VERSION          CONTAINER-RUNTIME
your_node   Ready    <none>   16d   v1.18.8   your_internal_ip   your_external_ip   Debian GNU/Linux 10 (buster)   4.19.0-10-cloud-amd64   docker://18.9.9
your_node   Ready    <none>   16d   v1.18.8   your_internal_ip   your_external_ip   Debian GNU/Linux 10 (buster)   4.19.0-10-cloud-amd64   docker://18.9.9
your_node   Ready    <none>   16d   v1.18.8   your_internal_ip   your_external_ip   Debian GNU/Linux 10
``` 

Copie uno de los your_external_ip valores.

Ahora abra un navegador web y navegue hasta .http://your_external_ip:30000

Aparecerá una página solicitando una contraseña de administrador e instrucciones sobre cómo recuperar esta contraseña de los registros de Jenkins Pod.

Usemos kubectl para extraer la contraseña de esos registros.

Primero, regresa a tu terminal y recupera el nombre de tu Pod:

``` 
kubectl get pods -n jenkins
``` 

Recibirá un resultado como este:

``` 
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-6fb994cfc5-twnvn   1/1     Running   0          9m54s
``` 

A continuación, verifique los registros del Pod para obtener la contraseña de administrador. Reemplace la sección resaltada con el nombre de su pod:

``` 
kubectl logs jenkins-6fb994cfc5-twnvn -n jenkins
``` 

Es posible que tengas que desplazarte hacia arriba o hacia abajo para encontrar la contraseña:

``` 
Running from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.get("JENKINS_HOME")
. . .

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

your_jenkins_password

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
. . .
``` 

Copiar your_jenkins_password. Ahora regrese a su navegador y péguelo en la interfaz de usuario de Jenkins.

Una vez que ingrese la contraseña, Jenkins le pedirá que instale complementos. Como no estás haciendo nada inusual, selecciona Instalar complementos sugeridos.

Después de la instalación, Jenkins cargará una nueva página y le pedirá que cree un usuario administrador. Complete los campos u omita este paso presionando el enlace omitir y continuar como administrador. Esto dejará su nombre de usuario como administrador y su contraseña como your_jenkins_password.

Aparecerá otra pantalla preguntando sobre la configuración de la instancia. Haga clic en el enlace Ahora no y continúe.

Después de esto, Jenkins creará un resumen de sus elecciones e imprimirá ¡Jenkins está listo! Haga clic en comenzar a usar Jenkins y aparecerá la página de inicio de Jenkins.

Ahora que ha instalado y configurado Jenkins en su clúster, demostremos sus capacidades y ejecutemos una canalización de muestra.


## 5. Ejecutar una canalización de muestra

Jenkins se destaca en la creación de canalizaciones y la gestión de flujos de trabajo de CI/CD. En este paso, construiremos una de las canalizaciones de muestra de Jenkins.

Desde la página de inicio de Jenkins, haga clic en el enlace Nuevo elemento en el menú de la izquierda.

Aparecerá una nueva página. Elija Pipeline y presione OK. 

Jenkins lo redirigirá a la configuración de la canalización. Busque la sección Canalización y seleccione Hola mundo en el menú desplegable de prueba de canalización de muestra. Este menú aparece en el lado derecho. Después de seleccionar Hola mundo, haga clic en el botón Guardar.

Jenkins lo redirigirá a la página de inicio del canal. Haga clic en construir ahora en el menú de la izquierda y observe cómo comienza a ejecutarse el proceso. El número 1 significa que esta es la primera compilación. Una vez que se complete la tarea, verá algunas estadísticas sobre la compilación.

También puede consultar la salida de la consola para ver qué sucedió mientras se ejecutaba la canalización. Pase el cursor sobre el n.º 1 y aparecerá un menú desplegable. Elija la salida de la consola para ver los detalles de la compilación.

Su proceso Hello World no es muy sofisticado, pero demuestra qué tan bien Jenkins puede crear y administrar flujos de trabajo de CI/CD.

