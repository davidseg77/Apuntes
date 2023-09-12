# Gestión de un clúster local de Kubernetes utilizando Kind y Terraform

Durante la siguiente práctica, crearemos un nuevo clúster local de Kubernetes utilizando Kind y Terraform para aprovisionar un servicio nginx. Por ende, tendremos un clúster de Kubernetes con 2 nodos, que están ejecutando el servicio de NGINX. Esto será creado desde Terraform y el cluster será gestionado por KIND.

## 1. Instalación de Kubectl, Terraform y Docker

* **Kubectl**

1. Actualizamos el índice de paquetes de apt e instalamos los paquetes necesarios para usar Kubectl con el repositorio apt:

```
sudo apt update
sudo apt-get install -y apt-transport-https cacertificates
curl
``` 

2. Descargamos la clave de firma pública de Google Cloud:

``` 
sudo curl -fsSLo /usr/share/keyrings/kubernetes archivekeyring.
gpg https://packages.cloud.google.com/apt/doc/aptkey.
gpg
``` 

3. Agregamos el repositorio de Kubectl a apt:

``` 
echo "deb [signed-by=/usr/share/keyrings/kubernetesarchive-
keyring.gpg] https://apt.kubernetes.io/
kubernetes-xenial main" | sudo tee
/etc/apt/sources.list.d/kubernetes.list
``` 

4. Actualizamos el índice de paquetes de apt con el nuevo repositorio e instale kubectl:

``` 
sudo apt-get update
sudo apt-get install -y kubectl
``` 

* **Terraform**

1. Descargamos el paquete binario Terraform mediante el comando wget:

``` 
wget -c
https://releases.hashicorp.com/terraform/0.13.5/terraform_
0.13.5_linux_amd64.zip
``` 

2. Descomprimimos el archivos descargado:

``` 
unzip terraform_0.13.5_linux_amd64.zip
``` 

3. Colocamos el binario de Terraform en el directorio raiz del sistema.

``` 
sudo mv terraform /usr/sbin/
``` 

4. Verificamos la versión que tenemos instalada:

``` 
terraform versión
``` 

* **Docker**

1. Para poder crear los servicios necesarios en Kubernetes, necesitamos los contenedores que proporciona Docker, para ello procedemos a instalarlo. Ejecutamos el siguiente comando para añadir la clave GPG para Docker.

``` 
curl -fsSL https://download.docker.com/linux/debian/gpg |
sudo gpg --dearmor -o /usr/share/keyrings/docker-archivekeyring.
gpg
``` 

2. Después, añadimos el repositorio de Docker, utilizando el siguiente comando.

``` 
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/dockerarchive-
keyring.gpg]
https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | sudo tee
/etc/apt/sources.list.d/docker.list > /dev/null
``` 

4. Ahora ejecuta el comando ‘apt update‘ para actualizar/refrescar todos los repositorios disponibles.
   
``` 
apt update
``` 

5. Ahora estás listo para instalar Docker en Debian 11 Bullseye.

``` 
sudo apt install docker-ce docker-ce-cli containerd.io
``` 

5. Habilitamos el servicio de docker.

``` 
sudo systemctl is-enabled docker
```

6. Habilitamos el servicio.

``` 
sudo systemctl is-enabled containerd
``` 

7. Con este comando verificamos el estatus del servicio docker.

```
sudo systemctl status docker containerd
``` 

## 2. Caso práctico

El primer paso sera crear un directorio donde trabajar:

``` 
mkdir terraform-kubernetes
``` 

Ahora tendremos que instalar KIND, para poder crear el cluster. Lo descargaremos desde su propio repositorio:

``` 
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kindlinux-
amd64
``` 

Al binario descargado le damos permiso de ejecución.

``` 
chmod +x ./kind
``` 

Movemos el binario dentro de la raíz del sistema.

``` 
sudo mv ./kind /usr/local/bin/kind
``` 

El siguiente paso seria descargar el archivo de configuración de Kind, que tenemos guardado en nuestro repositorio Git. Descargamos el directorio completo y luego movemos el archivo que nos interesa.

``` 
Git clone https://github.com/Domin01/terraform-kubernetes
``` 

``` 
mv terraform-kubernetes/kind_config.yaml
```

Si abrimos el archivo, podemos ver que es una configuración para nuestro cluster. Simplemente lo que estamos haciendo es crear el cluster y definir el puerto del mismo, ademas de una regla para que escuche en todas las
dirección.

``` 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30201
    hostPort: 30201
    listenAddress: "0.0.0.0"
``` 

Vamos a proceder a crear el cluster con el siguiente comando:

``` 
sudo kind create cluster --name terraform-kubernetes --
config kind_config.yaml
``` 

Para verificar que se ha creado correctamente ejecutamos el siguiente comando para ver el estado del cluster.

``` 
sudo kind get clusters
```

Ahora verificamos que kubectl puede ver nuestro cluster, ejecutamos el siguiente comando:

``` 
sudo kubectl cluster-info --context kind-terraformkubernetes
``` 

Nuestro siguiente paso sera trabajar con las credenciales que tiene el cluster incorporadas, para ello bajamos el archivo de configuración donde estarán las variables, para que Terraform pueda crear los objetos dentro del cluster.

``` 
mv terraform-kubernetes/kubernetes.tf
```

Dentro de este archivo estamos declarando nuestro proveedor, que en este caso es Kubernetes.

``` 
terraform {
  required_providers {
    kubernetes = {
      source = "hashicorp/kubernetes"
    }
  }
}
```

