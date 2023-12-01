# Cómo migrar servidores Linux, parte 2: transferir datos principales

## 1. Introducción

Hay muchos escenarios en los que es posible que deba trasladar sus datos y requisitos operativos de un servidor a otro. Es posible que necesite implementar sus soluciones en un nuevo centro de datos, actualizar a una máquina más grande o realizar la transición a un nuevo hardware o un nuevo proveedor de VPS.

Cualesquiera que sean sus motivos, hay muchas consideraciones diferentes que debe tener en cuenta al migrar de un sistema a otro. Obtener configuraciones funcionalmente equivalentes puede resultar difícil si no se trabaja con una solución de gestión de configuración como Chef, Puppet o Ansible. No solo necesita transferir datos, sino también configurar sus servicios para que funcionen de la misma manera en una máquina nueva.

En el último artículo, preparó su servidor para la migración de datos . En este punto, su sistema de origen y de destino deberían poder comunicarse; el sistema de destino debería tener acceso SSH al sistema de origen. También debe tener una lista del software y los servicios que necesita transferir, incluidos los números de versión. En esta guía, continuará donde lo dejó y comenzará la migración real a su nuevo servidor.


## 2. Crear un script de migración

Este tutorial y el siguiente se centrarán en crear y agregar un bashs cript de shell de migración a medida que avanza. Utilizará una serie de herramientas de Linux de bajo nivel y, en lugar de intentar ejecutarlas todas de forma interactiva, su objetivo debe ser terminar con un conjunto reproducible de pasos que puedan capturar las partes relevantes de la configuración de su servidor.

Mientras escribe este script, debería poder ejecutarlo de forma iterativa a medida que avanza. La mayoría de las herramientas utilizadas en este tutorial, como rsync, solo transferirán datos si se han modificado desde la última ejecución, de modo que pueda repetir comandos de forma segura sin tener que preocuparse por hacerlos redundantes. Debido a que configuró SSH para permitir la conexión a la máquina original (de origen) desde el nuevo servidor (de destino), debería trabajar desde el servidor de destino a lo largo de este tutorial.

Puede crear este script en su directorio de inicio, utilizando nano o su editor de texto favorito:

``` 
nano ~/sync.sh
``` 

En la primera línea del archivo, agregue un encabezado de secuencia de comandos, también conocido como shebang. Esto le dice al script con qué intérprete ejecutar de forma predeterminada. #!/bin/bash significa que el script utilizará de forma predeterminada el bash shell, que es el shell más potente y con mayor soporte disponible en la mayoría de los sistemas.

``` 
#!/bin/bash
``` 

Guarde y cierre el archivo por ahora. Si está utilizando nano, presione Ctrl+X, luego cuando se le solicite Y y luego Enter.

De vuelta en la línea de comando, haga que el script sea ejecutable usando chmod:

``` 
chmod 700 ~/sync.sh
``` 

Después de haber hecho el script ejecutable y haber agregado el shebang, puede ejecutarlo llamándolo directamente:

``` 
~/sync.sh
``` 

No producirá ningún resultado todavía, ya que el script está vacío. Debe probar el script periódicamente durante el resto de este tutorial según sea necesario. Como en el tutorial anterior de esta serie, es posible que necesite ejecutarlo con sudo permisos, según los pasos que agregue al script.


## 3. Instalar los programas y servicios necesarios

El primer paso que agregará a su secuencia de comandos de migración será restaurar los paquetes que marcó para la migración en el tutorial anterior.

### 3.1 Agregar repositorios adicionales

Antes de hacer eso, querrás conectarte nuevamente a tu servidor original (de origen) en una terminal separada, para verificar si has instalado software de algún repositorio de terceros. Si es así, no podrá reinstalar esos paquetes en su nuevo entorno sin configurar primero esas fuentes de paquetes adicionales.

En entornos Ubuntu y Debian, puede ver si hay repositorios alternativos en su sistema fuente investigando algunas ubicaciones:

