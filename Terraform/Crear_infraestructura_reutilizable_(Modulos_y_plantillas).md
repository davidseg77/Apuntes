# Cómo crear infraestructura reutilizable con módulos y plantillas de Terraform

## 1. Introducción

Uno de los principales beneficios de Infraestructura como Código (IAC) es la reutilización de partes de la infraestructura definida. En Terraform, puede utilizar módulos para encapsular componentes conectados lógicamente en una entidad y personalizarlos utilizando las variables de entrada que usted defina. Al utilizar módulos para definir su infraestructura en un alto nivel, puede separar los entornos de desarrollo, ensayo y producción pasando solo valores diferentes a los mismos módulos, lo que minimiza la duplicación de código y maximiza la concisión.

No está limitado a utilizar únicamente sus módulos personalizados. Terraform Registry está integrado en Terraform y enumera módulos y proveedores que puede incorporar en su proyecto de inmediato definiéndolos en la required_providers sección. Hacer referencia a módulos públicos puede acelerar su flujo de trabajo y reducir la duplicación de código. Si tiene un módulo útil y desea compartirlo con el mundo, puede considerar publicarlo en el Registro para que lo utilicen otros desarrolladores.

En este tutorial, explorará algunas de las formas de definir y reutilizar código en proyectos de Terraform. Hará referencia a módulos del Registro Terraform, separará entornos de desarrollo y producción utilizando módulos, aprenderá sobre las plantillas y cómo se usan, y especificará dependencias de recursos explícitamente utilizando el metaargumento depends_on.

## 2. Requisitos previos

* Terraform instalado en su máquina local.
* El droplet-lb módulo disponible modules en terraform-reusability. 
* Conocimiento de los enfoques de estructuración de proyectos Terraform. Para obtener más información, consulte nuestro tutorial Cómo estructurar un proyecto Terraform.


## 3. Separación de los entornos de desarrollo y producción

En esta sección, utilizará módulos para separar los entornos de implementación de destino. Los organizará según la estructura de un proyecto más complejo. Creará un proyecto con dos módulos: uno definirá los Droplets y los balanceadores de carga, y el otro configurará los registros de dominio DNS. Luego, escribirá la configuración para dos entornos diferentes (dev y prod), que llamarán a los mismos módulos.

### 3.1 Creando el dns-records módulo

Como parte de los requisitos previos, configuró el proyecto inicial terraform-reusability y creó el droplet-lb módulo en su propio subdirectorio en modules. Ahora configurará el segundo módulo, llamado dns-records, que contiene variables, resultados y definiciones de recursos. Desde el terraform-reusability directorio, cree dns-record sejecutando:

``` 
mkdir modules/dns-records
``` 

Navegue hasta él:

``` 
cd modules/dns-records
``` 

Este módulo contendrá las definiciones para su dominio y los registros DNS que luego señalará a los balanceadores de carga. Primero definirá las variables, que se convertirán en entradas que expondrá este módulo. Los almacenará en un archivo llamado variables.tf. Créelo para editarlo:

``` 
nano variables.tf
```

Agregue las siguientes definiciones de variables:

``` 
variable "domain_name" {}
variable "ipv4_address" {}
``` 

Guarde y cierre el archivo. Ahora definirá el dominio y los registros adjuntos A en CNAME un archivo llamado records.tf. Créelo y ábralo para editarlo ejecutando:

``` 
nano records.tf
``` 

Agregue las siguientes definiciones de recursos:

``` 
resource "digitalocean_domain" "domain" {
  name = var.domain_name
}

resource "digitalocean_record" "domain_A" {
  domain = digitalocean_domain.domain.name
  type   = "A"
  name   = "@"
  value  = var.ipv4_address
}

resource "digitalocean_record" "domain_CNAME" {
  domain = digitalocean_domain.domain.name
  type   = "CNAME"
  name   = "www"
  value  = "@"
}
``` 

Primero, agrega el nombre de dominio a su cuenta de DigitalOcean. La nube agregará automáticamente los tres servidores de nombres de DigitalOcean como NS registros. El nombre de dominio que proporcione a Terraform no debe estar presente en su cuenta de DigitalOcean, o Terraform mostrará un error durante la creación de la infraestructura.

Luego, define un A registro para su dominio y lo enruta (el @ as value significa el verdadero nombre de dominio, sin subdominios) a la dirección IP proporcionada como variable ipv4_address. La dirección IP real se pasará cuando inicialice una instancia del módulo. En aras de la exhaustividad, el CNAME registro que sigue especifica que el www subdominio también debe apuntar al mismo dominio. Guarde y cierre el archivo cuando haya terminado.

