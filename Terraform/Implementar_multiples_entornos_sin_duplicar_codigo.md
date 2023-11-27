# Cómo implementar múltiples entornos en su proyecto Terraform sin duplicar código

## 1. Introducción

Terraform ofrece funciones avanzadas que se vuelven cada vez más útiles a medida que su proyecto crece en tamaño y complejidad. Es posible aliviar el costo de mantener definiciones de infraestructura complejas para múltiples entornos estructurando su código para minimizar las repeticiones e introduciendo flujos de trabajo asistidos por herramientas para facilitar las pruebas y la implementación.

Terraform asocia un estado con un backend, que determina dónde y cómo se almacena y recupera el estado. Cada estado tiene un solo backend y está vinculado a una configuración de infraestructura. Ciertos servidores, como local o s3, pueden contener varios estados. En ese caso, la combinación de estado e infraestructura con el backend describe un espacio de trabajo . Los espacios de trabajo le permiten implementar múltiples instancias distintas de la misma configuración de infraestructura sin almacenarlas en backends separados.

En este tutorial, primero implementará varias instancias de infraestructura utilizando diferentes espacios de trabajo. Luego implementará un recurso con estado que, en este tutorial, será un volumen de DigitalOcean. Finalmente, hará referencia a módulos prediseñados de Terraform Registry, que puede utilizar para complementar los suyos.


## 2. Requisitos previos

Para completar este tutorial, necesitará:

* Terraform instalado en su máquina local y un proyecto configurado con el proveedor DO. 


## 3. Implementación de varias instancias de infraestructura mediante espacios de trabajo

Múltiples espacios de trabajo son útiles cuando desea implementar o probar una versión modificada de su infraestructura principal sin crear un proyecto separado y configurar claves de autenticación nuevamente. Una vez que haya desarrollado y probado una característica usando el estado separado, puede incorporar el nuevo código en el espacio de trabajo principal y posiblemente eliminar el estado adicional. Cuando crea init un proyecto de Terraform, independientemente del backend, Terraform crea un espacio de trabajo llamado default. Siempre está presente y nunca podrás eliminarlo.

Sin embargo, múltiples espacios de trabajo no son una solución adecuada para crear múltiples entornos, como por ejemplo para la puesta en escena y la producción. Por lo tanto, los espacios de trabajo, que solo rastrean el estado, no almacenan el código ni sus modificaciones.

Dado que los espacios de trabajo no rastrean el código real, debe administrar la separación del código entre múltiples espacios de trabajo en el nivel de control de versiones (VCS) cotejándolos con sus variantes de infraestructura. La forma de lograrlo depende de la propia herramienta VCS; por ejemplo, en Git las ramas serían una abstracción adecuada. Para que sea más fácil administrar el código para múltiples entornos, puede dividirlos en módulos reutilizables, de modo que evite repetir código similar para cada entorno.

### 3.1 Implementación de recursos en espacios de trabajo

Ahora creará un proyecto que implemente un Droplet, que aplicará desde múltiples espacios de trabajo.

Almacenará la definición de Droplet en un archivo llamado droplets.tf.

Suponiendo que esté en el terraform-advanced directorio, créelo y ábralo para editarlo ejecutando:

``` 
nano droplets.tf
```

Agregue las siguientes líneas:

``` 
resource "digitalocean_droplet" "web" {
  image  = "ubuntu-18-04-x64"
  name   = "web-${terraform.workspace}"
  region = "fra1"
  size   = "s-1vcpu-1gb"
}
``` 

Esta definición creará un Droplet que ejecuta Ubuntu 18.04 con un núcleo de CPU y 1 GB de RAM en la fra1 región. Su nombre contendrá el nombre del espacio de trabajo actual desde el que se implementa. Cuando haya terminado, guarde y cierre el archivo.

Aplicar el proyecto para Terraform para ejecutar sus acciones con:

``` 
terraform apply -var "do_token=${DO_PAT}"
``` 

El resultado será similar a este:

``` 
Output
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-18-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + ipv6_address_private = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-default"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "fra1"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
...
``` 

Ingrese yes cuando se le solicite implementar el Droplet en el default espacio de trabajo.

El nombre del Droplet será web-default, porque el espacio de trabajo con el que comienzas se llama default. Puede enumerar los espacios de trabajo para confirmar que es el único disponible:

``` 
terraform workspace list
``` 

El resultado será similar a este:

``` 
Output
* default
``` 

El asterisco ( *) significa que actualmente tiene ese espacio de trabajo seleccionado.

Cree y cambie a un nuevo espacio de trabajo llamado testing, que usará para implementar un Droplet diferente, ejecutando workspace new:

``` 
terraform workspace new testing
``` 

El resultado será similar a este:

``` 
Output
Created and switched to workspace "testing"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
``` 

Planifica la implementación del Droplet nuevamente ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado será similar a la ejecución anterior:

``` 
Output
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-18-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + ipv6_address_private = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-testing"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "fra1"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
...
```

Tenga en cuenta que Terraform planea implementar un Droplet llamado web-testing, cuyo nombre es diferente al de web-default. Esto se debe a que los espacios de trabajo default y testing tienen estados separados y no conocen los recursos de cada uno, aunque provengan del mismo código.

