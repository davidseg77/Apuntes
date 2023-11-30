# Cómo realizar tareas básicas de administración para dispositivos de almacenamiento en Linux

## 1. Encontrar capacidad de almacenamiento y uso con df

A menudo, la información más importante que necesitará sobre el almacenamiento de su sistema es la capacidad y la utilización actual de los dispositivos de almacenamiento conectados.

Para comprobar cuánto espacio de almacenamiento hay disponible en total y ver la utilización actual de sus unidades, utilice la utilidad df. De forma predeterminada, esto genera las medidas en bloques de 1K, lo que no siempre es útil. Agregue la -h bandera a la salida en unidades legibles por humanos:

``` 
df -h
Output
Filesystem      Size  Used Avail Use% Mounted on
udev            238M     0  238M   0% /dev
tmpfs            49M  624K   49M   2% /run
/dev/vda1        20G  1.1G   18G   6% /
tmpfs           245M     0  245M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           245M     0  245M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000
/dev/sda1        99G   60M   94G   1% /mnt/data
``` 

La /dev/vda1 partición, que está montada en /, está llena al 6% y tiene 18G de espacio disponible, mientras que la /dev/sda1 partición, que está montada en /mnt/data está vacía y tiene 94 G de espacio disponible. Las otras entradas utilizan tmpfs o devtmpfs sistemas de archivos, que es memoria volátil que se utiliza como si fuera almacenamiento permanente. Puede excluir estas entradas escribiendo:

``` 
df -h -x tmpfs -x devtmpfs
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.1G   18G   6% /
/dev/sda1        99G   60M   94G   1% /mnt/data
``` 

Esta salida ofrece una visualización más enfocada de la utilización actual del disco al eliminar algunos pseudodispositivos y dispositivos especiales.


## 2. Encontrar información sobre dispositivos de bloque con lsblk

Un dispositivo de bloque es un término genérico para un dispositivo de almacenamiento que lee o escribe en bloques de un tamaño específico. Este término se aplica a casi todos los tipos de almacenamiento no volátil, incluidas las unidades de disco duro (HDD), las unidades de estado sólido (SSD), etc. El dispositivo de bloque es el dispositivo físico donde está escrito el sistema de archivos. El sistema de archivos, a su vez, dicta cómo se almacenan los datos y archivos.

La utilidad lsblk se puede utilizar para mostrar información sobre dispositivos de bloque. Las capacidades específicas de la utilidad dependen de la versión instalada, pero en general, el lsblk comando se puede utilizar para mostrar información sobre la unidad en sí, así como la información de partición y el sistema de archivos que se ha escrito en ella.

Sin ningún argumento, lsblk mostrará los nombres de los dispositivos, los números mayores y menores asociados con un dispositivo (utilizados por el kernel de Linux para realizar un seguimiento de los controladores y dispositivos), si la unidad es extraíble, su tamaño, si está montada como de solo lectura, su tipo (disco o partición) y su punto de montaje. Algunos sistemas requieren sudoque esto se muestre correctamente:

``` 
sudo lsblk
Output
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  100G  0 disk 
vda    253:0    0   20G  0 disk 
└─vda1 253:1    0   20G  0 part /
``` 

De la salida que se muestra, las partes más importantes suelen ser el nombre, que se refiere al nombre del dispositivo en /dev, el tamaño, el tipo y el punto de montaje. Aquí puede ver que tiene un disco (/dev/vda) con una única partición (/dev/vda1) que se utiliza como /partición y otro disco ( /dev/sda) que no ha sido particionado.

Para obtener información más relevante sobre la administración de discos y particiones, puede pasar la --fs marca en algunas versiones:

``` 
sudo lsblk --fs
Output
NAME   FSTYPE LABEL  UUID                                 MOUNTPOINT
sda                                                       
vda                                                       
└─vda1 ext4   DOROOT c154916c-06ea-4268-819d-c0e36750c1cd /
``` 

Si el --fs indicador no está disponible en su sistema, puede replicar manualmente la salida utilizando el -o indicador para solicitar una salida específica. Puede utilizar -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT para obtener esta misma información.

Para obtener información sobre la topología del disco, escriba:

``` 
sudo lsblk -t
Output
NAME   ALIGNMENT MIN-IO OPT-IO PHY-SEC LOG-SEC ROTA SCHED    RQ-SIZE  RA WSAME
sda            0    512      0     512     512    1 deadline     128 128    2G
vda            0    512      0     512     512    1              128 128    0B
└─vda1         0    512      0     512     512    1              128 128    0B
``` 