A continuación, definirá las salidas para este módulo. Los resultados mostrarán el FQDN (nombre de dominio completo) de los registros creados. Crear y abrir outputs.tf para editar:

```
nano outputs.tf
``` 

Agregue las siguientes líneas:

``` 
output "A_fqdn" {
  value = digitalocean_record.domain_A.fqdn
}

output "CNAME_fqdn" {
  value = digitalocean_record.domain_CNAME.fqdn
}
``` 

Guarde y cierre el archivo cuando haya terminado.

Una vez definidas las variables, los registros DNS y las salidas, lo último que deberá especificar son los requisitos del proveedor para este módulo. Especificarás que el dns-records módulo requiere el digitalocean proveedor en un archivo llamado provider.tf. Créelo y ábralo para editarlo:

``` 
nano provider.tf
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

Cuando haya terminado, guarde y cierre el archivo. Ahora que digital ocean se ha definido el proveedor, el dns-records módulo está funcionalmente completo.

### 3.2 Creando diferentes ambientes

La estructura actual del terraform-reusability proyecto será similar a esta:

``` 
terraform-reusability/
├─ modules/
│  ├─ dns-records/
│  │  ├─ outputs.tf
│  │  ├─ provider.tf
│  │  ├─ records.tf
│  │  ├─ variables.tf
│  ├─ droplet-lb/
│  │  ├─ droplets.tf
│  │  ├─ lb.tf
│  │  ├─ outputs.tf
│  │  ├─ provider.tf
│  │  ├─ variables.tf
├─ provider.tf
``` 

Hasta ahora, tiene dos módulos en su proyecto: el que acaba de crear ( dns-records) y el que creó como parte de los requisitos previos ( droplet-lb).

Para facilitar diferentes entornos, almacenará los archivos de configuración del entorno dev y prod en un directorio llamado environments, que residirá en la raíz del proyecto. Ambos entornos llamarán a los mismos dos módulos, pero con diferentes valores de parámetros. La ventaja de esto es que cuando los módulos cambien internamente en el futuro, solo necesitará actualizar los valores que está pasando.

Primero, navegue hasta la raíz del proyecto ejecutando:

``` 
cd ../..
```

Luego, cree los directorios dev y al mismo tiempo:prod environments

``` 
mkdir -p environments/dev && mkdir environments/prod
``` 

El -p argumento ordena mkdir crear todos los directorios en la ruta dada.

Navegue hasta el dev directorio, ya que primero configurará ese entorno:

``` 
cd environments/dev
``` 

Almacenarás el código en un archivo llamado main.tf, así que créalo para editarlo:

``` 
nano main.tf
``` 

Agregue las siguientes líneas:


``` 
module "droplets" {
  source   = "../../modules/droplet-lb"

  droplet_count = 2
  group_name    = "dev"
}

module "dns" {
  source   = "../../modules/dns-records"

  domain_name   = "your_dev_domain"
  ipv4_address  = module.droplets.lb_ip
}
``` 

Aquí usted llama y configura los dos módulos droplet-lb y dns-records, que juntos darán como resultado la creación de dos Droplets. Están encabezados por un equilibrador de carga y los registros DNS para el dominio proporcionado están configurados para apuntar a ese equilibrador de carga. Recuerde reemplazarlo your_dev_domain con el nombre de dominio que desee para el dev entorno, luego guarde y cierre el archivo.

A continuación, configurará el proveedor de DigitalOcean y creará una variable para que pueda aceptar el token de acceso personal que creó como parte de los requisitos previos. Abra un nuevo archivo, llamado provider.tf, para editarlo:

``` 
nano provider.tf
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

variable "do_token" {}

provider "digitalocean" {
  token = var.do_token
}
``` 

En este código, necesita que el digital ocean proveedor esté disponible y pase la do_token variable a su instancia. Guarde y cierre el archivo.

Inicialice la configuración ejecutando:

``` 
terraform init
``` 

Recibirá el siguiente resultado:

``` 
Output
Initializing modules...
- dns in ../../modules/dns-records
- droplets in ../../modules/droplet-lb

Initializing the backend...

Initializing provider plugins...
- Finding digitalocean/digitalocean versions matching "~> 2.0"...
- Installing digitalocean/digitalocean v2.10.1...
- Installed digitalocean/digitalocean v2.10.1 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)

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

La configuración del prod entorno es similar. Navegue a su directorio ejecutando:

``` 
cd ../prod
``` 

Crear y abrir main.tf para editar:

