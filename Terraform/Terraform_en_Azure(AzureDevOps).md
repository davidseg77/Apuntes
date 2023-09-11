# Implementación de Terraform en Azure (DevOps Pipelines)


## 1. Introducción

En este proyecto se abordará el uso de Terraform conjuntamente con Azure y Azure DevOps. Se explicarán los conceptos teóricos fundamentales de cada una de estas herramientas y se realizarán pruebas prácticas para demostrar cómo se pueden utilizar juntas para crear y administrar recursos de infraestructura en la nube de Microsoft de forma automatizada.

También se explicará cómo se pueden definir recursos de Azure mediante código Terraform y cómo se pueden utilizar pipelines de Azure DevOps para implementar y administrar estos recursos de forma automatizada. Se realizarán pruebas prácticas para demostrar cómo se pueden crear y administrar recursos de Azure utilizando Terraform y Azure DevOps, lo que permitirá a los equipos de desarrollo y operaciones mejorar la eficiencia, la calidad y la seguridad de sus procesos.


## 2. Providers de Terraform para la infraestructura de Azure

• **AzureRM:** Administra recursos y funcionalidades estables de Azure, como máquinas virtuales, cuentas de almacenamiento e interfaces de red.

• **AzureAD:** Administra recursos de Azure Active Directory, como grupos, usuarios, entidades de servicio y aplicaciones.

• **AzureDevops:** Administra recursos de Azure DevOps, como agentes, repositorios, proyectos, canalizaciones y consultas.

• **AzAPI:** Permite administrar directamente los recursos y funcionalidades de Azure através de sus API de Azure Resource Manager. Esta herramienta es complementaria al proveedor de AzureRM y te permite gestionar recursos de Azure que no están disponibles públicamente.

• **Azure Stack:** Administra recursos de Azure Stack, como máquinas virtuales, DNS, red virtual y almacenamiento.


## 3. Sintaxis de Terraform

La sintaxis de Terraform se basa en un lenguaje específico de dominio llamado HCL (HashiCorp Configuration Language), que se utiliza para describir la configuración de la infraestructura como código (IaC).
La sintaxis de Terraform puede variar ligeramente dependiendo del proveedor de infraestructura en la nube con el que estés trabajando. Esto se debe a que cada proveedor de la nube tiene su propio conjunto de recursos y configuraciones específicas que se pueden crear y gestionar mediante Terraform. En este caso, mostraremos la
sintaxis que utilizan algunos de los elementos con el proveedor de Azure Cloud.

1. **Bloques:** Los bloques son la unidad básica de configuración en Terraform y se definen con llaves {}. Los bloques se utilizan para agrupar y configurar recursos o módulos en Terraform.

En Terraform se utilizan dos parámetros entre comillas a la hora de definir un recurso para especificar la "tipo" y el "nombre" del recurso. Por ejemplo: 

``` 
resource "azurerm_virtual_network" "ejemplo" {

# Configuración del recurso de red virtual de Azure
...
}
``` 

2. **Argumentos:** Los argumentos se definen dentro de los bloques y se utilizan para configurar los recursos. Se especifican como pares de clave = valor y se pueden utilizar para definir las propiedades y configuraciones del recurso. Por ejemplo:

``` 
resource "azurerm_virtual_network" "ejemplo" {

  name = "my-virtual-network"
  location = "West Europe"
  address_space = ["10.0.0.0/16"]
...
}
``` 

3. **Variables:** Las variables se utilizan para parametrizar la configuración en Terraform y se definen en un bloque variable. Las variables se utilizan para definir valores reutilizables que pueden ser pasados como parámetros a módulos o recursos. Por ejemplo:

``` 
variable "vm_size" {

  type = string
  default = "Standard_DS2_v2"
}
```

En este caso, el size hace referencia a la etiqueta predefinida por Azure que engloba el conjunto de características del que dispondrá la máquina virtual (Similar a un sabor o flavor en OpenStack).
Como vemos en el ejemplo anterior, dentro de la variable “vm_size” definimos varios campos, el campo “type” indicará que es una variable de tipo cadena. Y en el campo “default” seleccionamos la etiqueta del size que vamos a asignar a dicha variable.