Hay muchos otros atajos disponibles para mostrar características relacionadas con sus discos y particiones. Puede generar todas las columnas disponibles con la -O bandera o puede personalizar los campos para mostrar especificando los nombres de las columnas con la -o bandera. La -h bandera se puede utilizar para enumerar las columnas disponibles:

``` 
lsblk -h
Output
. . .

Available columns (for --output):
        NAME  device name
       KNAME  internal kernel device name

       . . .

  SUBSYSTEMS  de-duplicated chain of subsystems
         REV  device revision
      VENDOR  device vendor

For more details see lsblk(8).
``` 


## 3. Trabajar con montajes del sistema de archivos

Antes de poder utilizar un disco nuevo, normalmente hay que particionarlo, formatearlo con un sistema de archivos y luego montar la unidad o las particiones. La partición y el formateo suelen ser procedimientos que se realizan una sola vez. 

El montaje es algo que puedes hacer con más frecuencia. Al montar el sistema de archivos, estará disponible para el servidor en el punto de montaje seleccionado. Un punto de montaje es un directorio bajo el cual se puede acceder al nuevo sistema de archivos.

Para gestionar el montaje se utilizan principalmente dos comandos complementarios: mount y umount. El mount comando se utiliza para adjuntar un sistema de archivos al árbol de archivos actual. En un sistema Linux, se utiliza una única jerarquía de archivos unificada para todo el sistema, independientemente de cuántos dispositivos físicos esté compuesto. El umount comando (Nota: esto es umount, no unmount) se utiliza para desmontar un sistema de archivos. Además, el findmnt comando es útil para recopilar información sobre el estado actual de los sistemas de archivos montados.

### 3.1 Usando el comando de montaje

La forma más sencilla de utilizar mount es pasar un dispositivo o partición formateada y el punto de montaje donde se conectará:

``` 
sudo mount /dev/sda1 /mnt
``` 

El punto de montaje, el parámetro final que especifica en qué parte de la jerarquía de archivos se debe adjuntar el nuevo sistema de archivos, casi siempre debe ser un directorio vacío.

Por lo general, querrás seleccionar opciones más específicas al realizar el montaje. Aunque mount puede intentar adivinar el tipo de sistema de archivos, casi siempre es una mejor idea pasar el tipo de sistema de archivos con la -t opción. Para un sistema de archivos Ext4, esto sería:

``` 
sudo mount -t ext4 /dev/sda1 /mnt
``` 

Hay muchas otras opciones que afectarán la forma en que se monta el sistema de archivos. Hay opciones de montaje genéricas, que se pueden encontrar en la sección OPCIONES DE MONTAJE INDEPENDIENTES DEL SISTEMA DE ARCHIVOS del manual de montaje.

Pase otras opciones con la -o bandera. Por ejemplo, para montar una partición con las opciones predeterminadas (que significa rw, suid, dev, exec, auto, nouser, async), puede pasar -o defaults. Si necesita anular los permisos de lectura y escritura y montar como solo lectura, puede agregar ro como opción posterior, que anulará la rw opción defaults:

``` 
sudo mount -t ext4 -o defaults,ro /dev/sda1 /mnt
``` 

Para montar todos los sistemas de archivos descritos en el /etc/fstab archivo, puede pasar la -a opción:

``` 
sudo mount -a
```

### 3.2 Listado de opciones de montaje del sistema de archivos

Para mostrar las opciones de montaje utilizadas para un montaje específico, utilice el findmnt comando. Por ejemplo, si vieras el montaje de solo lectura del ejemplo anterior con findmnt, se vería así:

``` 
findmnt /mnt
Output
TARGET SOURCE    FSTYPE OPTIONS
/mnt   /dev/sda1 ext4   ro,relatime,data=ordered
``` 

Esto puede resultar útil si ha estado experimentando con múltiples opciones y finalmente ha descubierto un conjunto que le gusta. Puede encontrar las opciones que está utilizando findmnt para saber qué es apropiado agregar al /etc/fstab archivo para un montaje futuro.


### 3.3 Desmontar un sistema de archivos

El umount comando se utiliza para desmontar un sistema de archivos determinado. Una vez más, esto umount no es así unmount.

La forma general del comando es nombrar el punto de montaje o dispositivo de un sistema de archivos actualmente montado. Asegúrese de no estar utilizando ningún archivo en el punto de montaje y de que no tenga ninguna aplicación (incluido su shell actual) funcionando dentro del punto de montaje:

``` 
cd ~
sudo umount /mnt
``` 

Generalmente no hay opciones para agregar al comportamiento de desmontaje predeterminado.

