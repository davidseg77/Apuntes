# Cómo compartir datos entre contenedores Docker en Ubuntu 22.04

## 1. Requisitos previos

Para seguir este artículo, necesitará un servidor Ubuntu 22.04 con lo siguiente:

* Un usuario no root con privilegios sudo.
* Docker instalado


## 2. Crear un volumen independiente

Introducido en la versión 1.9 de Docker, el docker volume create comando le permite crear un volumen sin relacionarlo con ningún contenedor en particular. Utilizará este comando para agregar un volumen llamado DataVolume1:

``` 
docker volume create --name DataVolume1
``` 

Se muestra el nombre, lo que indica que el comando fue exitoso:

``` 
Output
DataVolume1
``` 

Para utilizar el volumen, creará un nuevo contenedor a partir de la imagen de Ubuntu y usará la --rm bandera para eliminarlo automáticamente cuando salga. También lo usarás -v para montar el nuevo volumen. -v requiere el nombre del volumen, dos puntos y luego la ruta absoluta donde debe aparecer el volumen dentro del contenedor. Si los directorios en la ruta no existen como parte de la imagen, se crearán cuando se ejecute el comando. Si existen, el volumen montado ocultará el contenido existente:

``` 
docker run -ti --rm -v DataVolume1:/datavolume1 ubuntu
``` 

Mientras esté en el contenedor, escriba algunos datos en el volumen:

``` 
echo "Example1" > /datavolume1/Example1.txt
``` 

Debido a que usó la --rm bandera, su contenedor se eliminará automáticamente cuando salga. Sin embargo, su volumen seguirá siendo accesible.

```
exit
``` 

Puede verificar que el volumen está presente en su sistema con docker volume inspect:

``` 
docker volume inspect DataVolume1
Output
[
    {
        "CreatedAt": "2018-07-11T16:57:54Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/DataVolume1/_data",
        "Name": "DataVolume1",
        "Options": {},
        "Scope": "local"
    }
]
``` 

**Nota:** Incluso puede consultar los datos del host en la ruta que figura como Mount point. Sin embargo, debe evitar modificarlo, ya que puede dañar los datos si las aplicaciones o los contenedores no se dan cuenta de los cambios.

A continuación, inicie un nuevo contenedor y adjunte DataVolume1:

```
docker run --rm -ti -v DataVolume1:/datavolume1 ubuntu
``` 

Verificar el contenido:

``` 
cat /datavolume1/Example1.txt
Output
Example1
``` 

Salir del contenedor:

``` 
exit
``` 

En este ejemplo, creó un volumen, lo adjuntó a un contenedor y verificó su persistencia.


## 3. Crear un volumen que persista cuando se retira el contenedor

En el siguiente ejemplo, creará un volumen al mismo tiempo que el contenedor, eliminará el contenedor y luego adjuntará el volumen a un contenedor nuevo.

Utilizará el docker run comando para crear un nuevo contenedor usando la imagen base de Ubuntu. -t nos dará una terminal, y -i nos permitirá interactuar con ella. Para mayor claridad, utilizará --name para identificar el contenedor.

La -v bandera nos permitirá crear un nuevo volumen, al que llamarás DataVolume2. Utilizará dos puntos para separar este nombre de la ruta donde se debe montar el volumen en el contenedor. Finalmente, especificará la imagen base de Ubuntu y confiará en el comando predeterminado en el archivo Docker de la imagen base de Ubuntu, bash para colocarnos en un shell:

``` 
docker run -ti --name=Container2 -v DataVolume2:/datavolume2 ubuntu
``` 

Mientras esté en el contenedor, escribirá algunos datos en el volumen:

``` 
echo "Example2" > /datavolume2/Example2.txt
``` 

``` 
cat /datavolume2/Example2.txt
Output
Example2
``` 

Salir del contenedor:

``` 
exit
``` 

Cuando reinicie el contenedor, el volumen se montará automáticamente:

``` 
docker start -ai Container2
``` 

Verifique que el volumen realmente se haya montado y que sus datos aún estén en su lugar:

``` 
cat /datavolume2/Example2.txt
Output
Example2
``` 

Finalmente, salga y limpie:

``` 
exit
``` 

Docker no nos permitirá eliminar un volumen si un contenedor hace referencia a él. Para ver qué sucede, intente:

``` 
docker volume rm DataVolume2
``` 

El mensaje nos indica que el volumen todavía está en uso y proporciona la versión larga del ID del contenedor:

``` 
Output
Error response from daemon: unable to remove volume: remove DataVolume2: volume is in use - [d0d2233b668eddad4986313c7a4a1bc0d2edaf0c7e1c02a6a6256de27db17a63]
```

Puede utilizar el ID del mensaje de error anterior para eliminar el contenedor:

``` 
docker rm d0d2233b668eddad4986313c7a4a1bc0d2edaf0c7e1c02a6a6256de27db17a63
Output
d0d2233b668eddad4986313c7a4a1bc0d2edaf0c7e1c02a6a6256de27db17a63
``` 

Quitar el contenedor no afectará el volumen. Puede ver que todavía está presente en el sistema enumerando los volúmenes con docker volume ls:

``` 
docker volume ls
Output
DRIVER              VOLUME NAME
local               DataVolume2
``` 

Y puedes usar docker volume rm para eliminarlo:

``` 
docker volume rm DataVolume2
``` 

En este ejemplo, creó un volumen de datos vacío al mismo tiempo que creó un contenedor. En el siguiente ejemplo, explorará lo que sucede cuando crea un volumen con un directorio contenedor que ya contiene datos.


## 4. Crear un volumen a partir de un directorio existente con datos

