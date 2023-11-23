# Cómo estructurar un proyecto Terraform

## 1. Introducción

Estructurar los proyectos Terraform de forma adecuada según sus casos de uso y la complejidad percibida es esencial para garantizar su mantenibilidad y extensibilidad en las operaciones diarias. Es necesario un enfoque sistemático para organizar adecuadamente los archivos de código para garantizar que el proyecto siga siendo escalable durante la implementación y utilizable para usted y su equipo.

En este tutorial, aprenderá cómo estructurar proyectos Terraform según su propósito general y complejidad. Luego, creará un proyecto con una estructura simple utilizando las características más comunes de Terraform: variables, locales, fuentes de datos y aprovisionadores. Al final, su proyecto implementará un servidor Ubuntu 20.04 (Droplet) en DigitalOcean, instalará un servidor web Apache y apuntará su dominio al servidor web.

## 2. Requisitos previos

* Un token de acceso personal de DigitalOcean, que puede crear a través del panel de control de DigitalOcean. 

* Una clave SSH sin contraseña agregada a su cuenta de DigitalOcean.

* Terraform instalado en su máquina local. 

* Python 3 instalado en su máquina local. 

* Un nombre de dominio completamente registrado agregado a su cuenta de DigitalOcean. 


## 3. Comprender la estructura de un proyecto Terraform

En esta sección, aprenderá qué considera Terraform un proyecto, cómo puede estructurar el código de infraestructura y cuándo elegir qué enfoque. También aprenderá sobre los espacios de trabajo de Terraform, qué hacen y cómo Terraform almacena el estado.

Un recurso es una entidad de un servicio en la nube (como DigitalOcean Droplet) declarada en código Terraform que se crea de acuerdo con propiedades especificadas e inferidas. Múltiples recursos forman infraestructura con sus conexiones mutuas.

Terraform utiliza un lenguaje de programación especializado para definir la infraestructura, llamado Hashicorp Configuration Language (HCL). El código HCL normalmente se almacena en archivos que terminan con la extensión tf. Un proyecto Terraform es cualquier directorio que contiene tfarchivos y que se ha inicializado mediante el initcomando, que configura las cachés de Terraform y el estado local predeterminado.

El estado de Terraform es el mecanismo mediante el cual realiza un seguimiento de los recursos que realmente están implementados en la nube. El estado se almacena en servidores (localmente en el disco o de forma remota en un servicio de almacenamiento de archivos en la nube o en un software de gestión de estado especializado) para lograr una redundancia y confiabilidad óptimas. Puede leer más sobre diferentes backends en la documentación de Terraform.

Los espacios de trabajo del proyecto le permiten tener múltiples estados en el mismo backend, vinculados a la misma configuración. Esto le permite implementar múltiples instancias distintas de la misma infraestructura. Cada proyecto comienza con un espacio de trabajo llamado default; este se usará si no crea o cambia explícitamente a otro.

Los módulos en Terraform (similares a las bibliotecas en otros lenguajes de programación) son contenedores de código parametrizado que contienen múltiples declaraciones de recursos. Le permiten abstraer una parte común de su infraestructura y reutilizarla más tarde con diferentes entradas.

Un proyecto Terraform también puede incluir archivos de código externos para usar con entradas de datos dinámicas, que pueden analizar la salida JSON de un comando CLI y ofrecerla para su uso en declaraciones de recursos. En este tutorial, harás esto con un script de Python.

Ahora que sabes en qué consiste un proyecto Terraform, revisemos dos enfoques generales para la estructuración de proyectos Terraform.

### 3.1 Estructura simple

Una estructura simple es adecuada para proyectos pequeños y de prueba, con algunos recursos de diferentes tipos y variables. Tiene algunos archivos de configuración, generalmente uno por tipo de recurso (o más archivos auxiliares junto con un archivo principal), y no tiene módulos personalizados, porque la mayoría de los recursos son únicos y no hay suficientes para generalizarlos y reutilizarlos. Después de esto, la mayor parte del código se almacena en el mismo directorio, uno al lado del otro. Estos proyectos suelen tener algunas variables (como una clave API para acceder a la nube) y pueden utilizar entradas de datos dinámicas y otras características de Terraform y HCL, aunque no de manera destacada.