4. **Expresiones:** Terraform permite el uso de expresiones para realizar cálculos y manipulaciones en la configuración. Las expresiones se utilizan para definir los valores de los argumentos o las variables. Por ejemplo:

``` 
resource "azurerm_virtual_machine" "ejemplo" {

  ...
  vm_size = "${var.vm_size}" # Uso de una variable en la configuración del recurso
  ...
}
``` 

5. **Referencias:** Las referencias se utilizan para hacer referencia a otros recursos o variables en la configuración de Terraform. Se utilizan para establecer dependencias entre recursos o para obtener valores de otros recursos o variables. Por ejemplo:

```
resource "azurerm_virtual_network" "ejemplo" {

  ...
}

resource "azurerm_subnet" "ejemplo" {
  virtual_network_name = azurerm_virtual_network.example.name # Referencia a otro recurso
  ...
}
``` 

Estos son algunos de los elementos básicos de la sintaxis de Terraform en HCL.


## 4. Casos prácticos


### 4.1 Configuración del entorno de desarrollo con Terraform y Azure.

#### 4.1.1 Máquina Windows

Para realizar la instalación de Terraform sincronización con una cuenta de Azure en una máquina Windows seguiremos los siguientes pasos:

* **Paso 1** (Descargar Terraform)
  
https://www.terraform.io/

* **Paso 2** (Añadir el ejecutable de Terraform al Path de Windows)
  
Descomprimimos el fichero descargado anteriormente (.zip) que contiene el binario de Terraform y lo movemos al path de Windows "C:\Windows\System32".

* **Paso 3** (Descargar e instalar Azure CLI)

La interfaz de línea de comandos (CLI) de Azure es una herramienta de línea de comandos multiplataforma para conectarse a Azure y ejecutar comandos administrativos en los recursos de Azure. Permite la ejecución de comandos a través de un terminal utilizando indicaciones de línea de comandos interactivas o un script.
Descargamos, en este caso el ejecutable .msi.

https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli

Una vez descargado, lo instalamos.

Para comprobar la versión que se ha instalado, abrimos una Powershell y ejecutamos el siguiente comando:

``` 
az version
``` 

El comando anterior nos devolverá la siguiente salida:

``` 
PS> az version
{
  "azure-cli": "2.47.0",
  "azure-cli-core": "2.47.0",
  "azure-cli-telemetry": "1.0.8",
  "extensions": {}
}
``` 

**Paso 4** (Instalamos un IDE para editar el código con el que vamos a trabajar)

En este caso, recomiendo usar Visual Studio Code y añadiendo las extensiones "Azure Account", "Terraform" y "Azure Terraform".

**Paso 5** (Nos logueamos desde la consola)

Abrimos la Powershell y ejecutamos el siguiente comando:

``` 
az login
``` 

Una vez ejecutemos el comando anterior, se abrirá una ventana en el navegador, donde requerirá una autenticación con nuestra cuenta de Microsoft Azure.

Una vez nos logueemos se mostrará el siguiente mensaje:

``` 
You have logged into Microsoft Azure!
```

En la terminal, se mostrará como salida un JSON similar al siguiente:

``` 
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "a0ce8a08-6fc1-4e4e-b707-b9d558a97fc2",
    "id": "d26cf210-db93-4db7-9cea-d9ddce1cd55d",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Azure subscription 1",
    "state": "Enabled",
    ...
   }
  }
]
``` 

Con esto ya podemos tendremos sincronizado Terraform con nuestra cuenta de Azure, y podremos comenzar a trabajar con una plantilla de terraform “main.tf”. Como vemos se ha asociará el ID de la correspondiente suscripción de Azure.

En caso de que queramos seleccionar una suscripción distinta (Porque dispongamos de varias), empleamos el siguiente comando:

``` 
az account set --subscription "<SUBSCRIPTION-ID>"
``` 

Además del “id” de la suscripción, podemos encontrar el “tenantId”. Un tenant de Azure es una instancia aislada y segura de la nube de Azure en la que una organización asociada puede crear, configurar y gestionar sus recursos y servicios en la nube.