También estamos declarando variables, como host, certificado de cliente, nuestra clave del cliente y nuestro certificado CA de cluster.

``` 
variable "host" {
  type = string
}

variable "client_certificate" {
  type = string
}

variable "client_key" {
  type = string
}

variable "cluster_ca_certificate" {
  type = string
}
``` 

Justo mas abajo del documento se declaran las variables, que se recogeran en el documento que creamos mas adelante.

``` 
provider "kubernetes" {
  host = var.host
  client_certificate = base64decode(var.client_certificate)
  client_key = base64decode(var.client_key)
  cluster_ca_certificate =
base64decode(var.cluster_ca_certificate)
}
``` 

Con el siguiente comando sacamos las credenciales del cluster, esta información la guardaremos en un archivo llamado terraform.tfvars. Donde se guardara el host, certificado de cliente, nuestra clave del cliente y nuestro certificado CA de cluster.

``` 
sudo kubectl config view --minify --flatten --
context=kind-terraform-kubernetes
``` 

Crearemos el fichero terraform.tfvars con las siguientes variables y la información recopilada anteriormente.

En primer lugar, vamos a crear la variable host = “”. Y vamos a copiar nuestra dirección de host. A continuación, vamos a crear el client_certificate = “”. Vamos a pegar los datos de nuestro cliente. Después vamos a crear la variable client_key = “”. Pegamos la información extraída. Y por último, creamos la variable cluster_ca_certificate = “” con su información correspondiente.

``` 
#Variables Terraform
host = ""
client_certificate = ""
client_key = ""
cluster_ca_certificate = ""
```

Guardamos el documento. Ahora deberíamos tener dos archivos uno con la configuración principal de Terraform llamado kubernetes.tf y otro archivo con las variables llamado terraform.tfvars. Ahora vamos a formatear los archivos de configuración al formato legible por terraform.

``` 
terraform fmt
``` 

Después inicializaremos el directorio, ejecutando el comando **terraform init**.

El siguiente paso sera empezar a programar el despliegue. Creamos el cluster con Terraform, descargamos del repositorio el archivo resource_nginx.tf.

Dentro de este archivo encontramos el despliegue de Kubernetes. Mas abajo aparecen las replicas especificadas, en este caso serán 2 y el puerto por donde mandará información fuera del contenedor, puerto 80 y por supuesto la imagen del contenedor como hemos dicho antes utilizaremos nginx.

``` 
resource "kubernetes_deployment" "nginx" {
  metadata {
    name = "nginx"
    labels = {
    App = "Nginx"
    }
  }
  spec {
    replicas = 2
    selector {
      match_labels = {
        App = "Nginx"
      }
    }
    template {
      metadata {
        labels = {
          App = "Nginx"
        }
      }
      spec {
        container {
          image = "nginx"
          name = "proyecto-nginx"
          port {
            container_port = 80
          }
        }
      }
    }
  }
}
```

A continuación, aplicaremos la configuración para programar el despliegue de NGINX. Pero antes que nada verificamos si la configuración esta bien establecida con el siguiente comando:

``` 
terraform validate
``` 

Ahora realizaremos el despliegue con el comando **terraform apply**.
Ponemos el parámetro “yes” y empezara a crearse automáticamente.
Verificamos que se ha creado correctamente.

Ahora para poder acceder desde fuera necesitamos un recurso para nuestro desligue de NGINX, en este caso el recurso NodePort que expondrá nuestro cluster al exterior con el numero de puertos 30201.
Descargamos el archivo de configuración llamado service_nginx.tf.

``` 
resource "kubernetes_service" "nginx" {
  metadata {
    name = "nginx-example"
  }
  spec {
    selector = {
      App =
kubernetes_deployment.nginx.spec.0.template.0.meta
data[0].labels.App
    }
    port {
      node_port = 30201
      port = 80
      target_port = 80
    }

    type = "NodePort"
  }
}
```

En la configuración podemos ver que el puerto que esta reenviando, es el puerto 80 al puerto 30201. Lo siguiente seria aplicar la configuración desde Terraform. Utilizando **terraform apply**.

Ahora verificamos que el servicio esta creado correctamente, ejecutando:

``` 
kubectl get services
```

Para verificar que todo esta funcionando correctamente entraremos en el navegador y pondremos la dirección IP de la maquina con el numero de puertos.

Comprobamos con ip a y ponemos la ip en el navegador seguido del puerto 30201.

Ahora para comprobar que con el recurso desplegado, se puede cambiar la configuración, vamos a cambiar el numero de despliegues a 6 y vamos a aplicar los cambios.

``` 
spec {
  replicas = 6
  selector {
    match_labels = {
      App = "Nginx"
    }
  }
}
``` 

Aplicamos los cambios y como vemos en la descripción del despliegue nos avisa que el numero de replicas ha cambiado y ahora el plan que tenemos no es para añadir, es para realizar cambios en la configuración.

Verificamos que realmente el numero de replicas ha cambiado.

``` 
sudo kubectl get deployments
```

Volvemos a verificar que el recurso sigue funcionando, con la ip en el navegador más el puerto.

El ultimo paso seria eliminar el proyecto y ver como se elimina por partes, el cluster y los servicios creados, utilizamos el siguiente comando y escribimos “yes”, para confirmar el borrado.

``` 
terraform destroy
``` 

Se eliminara el servicio y los deployments. Verificamos que después de destruir el plan, ya no podríamos acceder al recurso web desde el navegador.




