# Cómo utilizar minikube para el desarrollo y las pruebas de Kubernetes local

## 1. Introducción

Kubernetes es un sistema de orquestación de contenedores de código abierto para automatizar la implementación, el escalado y la administración de software. Se ha vuelto muy popular a nivel empresarial por facilitar el escalamiento horizontal de los recursos del servidor, y muchos proveedores de nube ofrecen su propia solución Kubernetes administrada.

Dado que una implementación de Kubernetes generalmente depende de varios servidores, puede requerir muchos recursos realizar el desarrollo y las pruebas de una pila de Kubernetes antes de implementarla en producción. Por esta razón, los autores de Kubernetes mantienen un proyecto complementario llamado minikube, que puede funcionar con un marco de contenedor como Docker para simular un clúster de Kubernetes que se ejecuta en una sola máquina.

En este tutorial, lo instalará minikubeen una computadora local o en un servidor remoto. También accederá al panel integrado de Kubernetes para explorar su clúster en un navegador. Una vez que su clúster se esté ejecutando, implementará una aplicación de prueba y explorará cómo acceder a ella a través de minikube. En la sección final de este tutorial, explorará cómo usar Minikube junto con clústeres remotos de Kubernetes usando perfiles de configuración.


## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Familiaridad con los conceptos de Kubernetes. 

* El marco del contenedor Docker instalado en el entorno Windows, Mac o Linux desde el que ejecutará minikube. 

* El administrador de paquetes Homebrew.

* Al menos 2 CPU, 2 GB de memoria y 20 GB de espacio en disco disponibles para el entorno donde está instalando Minikube.


## 3. instalar y ejecutar Minikube

Puede instalar minikubea través del administrador de paquetes Homebrew:

``` 
brew install minikube
Output
…
==> Installing minikube
==> Pouring minikube--1.25.2.x86_64_linux.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /home/sammy/.linuxbrew/etc/bash_completion.d
==> Summary
🍺  /home/sammy/.linuxbrew/Cellar/minikube/1.25.2: 9 files, 70.0MB
…
``` 

Para comenzar a usarlo minikube, puede ejecutarlo con el start comando, que creará automáticamente un clúster de Kubernetes local utilizando varios contenedores Docker y una versión estable reciente de Kubernetes.

``` 
minikube start
``` 

Esto tomará un momento y debería producir un resultado similar al siguiente, teniendo en cuenta que se kubectl ha configurado para usted.

``` 
Output
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.23.1 preload ...
    > preloaded-images-k8s-v16-v1...: 504.42 MiB / 504.42 MiB  100.00% 81.31 Mi
    > gcr.io/k8s-minikube/kicbase: 378.98 MiB / 378.98 MiB  100.00% 31.21 MiB p
🔥  Creating docker container (CPUs=2, Memory=1987MB) ...
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
``` 

La instalación minikubea través de Homebrew también proporciona kubectl la herramienta principal para administrar clústeres de Kubernetes a través de la línea de comandos. Ahora puede ejecutar kubectl get como lo haría con cualquier otro clúster de Kubernetes para enumerar todos los pods que se ejecutan en su clúster:

``` 
kubectl get pods -A
``` 

El -A argumento devolverá pods que se ejecutan en todos los espacios de nombres.

``` 
Output
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-64897985d-ttwl9            1/1     Running   0             46s
kube-system   etcd-minikube                      1/1     Running   0             57s
kube-system   kube-apiserver-minikube            1/1     Running   0             61s
kube-system   kube-controller-manager-minikube   1/1     Running   0             57s
kube-system   kube-proxy-ddtgd                   1/1     Running   0             46s
kube-system   kube-scheduler-minikube            1/1     Running   0             57s
kube-system   storage-provisioner                1/1     Running   1 (14s ago)   54s
``` 

Ahora tiene un clúster de Kubernetes ejecutándose localmente, con el que puede trabajar utilizando herramientas habituales de Kubernetes como kubectl. En los siguientes pasos de este tutorial, aprenderá cómo utilizar algunas de las funciones adicionales proporcionadas por minikube para monitorear y modificar su configuración local de Kubernetes.


## 4. Acceso al panel de Kubernetes

Minikube implementa el panel de Kubernetes de forma inmediata. Puede utilizar el panel de Kubernetes para monitorear el estado de su clúster o para implementar aplicaciones manualmente. Si implementó Minikube localmente, puede acceder al panel ejecutando el minikube dashboard comando:

``` 
minikube dashboard
``` 

Este comando iniciará automáticamente el panel, reenviará un puerto desde el interior de su clúster de Kubernetes para que pueda acceder a él directamente y abrirá un navegador web que apunte a ese puerto local.