Cuando una organización se registra para utilizar los servicios de Azure, se crea automáticamente un tenant de Azure para esa organización. Dentro de un tenant de Azure, una organización puede crear y gestionar una amplia gama de recursos, como máquinas virtuales, bases de datos, redes, almacenamiento y servicios de aplicaciones, entre otros. También se pueden configurar políticas de seguridad, gestionar el acceso a los recursos y establecer la facturación y la suscripción a los servicios.
En el panel de Azure Cloud, podemos encontrar el TenantId desde **“Azure Active Directory/Default Directory/Propiedades”.** (ID del inquilino)


#### 4.1.2 Máquina Linux

**Paso 1** (Descargar e instalar Terraform)

En este caso, se ha realizado una prueba de instalación sobre una máquina Ubuntu 22.04.2 LTS.
Podemos descargar el binario desde la página oficial. O realizar la instalación mediante los siguientes comandos:

``` 
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o
/usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg]
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee
/etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
``` 

Comprobamos la versión con el siguiente comando:

``` 
$ terraform --version
Terraform v1.4.5
on linux_amd64
``` 

**Paso 2** (Descargar e instalar Azure CLI)

Podemos instalarlo utilizando únicamente el siguiente comando:

``` 
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
``` 

Una vez instalado comprobamos la versión mediante el comando:

``` 
$ az version
{
  "azure-cli": "2.47.0",
  "azure-cli-core": "2.47.0",
  "azure-cli-telemetry": "1.0.8",
  "extensions": {}
}
``` 

**Paso 3** (Instalamos un IDE para editar el código con el que vamos a trabajar)

En este caso, realizaremos la instalación de “Visual Studio Code” añadiendo las extensiones "Terraform" y "Azure Terraform".

Descargamos la versión para Linux (Ubuntu) y posteriormente la instalamos mediante el siguiente comando:

``` 
dpkg -i code_1.77.3-1681292746_amd64.deb
``` 

**Paso 4** (Nos logueamos desde la consola)

``` 
az login
``` 

Después de ejecutar el comando anterior, se abrirá una ventana en el navegador, donde requerirá una autenticación con nuestra cuenta de Microsoft Azure.

Una vez nos logueemos se mostrará el siguiente mensaje:

``` 
You have logged into Microsoft Azure!
```

En la terminal, se mostrará como salida un JSON similar al siguiente:

``` 
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "a0ce8a08-6fc1-4e4e-b707-b9d558a97fc2",
    "id": "d26cf210-db93-4db7-9cea-d9ddce1cd55d",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Azure subscription 1",
    "state": "Enabled",
    ...
   }
  }
]
``` 

Una vez hecho esto, ya tendríamos sincronizado terraform con nuestra suscripción de azure.
Para habilitar el autocompletado de Terraform, debemos de instalar el siguiente paquete:

``` 
terraform -install-autocomplete
``` 


### 4.2 Ejemplo de despliegue de infraestructura

Vamos a realizar una prueba práctica de un despliegue de recursos en Azure mediante Terraform. Para ello, utilizaré una máquina Windows que hemos configurado previamente.

El Tenant ID es un número único que identifica a la organización o empresa que utiliza los servicios de Azure. Cada organización o empresa que utiliza los servicios de Azure tiene un Tenant ID, que se utiliza para autenticar y autorizar el acceso a los recursos de Azure. Primero realizaré un login desde la powershell seleccionando el tenantid correspondiente:

``` 
az login --tenant a0ce8a08-6fc1-4e4e-b707-b9d558a97fc2
``` 

En caso necesario seleccionamos el id de nuestra suscripción:

``` 
az account set --subscription d26cf210-db93-4db7-9cea-d9ddce1cd55d
``` 

En este caso realizaremos un ejemplo desplegando un servidor web mediante el uso de una máquina virtual y la infraestructura necesaria. El servidor devolverá "Hello, World" para la URL que escucha en el puerto 8080.
En este caso, partiremos de la siguiente plantilla de terraform (main.tf).