Como ejemplo de la estructura de archivos de este enfoque, así es como se verá al final el proyecto que creará en este tutorial:

```
.
└── tf/
    ├── versions.tf
    ├── variables.tf
    ├── provider.tf
    ├── droplets.tf
    ├── dns.tf
    ├── data-sources.tf
    └── external/
        └── name-generator.py
``` 

Dado que este proyecto implementará un Droplet del servidor web Apache y configurará registros DNS, las definiciones de las variables del proyecto, el proveedor de DigitalOcean Terraform, el Droplet y los registros DNS se almacenarán en sus respectivos archivos. Las versiones mínimas requeridas de los proveedores Terraform y DigitalOcean se especificarán en versions.tf, mientras que el script Python que generará un nombre para el Droplet (y se utilizará como fuente de datos dinámica en data-sources.tf) se almacenará en la externalcarpeta, para separarlo del código HCL.

### 3.2 Estructura compleja

Al contrario de la estructura simple, este enfoque es adecuado para proyectos grandes, con estructuras de subdirectorios claramente definidas que contienen múltiples módulos de distintos niveles de complejidad, además del código habitual. Estos módulos pueden depender unos de otros. Junto con los sistemas de control de versiones, estos proyectos pueden hacer un uso extensivo de los espacios de trabajo. Este enfoque es adecuado para proyectos más grandes que administran múltiples aplicaciones y al mismo tiempo reutilizan el código tanto como sea posible.

Las instancias de desarrollo, puesta en escena, control de calidad y infraestructura de producción también se pueden alojar en el mismo proyecto en diferentes directorios basándose en módulos comunes, lo que elimina el código duplicado y convierte al proyecto en la fuente central de verdad. Aquí está la estructura de archivos de un proyecto de ejemplo con una estructura más compleja, que contiene múltiples aplicaciones de implementación, módulos Terraform y entornos de nube de destino:

``` 
.
└── tf/
    ├── modules/
    │   ├── network/
    │   │   ├── main.tf
    │   │   ├── dns.tf
    │   │   ├── outputs.tf
    │   │   └── variables.tf
    │   └── spaces/
    │       ├── main.tf
    │       ├── outputs.tf
    │       └── variables.tf
    └── applications/
        ├── backend-app/
        │   ├── env/
        │   │   ├── dev.tfvars
        │   │   ├── staging.tfvars
        │   │   ├── qa.tfvars
        │   │   └── production.tfvars
        │   └── main.tf
        └── frontend-app/
            ├── env/
            │   ├── dev.tfvars
            │   ├── staging.tfvars
            │   ├── qa.tfvars
            │   └── production.tfvars
            └── main.tf
``` 

Ahora sabe qué es un proyecto Terraform, cómo estructurarlo mejor según la complejidad percibida y qué función cumplen los espacios de trabajo de Terraform. En los siguientes pasos, creará un proyecto con una estructura simple que aprovisionará un Droplet con un servidor web Apache instalado y registros DNS configurados para su dominio. Primero inicializará su proyecto con el proveedor y las variables de DigitalOcean y luego procederá a definir el Droplet, una fuente de datos dinámica para proporcionar su nombre y un registro DNS para la implementación.


## 4. Configurar su proyecto inicial

En esta sección, agregará el proveedor DigitalOcean Terraform a su proyecto, definirá las variables del proyecto y declarará una instancia del proveedor DigitalOcean, para que Terraform pueda conectarse a su cuenta.

Comience creando un directorio para su proyecto Terraform con el siguiente comando:

``` 
mkdir ~/apache-droplet-terraform
``` 

Navegue hasta él:

``` 
cd ~/apache-droplet-terraform
``` 

Dado que este proyecto seguirá el enfoque de estructuración simple, almacenará el proveedor, las variables, el Droplet y el código de registro DNS en archivos separados, según la estructura de archivos de la sección anterior. Primero, deberá agregar el proveedor DigitalOcean Terraform a su proyecto como proveedor obligatorio.

Cree un archivo llamado versions.tf y ábralo para editarlo ejecutando:

``` 
nano versions.tf
``` 

Agregue las siguientes líneas:

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

