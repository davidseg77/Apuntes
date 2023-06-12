# Curso Azure AZ-900 I

### Creación de una máquina virtual desde el portal

Dentro del buscador, Virtual Machine. Añadimos la suscripción correspondiente, y creamos o elegimos el grupo de recursos. Se indican aspectos como el SO, tamaño, usuario y contraseña...

También se indican los puertos de entrada. Hecho esto, pasamos a la opción de discos, de ahí a la de redes y posteriormente a la de administración. La opción de diagnóstico de arranque se puede deshabilitar.

Cuando se cree, si volvemos en el buscador a poner Virtual Machine, veremos como ya nos aparece. Si accedemos a ella, y le damos a Conectar, podremos hacerlo desde distintos protocolos, por ejemplo RDP. Habrá que descargarse el archivo que nos pide, y una vez introducida las credenciales accederemos a nuestra máquina.

En una máquina Windows Server, si queremos que pase a ser un servidor como tal tendremos que irnos a Power Shell en modo administrador e añadir el siguiente comando:

```
install-windowsfeature -name web-server -includemanagementtools
```

Con esto, convertimos nuestra máquina en un servidor web.

### Creación de una máquina virtual desde CLI

Para ello, vamos a Cloud Shell, cuyo icono está arriba junto al buscador. Eso si, habrá que crear un tipo de almacenamiento previamente y ya podremos acceder a la shell.

Podemos indicar si queremos trabajar en bash o power shell, preferiblemente bash.

Los comandos para crear la VM son los siguientes:

Para crear el grupo de recursos.

```
az group create --name x --location eastus (por ejemplo)
```

Para ver lo grupos de recursos creados.

```
az group list --output table
```

Para crear la máquina

```
az vm create \
--name x \
--resource-group X \
--image ubuntults \
--location eastus \
--admin-username x \
--admin-password x 
```

### Creación de una máquina virtual desde una plantilla

Plantillas de inicio rápido de Azure en google, accedemos a la web que nos indica y escogemos la opción Deploy a simple Windows VM. Implementar en Azure y ya pasamos a nuestro portal con la plantilla. Tendremos que decidir que parametros queremos pasarle a nuestra VM, para ello vamos a editar plantilla. Está en formato json.

Añadimos los valores oportunos y guardamos. La máquina se creará sin mayores dificultades.

### Creación de una máquina virtual desde Power Shell

Vamos a Cloud Shell. Cambiamos el entorno a Power Shell y añadimos los siguientes comandos:

Para crear el grupo de recursos.

```
new-azresourcegroup -name x -location eastus
```

Para asegurarnos que se ha creado correctamente

``` 
get-azresourcegroup | format-table
``` 

Para crear la máquina virtual

```
new-azvm -resourcegroup "myrgps" -name "myvmps" -location "east us" -virtualnetworkname "myvnetps" -subnetname "mysubnetps" -securitygroupname "mysrgps" -publicaddressname "mypublicps"
```

### Creación de una red virtual

Dentro del portal de Azure, buscamos redes virtuales. Damos en crear e indicamos suscripcion, grupo de recursos, región...

Nos muestra la dirección de red y subred, que se podría editar, y damos a crear. 

Ahora creamos una VM cuyo grupo de recursos va a ser el de la red virtual. Una vez dados los datos correspondientes, vamos al apartado Redes y vemos que por defecto la VM se halla asignada a la red virtual y subred creadas antes. 

Hacemos lo propio con otra VM y hacemos ping entre ellas para verificar que la red está operativa entre sendas máquinas.

### Creación de una base de datos SQL

Dentro del portal de Azure, buscamos base de datos SQL. Hacemos clic sobre SQL Database. 

Creamos un grupo de recursos, le damos un nombre a la base de datos y especificamos si tenemos un servidor. Si no lo tenemos, hacemos clic en crear nuevo servidor. Ahí le damos un nombre, usuario, contraseña, ubicación...

Hecho esto, vamos a la pestaña de Redes, donde le decimos que podemos conectarnos con nuestro servidor. Metodo de conectividad con un punto de conexión público. Permitimos que los servicios y recursos de Azure accedan a este servidor, pero no agregamos IP del cliente actual. 

Vamos a la configuración adicional, donde podemos usar datos existentes o los de muestra de Azure. Damos a crear.

En el menu de la izquierda, tenemos el Editor de consultas. En caso de salir un error de no conexión con el servidor, tendremos que hacer lo siguiente.

Vamos al apartado Información general y hacemos clic en la opción establecer Firewall del servidor. Es ahí donde podemos poner reglas de firewall. Damos en Agregar IP del cliente y guardar. Con esto, el error debería estar solventado.