``` 
nano main.tf
``` 

Agregue las siguientes líneas:

``` 
module "droplets" {
  source   = "../../modules/droplet-lb"

  droplet_count = 5
  group_name    = "prod"
}

module "dns" {
  source   = "../../modules/dns-records"

  domain_name   = "your_prod_domain"
  ipv4_address  = module.droplets.lb_ip
}
``` 

La diferencia entre esto y su dev código es que se implementarán cinco Droplets. Además, el nombre de dominio, que deberás reemplazar con tu prod nombre de dominio, será diferente. Guarde y cierre el archivo cuando haya terminado.

Luego, copie la configuración del proveedor de dev:

``` 
cp ../dev/provider.tf .
``` 

Inicialice esta configuración también:

``` 
terraform init
``` 

El resultado de este comando será el mismo que la vez anterior que lo ejecutó.

Puede intentar planificar la configuración para ver qué recursos crearía Terraform ejecutando:

``` 
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado prod será el siguiente:

``` 
Output
...
Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.dns.digitalocean_domain.domain will be created
  + resource "digitalocean_domain" "domain" {
      + id   = (known after apply)
      + name = "your_prod_domain"
      + urn  = (known after apply)
    }

  # module.dns.digitalocean_record.domain_A will be created
  + resource "digitalocean_record" "domain_A" {
      + domain = "your_prod_domain"
      + fqdn   = (known after apply)
      + id     = (known after apply)
      + name   = "@"
      + ttl    = (known after apply)
      + type   = "A"
      + value  = (known after apply)
    }

  # module.dns.digitalocean_record.domain_CNAME will be created
  + resource "digitalocean_record" "domain_CNAME" {
      + domain = "your_prod_domain"
      + fqdn   = (known after apply)
      + id     = (known after apply)
      + name   = "www"
      + ttl    = (known after apply)
      + type   = "CNAME"
      + value  = "@"
    }

  # module.droplets.digitalocean_droplet.droplets[0] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "prod-0"
...
    }

  # module.droplets.digitalocean_droplet.droplets[1] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "prod-1"
...
    }

  # module.droplets.digitalocean_droplet.droplets[2] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "prod-2"
...
    }

  # module.droplets.digitalocean_droplet.droplets[3] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "prod-3"
