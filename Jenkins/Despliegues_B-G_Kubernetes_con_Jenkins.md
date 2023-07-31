# Info extraida del taller Despliegues Blue/Green automatizados en Kubernetes con Jenkins

## Caso práctico

**Instalar Docker**

En primer lugar vamos a crear nuestro entorno, para eso necesitamos instalar: Docker, Jenkins y minikube.

Con este script, la instalación de la ventana acoplable está automatizada.

``` 
curl -fsSL https://get.docker.com | sudo bash
``` 

Si está utilizando Ubuntu 20.04:

``` 
apt update && apt install -y docker.io 
``` 

Agregue su usuario al grupo docker

```
sudo usermod -aG docker $USER
``` 

**Instalar Jenkins**

Vamos a usar docker para implementar Jenkins:

``` 
docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 5000:5000 jenkins/jenkins:lts
``` 

Vaya dentro de localhost: 8080 y haga este comando en shell:

``` 
docker exec <docker_container> cat /var/jenkins_home/secrets/initialAdminPassword
```

Inserte la contraseña, haga clic en instalar con complementos sugeridos y configure el usuario.

**Instalación de Minikube**

Ahora vamos a descargar el binario minikube y tenemos que dar permisos de ejecución.

``` 
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && mv minikube /usr/local/bin/
``` 

Ahora procedemos a desplegar minikube:

``` 
minikube start
``` 

E instale kubectl para operar el clúster:

``` 
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
``` 

Para asegurarnos de que todo marcha correctamente hacemos uso del siguiente comando:

``` 
kubectl get pod -n kube-system
```

Y todo ha de estar en estado running.

**Crear biblioteca compartida**

Ahora vamos a crear una biblioteca compartida, para esto necesitamos un repositorio nombrado jenkinsfile-shared-library con la siguiente estructura:

``` 
--src/com/mfranco/
--vars
--resources
``` 

Ahora nos vamos a Jenkins, Credentials y en Global Credentials las creamos, concretamente en Add credentials. Tendremos que poner el usuario de nuestro Github, la contraseña y un ID. Una vez que lo tengamos vamos a Jenkins, Administrar Jenkins, Configurar el sistema y en Global Pipeline Libraries las agregamos. En default ponemos master, por la rama a tener en cuenta, y activamos Modern SCM y Github.

Añadimos credenciales y URL del repositorio. Validamos, y guardamos. 


**Crear service account**

En primer lugar, clonamos el repositorio demo del taller mediante git clone. Aquí tendremos todo lo necesario para el taller.

```
git clone https://github.com/manu756/demo_repository/tree/master
``` 

Aquí vamos a crear una cuenta de servicio para implementar dentro del clúster, esto es manifests/serviceAccount.yaml y aplicarlo con:

``` 
kubectl apply -f manifests/serviceAccount.yaml
``` 

Ahora, necesito el token y el certificado creado para ejecutar los siguientes comandos:

Para token:

``` 
SECRET=$(kubectl get secrets | grep deployer | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.token}' | base64 -d
``` 

Para certificado:

``` 
SECRET=$(kubectl get secrets | grep deployer | awk '{print $1}') && kubectl get secret $SECRET -o jsonpath='{.data.ca\.crt}'
``` 

Ahora vamos a crear un archivo kubeconfig para poder acceder a nuestro clúster con los agentes para implementar pods:

Aquí tenemos la plantilla requerida: manifests/kubeconfig-template.yaml