En este terraform bloque, enumera los proveedores requeridos (DigitalOcean, versión 2.x). Cuando haya terminado, guarde y cierre el archivo.

Luego, defina las variables que su proyecto expondrá en el variables.tf archivo, siguiendo el enfoque de almacenar diferentes tipos de recursos en archivos de código separados:

``` 
nano variables.tf
``` 

Agregue las siguientes variables:

``` 
variable "do_token" {}
variable "domain_name" {}
``` 

Guarde y cierre el archivo.

La do_token variable contendrá su token de acceso personal de DigitalOcean y domain_name especificará el nombre de dominio que desee. El Droplet implementado tendrá la clave SSH, identificada por la huella digital SSH, instalada automáticamente.

A continuación, definamos la instancia del proveedor DigitalOcean para este proyecto. Lo almacenará en un archivo llamado provider.tf. Créelo y ábralo para editarlo ejecutando:

``` 
nano provider.tf
``` 

Agregue el proveedor:

``` 
provider "digitalocean" {
  token = var.do_token
}
``` 

Guarde y salga cuando haya terminado. Ha definido el digital ocean proveedor, que corresponde al proveedor requerido que especificó anteriormente en provider.tf, y ha establecido su token en el valor de la variable, que se proporcionará durante el tiempo de ejecución.

En este paso, creó un directorio para su proyecto, solicitó que el proveedor de DigitalOcean estuviera disponible, declaró las variables del proyecto y configuró la conexión a una instancia del proveedor de DigitalOcean para usar un token de autenticación que se proporcionará más adelante. Ahora escribirá un script que generará datos dinámicos para las definiciones de su proyecto.


## 5. Crear una secuencia de comandos de Python para datos dinámicos

Antes de continuar con la definición del Droplet, creará un script de Python que generará el nombre del Droplet dinámicamente y declarará un recurso de fuente de datos para analizarlo. El nombre se generará concatenando una cadena constante (web) con la hora actual de la máquina local, expresada en el formato de época UNIX. Un script de nomenclatura puede resultar útil cuando se generan varios Droplets según un esquema de nomenclatura, para poder diferenciarlos fácilmente.

Almacenará el script en un archivo llamado name-generator.py, en un directorio llamado external. Primero, cree el directorio ejecutando:

``` 
mkdir external
``` 

El external directorio reside en la raíz de su proyecto y almacenará archivos de código que no sean HCL, como el script de Python que escribirá.

Cree name-generator.py debajo external y ábralo para editarlo:

``` 
nano external/name-generator.py
``` 

Agregue el siguiente código:

``` 
import json, time

fixed_name = "web"
result = {
  "name": f"{fixed_name}-{int(time.time())}",
}

print(json.dumps(result))
``` 

Este script de Python importa los módulos json y time, declara un diccionario llamado result y establece el valor de la name clave en una cadena interpolada, que combina fixed_name con la hora UNIX actual de la máquina en la que se ejecuta. Luego, se result convierte a JSON y se genera en formato stdout. La salida será diferente cada vez que se ejecute el script:

``` 
Output
{"name": "web-1597747959"}
``` 

Cuando haya terminado, guarde y cierre el archivo.

Ahora que el script está listo, puede definir la fuente de datos, que extraerá los datos del script. Almacenará la fuente de datos en un archivo nombrado data-sources.tf en la raíz de su proyecto según el enfoque de estructuración simple.

Créelo para editarlo ejecutando:

``` 
nano data-sources.tf
``` 

Agregue la siguiente definición:

``` 
data "external" "droplet_name" {
  program = ["python3", "${path.module}/external/name-generator.py"]
}
``` 

Guarde y cierre el archivo.

Esta fuente de datos se llama droplet_name y ejecuta el name-generator.py script usando Python 3, que reside en el external directorio que acaba de crear. Analiza automáticamente su salida y proporciona los datos deserializados bajo su result atributo para su uso dentro de otras definiciones de recursos.

Con la fuente de datos ahora declarada, puede definir el Droplet en el que se ejecutará Apache.


## 6. Definir el droplet

En este paso, escribirá la definición del recurso Droplet y lo almacenará en un archivo de código dedicado a Droplets, según el enfoque de estructuración simple. Su nombre provendrá de la fuente de datos dinámica que acaba de crear y será diferente cada vez que se implemente.