``` 
# Crear backend de Terraform. 
# El fichero de configuración tfstate almacenará los metadatos para un entorno que se creará, actualizará y modificará. 
# Si no existe tfstate, Terraform intentará recrear toda la configuración en lugar de actualizarla.

terraform {
  backend "azurerm" {
    resource_group_name = "dev1"
    storage_account_name = "storageiesgn"
    container_name = "contenedoriesgn1"
    key = "terraform.tfstate"
    }
}

# Seleccionar el Azure Provider y la versión que usará
terraform {
  required_version = ">= 0.14"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.1.0"
    }
  }
}

# Configurar Microsoft Azure provider
provider "azurerm" {
  features {}
}

# Crear grupo de recursos en caso de que no exista
resource "azurerm_resource_group" "az_rg" {
  name     = var.ResourceGroupName
  location = var.ResourceGroupLocation
}

# Crear Virtual Network
resource "azurerm_virtual_network" "az_vnet" {
  name                = var.AzurermVirtualNetworkName
  location            = azurerm_resource_group.az_rg.location
  resource_group_name = azurerm_resource_group.az_rg.name
  address_space       = ["10.0.0.0/16"]

  tags = {
    environment = "my-terraform-env"
  }
}

# Crear Subnet en la Virtual Network
resource "azurerm_subnet" "az_subnet" {
  name                 = "my-terraform-subnet"
  resource_group_name  = azurerm_resource_group.az_rg.name
  virtual_network_name = azurerm_virtual_network.az_vnet.name
  address_prefixes     = ["10.0.2.0/24"]
}

# Crear IP Pública
resource "azurerm_public_ip" "az_public_ip" {
  name                = "my-terraform-public-ip"
  location            = azurerm_resource_group.az_rg.location
  resource_group_name = azurerm_resource_group.az_rg.name
  allocation_method   = "Static"

  tags = {
    environment = "my-terraform-env"
  }
}

# Crear Network Security Group y reglas
resource "azurerm_network_security_group" "az_net_sec_group" {
  name                = "my-terraform-nsg"
  location            = azurerm_resource_group.az_rg.location
  resource_group_name = azurerm_resource_group.az_rg.name

  security_rule {
    name                       = "HTTP"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "8080"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "${var.puerto_expuesto}"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

    security_rule {
    name                       = "SSH"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    environment = "my-terraform-env"
  }
}

# Crear Network Interface
resource "azurerm_network_interface" "az_net_int" {
  name                = "my-terraform-nic"
  location            = azurerm_resource_group.az_rg.location
  resource_group_name = azurerm_resource_group.az_rg.name

  ip_configuration {
    name                          = "my-terraform-nic-ip-config"
    subnet_id                     = azurerm_subnet.az_subnet.id
    private_ip_address_allocation = "Static"
    private_ip_address            = "${var.ip_privada}"
    public_ip_address_id          = azurerm_public_ip.az_public_ip.id
  }

  tags = {
    environment = "my-terraform-env"
  }
}

# Crear Network Interface Security Group association
resource "azurerm_network_interface_security_group_association" "az_net_int_sec_group" {
  network_interface_id      = azurerm_network_interface.az_net_int.id
  network_security_group_id = azurerm_network_security_group.az_net_sec_group.id
}

# Crear máquina virtual
resource "azurerm_linux_virtual_machine" "az_linux_vm" {
  name                            = "my-terraform-vm"
  location                        = azurerm_resource_group.az_rg.location
  resource_group_name             = azurerm_resource_group.az_rg.name
  network_interface_ids           = [azurerm_network_interface.az_net_int.id]
  size                            = "${var.size}"
  computer_name                   = "${var.hostname}"
  admin_username                  = "${var.usuario}"
  admin_password                  = "${var.contrasena}"
  disable_password_authentication = false

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  os_disk {
    name                 = "my-terraform-os-disk"
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
  
  tags = {
    environment = "my-terraform-env"
  }
}

# Configurar la ejecución de tareas automáticamente al iniciar la MV.

resource "azurerm_virtual_machine_extension" "az_vm_extension" {
  name                 = "hostname"
  virtual_machine_id   = azurerm_linux_virtual_machine.az_linux_vm.id
  publisher            = "Microsoft.Azure.Extensions"
  type                 = "CustomScript"
  type_handler_version = "2.1"

  settings = <<SETTINGS
    {
      "commandToExecute": "echo '${var.salida_echo}' > index.html ; nohup busybox httpd -f -p ${var.puerto_expuesto} &"
    }
  SETTINGS

  tags = {
    environment = "my-terraform-env"
  }
}

# Fuente de datos para acceder a las propiedades de una dirección IP pública de Azure existente.
data "azurerm_public_ip" "az_public_ip" {
  name                = azurerm_public_ip.az_public_ip.name
  resource_group_name = azurerm_linux_virtual_machine.az_linux_vm.resource_group_name
}

# Variable de salida: Dirección de IP pública
output "public_ip" {
  value = data.azurerm_public_ip.az_public_ip.ip_address
}
```

