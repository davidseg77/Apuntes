# Información del Curso de Terraform de OW

## 1. Instalación

Descargamos el fichero extraible de la página oficial de Terraform. En la terminal lo extraemos, y como recomendación lo movemos a un directorio específico, como por ejemplo /opt/.

Podemos irnos a /usr/bin/ y crear ahí un enlace simbólico que lleve al directorio de Terraform.

``` 
ln -s /opt/terraform terraform
``` 

Para comprobar que lo tenemos instalado

``` 
terraform version
```

**IDE**

IDE (entorno de desarrollo integrado) es una aplicación que hace que su experiencia de programación sea mucho más manejable. Por ejemplo, Visual Studio Code. En este caso, para trabajar con Terraform es recomendable integrar una IDE como podría ser PyCharm o mismamente VSC. 

Dentro de VSC instalamos el plugin que permitirá la integración con Terraform.

## 2. AWS Provider

Creamos un usuario en IAM dentro de AWS, y le asignamos el permiso de AdministratorAccess. AWS nos otorga un ID de Access Key y un secret Access Key. 

Hecho esto, nos instalamos el CLI de AWS en la terminal y en el home del usuario deberemos tener la carpeta .aws. Dentro de esa carpeta tendremos un fichero de credenciales, pues ahí debemos ingresar las ID dadas para nuestro usuario.

Si creamos un elastic service podemos asociarlo con nuestro usuario o perfil dado en el archivo de credenciales.

aws ec2 nombre --profile usuario o perfil dado en credenciales (por ej. davidseg).

Desde VSC podemos crear el archivo inicial de un proyecto de Terraform, su punto inicial (main.tf):

``` 
terraform {
  required_version = ">= 0.10.7"
}

provider "aws" {
  region = "eu-west-1"
  allowed_account_ids = ["723002569774"]
  profile = "openwebinars"
}
```

* *allowed_account_ids* hace referencia a la id de la cuenta de AWS nuestra, la permitida para este proyecto.

Y ahora inicializamos el proyecto en nuestra carpeta:

``` 
terraform init
``` 

**Crear nuestro primer recurso**

Como primer recurso, crearemos una VPC dentro bien del main.tf o en otro archivo dentro del mismo directorio llamado vpc.tf:

``` 
resource "aws_vpc" "vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support = true
  tags {
    Name = "profile dado en credenciales"
  }
}
``` 

Para aplicar esta definición, hacemos uso de terraform plan. Este comando se encarga de revisar si este recurso está ya en AWS y si ha de implantarlo o no:

```
terraform plan
``` 

Para aplicar los cambios después de la revisión del plan:

``` 
terraform apply 
```

**States (Ficheros de estado)**

Si hacemos cualquier cambio en nuestro archivo main.tf o el vpc.tf, con terraform plan nos dirá donde están los cambios y si los aplicamos se harán de manera automática. Lo comprobamos en AWS. 

Siempre que hagamos un cambio, Terraform nos crea un archivo backup para contener esa versión anterior del fichero, lo cual es muy aconsejable para hacer un rollback si fuera necesario. Solo habría que irse a ese archivo y desde el lanzar el plan y el apply.

**Variables**

Podemos especificar una serie de variables dentro de un archivo .tf. Esto es productivo a la hora de crear una plantilla extensa.

``` 
variable "cidr" {
  type = "string"
  default = "10.0.0.0/16"
}

variable "ami_id" {
  type = "string"
  default = "ami-acd005d5"
}

variable "instance_type" {
  type = "string"
  default = "t2.micro"
}
``` 

De hecho, si modificamos el fichero de creación de la vpc, podemos añadirle la variable:

``` 
resource "aws_vpc" "vpc" {
  cidr_block = "${var.cidr}"
  enable_dns_hostnames = false
  enable_dns_support = false
  tags {
    Name = "openwebinars"
  }
}
```

**Outputs**

Podemos crear un recurso de tipo output para dar forma a nuestra instancia de vpc. En el archivo de variables ya se definieron el tipo de instancia y la AMI.

