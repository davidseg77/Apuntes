# Cómo construir un módulo Terraform personalizado

## 1. Introducción

Los módulos de Terraform le permiten agrupar distintos recursos de su infraestructura en un único recurso unificado. Puede reutilizarlos más adelante con posibles personalizaciones, sin repetir las definiciones de recursos cada vez que los necesite, lo que resulta beneficioso para proyectos grandes y estructurados de forma compleja. Puede personalizar instancias de módulos utilizando variables de entrada que defina, así como extraer información de ellas mediante salidas. Además de crear sus propios módulos personalizados, también puede utilizar los módulos prediseñados publicados públicamente en Terraform Registry. Los desarrolladores pueden usarlos y personalizarlos utilizando entradas como los módulos que usted crea, pero su código fuente se almacena y se extrae de la nube.

En este tutorial, creará un módulo Terraform que configurará múltiples Droplets detrás de un Load Balancer para lograr redundancia. También utilizará las funciones de bucle for_each y count del lenguaje de configuración de Hashicorp (HCL) para implementar múltiples instancias personalizadas del módulo al mismo tiempo.

## 2. Requisitos previos

* Terraform instalado en su máquina local.
* Familiaridad con los tipos de datos y bucles de HCL. 
* Familiaridad con las salidas de Terraform y su uso. 

## 3. Estructura del módulo y beneficios

En esta sección, aprenderá qué beneficios aportan los módulos, dónde generalmente se ubican en el proyecto y cómo deben estructurarse.

Los módulos Terraform personalizados se crean para encapsular componentes conectados que se utilizan e implementan juntos con frecuencia en proyectos más grandes. Son autónomos y agrupan sólo los recursos, variables y proveedores que necesitan.

Los módulos generalmente se almacenan en una carpeta central en la raíz del proyecto, cada uno en su respectiva subcarpeta debajo. Para mantener una separación clara entre los módulos, diseñélos siempre para que tengan un único propósito y asegúrese de que nunca contengan submódulos.

Es útil crear módulos a partir de sus esquemas de recursos cuando se encuentra repitiéndolos con personalizaciones poco frecuentes. Empaquetar un único recurso como un módulo puede resultar superfluo y elimina gradualmente la simplicidad de la arquitectura general.

Para proyectos pequeños de desarrollo y pruebas no es necesario incorporar módulos porque no aportan mucha mejora en esos casos. Gracias a su capacidad de personalización, los módulos son el elemento constructivo de proyectos estructurados de forma compleja. Los desarrolladores utilizan módulos para proyectos más grandes debido a las importantes ventajas de evitar la duplicación de código. Los módulos también ofrecen la ventaja de que las definiciones solo necesitan modificaciones en un lugar, que luego se propagarán por el resto de la infraestructura.

A continuación, definirá, utilizará y personalizará módulos en sus proyectos de Terraform.


## 4. Creando un módulo

En esta sección, definirá varios Droplets y un Load Balancer como recursos de Terraform y los empaquetará en un módulo. También hará que el módulo resultante sea personalizable mediante las entradas del módulo.

Almacenará el módulo en un directorio llamado droplet-lb, bajo un directorio llamado modules. Suponiendo que se encuentra en el terraform-modules directorio que creó como parte de los requisitos previos, cree ambos a la vez ejecutando:

``` 
mkdir -p modules/droplet-lb
``` 

El -p argumento indica mkdir que se creen todos los directorios en la ruta proporcionada.

Navegue hasta él:

``` 
cd modules/droplet-lb
``` 

Como se señaló en la sección anterior, los módulos contienen los recursos y variables que utilizan. A partir de Terraform 0.13, también deben incluir definiciones de los proveedores que utilizan. Los módulos no requieren ninguna configuración especial para tener en cuenta que el código representa un módulo, ya que Terraform considera cada directorio que contiene código HCL como un módulo, incluso el directorio raíz del proyecto.

Las variables definidas en un módulo se exponen como sus entradas y se pueden usar en definiciones de recursos para personalizarlas. El módulo que creará tendrá dos entradas: la cantidad de Droplets a crear y el nombre de su grupo. Crea y abre para editar un archivo llamado variables.tf donde almacenarás las variables:

``` 
nano variables.tf
``` 

Agregue las siguientes líneas:

``` 
variable "droplet_count" {}
variable "group_name" {}
``` 

Guarde y cierre el archivo.

Almacenará la definición de Droplet en un archivo llamado droplets.tf. Créelo y ábralo para editarlo:

``` 
nano droplets.tf
``` 

Agregue las siguientes líneas:

``` 
resource "digitalocean_droplet" "droplets" {
  count  = var.droplet_count
  image  = "ubuntu-20-04-x64"
  name   = "${var.group_name}-${count.index}"
  region = "fra1"
  size   = "s-1vcpu-1gb"
}
``` 

Para el count parámetro, que especifica cuántas instancias de un recurso crear, se pasa la droplet_count variable. Su valor se especificará cuando se llame al módulo desde el código principal del proyecto. El nombre de cada uno de los Droplets implementados será diferente, lo que se logra agregando el índice del Droplet actual al nombre del grupo proporcionado. La implementación de los Droplets se realizará en la fra1 región y ejecutarán Ubuntu 20.04.

Cuando haya terminado, guarde y cierre el archivo.

Con los Droplets ahora definidos, puede continuar con la creación del Load Balancer. Almacenará su definición de recurso en un archivo llamado lb.tf. Créelo y ábralo para editarlo ejecutando:

``` 
nano lb.tf
``` 

Agregue su definición de recurso:

``` 
resource "digitalocean_loadbalancer" "www-lb" {
  name   = "lb-${var.group_name}"
  region = "fra1"

  forwarding_rule {
    entry_port     = 80
    entry_protocol = "http"

    target_port     = 80
    target_protocol = "http"
  }

  healthcheck {
    port     = 22
    protocol = "tcp"
  }

  droplet_ids = [
    for droplet in digitalocean_droplet.droplets:
      droplet.id
  ]
}
``` 

Usted define el Load Balancer con el nombre del grupo en su nombre para que sea distinguible. Lo implementas en la fra1 región junto con los Droplets. Las dos secciones siguientes especifican los puertos y protocolos de destino y de monitoreo.

El droplet_ids bloque resaltado incluye los ID de los Droplets, que deben ser administrados por el Load Balancer. Dado que hay varios Droplets y su recuento no se conoce de antemano, se utiliza un for bucle para recorrer la colección de Droplets ( digitalocean_droplet.droplets) y tomar sus ID. Rodea el for bucle entre corchetes ( []) para que la colección resultante sea una lista.

Guarde y cierre el archivo.

Ahora ha definido el Droplet, el Load Balancer y las variables para su módulo. Deberá definir los requisitos del proveedor, especificando qué proveedores utiliza el módulo, incluida su versión y dónde están ubicados. Desde Terraform 0.13, los módulos deben definir explícitamente las fuentes de los proveedores no mantenidos por Hashicorp que utilizan; esto se debe a que no los heredan del proyecto principal.

Almacenará los requisitos del proveedor en un archivo llamado provider.tf. Créelo para editarlo ejecutando:

``` 
nano provider.tf
``` 

Agregue las siguientes líneas para solicitar el digital ocean proveedor:

``` 
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}
``` 

Guarde y cierre el archivo cuando haya terminado. El droplet-lb módulo ahora requiere el digital ocean proveedor.

Los módulos también admiten salidas, que puede utilizar para extraer información interna sobre el estado de sus recursos. Definirá una salida que exponga la dirección IP del Load Balancer y la almacenará en un archivo denominado outputs.tf. Créelo para editarlo:

``` 
nano outputs.tf
``` 

Agregue la siguiente definición:

``` 
output "lb_ip" {
  value = digitalocean_loadbalancer.www-lb.ip
}
``` 

Esta salida recupera la dirección IP del Load Balancer. Guarde y cierre el archivo.

El droplet-lb módulo ahora está funcionalmente completo y listo para su implementación. Lo llamarás desde el código principal, que almacenarás en la raíz del proyecto. Primero, navegue hasta él yendo hacia arriba a través de su directorio de archivos dos veces:

``` 
cd ../..
``` 

Luego, crea y abre para editar un archivo llamado main.tf, en el que usarás el módulo:

``` 
nano main.tf
``` 

Agregue las siguientes líneas:

``` 
module "groups" {
  source = "./modules/droplet-lb"

  droplet_count = 3
  group_name    = "group1"
}

output "loadbalancer-ip" {
  value = module.groups.lb_ip
}
``` 

En esta declaración invocas el droplet-lb módulo ubicado en el directorio especificado como source. Usted configura la entrada que proporciona droplet_county group_name, que está configurada group1 para que luego pueda discernir entre instancias.

Dado que la salida IP del Load Balancer está definida en un módulo, no se mostrará automáticamente cuando aplique el proyecto. La solución a esto es crear otra salida recuperando su valor ( loadbalancer_ip).

Guarde y cierre el archivo cuando haya terminado.

Inicialice el módulo ejecutando:

``` 
terraform init
``` 

La salida se verá así:

``` 
Output
Initializing modules...
- groups in modules/droplet-lb

Initializing the backend...

Initializing provider plugins...
- Finding digitalocean/digitalocean versions matching "~> 2.0"...
- Installing digitalocean/digitalocean v2.19.0...
- Installed digitalocean/digitalocean v2.19.0 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)

...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Puede intentar planificar el proyecto para ver qué acciones tomaría Terraform ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado será similar a este:

``` 
Output...
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.groups.digitalocean_droplet.droplets[0] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "group1-0"
...
    }

  # module.groups.digitalocean_droplet.droplets[1] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "group1-1"
...
    }

  # module.groups.digitalocean_droplet.droplets[2] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "group1-2"
...
    }

  # module.groups.digitalocean_loadbalancer.www-lb will be created
  + resource "digitalocean_loadbalancer" "www-lb" {
...
      + name                     = "lb-group1"
...
    }

Plan: 4 to add, 0 to change, 0 to destroy.
...
```

Este resultado detalla que Terraform crearía tres Droplets, llamados group1-0, group1-1y group1-2, y también crearía un equilibrador de carga llamado group1-lb, que administrará el tráfico hacia y desde los tres Droplets.

Puedes intentar aplicar el proyecto a la nube ejecutando:

```
terraform apply -var "do_token=${DO_PAT}"
``` 

Ingrese yes cuando se le solicite. El resultado mostrará todas las acciones y también se mostrará la dirección IP del Load Balancer:

``` 
Output
module.groups.digitalocean_droplet.droplets[1]: Creating...
module.groups.digitalocean_droplet.droplets[0]: Creating...
module.groups.digitalocean_droplet.droplets[2]: Creating...
...
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

loadbalancer-ip = ip_address
``` 

Ha creado un módulo que contiene una cantidad personalizable de Droplets y un equilibrador de carga que se configurará automáticamente para administrar su tráfico entrante y saliente.

## 5. Cambiar el nombre de los recursos implementados

En la sección anterior, implementó el módulo que definió y lo llamó groups. Si alguna vez desea cambiar su nombre, simplemente cambiar el nombre de la llamada del módulo no producirá los resultados esperados. Cambiar el nombre de la llamada hará que Terraform destruya y vuelva a crear recursos, lo que provocará un tiempo de inactividad excesivo.

Por ejemplo, ábralo main.tf para editar ejecutando:

``` 
nano main.tf
``` 

Cambie el nombre del groups módulo a groups_renamed, como se resalta:

``` 
module "groups_renamed" {
  source = "./modules/droplet-lb"

  droplet_count = 3
  group_name    = "group1"
}

output "loadbalancer-ip" {
  value = module.groups_renamed.lb_ip
}
``` 

Guarde y cierre el archivo. Luego, inicialice el proyecto nuevamente:

```
terraform init
``` 

Ahora puedes planificar el proyecto:

```
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado será largo, pero tendrá un aspecto similar a este:

``` 
Output
...
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
  - destroy

Terraform will perform the following actions:
     # module.groups.digitalocean_droplet.droplets[0] will be destroyed
     ...
     # module.groups_renamed.digitalocean_droplet.droplets[0] will be created
     ...
```

Terraform le pedirá que destruya las instancias existentes y cree otras nuevas. Esto es destructivo e innecesario y puede provocar tiempos de inactividad no deseados.

En su lugar, utilizando el moved bloque, puede indicarle a Terraform que mueva recursos antiguos con el nuevo nombre. Ábralo main.tf para editarlo y agregue las siguientes líneas al final del archivo:

``` 
moved {
  from = module.groups
  to   = module.groups_renamed
}
```

Cuando haya terminado, guarde y cierre el archivo.

Ahora puedes planificar el proyecto:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

Cuando planifica con el moved bloque presente main.tf, Terraform quiere mover los recursos, en lugar de recrearlos:

``` 
Output
Terraform will perform the following actions:

     # module.groups.digitalocean_droplet.droplets[0] has moved to module.groups_renamed.digitalocean_droplet.droplets[0]
     ...
     # module.groups.digitalocean_droplet.droplets[1] has moved to module.groups_renamed.digitalocean_droplet.droplets[1]
     ...
``` 

Mover recursos cambia su lugar en el estado Terraform, lo que significa que los recursos reales de la nube no se modificarán, destruirán ni recrearán.

Debido a que modificará significativamente la configuración en el siguiente paso, destruya los recursos implementados ejecutando:

``` 
terraform destroy -var "do_token=${DO_PAT}"
``` 

Ingrese yes cuando se le solicite. La salida terminará en:

``` 
Output
...
Destroy complete! Resources: 4 destroyed.
``` 

En esta sección, cambió el nombre de los recursos en su proyecto Terraform sin destruirlos en el proceso. Ahora implementará varias instancias de un módulo desde el mismo código usando for_each y count.

## 6. Implementación de múltiples instancias de módulo

En esta sección, utilizará count y for_each implementará el droplet-lb módulo varias veces con personalizaciones.

### 6.1 Usando count

Una forma de implementar varias instancias del mismo módulo a la vez es pasar cuántas al count parámetro, que está disponible automáticamente para cada módulo. Abierto main.tf para editar:

``` 
nano main.tf
``` 

Modifíquelo para que se vea así, eliminando la definición de salida existente y moved el bloque:

``` 
module "groups" {
  source = "./modules/droplet-lb"

  count  = 3

  droplet_count = 3
  group_name    = "group1-${count.index}"
}
``` 

Al configurarlo count en 3, le indica a Terraform que implemente el módulo tres veces, cada una con un nombre de grupo diferente. Cuando haya terminado, guarde y cierre el archivo.

Planifique la implementación ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado será largo y se verá así:

``` 
Output
...
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.groups[0].digitalocean_droplet.droplets[0] will be created
...
  # module.groups[0].digitalocean_droplet.droplets[1] will be created
...
  # module.groups[0].digitalocean_droplet.droplets[2] will be created
...
  # module.groups[0].digitalocean_loadbalancer.www-lb will be created
...
  # module.groups[1].digitalocean_droplet.droplets[0] will be created
...
  # module.groups[1].digitalocean_droplet.droplets[1] will be created
...
  # module.groups[1].digitalocean_droplet.droplets[2] will be created
...
  # module.groups[1].digitalocean_loadbalancer.www-lb will be created
...
  # module.groups[2].digitalocean_droplet.droplets[0] will be created
...
  # module.groups[2].digitalocean_droplet.droplets[1] will be created
...
  # module.groups[2].digitalocean_droplet.droplets[2] will be created
...
  # module.groups[2].digitalocean_loadbalancer.www-lb will be created
...

Plan: 12 to add, 0 to change, 0 to destroy.
...
``` 

Terraform detalla en el resultado que cada una de las tres instancias de módulo tendría tres Droplets y un equilibrador de carga asociados.

### 6.2 Usando for_each

Puede usarlo for_each para módulos cuando requiera una personalización de instancias más compleja o cuando la cantidad de instancias dependa de datos de terceros (a menudo presentados como mapas) que no se conocen al escribir el código.

Ahora definirá un mapa que empareje los nombres de los grupos con los recuentos de Droplet e implementará instancias de droplet-lb acuerdo con él. Ábralo main.tf para editar ejecutando:

``` 
nano main.tf
``` 

Modifique el archivo para que se vea así:

``` 
variable "group_counts" {
  type    = map
  default = {
    "group1" = 1
    "group2" = 3
  }
}

module "groups" {
  source   = "./modules/droplet-lb"
  for_each = var.group_counts

  droplet_count = each.value
  group_name    = each.key
}
``` 

Primero define un mapa llamado group_counts que contiene cuántos Droplets debe tener un grupo determinado. Luego, invocas el módulo droplet-lb, pero especificas que el for_each bucle debe operar en var.group_counts el mapa que has definido justo antes. droplet_counttoma each.value, el valor del par actual, que es el recuento de Droplets para el grupo actual. group_name recibe el nombre del grupo.

Guarde y cierre el archivo cuando haya terminado.

Intente aplicar la configuración ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado detallará las acciones que Terraform tomaría para crear los dos grupos con sus Droplets y Load Balancers:

``` 
Output
...
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.groups["group1"].digitalocean_droplet.droplets[0] will be created
...
  # module.groups["group1"].digitalocean_loadbalancer.www-lb will be created
...
  # module.groups["group2"].digitalocean_droplet.droplets[0] will be created
...
  # module.groups["group2"].digitalocean_droplet.droplets[1] will be created
...
  # module.groups["group2"].digitalocean_droplet.droplets[2] will be created
...
  # module.groups["group2"].digitalocean_loadbalancer.www-lb will be created
...
``` 

En este paso, ha utilizado county for_each para implementar varias instancias personalizadas del mismo módulo a partir del mismo código.

## 7. Conclusión

En este tutorial, creó e implementó módulos Terraform. Utilizó módulos para agrupar recursos vinculados lógicamente y los personalizó para implementar múltiples instancias diferentes desde una definición de código central. También utilizó resultados para mostrar los atributos de los recursos contenidos en el módulo.