Una vez seleccionada la suscripción correspondiente como hemos visto anteriormente, crearemos un objeto de servicio. Un objeto de servicio es una aplicación dentro de Azure Active Directory con los tokens de autenticación que Terraform necesita para realizar acciones en su nombre.
Creamos el objeto de servicio mediante el siguiente comando:

``` 
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<SUBSCRIPTION_ID>"
``` 

Como vemos hemos asignado el rol “Contributor”. En Azure, el rol de "Contributor" es uno de los roles de acceso más comunes y permite a los usuarios realizar acciones específicas en los recursos de Azure, como crear y administrar recursos.

El parámetro "--scopes" especifica la suscripción de Azure a la que se le darán los permisos de colaborador al objeto de servicio de Azure AD recién creado. Una vez ejecutado el comando anterior obtendremos la siguiente salida:

``` 
{
"appId": "0b3b9036-f023-4593-9ecc-fa7dfc4538a7",
"displayName": "azure-cli-2023-05-03-07-32-08",
"password": "fXD8Q~5pH6hdZlF0yJAY6xng1paiirQTVqbUkb1r",
"tenant": "a0ce8a08-6fc1-4e4e-b707-b9d558a97fc2"
}
``` 

Estos valores los asignaremos posteriormente a unas variables del entorno que vamos a definir en el SO agente que estamos utilizando. Dependiendo del SO lo realizaremos de una forma u otra.

- Para Linux / MacOS:

``` 
export ARM_CLIENT_ID="<SERVICE_PRINCIPAL_APPID>"
export ARM_CLIENT_SECRET="<SERVICE_PRINCIPAL_PASSWORD>"
export ARM_SUBSCRIPTION_ID="<SUBSCRIPTION_ID>"
export ARM_TENANT_ID="<TENANT_ID>"
``` 

- Para Windows (PowerShell):

``` 
$env:ARM_CLIENT_ID="<SERVICE_PRINCIPAL_APPID>"
$env:ARM_CLIENT_SECRET="<SERVICE_PRINCIPAL_PASSWORD>"
$env:ARM_SUBSCRIPTION_ID="<SUBSCRIPTION_ID>"
$env:ARM_TENANT_ID="<TENANT_ID>"
``` 

Una vez realizada toda esta configuración previa de Azure, comenzamos a utilizar Terraform para desplegar la infraestructura. Para ello seguimos los siguientes pasos:

- Nos situamos en el directorio que contiene la plantilla de terraform, y inicializamos el directorio de trabajo de Terraform mediante el comando:

``` 
terraform init
``` 

- A continuación realizaremos un "terraform plan" previsualizar los cambios que se van a realizar antes de aplicarlos.

```
terraform plan
``` 

Al ejecutar el comando anterior, veremos un resumen del despliegue y en la última línea nos indicará los cambios previstos:

``` 
Plan: 9 to add, 0 to change, 0 to destroy.
```

- Como vemos el plan añadirá las 9 secciones que nos muestra por pantalla. Para aplicar los cambios, usaremos el comando "terraform apply" de la siguiente forma:

``` 
terraform apply
``` 

Si utilizaramos el parámetro "auto-approve", estaremos ejecutando el comando de forma automatizada sin necesidad de recurrir a ningún tipo de aprobación manual. El proceso de despliegue puede demorar bastante (Dependiendo del número de recursos). Como vemos, al terminar el despliegue nos devolverá la IP Pública.

Una vez terminado el despliegue, nos dirigimos al panel de Azure Portal y comprobamos los recursos. Además, para validar que todo funciona correctamente, accedemos a la IP Pública (Recurso) con el puerto correspondiente (8080). Deberíamos ver por pantalla el mensaje “Hello World”.

Para limpiar los recursos creados, nos dirigimos al directorio de trabajo que contiene la plantilla terraform y ejecutamos el comando:

``` 
terraform destroy
``` 


## 5. Prueba práctica del proyecto

### 5.1 Objetivo propuesto

En este proyecto, vamos a realizar una demostración práctica en la que implementaremos un código de una plantilla de Terraform en una pipeline de Azure DevOps. La plantilla dependerá de un fichero para almacenar variables.