``` 
cat /etc/apt/sources.list
Output
…
## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu impish partner
# deb-src http://archive.canonical.com/ubuntu impish partner

deb http://security.ubuntu.com/ubuntu impish-security main restricted
# deb-src http://security.ubuntu.com/ubuntu impish-security main restricted
deb http://security.ubuntu.com/ubuntu impish-security universe
# deb-src http://security.ubuntu.com/ubuntu impish-security universe
deb http://security.ubuntu.com/ubuntu impish-security multiverse
# deb-src http://security.ubuntu.com/ubuntu impish-security multiverse
``` 

Esta es la lista de fuentes del paquete principal; debido a que es un archivo único, puede usarlo catpara generar su contenido. Si la última línea del archivo contiene una ubuntu.com dirección, entonces probablemente no haya agregado ningún repositorio de terceros a este archivo. También se pueden enumerar repositorios adicionales en el sources.list.d directorio:

``` 
ls /etc/apt/sources.list.d
Output
droplet-agent.list  elastic-7.x.list  nodesource.list
``` 

Si este directorio no está vacío, puede cat verificar los archivos individuales en cada uno de los repositorios:

``` 
cat /etc/apt/sources.list.d/elastic-7.x.list
Output
deb https://artifacts.elastic.co/packages/7.x/apt stable main
``` 

Esto le indicará la URL del repositorio que deberá volver a agregar a su máquina de destino. En la mayoría de los casos, puedes hacerlo con el add-apt-repository comando:

``` 
sudo add-apt-repository repo_url
``` 

En RHEL, Rocky o Fedora Linux, puede utilizar dnf para enumerar los repositorios configurados para el servidor:

``` 
dnf repolist enabled
```

Luego puede agregar repositorios adicionales a su sistema de destino usando dnf config-manager:

``` 
sudo dnf config-manager --add-repo repo_url
```

Si realiza algún cambio en su lista de origen, agréguelo como comentarios en la parte superior de su secuencia de comandos de migración en su sistema de destino. De esta manera, si tiene que comenzar desde una instalación nueva, sabrá qué procedimientos deben realizarse antes de intentar una nueva migración.

``` 
nano ~/sync.sh

#!/bin/bash

#############
# Prep Steps
#############

# Add additional repositories to /etc/apt/source.list
#       deb http://example.repo.com/linux/deb stable main non-free
``` 

Luego, guarde y cierre el archivo.

### 3.2 Especificación de restricciones de versión e instalación

Ahora tiene las fuentes de su paquete actualizadas en su máquina de destino para que coincidan con su máquina de origen.

En máquinas Ubuntu o Debian, ahora puede instalar las versiones del software que necesita en su máquina de destino escribiendo:

``` 
sudo apt update
sudo apt install package_name=version_number
``` 

Si la versión del paquete que intenta hacer coincidir tiene más de unos meses, es posible que se haya eliminado de los repositorios oficiales. En este caso, podría intentar buscar la versión anterior de los .deb paquetes (por ejemplo, explorando repositorios anteriores más antiguos o PPA de terceros) y sus dependencias e instalarlos manualmente con:

``` 
sudo dpkg -i package.deb
``` 

Sin embargo, debe hacer esto con mucha moderación para evitar crear una situación en la que tenga demasiados paquetes con versiones que no coinciden. Si las versiones anteriores de software no están disponibles, pruebe primero las versiones más recientes disponibles para ver si aún satisfacen sus necesidades, para evitar imponer requisitos obsoletos.

Para RHEL, Rocky o Fedora Linux, puede instalar versiones específicas de software escribiendo:

``` 
Sudo dnf install package_name-version_number
``` 

Si necesita buscar archivos rpm que se han eliminado del repositorio y sustituirlos por versiones más nuevas, puede instalarlos con dnf:

``` 
dnf install package_name.rpm
``` 

Nuevamente, realice un seguimiento de las operaciones que está realizando aquí. Puedes incluirlos como comentarios en el script que estás creando:

``` 
nano ~/sync.sh
```

``` 
#!/bin/bash

#############
# Prep Steps
#############

# Add additional repositories to /etc/apt/source.list
#       deb http://example.repo.com/linux/deb stable main non-free

# Install necessary software and versions
#       apt-get update
#       apt-get install apache2=2.2.22-1ubuntu1.4 mysql-server=5.5.35-0ubuntu0.12.04.2 libapache2-mod-auth-mysql=4.3.9-13ubuntu3 php5-mysql=5.3.10-1ubuntu3.9 php5=5.3.10-1ubuntu3.9 libapache2-mod-php5=5.3.10-1ubuntu3.9 php5-mcrypt=5.3.5-0ubuntu1
```

