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


## 5. Security Groups y reglas

Vamos a crear los security groups y reglas para las máquinas EC2. Lo hacemos bajo el siguiente fichero (security_groups.tf):

```
# Default Security Group
resource "aws_default_security_group" "default" {
  vpc_id = aws_vpc.informe_nube.id

  ingress {
    protocol  = -1
    self      = true
    from_port = 0
    to_port   = 0
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name     = "Default"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube]
}

# Security Group
resource "aws_security_group" "servidor_web" {
  name        = "Servidor Web"
  description = "Grupo de seguridad para las intancias EC2"
  vpc_id      = aws_vpc.informe_nube.id

  tags = {
    Name     = "Servidor Web"
    Episodio = "Informe Nube 4"
  }
}

resource "aws_security_group_rule" "http" {
  type              = "ingress"
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  ipv6_cidr_blocks  = ["::/0"]
  security_group_id = aws_security_group.servidor_web.id
  description       = "Permitir conexiones al puerto HTTP desde cualquier IP"

  depends_on = [aws_security_group.servidor_web]
}

resource "aws_security_group_rule" "https" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  ipv6_cidr_blocks  = ["::/0"]
  security_group_id = aws_security_group.servidor_web.id
  description       = "Permitir conexiones al puerto HTTP desde cualquier IP"

  depends_on = [aws_security_group.servidor_web]
}

resource "aws_security_group_rule" "ssh" {
  type              = "ingress"
  from_port         = 22
  to_port           = 22
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  ipv6_cidr_blocks  = ["::/0"]
  security_group_id = aws_security_group.servidor_web.id
  description       = "Permitir conexiones al puerto SSH desde cualquier IP"

  depends_on = [aws_security_group.servidor_web]
}

# Security Group
resource "aws_security_group" "efs" {
  name        = "EFS"
  description = "Grupo de seguridad para el disco EFS"
  vpc_id      = aws_vpc.informe_nube.id

  tags = {
    Name     = "EFS"
    Episodio = "Informe Nube 4"
  }
}

resource "aws_security_group_rule" "efs" {
  count             = length(data.aws_availability_zones.available.zone_ids)
  type              = "ingress"
  from_port         = 2049
  to_port           = 2049
  protocol          = "tcp"
  cidr_blocks       = [element(aws_subnet.public[*].cidr_block, count.index)]
  security_group_id = aws_security_group.efs.id
  description       = "Permitir conexiones al puerto HTTP desde cualquier IP"

  depends_on = [aws_security_group.efs]
}
```

El recurso **efs** es el encargado de la compartición de archivos file system de AWS. 


## 6. Crear una máquina en EC2

Lo hacemos en base al siguiente archivo (ec2_instance_simple.tf):

```
# Elastic IPS
resource "aws_eip" "simple" {
  vpc = true

  tags = {
    Name    = "IP elastica"
    Episodio = "Informe Nube 4"
  }

  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_instance" "simple" {
  count                       = 1
  availability_zone           = "eu-west-1a"
  ami                         = "ami-022e8cc8f0d3c52fd" // AMI son regionales (distintas IDS por region)
  instance_type               = "t3a.small"
  subnet_id                   = aws_subnet.public[0].id
  vpc_security_group_ids      = concat([aws_security_group.servidor_web.id], [aws_default_security_group.default.id])
  key_name                    = aws_key_pair.devago.id (Asociación a la clave SSH)

  lifecycle {
    create_before_destroy = true
  }

  root_block_device {
    volume_type           = "gp3"
    volume_size           = "50"
    delete_on_termination = true
  }
#User data is limited to 16 KB
  user_data = <<-EOF
                #!/bin/bash
                # ---> Updating, upgrating and installing the base
                apt update
                apt install python3-pip apt-transport-https ca-certificates curl software-properties-common -y
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
                apt update && apt upgrade -y
                apt install docker-ce -y
                systemctl status docker
                usermod -aG docker ubuntu
                docker run -p 80:80 -d nginxdemos/hello
                EOF

  tags = {
    Name    = "EC2 simple en ${aws_subnet.public[count.index].availability_zone}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube, aws_key_pair.devago, aws_security_group.servidor_web]
}

resource "aws_eip_association" "simple" {
  instance_id   = aws_instance.simple[0].id
  allocation_id = aws_eip.simple.id
}

output "simple_dns" {
  value = aws_eip.simple.public_dns
}
```

Lo registrado en el apartado **key name** lo tenemos registrado en otro fichero (ssh_keys.tf):