En esta prueba práctica, subiremos el código a un repositorio de “Azure Repos” y configuraremos una pipeline mediante un fichero YAML, de forma que al realizar un cambio en el repositorio se realice el despliegue o los cambios definidos en la infraestructura.

En definitiva, HashiCorp Terraform, usado con Microsoft Azure DevOps, proporciona una forma de configurar implementaciones automatizadas de infraestructura como código.

### 5.2 Prerrequisitos

Antes de iniciar con la práctica propuesta, debemos de contar con una serie de
prerrequisitos:

• Una cuenta de Azure.
• Una suscripción de Azure para poder desplegar los recursos necesarios.
• Una cuenta de Azure DevOps.
• Una organización y un proyecto de Azure DevOps.
• Un agente autohospedado o hospedado por Microsoft para realizar trabajos paralelos.
• Repositorio de código fuente.
• Un editor de texto (En este caso Visual Studio Code).

### 5.3 Pasos a seguir para la implementación

#### 5.3.1 Creación de un Backend en Azure

Una vez contamos con la cuenta de Azure, lo primero que haremos será crear un backend para mantener el estado de la infraestructura tras aplicar los cambios deseados. Si no disponemos de un backend, deberíamos de importar cada uno de los recursos existentes manualmente dentro del código de terraform para que los incluya en el proyecto. Esto tomaría bastante más tiempo que realizar una consulta de estado mediante
el backend. Para crear e inicializar un backend seguiremos los siguientes pasos:

1. Nos conectamos a nuestra cuenta de azure desde la Powershell:

``` 
az login
``` 

En caso necesario seleccionamos la suscripción correspondiente:

``` 
az account set --subscription "<SUBSCRIPTION-ID>"
``` 

2. Creamos un grupo de recursos mediante el comando:

``` 
az group create --name dev1 --location eastus
``` 

Como vemos, el grupo de recursos se llamará dev1, con localización eastus.
En este punto contamos con el grupo de recursos, pero sin cuenta de almacenamiento.

Suponiendo que la cuenta de almacenamiento se llamará “storageiesgn”, podemos verificarlo con:

``` 
az storage account show --name storageiesgn --resource-group dev1
```

En la salida del comando, veremos que no existe ninguna cuenta de almacenamiento con ese nombre.
También podemos crear el grupo de recursos directamente desde el portal de azure usando un navegador web:

3. Creamos una cuenta de almacenamiento desde el portal de azure, con el nombre que tenemos definido en la sección del backend dentro del main.tf (storageiesgn).

A la hora de crear la cuenta de almacenamiento, seleccionamos el grupo de recursos creado en el paso anterior.

También deberemos especificar algunos parámetros como la localización, y la redundancia.
También podemos hacerlo mediante el comando:

``` 
az storage account create --name storageiesgn --resource-group dev1 --location
eastus --sku Standard_LRS
``` 

4. A continuación, desde el portal de azure he accedido a la “cuenta de almacenamiento/blob service” y hemos agregado un contenedor con el nombre que tenemos definido en la sección del backend dentro del main.tf.

También podemos hacerlo mediante el comando:

``` 
az storage container create --name contenedoriesgn1 --account-name storageiesgn
--account-key <storage-account-key>
``` 

Al hacerlo mediante el comando anterior, requerimos una clave de acceso para la cuenta de almacenamiento (Cada clave dispone de una serie de permisos). Podemos obtenerla con el siguiente comando:

```
az storage account keys list --account-name storageiesgn --resource-group dev1
``` 

Con esto ya tenemos el backend creado, para inicializarlo tan solo debemos de ejecutar "terraform init" en la máquina agente (La cual tiene terraform instalada). En este caso, este comando irá incluido dentro de la pipeline.


#### 5.3.2 Creación de un proyecto en Azure DevOps.

Lo primero que haremos será crear una organización. Para ello, iniciamos sesión en Azure DevOps y en el panel principal pulsamos en “New organization”.

Por último definimos el nombre y la ubicación de los proyectos de dicha organización.

Para crear un proyecto, accedemos a la pestaña de la organización correspondiente y pulsamos en “New Project”. Crearemos un proyecto “Privado”.