Cree y abra el droplets.tf archivo para editarlo:

``` 
nano droplets.tf
``` 

Agregue la siguiente definición de recurso Droplet:

``` 
data "digitalocean_ssh_key" "ssh_key" {
  name = "your_ssh_key_name"
}

resource "digitalocean_droplet" "web" {
  image  = "ubuntu-20-04-x64"
  name   = data.external.droplet_name.result.name
  region = "fra1"
  size   = "s-1vcpu-1gb"
  ssh_keys = [
    data.digitalocean_ssh_key.ssh_key.id
  ]
}
``` 

Primero declara un recurso de clave SSH de DigitalOcean llamado ssh_key, que recuperará una clave de su cuenta por su nombre. Asegúrese de reemplazar el código resaltado con el nombre de su clave SSH.

Luego, declaras un recurso Droplet, llamado web. Su nombre real en la nube será diferente porque se solicita desde la droplet_name fuente de datos externa. Para iniciar el recurso Droplet con una clave SSH cada vez que se implementa, el ID del recurso ssh_key se pasa al ssh_keys parámetro, de modo que DigitalOcean sepa qué clave aplicar.

Por ahora, esto es todo lo que necesita configurar en relación con droplet.tf, así que guarde y cierre el archivo cuando haya terminado.

Ahora escribirá la configuración para el registro DNS que apuntará su dominio al Droplet recién declarado.


## 7. Definición de registros DNS

El último paso del proceso es configurar el registro DNS que apunta al Droplet de su dominio.

Almacenará la configuración de DNS en un archivo llamado dns.tf, porque es un tipo de recurso independiente de los demás que creó en los pasos anteriores. Créelo y ábralo para editarlo:

``` 
nano dns.tf
``` 

Agregue las siguientes líneas:

``` 
resource "digitalocean_record" "www" {
  domain = var.domain_name
  type   = "A"
  name   = "@"
  value  = digitalocean_droplet.web.ipv4_address
}
``` 

Este código declara un registro DNS de DigitalOcean en su nombre de dominio (pasado usando la variable), de tipo A. El registro tiene el nombre de @, que es un marcador de posición que enruta al propio dominio y con la dirección IP del Droplet como value. Puede reemplazar el name valor con otra cosa, lo que dará como resultado la creación de un subdominio.

Cuando haya terminado, guarde y cierre el archivo.

Ahora que ha configurado el Droplet, la fuente de datos del generador de nombres y un registro DNS, pasará a implementar el proyecto en la nube.


## 8. Planificación y aplicación de la configuración

En esta sección, inicializará su proyecto Terraform, lo implementará en la nube y comprobará que todo se haya aprovisionado correctamente.

Ahora que la infraestructura del proyecto está completamente definida, todo lo que queda por hacer antes de implementarla es inicializar el proyecto Terraform. Hágalo ejecutando el siguiente comando:

``` 
terraform init
``` 

Recibirá el siguiente resultado:

``` 
Output
Initializing the backend...

Initializing provider plugins...
- Finding digitalocean/digitalocean versions matching "~> 2.0"...
- Finding latest version of hashicorp/external...
- Installing digitalocean/digitalocean v2.10.1...
- Installed digitalocean/digitalocean v2.10.1 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)
- Installing hashicorp/external v2.1.0...
- Installed hashicorp/external v2.1.0 (signed by HashiCorp)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
``` 

Ahora podrá implementar su Droplet con un nombre generado dinámicamente y un dominio adjunto en su cuenta de DigitalOcean.

Comience definiendo el nombre de dominio, la huella digital de la clave SSH y su token de acceso personal como variables de entorno, de modo que no tenga que copiar los valores cada vez que ejecute Terraform. Ejecute los siguientes comandos, reemplazando los valores resaltados:

``` 
export DO_PAT="your_do_api_token"
export DO_DOMAIN_NAME="your_domain"
``` 

Puede encontrar su token API en su Panel de control de DigitalOcean.

Ejecute el plan comando con los valores de las variables pasados ​​para ver qué pasos seguiría Terraform para implementar su proyecto:

``` 
terraform plan -var "do_token=${DO_PAT}" -var "domain_name=${DO_DOMAIN_NAME}"
``` 

El resultado será similar al siguiente:

``` 
Output
Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-1625908814"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "fra1"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + ssh_keys             = [
          + "...",
        ]
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

  # digitalocean_record.www will be created
  + resource "digitalocean_record" "www" {
      + domain = "your_domain'"
      + fqdn   = (known after apply)
      + id     = (known after apply)
      + name   = "@"
      + ttl    = (known after apply)
      + type   = "A"
      + value  = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
...
``` 

Las líneas que comienzan con verde + significan que Terraform creará cada uno de los recursos que siguen, que es exactamente lo que debería suceder, para que pueda realizar apply la configuración:

``` 
terraform apply -var "do_token=${DO_PAT}" -var "domain_name=${DO_DOMAIN_NAME}"
``` 

El resultado será el mismo que antes, excepto que esta vez se le pedirá que confirme:

```
Output
Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: `yes`
Ingrese yesy Terraform aprovisionará su Droplet y el registro DNS:

Output
digitalocean_droplet.web: Creating...
...
digitalocean_droplet.web: Creation complete after 33s [id=204432105]
digitalocean_record.www: Creating...
digitalocean_record.www: Creation complete after 1s [id=110657456]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
``` 

Terraform ahora ha registrado los recursos desplegados en su estado. Para confirmar que los registros DNS y el Droplet se conectaron correctamente, puede extraer la dirección IP del Droplet del estado local y verificar si coincide con los registros DNS públicos de su dominio. Ejecute el siguiente comando para obtener la dirección IP:

``` 
terraform show | grep "ipv4"
``` 

Recibirás la dirección IP de tu Droplet:

``` 
Output
ipv4_address       = "your_Droplet_IP"
...
``` 

Puede verificar los registros A públicos ejecutando:

``` 
nslookup -type=a your_domain | grep "Address" | tail -1
``` 

El resultado mostrará la dirección IP a la que apunta el registro A:

``` 
Output
Address: your_Droplet_IP
``` 
Son iguales, como deberían ser, lo que significa que el registro Droplet y DNS se aprovisionaron correctamente.

Para que se realicen los cambios en el siguiente paso, destruya los recursos implementados ejecutando:

``` 
terraform destroy -var "do_token=${DO_PAT}" -var "domain_name=${DO_DOMAIN_NAME}"
```

Cuando se le solicite, ingrese yes para continuar.

En este paso, creó su infraestructura y la aplicó a su cuenta de DigitalOcean. Ahora lo modificará para instalar automáticamente el servidor web Apache en el Droplet aprovisionado utilizando los aprovisionadores de Terraform.


## 9. Ejecutar código usando aprovisionadores

Ahora configurará la instalación del servidor web Apache en su Droplet implementado utilizando el remote-exec aprovisionador para ejecutar comandos personalizados.

Los aprovisionadores de Terraform se pueden utilizar para ejecutar acciones específicas en recursos remotos creados (el remote-exec  aprovisionador) o en la máquina local en la que se ejecuta el código (usando el local-exec aprovisionador). Si falla un aprovisionador, el nodo se marcará como contaminado en el estado actual, lo que significa que se eliminará y se volverá a crear durante la siguiente ejecución.

Para conectarse a un Droplet aprovisionado, Terraform necesita la clave SSH privada de la configurada en el Droplet. La mejor manera de pasar la ubicación de la clave privada es mediante el uso de variables, así que ábralas variables.tf para editarlas:

``` 
nano variables.tf
```

Agregue la línea resaltada:

``` 
variable "do_token" {}
variable "domain_name" {}
variable "private_key" {}
``` 

Ahora ha agregado una nueva variable, llamada private_key, a su proyecto. Guarde y cierre el archivo.

A continuación, agregará los datos de conexión y las declaraciones del aprovisionador remoto a su configuración de Droplet. Ábralo droplets.tf para editar ejecutando:

``` 
nano droplets.tf
``` 

Amplíe el código existente con las líneas resaltadas:

``` 
data "digitalocean_ssh_key" "ssh_key" {
  name = "your_ssh_key_name"
}

resource "digitalocean_droplet" "web" {
  image  = "ubuntu-20-04-x64"
  name   = data.external.droplet_name.result.name
  region = "fra1"
  size   = "s-1vcpu-1gb"
  ssh_keys = [
    data.digitalocean_ssh_key.ssh_key.id
  ]

  connection {
    host        = self.ipv4_address
    user        = "root"
    type        = "ssh"
    private_key = file(var.private_key)
    timeout     = "2m"
  }

  provisioner "remote-exec" {
    inline = [
      "export PATH=$PATH:/usr/bin",
      # Install Apache
      "apt update",
      "apt -y install apache2"
    ]
  }
}
``` 

El connection bloque especifica cómo Terraform debe conectarse al Droplet de destino. El provisioner bloque contiene la matriz de comandos, dentro del inline parámetro, que ejecutará después del aprovisionamiento. Es decir, actualizar la caché del administrador de paquetes e instalar Apache. Guarde y salga cuando haya terminado.

También puede crear una variable de entorno temporal para la ruta de la clave privada:

``` 
export DO_PRIVATE_KEY="private_key_location"
``` 

Intente aplicar la configuración nuevamente:

``` 
terraform apply -var "do_token=${DO_PAT}" -var "domain_name=${DO_DOMAIN_NAME}" -var "private_key=${DO_PRIVATE_KEY}"
```

Ingrese yes cuando se le solicite. Recibirá un resultado similar al anterior, pero seguido de un resultado largo del remote-exec aprovisionador:

``` 
Output
digitalocean_droplet.web: Creating...
digitalocean_droplet.web: Still creating... [10s elapsed]
digitalocean_droplet.web: Still creating... [20s elapsed]
digitalocean_droplet.web: Still creating... [30s elapsed]
digitalocean_droplet.web: Provisioning with 'remote-exec'...
digitalocean_droplet.web (remote-exec): Connecting to remote host via SSH...
digitalocean_droplet.web (remote-exec):   Host: ...
digitalocean_droplet.web (remote-exec):   User: root
digitalocean_droplet.web (remote-exec):   Password: false
digitalocean_droplet.web (remote-exec):   Private key: true
digitalocean_droplet.web (remote-exec):   Certificate: false
digitalocean_droplet.web (remote-exec):   SSH Agent: false
digitalocean_droplet.web (remote-exec):   Checking Host Key: false
digitalocean_droplet.web (remote-exec): Connected!
...
digitalocean_droplet.web: Creation complete after 1m5s [id=204442200]
digitalocean_record.www: Creating...
digitalocean_record.www: Creation complete after 1s [id=110666268]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
``` 

Ahora puede navegar a su dominio en un navegador web. Verá la página de bienvenida predeterminada de Apache.

Esto significa que Apache se instaló correctamente y que Terraform aprovisionó todo correctamente.

Para destruir los recursos implementados, ejecute el siguiente comando e ingrese yes cuando se le solicite:

``` 
terraform destroy -var "do_token=${DO_PAT}" -var "domain_name=${DO_DOMAIN_NAME}" -var "private_key=${DO_PRIVATE_KEY}"
```

Ahora ha completado un pequeño proyecto Terraform con una estructura simple que implementa el servidor web Apache en un Droplet y configura registros DNS para el dominio deseado.


## 10. Conclusión

Ha aprendido acerca de dos enfoques generales para estructurar sus proyectos Terraform, según su complejidad. Siguiendo el enfoque de estructuración simple y utilizando el remote-exec aprovisionador para ejecutar comandos, implementó un Droplet que ejecuta Apache con registros DNS para su dominio.

Como referencia, aquí está la estructura de archivos del proyecto que creó en este tutorial:

``` 
.
└── tf/
    ├── versions.tf
    ├── variables.tf
    ├── provider.tf
    ├── droplets.tf
    ├── dns.tf
    ├── data-sources.tf
    └── external/
        └── name-generator.py
``` 

Los recursos que definió (el Droplet, el registro DNS y la fuente de datos dinámicos, el proveedor y las variables de DigitalOcean) se almacenan cada uno en su propio archivo separado, de acuerdo con la estructura simple del proyecto descrita en la primera sección de este tutorial.