En primer lugar, vamos a dar forma a un archivo para crear un recurso que vamos a llamar webserver.tf. Dentro de este archivo, también podemos introducir el recurso source data, que busca los recursos, por ejemplo, de las zonas de disponibilidad de AWS. Esto lo insertamos mediante la directiva data. Estos datos los tenemos a modo de consulta en la página de Terraform.

``` 
data "aws_availability_zones" "az"{

}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners = ["099720109477"] (El owner se saca yendo a AWS y buscando la ID de la máquina en cuestión)

  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*"]
  }
}

data "template_file" "userdata" {
  template = "${file("templates/userdata.tpl")}"
  vars {
    webserver = "apache2"
  }
}

resource "aws_instance" "web-server" {
  ami = "${data.aws_ami.ubuntu.id}"
  instance_type = "${var.instance_type}"
  user_data = "${data.template_file.userdata.rendered}"
}
``` 

Y trás el plan y el apply, creamos el archivo de output.tf:

```
output "server-ip" {
  value = "${aws_instance.web-server.public_ip}"
}

output "az" {
  value = "${data.aws_availability_zones.az.names}"
}
```

Si quisiera recuperar el output de una plantilla de terraform:

```
terraform output
``` 

**Templates**

Pueden ser creados para personalizar cualquier aspecto de nuestra infraestructura. En este caso, vamos a crear un directorio template y dentro añadimos nuestro archivo de plantilla, un simple script a modo de ejemplo. 

* template.tf

``` 
#!/bin/bash
yum install ${webserver} --assumeyes
service ${webserver} start
``` 

como vemos, no es más que una simple instalación y el inicio del servicio a instalar. Lo declaramos mediante la variable webserver, variable que ahora llevaremos a nuestro archivo anterior, el webserver.tf. Introducimos lo siguiente en dicho archivo.

``` 
data "template_file" "userdata" {
  template = "${file("templates/userdata.tpl")}" (Ruta donde se aloja el archivo template)
  vars {
    webserver = "apache2"
  }
}
``` 

## 3. Importar recursos

Para importar un recurso externo a Terraform el primer requisito es tenerlo declarado. Como ejemplo, vamos a declarar un servicio de tipo ec2:

* ec2.tf

``` 
resource "aws_instance" "web" {
  ami = "ami-acd005d5"
  instance_type = "t2.small"
  tags {
    Name = "openwebinars-webserver"
  }
}
```

Le añadimos, logicamente, el main.tf:

```
terraform {
  required_version = ">= 0.10.7"
}

provider "aws" {
  region = "eu-west-1"
  allowed_account_ids = ["723002569774"]
  profile = "openwebinars"
}
``` 

Y ahora queremos importar una instancia de AWS, pero primero hemos de iniciar esta nueva plantilla:

``` 
terraform init

terraform import + tipo de recurso seguido de punto y nombre del recurso (aws_instance.web) + ID del recurso dentro de AWS.
``` 

Y la instancia se importará al archivo de estado de Terraform.

## 4. Desarrollo de una plantilla compleja

**Crear N recursos**

En este caso, podemos crear una cantidad deseada de un mismo recurso dentro un único archivo. Para ello defino las variables necesarias (variables.tf)

``` 
variable "instance_type" {
  type = "string"
  default = "t2.micro"
}

variable "region" {
  type = "string"
  default = "eu-west-1"
}

variable "aws_id" {
  type = "string"
  default = "723002569774"
}

variable "aws_amis" {
  type = "map"
  default = {
    "eu-west-1" = "ami-acd005d5"
    "us-east-1" = "ami-8c1be5f6"
    "eu-central-1" = "ami-c7ee5ca8"
  }
}
``` 

* Tipo de instancia
* Región
* Id de cuenta AWS
* AMI de la VM según región

Creamos el main.tf:

``` 
terraform {
  required_version = ">= 0.10.7"
}

provider "aws" {
  region = "${var.region}"
  allowed_account_ids = ["${var.aws_id}"]
  profile = "openwebinars"
}
```

Y, por último, el archivo con la plantilla a definir (por ej, webserver.tf):

