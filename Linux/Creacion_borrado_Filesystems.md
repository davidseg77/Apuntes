# Info extraida del taller Creación y borrado de Filesystems en Linux

## 1. Caso práctico

### 1.1 Crear un archivo de swap

``` 
sudo dd if=/dev/zero of=/swapfile bs=1M count=10
```

Con este comando, asignamos cero al archivo dev que no contiene nada y lo copia al swapfile que es un archivo que estamos creando y lo llena de bloques de diez unidades de un megabyte. Serían diez en total.

Hemos creado el archivo y ahora tenemos que formatearlo como un archivo de tipo swap:

``` 
sudo mkswap /swapfile
```

Lo hemos formateado y ahora lo activamos:

``` 
swapon /swapfile
```

Con swapon vemos los archivos de tipo swap que tenemos en el sistema. Para desactivarlo sería con swapoff. 

### 1.2 Creación y destrucción de particiones

Para ello, vamos a crear un disco duro en nuestra VM aparte del que ya tenemos. En primer lugar, apagamos nuestra VM. Cuando este apagada, vamos a Storage y abajo damos en añadir controlador de Storage. Escogemos el controlador SATA (AHCI), y dentro del controlador damos a añadir disco duro.

Hacemos clic en crear y creamos uno de 100 MB. Aceptamos lo que tiene por defecto y dinamicamente queda asignado. Clic en Choose y OK. 

Volvemos a encender nuestra VM con su disco adicional. Para comprobar mis dispositivos de almacenamiento y particiones:

``` 
sudo fdisk -l
```

Ahora creamos nuestra partición en el disco:

```
sudo fdisk /dev/sdb
```

Nos surgirá una pregunta, mejor dicho un mensaje en el cual hemos de introducir una letra. Nosotros marcaremos la n, que significa añadir una nueva partición. Y en tipo de partición, no escogemos nada, aceptamos todo mejor dicho.

Damos a la w después para escribir la partición y guardar. Y tendremos nuestra partición creada. Comprobamos:

``` 
sudo fdisk -l
```

Vamos al directorio /mnt y creamos un directorio llamado part. Es el que voy a usar para montar la partición. 

``` 
sudo mkfs.ext4 /dev/sdb1
```

Con ese comando creamos un filesystem de formato ext4 para el disco duro. Para montar la partición:

``` 
sudo mount -t ext4 /dev/sdb1 /mnt/part
```

Para eliminar la partición:

``` 
sudo fdisk /dev/sdb
```

Y damos la opción dd (delete).


### 1.3 Crear particiones persistentes

Para ello, editamos el archivo /etc/fstab. Y añadimos la partición que tenemos creada junto con la ruta de montaje y el tipo de sistema de archivos.

``` 
/dev/sdb1   /mnt/part   ext4    defaults    0 0
```

Reiniciamos el equipo para que los cambios se produzcan. 

Para que sean aun más persistentes, hacemos lo siguiente:

```
sudo blkid
```

Con este comando, vemos los UUID de cada partición. Lo ideal para tener mayor persistencia es copiar la UUID de nuestra partición y reemplazarla en el fichero en sustitución del nombre de la partición:

``` 
UUID=Código UUID  /mnt/part   ext4    defaults    0 0
```

Volvemos a reiniciar y listo.