El reenvío de puertos bloqueará el terminal en el que se está ejecutando mientras esté activo, por lo que probablemente querrás ejecutar esto en una nueva ventana de terminal mientras continúas trabajando. Puede presionar Ctrl+C para salir elegantemente de un proceso de bloqueo como este cuando desee dejar de reenviar el puerto.

Si está ejecutando minikube en un servidor remoto donde no puede acceder fácilmente a un navegador web, puede ejecutar minikube dashboard con la --url opción adjunta. Esta opción iniciará el proceso de reenvío de puertos y proporcionará una URL que puede usar para acceder al panel, en lugar de abrir un navegador directamente:

``` 
minikube dashboard --url
Output
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:34197/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
``` 

Tenga en cuenta el número de puerto que devolvió este comando, ya que será diferente en su sistema.

Sin embargo, la configuración de seguridad predeterminada de Kubernetes impedirá que se pueda acceder a esta URL desde una máquina remota. Deberá crear un túnel SSH para acceder a la URL del panel. Para crear un túnel desde su máquina local a su servidor, ejecútelo ssh con la -L bandera. Proporcione el número de puerto que anotó en el resultado del proceso de reenvío junto con la dirección IP de su servidor remoto:

``` 
ssh -L 34197:127.0.0.1:34197 sammy@your_server_ip
``` 

Luego debería poder acceder al panel en un navegador en .http://127.0.0.1:34197/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

Ahora que ha visto más formas de trabajar con minikube un clúster de Kubernetes completo, en el siguiente paso, implementará y accederá a una aplicación de muestra para verificar que su clúster de Minikube esté funcionando como se esperaba.


## 5. Implementación y prueba de una aplicación de muestra

Puede utilizar el kubectl comando para implementar una aplicación de prueba en su clúster Minikube. El siguiente comando recuperará e implementará una aplicación Kubernetes de muestra (en este caso, la de Google) hello-app.

```
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
``` 

Este comando crea una implementación, a la que usted llama web dentro de su clúster, desde una imagen remota llamada hello-app, gcr.io el registro de contenedor de Google.

A continuación, exponga la web implementación como un servicio Kubernetes, especificando un puerto estático donde será accesible con --type=NodePorty --port=8080:

``` 
kubectl expose deployment web --type=NodePort --port=8080
``` 

Ahora puedes comprobar si el servicio se está ejecutando con el kubectl get service comando:

``` 
kubectl get service web
``` 

Recuerde, los NodePorts de Kubernetes usan puertos aleatorios y su resultado será diferente:

``` 
Output
NAME   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.109.254.242   <none>        8080:31534/TCP   10s
``` 

Ahora puede utilizar minikube para recuperar una URL a la que se pueda acceder fuera del contenedor. Esta URL le permitirá acceder al hello-app servicio que se ejecuta en el puerto 8080 dentro de su clúster. Si está ejecutando Minikube localmente, no necesitará realizar ningún reenvío de puerto activo utilizando este método. Para recuperar la URL de su aplicación de muestra, ejecute el siguiente minikube service web –url comando:

``` 
minikube service web --url
Output
http://192.168.49.2:31534
``` 

Ahora puedes intentar conectarte a esa URL. Para hacer esto, utilizará un programa de línea de comandos llamado curl, que es popular para realizar diferentes tipos de solicitudes web. En general, si desea verificar si una conexión determinada debería funcionar en un navegador en circunstancias ideales, siempre debe probar primero con curl.

``` 
curl http://192.168.49.2:31534
Output
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-j92tb
``` 

Si está ejecutando minikube en una máquina local, también puede visitar esta URL en un navegador y debería devolver el mismo texto plano sin estilo. Si está ejecutando en una máquina remota, puede usar el túnel SSH nuevamente como en el paso 2 si desea ver este resultado en un navegador.

Ahora tiene un ejemplo mínimo de una aplicación implementada mediante minikube. Por lo general, en un clúster de Kubernetes de producción, proporcionaría un acceso más detallado a cualquier punto final accesible desde la web, pero eso no depende de ninguna minikube funcionalidad específica, cuyos fundamentos ha visto aquí.

En el siguiente paso de este tutorial, utilizará algunas de las herramientas integradas de Minikube para cambiar algunos de los valores de configuración predeterminados de su clúster.


## 6. Gestión de los recursos y el sistema de archivos de Minikube

El minikube comando proporciona varios subcomandos para ayudar a administrar su clúster. Por ejemplo, si alguna vez necesita cambiar la cantidad de memoria disponible en su clúster, puede usar la minikube config para ajustar la cantidad predeterminada. De forma predeterminada, este valor se proporciona en MB, por lo que minikube config 4096 proporcionaría el equivalente a 4 GB a su clúster:

``` 
minikube config set memory 4096
Output
❗  These changes will take effect upon a minikube delete and then a minikube start
``` 

El resultado indica que deberá volver a implementar su clúster para que el cambio surta efecto.

``` 
minikube delete
Output
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /home/sammy/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
``` 