``` 
resource "aws_instance" "webservers" {
  ami = "${lookup(var.aws_amis, var.region)}" (Lookup busca entre la variable aws_amis al ser de type map.)
  instance_type = "${var.instance_type}"
  count = 3
  tags {
    Name = "webservers"
  }
}
```

Mediante esta plantilla indicamos que queremos lanzar tres instancias iguales con la imagen y región definidas. Iniciamos con terraform init, y lanzamos el terraform plan. 

**Reutilizar una plantilla**

Para este caso, vamos a hacer uso del mismo main.tf. Ahora queremos lanzar un servicio de balanceador de carga de AWS, definido en un archivo (elb.tf):

```
data "aws_availability_zones" "az" {}

resource "aws_elb" "elb-web" {
  name_prefix = "${var.project}-" (Es mejor hacer uso de la opción name_prefix en lugar de variable)

  "listener" {
    instance_port = 50
    instance_protocol = "http"
    lb_port = 80
    lb_protocol = "http"
  }

  availability_zones = ["${data.aws_availability_zones.az.names}"] (Esto devuelve una lista con las zonas disponibles)
}
``` 

Y hacemos uso de una serie de variables (variables.tf), que a fin de cuentas será el que ya teniamos salvo que se añade una variable, que he llamado project, donde damos nombre al balanceador:

``` 
variable "instance_type" {
  type = "string"
  default = "t2.micro"
}

variable "region" {
  type = "string"
  default = "eu-west-1"
}

variable "aws_id" {
  type = "string"
  default = "723002569774"
}

variable "aws_amis" {
  type = "map"
  default = {
    "eu-west-1" = "ami-acd005d5"
    "us-east-1" = "ami-8c1be5f6"
    "eu-central-1" = "ami-c7ee5ca8"
  }
}

variable "project" {
  type = "string"
  default = "web"
}

variable "environment" {  (En realidad, esta variable no la uso y si la opción name_prefix en elb.tf)
  type = "string"
  default = "prod"
}
``` 

Hecho todo esto, terraform plan y apply para ejecutar todo. 


**Crear y asociar varios recursos entre si**

Vamos a crear una VPC con subnets, dos públicas y dos privadas, una puerta de enlace y una ruta.

* vpc.tf

``` 
resource "aws_vpc" "vpc" {
  cidr_block = "${var.cidr}"
  enable_dns_hostnames = false
  enable_dns_support = false
  tags {
    Name = "openwebinars"
  }
}

resource "aws_subnet" "pub1" {
  cidr_block = "${var.pub1_cidr}"
  vpc_id = "${aws_vpc.vpc.id}"
  map_public_ip_on_launch = true
  availability_zone = "${data.aws_availability_zones.az.names[0]}"
  tags {
    Name = "pub1"
  }
}

resource "aws_subnet" "pub2" {
  cidr_block = "${var.pub2_cidr}"
  vpc_id = "${aws_vpc.vpc.id}"
  map_public_ip_on_launch = true
  availability_zone = "${data.aws_availability_zones.az.names[1]}"
  tags {
    Name = "pub2"
  }
}

resource "aws_subnet" "pri1" {
  cidr_block = "${var.pri1_cidr}"
  vpc_id = "${aws_vpc.vpc.id}"
  map_public_ip_on_launch = true
  availability_zone = "${data.aws_availability_zones.az.names[0]}"
  tags {
    Name = "pri1"
  }
}

resource "aws_subnet" "pri2" {
  cidr_block = "${var.pri2_cidr}"
  vpc_id = "${aws_vpc.vpc.id}"
  map_public_ip_on_launch = true
  availability_zone = "${data.aws_availability_zones.az.names[1]}"
  tags {
    Name = "pri2"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = "${aws_vpc.vpc.id}"
}

resource "aws_route" "default_route" {
  route_table_id = "${aws_vpc.vpc.default_route_table_id}"
  destination_cidr_block = "0.0.0.0/0"
  gateway_id = "${aws_internet_gateway.igw.id}"
}
``` 

Creamos dos recursos security group, uno para el balanceador y otro para las instancias.