``` 
apiVersion: v1
kind: Config
users:
- name: deployer
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IldQc3AwWGFfMlF2Mk5RYkpmTkM5VDBON3VnV19qWGZKWElHbDZfN3dlRzQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlcGxveWVyLXRva2VuLXI3ODd3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlcGxveWVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMzExNmFlMzItYzcyMC00NTNjLWI4NTctY2I1Mzk1YzZiMTJkIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6ZGVwbG95ZXIifQ.JrQ3bqTnx0ZlD_w1pjic2m8w471nH-2JWc7NINB2JTV18MxHuSt7rXbcSL-SiMAteCsFuSQGRt0uljgh58ibbH-rLkRtRwzYUJaGUs3R30lxse6mhTL22PRNr17K-Uk7QIpz4aYKMWp_raZdfWVkPPyE4AZED9h54M4Cts5GkuVKf1mvNwl0W6SMZ8lkfIpsdhu6uC8VYIuSn9ACxRXMQUbHh4QBvshUowjqMG8ohevut1GjfpuFg9C_Mf54XCl1C6ycXR8t0axX_EgxmbYx3zpBpRkZvtKf17b1QuPJJJdX0n9UC1pDu3Lq2LfHan676eyuARkytXx199oEAUlJNQ
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJd01EUXlOakU0TXpVd05Gb1hEVE13TURReU5URTRNelV3TkZvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS2RVCi9nVGhyVVdiVDZQVkpSZmhTSm1rdkdZaFFmbzdScmUvUm5abmVrVkNxZzFrK2Z6RDBIa2Y5aXhoOHpXbHZNSkUKbysydmFVdnkxc2FwSW84ZllESVR0QTA3VFVIMHI3VXU0ck9ZNngzQ1BUSDhHVG4xT2I4cyt2WHpWcU0yQXZ0ZgpBejBTV1FiNGRYbk5KdktndXo3RFd0MW0wSE1HSVFPemVoZ2F5WGhzZS9NWEs3ZGNWV0hzNjRmQ0dpbENiNmN4CmJrY3ZvdE5oTVVVOWlqL05CV2lSblV4ZU1XR3FMMk5wUVZscFgwNUdESHBBNEJISnRtcnN3RDczTVZpMmNjekcKaVdpL0NhazZaamc3MTBwVUlHMk9UL05YbUI2ak4wR3RSREVqUURSTzhYbnR0c1hsY1hVOVdPSWx3VUwwTWJEMgpNY3FvSlZySmxyRVd3RlNrUmRFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCNzFoaHgySXJlbGd6dVlJOFM4Q1J0VUE1b0ZibGpFS2VDSjYzVDRiSG05YWs5OC8vSAp3VWF1Y2J4QTF4UEo1YjUzaFBreUhTSC9PL2g4R2NCREpYRTRQZ2dNZGd3UkJ3ZHJyR2prMGEwZzlYZm80ZXVGClBFS3drSUF1djZxRUhhbnJYTnJ2YmVJU2FIeW5mUjBzSHpVbmJLK2pDS1BmNHBpcS8zMEVkcnUxR1FKSUxKd1QKb2t5VUV0YkdjeVRsZjdIOVo3aXpPT1lFZXhSWVRieGk5N1NDYzl2MnVHRzFJTXFHUlJXa2dhUndCZkx4S1FoQQpPSDJONURQRXpCMmR5dGRHcm9WTE00U0ZhNFRxY211UHd1cmVWOGpJa21VVmphTXBsZlA1bXVlTmRXclRVTlBPCllDOWNEZDNKVEtKZ3VNa2RvdmRLc2M5YWJNTXZyS0ZLYkROcwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://172.17.0.4:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: deployer
    namespace: default
  name: default
current-context: default
```

Ya tenemos nuestro template de Kubeconfig, pero ahora debemos de irnos a Jenkins, a Credentials y en Global Credential añadir una nueva de tipo Secret File. 

Ahi vamos a seleccionar el archivo recien creado, el yaml de kubeconfig-template.
Le ponemos de id kubeconfig.


**Crear un agente para los contenedores docker**

Para ejecutar este agente debemos crear un usuario llamado jenkins. Este usuario tiene su clave ssh. Copiamos su clave pública y vamos a Administrar Jenkins, a Administrar nodos, Nuevo nodo y marco Agente permanente. 

Le vamos a poner de nombre Docker, número de ejecutores 1, directorio raiz remoto /home/jenkins, en Etiquetas pongo Docker. Usar en modo Utilizar este nodo tanto como sea posible, bajo el metodo Arrancar agentes remotos en máquinas Unix via SSH. El nombre va a tomar la ip por defecto de contenedores Docker (172.17.0.1). Damos en Add credentials justo debajo de la ip y agregamos lo siguiente:

* SSH username with private key

Será global, con ID dockerNodeSSHKey, username Jenkins y pegamos la clave privada del usuario. 

Por último, debajo de la agregación de las credenciales tenemos que marcar como Host Key Verification Strategy la opción Manually trusted key verification strategy. Y activamos la casilla de require manual y guardamos.

Se va a lanzar pero la primera vez dará error. En la ventana de Jenkins, en el menu de la izquierda vamos a Trust SSH Host Key, Yes y Relanzar agente.

Para este taller debemos tener instalado Java en el equipo donde se halla el agente Jenkins.

**Mapear nuestro repo a Jenkins**

Para ello, copiamos el enlace de nuestro repo y en Jenkins damos Nueva tarea, con la opción Multibranch Pipeline. Decimos que la fuente de la rama va a ser Github, ponemos nuestras credenciales y la URL del repo. Guardamos y el repo será generado y lo construirá. Para saber si todo ha ido OK, vamos al navegador y ponemos la ip de nuestro servidor (172.17.0.4:30001).

**Subir el repositorio como si fuera un flujo de desarrollo**

Por ejemplo, cambiando el archivo app/app.js. Cambiamos la versión y en el Jenkinsfile hacemos lo propio.

Hacemos un commit y un push. 

Y vamos a la tarea de Jenkins y volvemos a construir. Si el Jenkins fuera público, tendría un webhook que haría el despliegue automático.

Comprobamos con una secuencia de comandos:

``` 
kubectl get pods
kubectl get deploy
kubectl port-forward deployment/nodejs-app-v4 3000:3000
```

Si voy al navegador y compruebo, tendremos nuestra nueva versión operativa.

**Blue/Green**

Si hubiera un error durante el despliegue de una nueva versión, solo tenemos que ir dentro de la tarea de Jenkins a la pestaña de dicha versión, Restart from stage y opción Do Blue/Green. Damos en Run y se iniciará el proceso de construcción de nuevo. 