Para confirmar que está en el testing espacio de trabajo, genere el actual en el que se encuentra workspace show:

``` 
terraform workspace show
```

El resultado será el nombre del espacio de trabajo actual:

``` 
Output
testing
``` 

Para eliminar un espacio de trabajo, primero debe destruir todos sus recursos implementados. Luego, si está activo, deberás cambiar a otro usando workspace select. Dado que el testing espacio de trabajo aquí está vacío, puedes cambiar a default:

``` 
terraform workspace select default
``` 

Recibirá un resultado de Terraform confirmando el cambio:

``` 
Output
Switched to workspace "default".
``` 

Luego puedes eliminarlo ejecutando workspace delete:

``` 
terraform workspace delete testing
``` 

Terraform luego realizará la eliminación:

``` 
Output
Deleted workspace "testing"!
``` 

Puedes destruir el Droplet que has implementado en el default espacio de trabajo ejecutando:

``` 
terraform destroy -var "do_token=${DO_PAT}"
``` 

Ingrese yes cuando se le solicite finalizar el proceso.

En esta sección, ha trabajado en múltiples espacios de trabajo de Terraform. En la siguiente sección, implementará un recurso con estado.


## 4. Implementación de recursos con estado

Los recursos sin estado no almacenan datos, por lo que puedes crearlos y reemplazarlos rápidamente, ya que no son únicos. Los recursos con estado, por otro lado, contienen datos que son únicos o no simplemente recreables; por lo tanto, requieren un almacenamiento de datos persistente.

Dado que puede terminar destruyendo dichos recursos, o que varios recursos requieran sus datos, es mejor almacenarlos en una entidad separada, como DigitalOcean Volumes.

Los volúmenes proporcionan espacio de almacenamiento adicional. Se pueden adjuntar a Droplets (servidores), pero están separados de ellos. En este paso, definirá el volumen y lo conectará a un Droplet en formato droplets.tf.

Ábrelo para editarlo:

``` 
nano droplets.tf
``` 

Agregue las siguientes líneas:

``` 
resource "digitalocean_droplet" "web" {
  image  = "ubuntu-18-04-x64"
  name   = "web-${terraform.workspace}"
  region = "fra1"
  size   = "s-1vcpu-1gb"
}

resource "digitalocean_volume" "volume" {
  region                  = "fra1"
  name                    = "new-volume"
  size                    = 10
  initial_filesystem_type = "ext4"
  description             = "New Volume for Droplet"
}

resource "digitalocean_volume_attachment" "volume_attachment" {
  droplet_id = digitalocean_droplet.web.id
  volume_id  = digitalocean_volume.volume.id
}
```

Aquí define dos nuevos recursos, el volumen en sí y un volumen adjunto. El volumen será de 10 GB, tendrá el formato ext4, se llamará new-volume y estará ubicado en la misma región que el Droplet. Dado que el Volumen y el Droplet son entidades separadas, necesitarás definir un objeto adjunto de Volumen para conectarlos. volume_attachment toma los ID de Droplet y Volumen e indica a la nube de DigitalOcean que ponga el volumen a disposición del Droplet como un dispositivo de disco.

Cuando haya terminado, guarde y cierre el archivo.

Planifique esta configuración ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

Las acciones que planificará Terraform serán las siguientes:

```
OutputTerraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-18-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + ipv6_address_private = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-default"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "fra1"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

  # digitalocean_volume.volume will be created
  + resource "digitalocean_volume" "volume" {
      + description             = "New Volume for Droplet"
      + droplet_ids             = (known after apply)
      + filesystem_label        = (known after apply)
      + filesystem_type         = (known after apply)
      + id                      = (known after apply)
      + initial_filesystem_type = "ext4"
      + name                    = "new-volume"
      + region                  = "fra1"
      + size                    = 10
      + urn                     = (known after apply)
    }

  # digitalocean_volume_attachment.volume_attachment will be created
  + resource "digitalocean_volume_attachment" "volume_attachment" {
      + droplet_id = (known after apply)
      + id         = (known after apply)
      + volume_id  = (known after apply)
    }

Plan: 3 to add, 0 to change, 0 to destroy.
...
```

El resultado detalla que Terraform crearía un Droplet, un Volumen y un archivo adjunto de Volumen, que conecta el Volumen al Droplet.

Ahora ha definido y conectado un Volumen (un recurso con estado) a un Droplet. En la siguiente sección, revisará los módulos Terraform públicos prediseñados que puede incorporar a su proyecto.


**Conclusión**

Los proyectos más grandes pueden hacer uso de algunas funciones avanzadas que ofrece Terraform para ayudar a reducir la complejidad y facilitar el mantenimiento. Los espacios de trabajo le permiten probar nuevas incorporaciones a su código sin tocar las implementaciones principales estables. También puede vincular espacios de trabajo con un sistema de control de versiones para realizar un seguimiento de los cambios de código. El uso de módulos prefabricados también puede acortar el tiempo de desarrollo, pero puede generar gastos o tiempo adicionales en el futuro si el módulo queda obsoleto.