Generalmente, crear un volumen de forma independiente docker volume create y crear uno mientras se crea un contenedor son equivalentes, con una excepción. Si crea un volumen al mismo tiempo que crea un contenedor y proporciona la ruta a un directorio que contiene datos en la imagen base, esos datos se copiarán en el volumen.

Como ejemplo, creará un contenedor y agregará el volumen de datos en /var, un directorio que contiene datos en la imagen base:

``` 
docker run -ti --rm -v DataVolume3:/var ubuntu
``` 

Todo el contenido del /var directorio de la imagen base se copia en el volumen y puede montar ese volumen en un nuevo contenedor.

Salga del contenedor actual:

``` 
exit
``` 

Esta vez, en lugar de confiar en el bash comando predeterminado de la imagen base, emitirá su propio ls comando, que mostrará el contenido del volumen sin ingresar al shell:

``` 
docker run --rm -v DataVolume3:/datavolume3 ubuntu ls datavolume3
``` 

El directorio datavolume3 ahora tiene una copia del contenido del /var directorio de la imagen base:

``` 
Output
backups
cache
lib
local
lock
log
mail
opt
run
spool
tmp
```

Es poco probable que desee realizar el montaje /var/ de esta manera, pero puede resultar útil si ha creado su propia imagen y desea una forma sencilla de conservar los datos. En el siguiente ejemplo, demostrará cómo se puede compartir un volumen entre varios contenedores.


## 5. Compartir datos entre varios contenedores Docker

Hasta ahora, ha adjuntado un volumen a un contenedor a la vez. A menudo, querrás adjuntar varios contenedores al mismo volumen de datos. Esto es relativamente sencillo de lograr, pero hay una advertencia crítica: en este momento, Docker no maneja el bloqueo de archivos. Si necesita que varios contenedores escriban en el volumen, las aplicaciones que se ejecutan en esos contenedores deben diseñarse para escribir en almacenes de datos compartidos para evitar la corrupción de datos.

### 5.1 Crear Container4 y DataVolume4

Úselo docker run para crear un nuevo contenedor Container4 con un nombre y un volumen de datos adjunto:

``` 
docker run -ti --name=Container4 -v DataVolume4:/datavolume4 ubuntu
``` 

A continuación, creará un archivo y agregará algo de texto:

``` 
echo "This file is shared between containers" > /datavolume4/Example4.txt
```

Luego, saldrás del contenedor:

``` 
exit
``` 

Esto nos regresa al símbolo del sistema del host, donde creará un nuevo contenedor que monte el volumen de datos desde Container4.

### 5.2 Cree Container5 y monte volúmenes desde Container4

Vas a crear Container5 y montar los volúmenes desde Container4:

``` 
docker run -ti --name=Container5 --volumes-from Container4 ubuntu
``` 

Verifique la persistencia de los datos:

``` 
cat /datavolume4/Example4.txt
Output
This file is shared between containers
``` 

Ahora, agregue algo de texto de Container5:

``` 
echo "Both containers can write to DataVolume4" >> /datavolume4/Example4.txt
```

Finalmente, saldrás del contenedor:

``` 
exit
``` 

A continuación, comprobará que sus datos todavía estén presentes en Container4.

### 5.3 Ver cambios realizados en Container5

Ahora, verifique los cambios que se escribieron en el volumen de datos Container5 reiniciando Container4:

``` 
docker start -ai Container4
``` 

Consulta los cambios:

``` 
cat /datavolume4/Example4.txt
Output
This file is shared between containers
Both containers can write to DataVolume4
```

Ahora que ha verificado que ambos contenedores pudieron leer y escribir desde el volumen de datos, saldrá del contenedor:

``` 
exit
``` 

Nuevamente, Docker no maneja ningún bloqueo de archivos, por lo que las aplicaciones deben tener en cuenta el bloqueo del archivo. Es posible montar un volumen Docker como de solo lectura para garantizar que la corrupción de datos no ocurra por accidente cuando un contenedor requiere acceso de solo lectura agregando :ro. Ahora verás cómo funciona esto.

### 5.4 Inicie el contenedor 6 y monte el volumen de solo lectura

Una vez que se ha montado un volumen en un contenedor, en lugar de desmontarlo como lo haría con un sistema de archivos típico de Linux, puede crear un nuevo contenedor montado de la manera que desee y, si es necesario, eliminar el contenedor anterior. Para que el volumen sea de solo lectura, agregue :ro al final del nombre del contenedor:

``` 
docker run -ti --name=Container6 --volumes-from Container4:ro ubuntu
``` 

Verificará el estado de solo lectura intentando eliminar su archivo de ejemplo:

``` 
rm /datavolume4/Example4.txt
Output
rm: cannot remove '/datavolume4/Example4.txt': Read-only file system
``` 

Finalmente, saldrá del contenedor y limpiará los contenedores y volúmenes de prueba:

``` 
exit
``` 

Ahora que ha terminado, limpie sus contenedores y volumen:

``` 
docker rm Container4 Container5 Container6
docker volume rm DataVolume4
``` 

En este ejemplo, mostró cómo compartir datos entre dos contenedores usando un volumen de datos y cómo montar un volumen de datos como de solo lectura.


## 6. Conclusión

En este tutorial, creó un volumen de datos que permitió que los datos persistieran mediante la eliminación de un contenedor. Compartió volúmenes de datos entre contenedores, con la salvedad de que las aplicaciones deberán diseñarse para manejar el bloqueo de archivos para evitar la corrupción de datos. Finalmente, mostró cómo montar un volumen compartido en modo de solo lectura. 