...
    }

  # module.droplets.digitalocean_droplet.droplets[4] will be created
  + resource "digitalocean_droplet" "droplets" {
...
      + name                 = "prod-4"
...
    }

  # module.droplets.digitalocean_loadbalancer.www-lb will be created
  + resource "digitalocean_loadbalancer" "www-lb" {
...
      + name                     = "lb-prod"
...

Plan: 9 to add, 0 to change, 0 to destroy.
...
``` 

Esto implementaría cinco Droplets con un Load Balancer. También crearía el prod dominio que especificó con los dos registros DNS apuntando al equilibrador de carga. También puede intentar planificar la configuración del dev entorno; observará que se planificaría la implementación de dos Droplets.

**Nota:** Puede aplicar esta configuración para los entornos devy prodcon el siguiente comando:

``` 
terraform apply -var "do_token=${DO_PAT}"
``` 

Para destruirlo, ejecute el siguiente comando e ingrese yescuando se le solicite:

``` 
terraform destroy -var "do_token=${DO_PAT}"
``` 

A continuación se muestra cómo ha estructurado este proyecto:

``` 
terraform-reusability/
├─ environments/
│  ├─ dev/
│  │  ├─ main.tf
│  │  ├─ provider.tf
│  ├─ prod/
│  │  ├─ main.tf
│  │  ├─ provider.tf
├─ modules/
│  ├─ dns-records/
│  │  ├─ outputs.tf
│  │  ├─ provider.tf
│  │  ├─ records.tf
│  │  ├─ variables.tf
│  ├─ droplet-lb/
│  │  ├─ droplets.tf
│  │  ├─ lb.tf
│  │  ├─ outputs.tf
│  │  ├─ provider.tf
│  │  ├─ variables.tf
├─ provider.tf
``` 

La adición es el environments directorio, que contiene el código para los entornos dev y prod.

El beneficio de este enfoque es que los cambios adicionales en los módulos se propagan automáticamente a todas las áreas de su proyecto. Salvo posibles personalizaciones de las entradas del módulo, este enfoque no es repetitivo y promueve la reutilización tanto como sea posible, incluso en entornos de implementación. En general, esto reduce el desorden y le permite rastrear las modificaciones utilizando un sistema de control de versiones.

En las dos últimas secciones de este tutorial, revisará el depends_on metaargumento y la templatefile función.

## 4. Declarar dependencias para construir infraestructura en orden

Mientras planifica acciones, Terraform intenta automáticamente identificar las dependencias existentes y las integra en su gráfico de dependencia. Las principales dependencias que puede detectar son referencias claras; por ejemplo, cuando un valor de salida de un módulo se pasa a un parámetro de otro recurso. En este escenario, el módulo primero debe completar su implementación para proporcionar el valor de salida.

Las dependencias que Terraform no puede detectar están ocultas: tienen efectos secundarios y referencias mutuas que no se pueden deducir del código. Un ejemplo de esto es cuando un objeto depende no de la existencia, sino del comportamiento de otro, y no accede a sus atributos desde el código. Para superar esto, puede utilizar depends_on para especificar manualmente las dependencias de forma explícita. Desde Terraform 0.13, también puede utilizar depends_on módulos on para forzar que los recursos enumerados se implementen por completo antes de implementar el módulo en sí. Es posible utilizar el depends_on metaargumento con cada tipo de recurso. depends_on también aceptará una lista de otros recursos de los que depende su recurso especificado.

depends_on acepta una lista de referencias a otros recursos. Su sintaxis se ve así:

``` 
resource "resource_type" "res" {
  depends_on = [...] # List of resources

  # Parameters...
}
``` 

Recuerda que sólo debes utilizarlo depends_on como último recurso. Si se utiliza, se debe mantener bien documentado, porque el comportamiento del que dependen los recursos puede no ser inmediatamente obvio.

En el paso anterior de este tutorial, no especificó ninguna dependencia explícita usando depends_on, porque los recursos que creó no tienen efectos secundarios que no se puedan deducir del código. Terraform puede detectar las referencias realizadas a partir del código que ha escrito y programará la implementación de los recursos en consecuencia.

## 5. Uso de plantillas para personalización

En Terraform, la creación de plantillas consiste en sustituir resultados de expresiones en lugares apropiados, como cuando se establecen valores de atributos en recursos o se construyen cadenas. Lo ha utilizado en los pasos anteriores y en los requisitos previos del tutorial para generar dinámicamente nombres de droplets y otros valores de parámetros.

Al sustituir valores en cadenas, los valores se especifican y están rodeados por ${}. La sustitución de plantillas se utiliza a menudo en bucles para facilitar la personalización de los recursos creados. También permite la personalización del módulo sustituyendo entradas en los atributos de recursos.

Terraform ofrece la templatefile función, que acepta dos argumentos: el archivo del disco a leer y un mapa de variables emparejadas con sus valores. El valor que devuelve es el contenido del archivo representado con la expresión sustituida, tal como lo haría normalmente Terraform al planificar o aplicar el proyecto. Debido a que las funciones no forman parte del gráfico de dependencia, el archivo no se puede generar dinámicamente desde otra parte del proyecto.

Imagine que el contenido del archivo de plantilla llamado droplets.tmpl es el siguiente:

``` 
%{ for address in addresses ~}
${address}:80
%{ endfor ~}
``` 

Las declaraciones más largas deben estar rodeadas por %{}, como es el caso de las declaraciones for y endfor, que indican el inicio y el final del for ciclo respectivamente. El contenido y el tipo de la droplets variable no se conocen hasta que se llama a la función y se proporcionan los valores reales, así:

``` 
templatefile("${path.module}/droplets.tmpl", { addresses = ["192.168.0.1", "192.168.1.1"] })
``` 

Esta templatefile llamada devolverá el siguiente valor:

``` 
Output
192.168.0.1:80
192.168.1.1:80
``` 

Esta función tiene sus casos de uso, pero son poco comunes. Por ejemplo, podrías usarlo cuando parte de la configuración debe existir en un formato propietario, pero depende del resto de los valores y debe generarse dinámicamente. En la mayoría de los casos, es mejor especificar todos los parámetros de configuración directamente en el código Terraform, siempre que sea posible.


## 6. Conclusión

En este artículo, ha maximizado la reutilización de código en un proyecto Terraform de ejemplo. La forma principal es empaquetar funciones y configuraciones de uso frecuente como un módulo personalizable y utilizarlo cuando sea necesario. Al hacerlo, no duplica el código subyacente (que puede ser propenso a errores) y permite tiempos de respuesta más rápidos, ya que modificar el módulo es casi todo lo que necesita hacer para introducir cambios.

No estás limitado a tus propios módulos. Como ha visto, Terraform Registry proporciona módulos y proveedores de terceros que puede incorporar a su proyecto.