Como paso adicional, vamos a solicitar a Microsoft, la utilización los repositorios y de trabajos paralelos, ya que por defecto viene desactivada para Agentes hospedados por microsoft (Esto se debe a la criptominería).

Para ello accedemos a la organización correspondiente, y desde aquí nos dirigimos a “Organization Settings/Pararell Jobs/Private Projects/Microsoft hosted”. Una vez aquí, pulsamos en “change” y activamos los repositorios mediante la siguiente opción:

``` 
Basic + Test Plans      Start Free Trial
```

Si disponemos de una suscripción de pago de Azure, podemos vincularla a Azure DevOps, y ya no necesitaremos hacer uso de la opción anterior.

Además, en el caso de los proyectos privados, podemos obtener un trabajo gratuito que se pueda ejecutar durante un máximo de 60 minutos cada vez. Cuando creamos una nueva organización de Azure DevOps, puede que no siempre realice esta concesión gratuita de forma predeterminada. Por lo que debemos de solicitarla en el siguiente enlace.

https://forms.office.com/pages/responsepage.aspx?id=v4j5cvGGr0GRqy180BHbR63mUWPlq7NEsFZhkyH8jChUMlM3QzdDMFZOMkVBWU5BWFM3SDI2QlRBSC4u


#### 5.3.3 Añadir una conexión de servicio con Azure Cloud.

Normalmente, para logearnos con nuestra cuenta de Azure, usaríamos el comando “az login”. Sin embargo, este comando abriría nuestro navegador web para introducir nuestra cuenta de Azure. En el agente que ejecuta la pipeline, no podemos hacer esto.

Como alternativa ejecutar el comando con el siguiente parámetro:

``` 
az login –use-device-code
``` 

Este nos proporcionará un código en la propia terminal y un enlace, tan solo debemos de copiar el enlace, abrirlo en un navegador (De cualquier dispositivo) e introducir el código.

Seguidamente introducimos las credenciales de nuestra cuenta de azure y nos habremos logueado. Este es un buen método por ejemplo para enviar un SMS con dicho mensaje y así contar con una autenticación de doble factor cada vez que ejecutemos la pipeline.

La última opción, es utilizar una conexión de servicio. En Azure DevOps, una conexión de servicio es una configuración que permite a los proyectos y pipelines de Azure DevOps interactuar con servicios externos. Estas conexiones de servicio se utilizan para autenticar y autorizar el acceso a recursos externos, como repositorios de código fuente, servicios de compilación, despliegue y pruebas, entre otros.

Para habilitar una conexión de servicio, nos dirigimos a nuestro proyecto de Azure Devops, y luego a “Project Settings / Service Connections / New service connection”. En este caso seleccionamos la conexión de tipo Azure Resource Manager, nos aparecerá varias formas de añadirla (Como la automática en la que iniciamos sesión en Azure y la seleccionamos o introduciendo los parámetros necesarios manualmente).


#### 5.3.4 Preconfiguración, creación de la pipeline (YAML).

Desde el panel de Azure DevOps, dentro de nuestro proyecto, nos dirigimos a “Pipelines”. Una vez aquí, pulsamos en “Create Pipeline”.

A continuación nos pedirá la fuente del código, en este caso seleccionaremos “Azure Repos Git”.
Y seguidamente, seleccionamos el repositorio correspondiente.

Una vez seleccionado el repositorio, nos dará la opción de crear una pipeline desde cero, partiendo de una plantilla que nos proporciona, o seleccionar una pipeline existente.

En este caso, he optado por la 2º opción, ya que cuento con una pipeline predefinida anteriormente con visual studio code. Nos pedirá que seleccionemos la rama y la ruta del fichero yaml sobre el que se ejecutará la pipeline. En este caso, cuento con más de un fichero yaml, ya que he usado una opción que me permite llamar a un fichero desde otro (template). Por lo que selecciono el fichero principal (terraform-plan.yaml).