``` 
minikube start
``` 

minikube También proporciona la capacidad de montar temporalmente un directorio desde su sistema de archivos local en el clúster. Puede exportar un directorio a su clúster usando el minikube mount comando.

La sintaxis del mount comando utiliza la siguiente sintaxis: local_path:minikube_host_path. La local_path parte del comando es su directorio local que desea montar en el clúster. La minikube_host_path parte del comando es la ubicación en el contenedor Minikube o VM donde le gustaría acceder a los archivos.

Este comando de muestra montará su directorio de inicio local en su minikube clúster en la /host ruta:

``` 
minikube mount $HOME:/host
Output
📁  Mounting host path /home/sammy into VM as /host ...
    ▪ Mount type:
    ▪ User ID:      docker
    ▪ Group ID:     docker
    ▪ Version:      9p2000.L
    ▪ Message Size: 262144
    ▪ Options:      map[]
    ▪ Bind Address: 192.168.49.1:43605
🚀  Userspace file server: ufs starting
✅  Successfully mounted /home/sammy to /host

📌  NOTE: This process must stay alive for the mount to be accessible ...
``` 

Esto puede resultar útil si desea conservar la entrada o la salida, como el registro de un minikube clúster.

Al igual que con el reenvío de puertos, esto se ejecutará como un proceso de bloqueo en esta terminal hasta que envíe un Ctrl+C comando. En el siguiente y último paso, aprenderá cómo cambiar de manera eficiente entre minikube un clúster de Kubernetes remoto y completo.


## 7. (Opcional) Trabajar con varios clústeres de Kubernetes

Para ejecutar varios clústeres de Kubernetes localmente, minikube configure varios perfiles. Por ejemplo, si desea trabajar y probar varias versiones de Kubernetes, puede crear varios clústeres de Kubernetes y alternar entre ellos usando la marca --profile o -p.

Si planea trabajar con un perfil específico por un tiempo, el minikube profile comando le permite configurar el perfil predeterminado que le gustaría usar, en lugar de especificarlo con la --profile bandera con cada comando.

Puedes iniciar Minikube con un nuevo perfil ejecutando minikube start con la -p bandera:

``` 
minikube start -p new-profile
``` 

Luego puedes cambiar el perfil activo de Minikube con minikube profile:

```
minikube profile new-profile
Output
✅  minikube profile was successfully set to new-profile
``` 

También puedes recuperar el perfil actual con el que estás trabajando con el get profile comando:

``` 
minikube config get profile
Output
New-profile
``` 

Ya sea que esté utilizando varios perfiles o no, minikube crea automáticamente archivos de configuración en sus ubicaciones predeterminadas donde kubectl otras herramientas de Kubernetes pueden analizarlos. Por ejemplo, si ejecutara kubectl get nodes, analizaría minikube la configuración de su clúster y devolvería un solo nodo:

``` 
kubectl get nodes
Output
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   3h18m   v1.23.1
``` 

Al ejecutarlo kubectl, puede especificar la ruta a un kubeconfig archivo diferente al predeterminado ~/.kube/config. kubectl utilizará las credenciales del clúster especificadas en esa configuración en lugar de las predeterminadas. Por ejemplo, si tiene otra configuración de clúster en un archivo llamado remote-kubeconfig.yaml, puede recuperar los nodos de ese clúster usando el siguiente comando:

``` 
kubectl --kubeconfig=remote-kubeconfig.yaml get nodes
``` 

Estos nodos que no son Minikube se ejecutan de forma remota:

``` 
Output
NAME                   STATUS   ROLES    AGE    VERSION
pool-xr6rvqbox-uha8f   Ready    <none>   2d2h   v1.21.9
pool-xr6rvqbox-uha8m   Ready    <none>   2d2h   v1.21.9
pool-xr6rvqbox-uha8q   Ready    <none>   2d2h   v1.21.9
``` 

Kubernetes generalmente está diseñado para funcionar con un archivo de configuración por clúster, de modo que se puedan pasar a kubectl otros comandos en tiempo de ejecución. Si bien es posible fusionar configuraciones, las mejores prácticas variarán según su uso de Kubernetes y no es necesario hacerlo. Quizás también quieras investigar el uso de Krew, un administrador de paquetes para complementos de Kubectl.


## 8. Conclusión

En este tutorial, instaló Minikube y configuró el panel integrado de Kubernetes para monitorear e implementar aplicaciones. También exploró algunas de las mejores prácticas para trabajar simultáneamente con una instancia de prueba local minikube y una instancia remota de Kubernetes utilizando perfiles de Minikube y la kubectl --kubeconfig bandera. Probar y evaluar las configuraciones de Kubernetes de minikube forma local puede ser de gran ayuda para determinar si está preparado para implementar Kubernetes en producción y cuándo hacerlo.