```
# SSH Keys
resource "aws_key_pair" "devago" {
  key_name   = "devago"
  public_key = file(var.ssh_pub_path) (Esta variable la tenemos en el archivo vars.tf)

  tags = {
    Name     = "devago"
    Usuario  = "Antony R. Goetzschel <ago>"
    Episodio = "Informe Nube 4"
  }
}
```

## 7. Script de inicio User Data

Ahora vamos a crear un script que se ejecutará al inicio de una máquina EC2, User Data.

Dentro del archivo tf que vimos anteriormente (ec2_instance_simple.tf), introducimos el script en la parte final. Volvemos a verlo:

```
#User data is limited to 16 KB
  user_data = <<-EOF
                #!/bin/bash
                # ---> Updating, upgrating and installing the base
                apt update
                apt install python3-pip apt-transport-https ca-certificates curl software-properties-common -y
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
                apt update && apt upgrade -y
                apt install docker-ce -y
                systemctl status docker
                usermod -aG docker ubuntu
                docker run -p 80:80 -d nginxdemos/hello
                EOF

  tags = {
    Name    = "EC2 simple en ${aws_subnet.public[count.index].availability_zone}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube, aws_key_pair.devago, aws_security_group.servidor_web]
}

resource "aws_eip_association" "simple" {
  instance_id   = aws_instance.simple[0].id
  allocation_id = aws_eip.simple.id
}

output "simple_dns" {
  value = aws_eip.simple.public_dns
}
```

## 8. Servicio Web en HA

Ya tenemos, por lo tanto, una máquina EC2 corriendo con su usuario y un servicio Docker activo. Ahora, para darle alta disponibilidad al servicio, deberíamos crear una máquina para cada zona de la región. Una réplica por zona. 

Lo hacemos en base al siguiente archivo (ec2_instance_multiple.tf):

```
# Elastic IPS
resource "aws_eip" "multiple" {
  count = length(data.aws_availability_zones.available.zone_ids)
  vpc = true

  tags = {
    Name    = "IP elastica"
    Episodio = "Informe Nube 4"
  }

  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_instance" "multiple" {
  count                       = length(data.aws_availability_zones.available.zone_ids)
  availability_zone           = element(data.aws_availability_zones.available.names, count.index)
  ami                         = "ami-022e8cc8f0d3c52fd" // AMI son regionales (distintas IDS por region)
  instance_type               = "t3a.small"
  subnet_id                   = aws_subnet.public[count.index].id
  vpc_security_group_ids      = concat([aws_security_group.servidor_web.id], [aws_default_security_group.default.id])
  key_name                    = aws_key_pair.devago.id

  lifecycle {
    create_before_destroy = true
  }

  root_block_device {
    volume_type           = "gp2"
    volume_size           = "50"
    delete_on_termination = true
  }
#User data is limited to 16 KB
  user_data = <<-EOF
                #!/bin/bash
                # ---> Updating, upgrating and installing the base
                apt update
                apt install python3-pip apt-transport-https ca-certificates curl software-properties-common -y
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
                apt update && apt upgrade -y
                apt install docker-ce -y
                systemctl status docker
                usermod -aG docker ubuntu
                docker run -p 80:80 -d nginxdemos/hello
                EOF

  tags = {
    Name    = "EC2 multiple en ${aws_subnet.public[count.index].availability_zone}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_vpc.informe_nube, aws_key_pair.devago, aws_security_group.servidor_web]
}

resource "aws_eip_association" "multiple" {
  count         = length(data.aws_availability_zones.available.zone_ids)
  instance_id   = aws_instance.multiple[count.index].id
  allocation_id = aws_eip.multiple[count.index].id
}

output "multiple_dns" {
  value = aws_eip.multiple[*].public_dns
}
```

Viene a ser el mismo archivo para la instancia simple, salvo que dentro del recurso de instancia multiple en el contador ponemos el parámetro length para decirle que cree una máquina por cada zona de nuestra región. 

Y dentro del script de user data, en tags especificamos que es para una EC2 multiple bajo el indice del contador de zonas de la región. 


## 9. Montar un volumen EFS en una máquina EC2

* **ec2_instance_persistent.tf**