* security_groups.tf

```
resource "aws_security_group" "elb-sg" {
  name = "elb-sg"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
    from_port = 80
    protocol = "tcp"
    to_port = 80
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port = 0
    protocol = "-1" (-1 significa a cualquier protocolo)
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "web-sg" {
  name = "web-sg"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
    from_port = 80
    protocol = "tcp"
    to_port = 80
    security_groups = ["${aws_security_group.elb-sg.id}"]
  }
  egress {
    from_port = 0
    protocol = "-1"
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Definimos el balanceador de carga (elb.tf):

``` 
resource "aws_elb" "elb-web" {
  name = "${var.environment}-${var.project}"
  cross_zone_load_balancing = true
  subnets = ["${aws_subnet.pub1.id}", "${aws_subnet.pub2.id}"]

  "listener" {
    instance_port = 80
    instance_protocol = "http"
    lb_port = 80
    lb_protocol = "http"
  }
  security_groups = ["${aws_security_group.elb-sg.id}"]
  instances = ["${aws_instance.webservers.*.id}"]
}
``` 

Y el main.tf quedaría del siguiente modo:

``` 
terraform {
  required_version = ">= 0.10.7"
}

provider "aws" {
  region = "${var.region}"
  allowed_account_ids = ["${var.aws_id}"]
  profile = "openwebinars"
}

data "aws_availability_zones" "az" {}
``` 

Ahora mismo, pese a todo, el balanceador de carga no tiene instancias asociadas. Por ende, hacemos uso del recurso aws_instance dentro de un archivo. Antes, crearemos una variable para la region.

* variables.tf

```  
variable "aws_region" {
  type = "string"
  default = "eu-west-1"
}
variable "cidr" {
  type = "string"
  default = "10.0.0.0/16"
}

variable "pub1_cidr" {
  type = "string"
  default = "10.0.0.0/24"
}

variable "pub2_cidr" {
  type = "string"
  default = "10.0.1.0/24"
}

variable "pri1_cidr" {
  type = "string"
  default = "10.0.10.0/24"
}

variable "pri2_cidr" {
  type = "string"
  default = "10.0.11.0/24"
}

variable "instance_type" {
  type = "string"
  default = "t2.micro"
}

variable "region" {
  type = "string"
  default = "eu-west-1"
}

variable "aws_id" {
  type = "string"
  default = "723002569774"
}

variable "aws_amis" {
  type = "map"
  default = {
    "eu-west-1" = "ami-acd005d5"
    "us-east-1" = "ami-8c1be5f6"
    "eu-central-1" = "ami-c7ee5ca8"
  }
}

variable "project" {
  type = "string"
  default = "web"
}

variable "environment" {
  type = "string"
  default = "prod"
}
``` 

Y, ahora sí, creamos el archivo que va a permitir asociar las instancias al balanceador de carga. Le volvemos a llamar *webserver.tf*.

``` 
resource "aws_instance" "webservers" {
  ami = "${lookup(var.aws_amis, var.aws_region)}"
  instance_type = "${var.instance_type}"
  vpc_security_group_ids = ["${aws_security_group.web-sg.id}"]
  subnet_id = "${aws_subnet.pri1.id}"
  count = 2
  tags {
    Name = "${var.environment}-webservers"
  }
}
``` 

Y ejecutamos todo el proceso:

``` 
terraform init
terraform plan
terraform apply
``` 

**Crear y asociar varios recursos entre si II**

En este paso, vamos a implantar un servicio de bases de datos, añadiendose a lo anterior. En primer término, voy a definir dos variables más, una para el username y otra para el password.

* variables.tf

``` 
variable "aws_region" {
  type = "string"
  default = "eu-west-1"
}
variable "cidr" {
  type = "string"
  default = "10.0.0.0/16"
}

variable "pub1_cidr" {
  type = "string"
  default = "10.0.0.0/24"
}

variable "pub2_cidr" {
  type = "string"
  default = "10.0.1.0/24"
}

variable "pri1_cidr" {
  type = "string"
  default = "10.0.10.0/24"
}

