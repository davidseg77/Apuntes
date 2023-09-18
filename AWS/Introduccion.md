# Curso de Introducción a AWS (OW)

## 1. Networking

### 1.1 VPC

Dentro de la VPC podemos crear las subnets. Asociamos las subnets con la VPC y las zonas de disponibilidad.
Para que las subnets públicas tengan asignadas automaticamente una IP pública tenemos que ir dentro de la subnet en cuestión a Subnet Actions y ahí activar en Modify auto-assign IP settings la opción.

### 1.2 Routing Table

Dentro de Route Table, lo creamos, le damos nombre y lo asociamos con la VPC. Se puede crear una para subnets públicas y otra para privadas. 

Por ejemplo, creo una route table para una subnet privada y después vamos a Subnet Associations dentro de la route table, Edit y seleccionamos las subnets privadas requeridas.

### 1.3 Internet Gateway

Conecta la PVC con Internet. Se referencia en la tabla de rutas y solo pueden llegar a Internet los recursos que tengan IP pública.

Vamos a Internet Gateway y le damos nombre. Vamos a Attach to VPC para asociarlo a una VPC. Acto seguido, habrá que asociarlo como ruta por defecto en la tabla de rutas. Para ello, vamos a la tabla de rutas de la subnet pública y en rutas, en edit añadimos una nueva ruta, la de nuestro IG. Por ej. 0.0.0.0/0.

### 1.4 NAT Gateway

Gateway de acceso a Internet para las subnets privadas. Ha de estar creada en una subnet pública con una tabla de rutas que tenga como ruta por defecto el IG.

Vamos a NAT Gateway, asociamos en que subnet ha de crearse (pública) y añadimos una IP elástica dando en crear new eip. 

Después vamos a Table Route y vamos a la route private. Vamos a add another route y en rutas, en edit añadimos una nueva ruta. Por ej. 0.0.0.0/0 en destination y en target añadimos la NAT Gateway creada.

### 1.5 Networks ACL

En Security, hallamos las Networks ACLs. Solo habría que darles nombre y asociarlas con la VPC requerida. En rule damos la prioridad, a menor número mayor prioridad. Preferiblemente menor de 100.

## 2. Route53

DNS de AWS. Podemos hallarlo en Servicios. Creamos una hosted zone con el dominio deseado e indicamos el tipo, bien público o privado desde una VPC. 

## 3. EC2

Máquinas virtuales de AWS. Cuenta con distintos apartados:

### 3.1 Security Group

Creamos dandole un nombre, una descripción y asociamos la VPC. En Inbound añadimos las reglas de entrada, en Outbound las de salida. 

### 3.2 Elastic IP

IP públicas que asigna Amazon. Para crearla vamos al menu de la izquierda a IP elasticas, vamos a Allocate new address y cuando tengamos la ip vamos a Actions y en associate address liberamos la ip completamente para que pueda ser utilizada.

Para asociar esta ip elastica a una instancia recien creada, vamos a Elastic IP en el menu y seleccionamos la IP deseada. En Actions, associate address e indicamos la instancia con la cual asociar. 

### 3.3 EBS

Elastic Block Storage. Proporciona volumenes de almacenamiento de bloques persistentes para las instancias EC2. Para ello, nos vamos a volumenes, Create Volume e importante es que estén siempre en la misma zona de disponibilidad que la instancia. Despues de crear todos los detalles, vamos a Actions, Attach volume y lo asociamos a la instancia. Con df -h podemos observar en la instancia nuestro volumen. 

Despues hacemos lo siguiente:

* Listamos los volumenes.

```
lsblk
```

* Creamos una partición con un formato si el volumen es nuevo

```
mkfs -t ext4 /dev/xvdf
```

* Y montamos el volumen

```
mount /dev/xvdf /mnt/
```

* Verificamos que está todo correcto

```
df -h 
```

## 4. Autoscaling

Dentro de Autoscaling, en Launch Configuration damos en Create Autoscaling Group. Antes de ello, habremos de crear un launch configuration. Es muy parecido a lanzar una instancia con respecto a las AMI. Podemos elegir una AMI ya en uso si se quisiera. Para que nos entendamos, esto sería como crear un template. 

Hecho esto, en Autoscaling Group podemos crearlo desde un Launch Configuration ya existente. Después, tendremos que ponerle un nombre, el tamaño de escala, la red y la subred a escalar. Creado todo, vamos a instancias y comprobamos si el autoscaling está funcionando.

## 5. ELB

Elastic Load Balancing. Para crearlo, previamente tendremos que crear un security group para este balanceador con su nombre, descripción, VPC... 

Tras esto, vamos a Load Balancers y damos nombre, asociamos VPC, creamos reglas de protocolos de puerto, por ejemplo para redireccionar. Luego indicamos las subredes sobre las que actuar, deben ser públicas. No asociamos instancias, ya que interesa asociar el ELB al Autoscaling. 

Para asociar el ELB al Autoscaling, vamos a Autoscaling Group y en Editar indicamos el Load Balancer. 

## 6. Cloudwatch

Servicio de monitorización. Se ve dentro de las instancias en Monitoring. Dentro de Autoscaling Group, podemos crear una politica de monitoreo mediante Scaling Policies. 

## 7. Bases de datos

RDS (Relational Databases Services). 
Antes de crear una RDS, habremos de crear un Security Group para ello. Damos nombre, VPC a asociar, y regla para abrir el puerto de MYSQL, por ejemplo, a nuestra IP. 

Cuando se crea, el siguiente paso es ir a RDS en servicios. En RDS creamos un DB Subnet Group, donde indicamos la VPC, la zona y la subnet deseada. 

Hecho esto, vamos a instancias y lanzamos una instancia RDS. Elegimos la base de datos, por ejemplo MySQL, el plan a usar, como DevTest, especificamos el motor de la base de datos, tipo de instancia y los datos de la base de datos. 

En Configuración avanzada, indicamos la VPC, el subnet group, Publicy Accessible en yes, El VPC security group y creamos.

Accedemos a la terminal y para conectarnos a la base de datos es importante añadir el endpoint que nos da la instancia de la base de datos. 

```
mysql -h endpoint -u root -p 
```

### 7.1 Crear cluster con ElasticCache

Creamos primero un Security Group con su nombre, VPC asociada y las reglas para conectarse. Custom TCP rule, TCP... 

Ahora vamos a ElasticCache en Servicios, creamos el subnet group e indicamos nombre, VPC, zona y subnet privadas. (Aquí ha de ser así)

Seleccionamos motor, memcache, le ponemos nombre, puerto 11211, subnet group, security group...

## 8. S3

Storage Service. En Servicios, S3, creamos un Bucket, damos nombre y región. Una vez creado, en properties, Static Website Hosting podemos indicar que este bucket va a ser host para una web. El documento index podría ser index.html, por ejemplo.

## 9. Usuarios y roles. AWS CLI

IAM. Identity and Access Management. 

En Servicios, en IAM, podemos crear un usuario y sus tipos de acceso a los recursos de AWS. Podemos añadirlo a un grupo, asignarle una serie de politicas... 

Cuando se crea el usuario, nos da la key secreta y la ID de access key. Con el comando **aws-configure**, podemos en la terminal dar acceso al usuario.
En el home del usuario se crea una carpeta .aws. Ahi estan las credenciales recien creadas y en el config la configuracion del output y de la región del usuario.