``` 
name: "Terraform Plan"

trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: TerraformInstaller@0
  displayName: "Install Terraform"
  inputs:
    terraformVersion: '1.4.6'
    terraformDownloadLocation: 'https://releases.hashicorp.com/terraform'


- task: TerraformTaskV4@4
  inputs:
    provider: 'azurerm'
    command: 'init'
    workingDirectory: '$(System.DefaultWorkingDirectory)/Templates'
    backendServiceArm: 'Suscripción de Azure 1(d26cf210-db93-4db7-9cea-d9ddce1cd55d)'
    backendAzureRmResourceGroupName: 'dev1'
    backendAzureRmStorageAccountName: 'storageiesgn'
    backendAzureRmContainerName: 'contenedoriesgn1'
    backendAzureRmKey: 'terraform.tfstate'
  displayName: 'Terraform Init'

- task: TerraformTaskV4@4
  inputs:
    provider: 'azurerm'
    command: 'plan'
    workingDirectory: '$(System.DefaultWorkingDirectory)/Templates'
    commandOptions: '-out=terraform.tfplan'
    environmentServiceNameAzureRM: 'Suscripción de Azure 1(d26cf210-db93-4db7-9cea-d9ddce1cd55d)'
  displayName: 'Terraform Plan'

- task: PublishBuildArtifacts@1
  inputs:
    pathtoPublish: '$(Build.SourcesDirectory)/Templates/terraform.tfplan'
    artifactName: 'terraformPlan'
    publishLocation: 'Container'
  displayName: 'Publish Terraform plan'

- template: terraform-apply.yaml
```

Otro detalle importante, es disponer de las extensiones necesarias añadidas al proyecto.
Por ejemplo, en la pipeline que vamos a usar, tenemos una tarea para instalar Terraform en la máquina agente (TerraformInstaller@0).

```
- task: TerraformInstaller@0
displayName: "Install Terraform"
inputs:
terraformVersion: '1.4.6'
terraformDownloadLocation: 'https://releases.hashicorp.com/terraform'
``` 

Esta tarea no funcionará si no hemos añadido esta extensión al proyecto. Para añadirla, desde el panel de Azure DevOps, hacemos clic en la bolsa de la compra que aparece en la esquina superior derecha, y posteriormente nos dirigimos a Browse Marketplace.

Una vez aquí, buscamos la extensión correspondiente para la instalación de Terraform y la añadimos al proyecto.

Una vez hemos realizado toda la configuración anterior, deberíamos tener:

• La ejecución de trabajos paralelos habilitada.
• El repositorio de Azure Repos habilitado.
• Nuestro código en el repositorio de Azure Repos (main.tf y variables).
• Nuestra pipeline en dicho repositorio.
• Las extensiones necesarias añadidas al proyecto para la ejecución de la pipeline.


#### 5.3.5 Ejecución de la pipeline.

Nos dirigimos a la sección pipelines y pulsamos en “Run” sobre la pipeline definida. Mediante la tarea “PublishBuildArtifacts@1” estamos generando un artefacto, que contendrá un fichero terraform.tfplan (Salida de Terraform plan). Este fichero está en formato binario y no se puede leer directamente como texto plano. Debemos utilizar comandos específicos de Terraform para mostrar y revisar su contenido de manera legible.

Como vemos la ejecución de la pipeline se ha completado correctamente, ahora realizaremos la comprobación de los recursos desplegados en Azure Portal.

Podemos observar como la infraestructura se ha desplegado correctamente.

En caso de realizar alguna modificación al repositorio (En la rama main), se volverá a ejecutar la pipeline gracias a la siguiente configuración del fichero YAML:

``` 
trigger:
branches:
include:
- main
``` 

Además podrá saber el estado actual de la infraestructura gracias al fichero de estado que se ha generado en el contenedor que creamos para el backend, mediante el siguiente bloque de codigo (Dentro del main.tf):

``` 
terraform {
  backend "azurerm" {
    resource_group_name = "dev1"
    storage_account_name = "storageiesgn"
    container_name = "contenedoriesgn1"
    key = "terraform.state"
    }
}
``` 

Como vemos el fichero de estado se llamará terraform.state. Vamos a comprobar desdeel portal de Azure que se haya generado dentro del contenedor (blob).

Gracias a este fichero de estado, cuando volvamos a realizar un cambio en el código de la infraestructura, no se necesitará importar cada uno de los recursos al proyecto de Terraform del agente autohospedado por Microsoft (Ya que cada vez que se ejecute la pipeline, es un agente distinto).

Si tuviésemos configurado un agente autohospedado (Sabemos que no va a cambiar), no sería necesario (Aunque recomendable) el fichero de estado.

















