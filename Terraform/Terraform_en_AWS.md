# Terrraform en AWS

## 1. Manejo de credenciales

Empezamos con las lecciones de Terraform en AWS, esta vez creando las credenciales necesarias para conectar a Terraform con AWS mediante IAM.

Creamos usuario, lo pongo en tipo de acceso en Programmatic access y lo añado a su grupo correspondiente. Y añadimos tres políticas, las siguientes:

* AmazonEC2FullAccess
* AmazonVPCFullAccess
* IAMUserSSHKeys
* AmazonS3FullAccess

Son políticas de EC2, VPC y para las claves SSH. También de S3 para guardar los estados de Terraform y que no se pierda la infraestructura. 

## 2. Crear un Bucket S3 para estados remotos

Vamos a crear un Bucket S3 para almacenar los estados remotos de terraform. 

A la hora de crear un bucket es importante escoger bien la región, tenerlo claro, al igual que se ha de tener cuidado con el acceso público. Para manejar el estado remoto de terraform, conviene dejar activado el bloqueo de acceso público (Block all public access). 

Activamos el Bucket versioning, damos en Tags un Key y un value. Por ejemplo, Name y Test. Habilitamos el Server-side encryptation, con Amazon S3 key, para que todos los objetos que vamos a almacenar de Terraform esten encriptados dentro del bucket. Y damos en Crear.

## 3. Primeros Script con Backend S3 - Estados Remotos

Empezamos a escribir los script de terraform, enlazando el backend S3 para almacenar los estados remotos y el proveedor de AWS y las credenciales.

Nuestro script es el siguiente (main.tf):

```
# Backends variables and configuration
terraform {
  backend "s3" {
    bucket                  = "informe-nube-tf-states"
    key                     = "terraform.tfstate"
    encrypt                 = "true"
    region                  = "eu-west-1"
    profile                 = "nombredemiyperfil"
    shared_credentials_file = "ruta/a/credentciales/de/aws"
  }
}

# Provider Configuration
provider "aws" {
  region                  = "eu-west-1"
  profile                 = "nombredemiyperfil"
  shared_credentials_file = "ruta/a/credentciales/de/aws"
}
```

Con el bloque de provider establecemos la comunicación entre Terraform y AWS. Si tuvieramos un dominio de S3 nuestro propio, incluiriamos la línea endpoint justo debajo de encrypt:

``` 
endpoint = "" (Entre comillas meteriamos la URL de ese dominio propio externo del creado por AWS)
```

Hecho esto, cargamos las variables de entorno en la terminal, justo en el directorio donde tenemos nuestros archivos en el repo (.env) 

``` 
AWS_ACCESS_KEY_ID=KEYDEACCESODEEJEMPLO
AWS_SECRET_ACCESS_KEY=KeySECRETAdeACCESO
AWS_DEFAULT_REGION=eu-west-1
```

Introducimos el comando para cargar estas variables:

``` 
source ../.env
```

Y lanzamos el **terraform init**, solo el init para inicializar ese script junto con las variables de credenciales para permitir ya si la conexión entre Terraform y AWS de manera remota junto con sus respectivas credenciales. 


## 4. VPC Redes Privadas Cloud

Vamos a crear una VPC en AWS. Para ello, tenemos el siguiente archivo de Terraform (vpc.tf):

``` 
# VPC
resource "aws_vpc" "informe_nube" {
  cidr_block                       = var.cidr
  assign_generated_ipv6_cidr_block = false
  enable_dns_hostnames             = true

  tags = {
    Name     = "VPC Tests"
    Episodio = "Informe Nube 4"
  }

  lifecycle {
    prevent_destroy = false // para entornos de produccion colocarlo en true
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.informe_nube.id
  count                   = length(data.aws_availability_zones.available.zone_ids) -- Estos parámetros tienen que ver con lo introducido en otro archivo que dejaré debajo de este. (data.tf)
  cidr_block              = cidrsubnet(var.cidr, 4, 0 + count.index)
  map_public_ip_on_launch = false
  availability_zone       = element(data.aws_availability_zones.available.names, count.index)

  tags = {
    Name     = "Red publica-${count.index}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube]
}

resource "aws_subnet" "privada" {
  vpc_id                  = aws_vpc.informe_nube.id
  count                   = length(data.aws_availability_zones.available.zone_ids)
  cidr_block              = cidrsubnet(var.cidr, 4, 3 + count.index)
  map_public_ip_on_launch = false
  availability_zone       = element(data.aws_availability_zones.available.names, count.index)

  tags = {
    Name     = "Red privada-${count.index}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube]
}

# Internet Gateway
resource "aws_internet_gateway" "informe_nubec" {
  vpc_id = aws_vpc.informe_nube.id

  tags = {
    Name = "Internet Gateway"
    Episodio = "Informe Nube 4"
  }

  lifecycle {
    prevent_destroy = false // para entornos de produccion colocarlo en true
  }

  depends_on = [aws_vpc.informe_nube]
}

# Routes
resource "aws_route_table" "public" {
  vpc_id           = aws_vpc.informe_nube.id
  # Internet
  route {
    ipv6_cidr_block = "::/0"
    gateway_id = aws_internet_gateway.informe_nubec.id
  }
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.informe_nubec.id
  }

  tags = {
    Name = "Tabla de Route para las redes public"
    Episodio = "Informe Nube 4"
  }

}

# Route Association
resource "aws_route_table_association" "public" {
  count          = length(data.aws_availability_zones.available.zone_ids)
  route_table_id = aws_route_table.public.id
  subnet_id      = element(aws_subnet.public[*].id,count.index)

  depends_on = [aws_route_table.public]
}
```

Dentro de nuestro archivo de variables (vars.tf), añadimos aspectos como el cidr que tiene que ver con la ip que se le va a asignar a nuestra vpc. 

``` 
variable "cidr" {
  description = "Direcciones IP privadas a utilizar en el VPC, en notacion CIDR"
  default     = "10.0.0.0/20"
}

variable "ssh_pub_path" {
  description = "Directorio de la llave SSH publica"
  default     = "~/.ssh/id_rsa.pub"
}
```

* data.tf

```
data "aws_region" "current" {}
data "aws_availability_zones" "available" {
  state = "available"
}
```

Con este archivo se consigue que se devuelvan las zonas disponibles en ese preciso momento. Puede ser que alguna esté caida temporalmente. 