```
# Elastic IPS
resource "aws_eip" "persistent" {
  vpc = true

  tags = {
    Name    = "IP elastica"
    Episodio = "Informe Nube 4"
  }

  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_efs_file_system" "persistent" {
  creation_token = "InformeNube4"
  encrypted = true
  performance_mode = "generalPurpose"

  tags = {
    Name    = "EFS"
    Episodio = "Informe Nube 4"
  }
}

resource "aws_efs_mount_target" "persistent" {
  count           = length(data.aws_availability_zones.available.zone_ids)
  file_system_id  = aws_efs_file_system.persistent.id
  subnet_id       = element(aws_subnet.privada.*.id,count.index)
  security_groups = [aws_security_group.efs.id]
}


resource "aws_instance" "persistent" {
  availability_zone      = "eu-west-1b"
  ami                    = "ami-022e8cc8f0d3c52fd"
  count                  = 1
  instance_type          = "t3a.small"
  subnet_id              = aws_subnet.public[1].id
  vpc_security_group_ids = concat([aws_security_group.servidor_web.id], [aws_security_group.efs.id], [aws_default_security_group.default.id])
  key_name               = aws_key_pair.devago.id

  lifecycle {
    create_before_destroy = true
  }

  root_block_device {
    volume_type = "gp2"
    volume_size = "10"
    delete_on_termination = true
  }

  user_data = <<-EOF
                #!/bin/bash
                # ---> Updating, upgrating and installing the base
                apt update
                apt install python3-pip apt-transport-https ca-certificates curl software-properties-common nfs-common -y
                mkdir /var/lib/docker
                echo "${aws_efs_file_system.persistent.dns_name}:/  /var/lib/docker    nfs4   nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 2" >> /etc/fstab
                mount -a
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
                apt update && apt upgrade -y
                apt install docker-ce -y
                systemctl status docker
                usermod -aG docker ubuntu
                docker run -p 80:80 -d nginxdemos/hello
                EOF

  tags = {
    Name    = "EC2 con persistencia en ${aws_subnet.public[count.index].availability_zone}"
    Episodio = "Informe Nube 4"
  }

  depends_on = [aws_efs_file_system.persistent, aws_efs_mount_target.persistent]
}

resource "aws_eip_association" "persistent" {
  instance_id   = aws_instance.persistent[0].id
  allocation_id = aws_eip.persistent.id
}

output "persistent_dns" {
  value = aws_eip.persistent.public_dns
}
```

### 9.1 Conclusión (Estructura directorio)

Quedaría de la siguiente manera:

.env
README.md
data.tf
ec2_instance_multiple.tf
ec2_instance_persistent.tf
ec2_instance_simple.tf
main.tf
security_groups.tf
ssh_keys.tf
vars.tf
vpc.tf


## 10. VPs en EC2 (2º caso práctico)

El directorio quedaría configurado con los siguientes archivos:

README.md
main.tf
terraform.tfvars.example (Habrá que copiar este archivo para el terraform.tfvars)
versions.tf

* **main.tf**

``` 
provider "aws" {
  region = "eu-west-3"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

variable "ssh_key_path" {}
variable "vpc_id" {}

resource "aws_key_pair" "deployer" {
  key_name   = "deployer-key"
  public_key = file(var.ssh_key_path)
}

resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow SSH inbound traffic"
  vpc_id      = var.vpc_id

  ingress {
    description = "SSH from VPC"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow_ssh"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  key_name = aws_key_pair.deployer.key_name
  vpc_security_group_ids = [
    aws_security_group.allow_ssh.id
  ]
  tags = {
    Name = "HelloWorld"
  }
}

output "ip_instance" {
  value = aws_instance.web.public_ip
}

output "ssh" {
  value = "ssh -l ubuntu ${aws_instance.web.public_ip}"
}
```

* **terraform.tfvars.example**

``` 
# Aquí va la clave SSH pública
ssh_key="ruta fichero ssh.pub"
vpc_id="vpc-XXXXXX"
```

* **versions.tf**

``` 
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
  required_version = ">= 0.13"
}
```

Configurado todo esto, los pasos a dar serían estos:

1. **Copia el fichero terraform.tfvars.examples**
   
``` 
cp terraform.tfvars.example terraform.tfvars
``` 

Edita el fichero terraform.tfvars y pon tus propios valores, sobre todo el vpc_id y la clave ssh

2. **Inicialización del despliegue**

``` 
terraform init
``` 

3. **Planificación del despliegue**
   
``` 
terraform plan
``` 

4. **Despliegue de la infraestructura**
   
``` 
terraform apply
``` 

5. **Destrucción de la infraestructura**
   
``` 
terraform destroy
``` 

## 11. Clúster en AWS EKS

El directorio quedaría configurado de la siguiente manera:

README.md
kubernetes-dashboard-admin.rbac.yaml
main.tf
outputs.tf
security-groups.tf
terraform.tfvars.example
versions.tf
vpc.tf

Para este caso en concreto, usaremos módulos de terraform en algunos aspectos para poder lidiar con la complejidad que supone la creación de un clúster.

Y ahora veamos el contenido de cada uno de los archivos en detalle:

* **kubernetes-dashboard-admin.rbac.yaml**

``` 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
# Create ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kube-system
```

