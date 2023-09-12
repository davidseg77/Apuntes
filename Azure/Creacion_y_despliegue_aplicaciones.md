# Creación de una aplicación en varios contenedores.

## 1. Creación de una aplicación en varios contenedores (Caso práctico)

Vamos a realizar la implementación de una aplicación wordpress de varios contenedores en Web App for Containers de Azure desde la Cloud Shell, para ver otra línea de comando más amigable. Para ello vamos a hacer uso de Docker Compose. Éste será el fichero:

``` 
version: '3.3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
``` 

Desde el portal de Azure, abrimos una Cloud Shell. Esto se hace en los iconos de la barra superior de Azure, junto a la barra de busqueda.

Nos pedirá que creemos una cuenta de almacenamiento, la cual ya tengo creada para mi suscripción. Se nos abre una terminal bash en la que podremos cargar ficheros que tenemos en nuestro equipo local.

Cargamos el fichero compose-wordpress.yaml, desde el icono que aparece en la parte superior en forma de página con una esquina plegada.
Cuando se complete aparecerá una ventana de diálogo.
Comprobamos que se ha transferido el fichero yaml.

Cloud Shell ya tiene integrada la CLI de Azure, por tanto, procedemos a crear un nuevo grupo de recursos para esta aplicación, o podemos usar un grupo que ya tengamos creado:

``` 
az group create –location westeurope –name recurso2
``` 

Una vez creado, creamos un plan de Azure App Service en el grupo de recursos que hemos creado:

``` 
az appservice plan create –name plan2 –resource-group recurso2 –sku F1
–is-linux
``` 

Ahora vamos a crear la aplicación utilizando el fichero de Docker compose:

``` 
az webapp create --resource-group recurso2 –-plan plan2 –-name davidwp
--multicontainer-config-type compose --multicontainer-config-file
compose-wordpress.yaml
``` 

Cuando se haya creado, ya podremos acceder desde el navegador a nuestra aplicación.

No se recomienda ejecutar la base de datos en un contenedor en producción, puesto que los contenedores locales no son escalables. Vamos a usar Azure Database for MySQL que sí se pueden escalar.
Para ello, creamos un servidor de Azure Database for MySQL con el comando ‘az mysql server create’:

``` 
az mysql server create --resource-group recurso2 --name mydavid --
location westeurope --admin-user adminuser --admin-password
Adminp@ssword --sku-name B_Gen5_1 --version 5.7
``` 

Ahora debemos añadir una nueva regla al firewall para que la base de datos permita el acceso solo para los otros recursos de Azure, y eso se consigue poniendo IP de inicio 0.0.0.0 e IP de fin 0.0.0.0:

``` 
az mysql server firewall-rule create --name AzureIPs --server mydavid -
-resource-group recurso2 --start-ip-address 0.0.0.0 --end-ip-address
0.0.0.0
```

Una vez creado el servidor, procedemos a crear la base de datos:

``` 
az mysql db create --resource-group recurso2 --server-name mydavid --
name wordpress
``` 

Y también tenemos que crear las variables de entorno para la base de datos:

``` 
az webapp config appsettings set --resource-group recurso2 --name
davidwp --settings WORDPRESS_DB_HOST="mydavid.mysql.database.azure.com"
WORDPRESS_DB_USER="adminuser@mydavid"
WORDPRESS_DB_PASSWORD="DBp@ssword" WORDPRESS_DB_NAME="wordpress"
``` 

Ahora nuestra aplicación guarda la información en la base de datos creada mediante Azure Database for MySQL. Ya podemos proceder a la instalación de wordpress.


## 2. Despliegue sencillo de una página web estática.

Procedemos al despliegue de una página web estática de una manera simple y sencilla. Para ello, tengo preparado un repositorio en github, con el código html necesario. Vamos a realizar, desde la Cloud Shell, una clonación de dicho repositorio:

``` 
git clone https://github.com/FranBuzon/html-docs-hello-world
``` 

Ahora realizamos, dentro del nuevo directorio, la creación de una web app con el comando ‘az webapp up’, y como parámetros solo debemos especificar la localización, el nombre de la aplicación y que el código es html:

``` 
az webapp up --location westeurope --name staticfranpage –-html
``` 

Este comando creará un grupo de recursos con un nombre predeterminado, un plan de App Service predeterminado, una aplicación con el nombre que hemos puesto e implementará con ZIP los archivos desde el directorio local hasta la aplicación web.

Entramos en la URL para comprobar.

Ahora bien, si queremos realizar algún cambio en nuestra página estática, nos vamos al Cloud Shell de nuevo, editamos el fichero index.html, y volvemos a lanzar el comando anterior para actualizar los cambios:

``` 
az webapp up --location westeurope --name staticfranpage –-html
``` 

Vemos que dice que esa aplicación ya existe y que ejecutará el contenido en ella. Cuando finalice solo debemos recargar nuestra página para ver los cambios realizados.


