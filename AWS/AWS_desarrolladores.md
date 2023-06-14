# Curso de AWS para desarrolladores (OW)

### Lambda

En Servicios, Lambda. Creamos función, author from scratch, ya que queremos crearla desde cero, damos nombre y escogemos runtime. Podemos añadir permisos, asociar a una VPC... Creamos.

En Code, podemos manejar el código y testearlo. Para poder ejecutarlo externamente, añadimos trigger y seleccionamos API Gateway, Cloudwatch, LoadBalancer...

Dentro de API Gateway escogemos una API de tipo HTTP. Damos nombre, y añadimos el trigger.

Si ahora cambiamos el código y damos a Deploy, podrá mostrarse el contenido externamente.

### SQS

Amazon Simple Queue Service (SQS) es un servicio de mensajería completamente administrado que permite enviar, almacenar y recibir mensajes entre componentes de aplicaciones distribuidas y sistemas. SQS forma parte de Amazon Web Services (AWS) y está diseñado para facilitar la construcción de sistemas escalables y tolerantes a errores.

En servicios, Simple Queue Service. Creamos Queue, le damos nombre y en política de acceso le damos avanzado. Creamos, copiamos la URL que nos da y vamos a la terminal. 

### Despliegue de aplicación con Elastic Beanstalk

Vamos a Servicios, Elastic Beanstalk, creamos aplicación y le damos nombre. 

Elegimos plataforma, por ejemplo docker, la rama de plataforma y versión. Podemos subir nuestro propio código, e indicar la configuración deseada. Single instance (Free tier) es la más simple. 

Creamos app, y a su vez nos creará un entorno.

Si vamos a EC2, en instancias ya tendremos nuestra aplicación bajo una instancia. Nos creará automáticamente una VPC, con subnet, security group...

Si volvemos a Elastic Beanstalk, tendremos una URL que si la llevamos al navegador nos mostrará nuestra aplicación corriendo. 