Nuevamente, guarde y cierre el archivo.


## 4. Comience a transferir datos

La transferencia real de datos no suele ser la parte de la migración que requiere más mano de obra, pero puede ser la que requiere más tiempo. Si está migrando un servidor con una gran cantidad de datos, probablemente sea una buena idea comenzar a transferir datos lo antes posible.

Rsync es una poderosa herramienta que proporciona una amplia gama de opciones para replicar archivos y directorios en muchos entornos diferentes, con validación de suma de comprobación incorporada y otras características. Identifique los directorios cuyos datos desee transferir y agregue rsync comandos a su secuencia de comandos de migración.

Un rsync comando de muestra se ve así:

``` 
rsync -azvP --progress source_server:/path/to/directory/to/transfer /path/to/local/directory
```

-**azvP** es un conjunto típico de opciones de Rsync. Como desglose de lo que hace cada uno de ellos:

* **a** habilita el “Modo de archivo” para esta operación de copia, que conserva los tiempos de modificación de los archivos, los propietarios, etc. También es el equivalente a proporcionar cada una de las -rlptgoD opciones individualmente (sí, de verdad). En particular, la -r opción le dice a Rsync que recurra a subdirectorios para copiar también archivos y carpetas anidados. Esta opción es común a muchas otras operaciones de copia, como cpy scp.
  
* **z** comprime los datos durante la propia transferencia, si es posible. Esto es útil para cualquier transferencia a través de conexiones lentas, especialmente cuando se transfieren datos que se comprimen de manera muy efectiva, como registros y otro texto.
  
* **v** habilita el modo detallado, para que pueda leer más detalles de su transferencia mientras está en progreso.
  
* **P** le dice a Rsync que conserve copias parciales de cualquier archivo que no se transfiera por completo, para que las transferencias puedan reanudarse más tarde.
  
Con la adición de rsync comandos, su secuencia de comandos de sincronización ahora podría verse así:

```
#!/bin/bash

#############
# Prep Steps
#############

# Add additional repositories to /etc/apt/source.list
#       deb http://example.repo.com/linux/deb stable main non-free

# Install necessary software and versions
#       apt-get update
#       apt-get install apache2=2.2.22-1ubuntu1.4 mysql-server=5.5.35-0ubuntu0.12.04.2 libapache2-mod-auth-mysql=4.3.9-13ubuntu3 php5-mysql=5.3.10-1ubuntu3.9 php5=5.3.10-1ubuntu3.9 libapache2-mod-php5=5.3.10-1ubuntu3.9 php5-mcrypt=5.3.5-0ubuntu1

#############
# File Transfer
#############


# Rsync web root
rsync -azvP --progress source_server:/var/www/site1 /var/www/

# Rsync home directories
. . .
``` 

Recuerde que estos comandos se pueden volver a ejecutar y no transferirán ningún dato nuevo a menos que los archivos de origen hayan cambiado, por lo que puede agregar a este script de forma incremental a medida que lo prueba. Sea conservador e iterativo sobre qué directorios incluye.

### 4.1 Migrar bases de datos y otros datos que no son archivos

Tenga en cuenta que no necesariamente puede copiar todos sus datos rsync sin ninguna preparación adicional. Muchas aplicaciones, como las bases de datos, almacenan sus datos relevantes en múltiples "archivos" reales en su sistema de archivos, para optimizar el acceso utilizando técnicas como Database Sharding. Por lo general, no se debe acceder a estos archivos ni copiarlos tal como están; en su lugar, una base de datos expone datos a través de una interfaz de consulta.

Afortunadamente, casi todas las aplicaciones que implementan su propio almacenamiento incluirán algún mecanismo de exportación e importación de datos a archivos normales, para que puedan copiarse normalmente durante migraciones como esta. Por ejemplo, si está utilizando MySQL, puede revisar Cómo importar y exportar bases de datos. Luego puede transferir estas exportaciones entre servidores usando rsynco scp.