variable "pri2_cidr" {
  type = "string"
  default = "10.0.11.0/24"
}

variable "instance_type" {
  type = "string"
  default = "t2.micro"
}

variable "region" {
  type = "string"
  default = "eu-west-1"
}

variable "aws_id" {
  type = "string"
  default = "723002569774"
}

variable "aws_amis" {
  type = "map"
  default = {
    "eu-west-1" = "ami-acd005d5"
    "us-east-1" = "ami-8c1be5f6"
    "eu-central-1" = "ami-c7ee5ca8"
  }
}

variable "project" {
  type = "string"
  default = "web"
}

variable "environment" {
  type = "string"
  default = "prod"
}

variable "rds_username" {
  type = "string"
  default = "root"
}

variable "rds_passwd" {
  type = "string"
  default = "0penW3b1n4r$"
}
``` 

También vamos a tener que crear un security group adicional para nuestro RDS.

* security_groups.tf

``` 
resource "aws_security_group" "elb-sg" {
  name = "elb-sg"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
    from_port = 80
    protocol = "tcp"
    to_port = 80
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port = 0
    protocol = "-1"
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "web-sg" {
  name = "web-sg"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
    from_port = 80
    protocol = "tcp"
    to_port = 80
    security_groups = ["${aws_security_group.elb-sg.id}"]
  }
  egress {
    from_port = 0
    protocol = "-1"
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "rds-sg" {
  name = "rds-sg"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
    from_port = 3306
    protocol = "tcp"
    to_port = 3306
    security_groups = ["${aws_security_group.web-sg.id}"]
  }
  ingress {
    from_port = 3306
    protocol = "tcp"
    to_port = 3306
    cidr_blocks = ["85.137.199.237/32"]
  }
  egress {
    from_port = 0
    protocol = "-1"
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}
``` 

Hechas estas asignaciones, creamos el archivo .tf del servicio RDS.

* rds.tf

``` 
resource "aws_db_subnet_group" "subn-groups" {
  subnet_ids = ["${aws_subnet.pri1.id}", "${aws_subnet.pri2.id}"]
}

resource "aws_db_instance" "mydb" {
  instance_class = "db.t2.micro"
  identifier = "mydb"
  username = "${var.rds_username}"
  password = "${var.rds_passwd}"
  engine = "mysql"
  allocated_storage = 10
  storage_type = "gp2"
  multi_az = false
  db_subnet_group_name = "${aws_db_subnet_group.subn-groups.name}"
  vpc_security_group_ids = ["${aws_security_group.rds-sg.id}"]
  publicly_accessible = true
  skip_final_snapshot = true
}
``` 

¿Y qué quedaría? Pues generar el autoscaling para tener un servicio RDS de alta disponibilidad. En primer lugar, creamos en el archivo un launch configuration, que es una plantilla en base a la cual el autoscaling va a lanzar instancias. 

* webservers.tf

``` 
resource "aws_launch_configuration" "web-server" {
  name_prefix = "web-server-"
  image_id = "${lookup(var.aws_amis, var.aws_region)}"
  instance_type = "${var.instance_type}"
  key_name = "openwebinars"
  security_groups = ["${aws_security_group.web-sg.id}"]
  user_data = "${file("templates/userdata.tpl")}"
  lifecycle {
    create_before_destroy = true
  }
}
resource "aws_autoscaling_group" "as-web" {
  name = "${aws_launch_configuration.web-server.name}"
  launch_configuration = "${aws_launch_configuration.web-server.name}"
  max_size = 1
  min_size = 1
  load_balancers = ["${aws_elb.elb-web.id}"]
  vpc_zone_identifier = ["${aws_subnet.pub2.id}", "${aws_subnet.pub1.id}"]
  wait_for_elb_capacity = 1
  tag {
    key = "Name"
    propagate_at_launch = true
    value = "web-server"
  }
  lifecycle {
    create_before_destroy = true
  }
}
``` 

Y ejecutamos el proceso:

``` 
terraform init
terraform plan
terraform apply
```

Para eliminar la plantilla completa:

``` 
terraform destroy
``` 




