* **main.tf**

``` 
variable "project_name" {}
variable "region" {}

provider "aws" {
  region      = var.region
}

module "eks" {
  source       = "terraform-aws-modules/eks/aws"
  cluster_name = "${var.project_name}-${local.cluster_name}"
  subnets      = module.vpc.private_subnets
  cluster_version = "1.18"
  tags = {
    Environment = "${var.project_name}-testing"
    GithubRepo  = "terraform-aws-eks"
    GithubOrg   = "terraform-aws-modules"
  }

  vpc_id = module.vpc.vpc_id

  worker_groups = [
    {
      name                          = "worker-group-1"
      instance_type                 = "t3.small"
      additional_userdata           = "echo foo bar"
      asg_desired_capacity          = 1
      additional_security_group_ids = [aws_security_group.worker_group_mgmt_one.id]
    },
    {
      name                          = "worker-group-2"
      instance_type                 = "t3.medium"
      additional_userdata           = "echo foo bar"
      additional_security_group_ids = [aws_security_group.worker_group_mgmt_two.id]
      asg_desired_capacity          = 3
    },
  ]
  //workers_group_defaults = {
  //  root_volume_type = "gp2"
  //}
}

data "aws_eks_cluster" "cluster" {
  name = module.eks.cluster_id
}

data "aws_eks_cluster_auth" "cluster" {
  name = module.eks.cluster_id
}

provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  token                  = data.aws_eks_cluster_auth.cluster.token
  load_config_file       = false
  version                = "~> 1.9"
}
```

* **outputs.tf**

``` 
output "cluster_endpoint" {
  description = "Endpoint for EKS control plane."
  value       = module.eks.cluster_endpoint
}

output "cluster_security_group_id" {
  description = "Security group ids attached to the cluster control plane."
  value       = module.eks.cluster_security_group_id
}

output "kubectl_config" {
  description = "kubectl config as generated by the module."
  value       = module.eks.kubeconfig
}

output "config_map_aws_auth" {
  description = "A kubernetes configuration to authenticate to this EKS cluster."
  value       = module.eks.config_map_aws_auth
}

output "region" {
  description = "AWS region"
  value       = var.region
}

output "cluster_name" {
  description = "Kubernetes Cluster Name"
  value       = local.cluster_name
}
```

* **security-groups.tf**

``` 
resource "aws_security_group" "worker_group_mgmt_one" {
  name_prefix = "worker_group_mgmt_one"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"

    cidr_blocks = [
      "10.0.0.0/8",
    ]
  }
}

resource "aws_security_group" "worker_group_mgmt_two" {
  name_prefix = "worker_group_mgmt_two"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"

    cidr_blocks = [
      "192.168.0.0/16",
    ]
  }
}

resource "aws_security_group" "all_worker_mgmt" {
  name_prefix = "all_worker_management"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"

    cidr_blocks = [
      "10.0.0.0/8",
      "172.16.0.0/12",
      "192.168.0.0/16",
    ]
  }
}
```

* **terraform.tfvars.example**

``` 
project_name="terraform"
region_name="eu-west-3"
```

* **versions.tf**

```
terraform {
  required_version = ">= 0.12"
}

provider "random" {
  version = "~> 2.1"
}

provider "local" {
  version = "~> 1.2"
}

provider "null" {
  version = "~> 2.1"
}

provider "template" {
  version = "~> 2.1"
}
```

* **vpc.tf**

``` 
data "aws_availability_zones" "available" {}

resource "random_string" "suffix" {
  length  = 8
  special = false
}

locals {
  cluster_name = "${var.project_name}-eks-${random_string.suffix.result}"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.77.0"

  name                 = "${var.project_name}-vpc"
  cidr                 = "10.0.0.0/16"
  azs                  = data.aws_availability_zones.available.names
  private_subnets      = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets       = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
  }

  public_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/elb-${local.cluster_name}"                      = "1"
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb-${local.cluster_name}"             = "1"
  }
}
```

Y después daríamos los mismos pasos más o menos vistos en el anterior apartado:

1. **Copia el fichero terraform.tfvars.examples**
   
``` 
cp terraform.tfvars.example terraform.tfvars
``` 

Edita el fichero terraform.tfvars y pon tus propios valores, sobre todo el vpc_id y la clave ssh

2. **Inicialización del despliegue**

``` 
terraform init
``` 

3. **Planificación del despliegue**

```    
terraform plan
``` 

4. **Despliegue de la infraestructura**

```    
terraform apply
``` 

5. **Conexión SSH**

``` 
ssh -l ubuntu EIP
``` 

6. **Destrucción de la infraestructura**

```    
terraform destroy
``` 