## 5. Modificar los archivos de configuración

Aunque parte del software volverá a funcionar correctamente después de transferir los detalles y datos de configuración relevantes del servidor original, será necesario modificar muchas configuraciones.

Esto presenta un pequeño problema para el script de sincronización. Si ejecuta el script para sincronizar sus datos y luego modifica los valores para reflejar la información correcta para su nuevo destino, estos cambios se borrarán la próxima vez que ejecute el script. Para resolver este problema, puede agregar pasos adicionales al script que modificarán esos datos después de transferirlos.

Linux incluye una serie de utilidades principales que son muy útiles para este tipo de secuencias de comandos de texto. Dos de ellos son sed y awk. En general, sed es más sencillo de usar si realiza modificaciones en texto no estructurado mediante expresiones regulares y awk es más útil para análisis más complejos de texto formateado o datos tabulares. 

De esta manera, su secuencia de comandos de sincronización puede ejecutar comandos sed o awk inmediatamente después rsync, de modo que sus archivos se modifiquen automáticamente según sea necesario después de ser transferidos.

sed la sintaxis se ve así:

```
sed -i 's/string_to_match/string_to_replace_it_with/g' file_to_edit
``` 

La -i bandera significa que el archivo se modificará en su lugar en lugar de crear un archivo de salida separado. Los s y g no cambian y son una sedconvención regular. También puede utilizar expresiones regulares dentro del cadena_para_coincidir. Intente agregar un sed comando a su sync.sh:

``` 
rsync -avz --progress source_server:/etc/mysql/* /etc/mysql/

# Change socket to '/mysqld/mysqld.sock'
sed -i 's/\/var\/run\/mysqld\/mysqld.sock/\/mysqld\/mysqld.sock/g' /etc/mysql/my.cnf
``` 

Esto cambiará cada instancia de /var/run/mysqld/mysqld.sock in /etc/mysql/my.cnf a /mysqld/mysqld.sock/g. El \ carácter se utiliza para preceder a los / caracteres porque, de lo contrario, se analizarían como el final de la sed expresión. Esto se conoce como escape de caracteres especiales. Asegúrese de que sus sed comandos vengan después de los rsync comandos.

Puede utilizarlo awk para texto formateado de la misma manera que utilizó sed para texto no estructurado. Por ejemplo, el /etc/shadow archivo se divide en pestañas delimitadas por el carácter de dos puntos (:), que se ven así:

``` 
vault:!:18941::::::
stunnel4:!:18968:0:99999:7:::
sammy:$6$bscTWIVxvy.KhkO8$NJNhpABhJoybG/vDiRzQ2y9OFEd6XtqgCUG4qkuWfld97VEXH8/jUtc7oMtOC34V47VE7HjdpMMv37Aftdb7C/:18981:0:99999:7:::
```

Podrías usar awk para eliminar los datos de la segunda “columna” (es decir, entre el primer y segundo :carácter), así:

``` 
awk 'BEGIN { OFS=FS=":"; } $1=="root" { $2=""; } { print; }' /etc/shadow > shadow.tmp && mv shadow.tmp /etc/shadow
``` 

Este comando le dice a awk que tanto el delimitador de entrada como el de salida deben analizarse como:. Luego especifica que si la columna 1 es igual a "raíz", entonces la columna 2 debe establecerse como una cadena vacía. A diferencia de sed, awk no admite directamente la edición de archivos en el lugar, por lo que este script realiza pasos equivalentes a escribir en un archivo temporal y luego usar mv para sobrescribir la entrada original con el archivo temporal.

Siempre puede agregar comentarios a su script de migración (en líneas precedidas por #) para documentar correcciones o cambios en curso en sus archivos.


## 6. Conclusión

Ahora debería tener toda la información que necesita para migrar sus entornos de aplicaciones y sus datos a su nuevo servidor. También debe tener documentación buena y reproducible para este proceso en caso de que alguna vez necesite volver a implementar su pila en un nuevo sistema.

En el tutorial final de esta serie, revisará cómo transferir y probar cualquier servicio del sistema persistente en su nuevo servidor.




