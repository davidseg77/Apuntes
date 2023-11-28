# Cómo particionar y formatear dispositivos de almacenamiento en Linux

## 1. Introducción

Preparar un disco nuevo para usarlo en un sistema Linux es un proceso sencillo. Existen muchas herramientas, formatos de sistemas de archivos y esquemas de partición que pueden cambiar el proceso si tiene necesidades especializadas, pero los fundamentos siguen siendo los mismos.

Esta guía cubrirá el siguiente proceso:

* Identificando el nuevo disco en el sistema.
* Crear una única partición que abarque todo el disco (la mayoría de los sistemas operativos esperan un diseño de partición, incluso si solo hay un sistema de archivos presente)
* Formatear la partición con el sistema de archivos Ext4 (el predeterminado en la mayoría de las distribuciones modernas de Linux)
* Montaje y configuración Montaje automático del sistema de archivos en el arranque


## 2. Instalar Parted

Para particionar la unidad, utilizará la parted utilidad. La mayoría de los comandos necesarios para interactuar con un sistema de archivos de bajo nivel están disponibles de forma predeterminada en Linux. parted, que crea particiones, es una de las únicas excepciones ocasionales.

Si estás en un servidor Ubuntu o Debian y parted no lo tienes instalado, puedes instalarlo escribiendo:

``` 
sudo apt update
sudo apt install parted
``` 
Si está en un servidor RHEL, Rocky Linux o Fedora, puede instalarlo escribiendo:

``` 
sudo dnf install parted
``` 

Todos los demás comandos utilizados en este tutorial deben estar preinstalados, para que pueda continuar con el siguiente paso.

## 3. Identificar el nuevo disco en el sistema

Antes de configurar la unidad, debe poder identificarla correctamente en el servidor.

Si se trata de una unidad completamente nueva, una forma de identificarla en su servidor es buscar la ausencia de un esquema de partición. Si solicita parted enumerar el diseño de partición de sus discos, se producirá un error para los discos que no tengan un esquema de partición válido. Esto se puede utilizar para ayudar a identificar el nuevo disco:

``` 
sudo parted -l | grep Error
``` 

Debería ver un unrecognized disk label error para el disco nuevo sin particionar:

``` 
Output
Error: /dev/sda: unrecognized disk label
``` 

También puedes usar el lsblk comando y buscar un disco del tamaño correcto que no tenga particiones asociadas:

``` 
lsblk
Output
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
vda    253:0    0    20G  0 disk 
└─vda1 253:1    0    20G  0 part /
``` 

**Nota:** Recuerde verificar lsblk cada vez que se vuelva a conectar a su servidor antes de realizar cambios. Los identificadores de disco /dev/sd* y /dev/hd* no necesariamente serán consistentes entre los arranques, lo que significa que existe cierto peligro de particionar o formatear el disco incorrecto si no verifica el identificador del disco correctamente.

Cuando sepa el nombre que el kernel le ha asignado a su disco, podrá particionar su unidad.


## 4. Particionar la nueva unidad

Como se mencionó en la introducción, en esta guía creará una única partición que abarque todo el disco.

**Elija un estándar de partición**

Para hacer esto, primero debe especificar el estándar de partición que desea utilizar. Hay dos opciones: GPT y MBR. GPT es un estándar más moderno, mientras que MBR tiene un soporte más amplio entre los sistemas operativos más antiguos. Para un servidor en la nube típico, GPT es una mejor opción.

Para elegir el estándar GPT, pase el disco que identificó parted con mklabel gpt:

``` 
sudo parted /dev/sda mklabel gpt
``` 

Para utilizar el formato MBR, utilice mklabel msdos:

``` 
sudo parted /dev/sda mklabel msdos
``` 

**Crear la nueva partición**

Una vez seleccionado el formato, puede crear una partición que abarque todo el disco usando parted -a:

``` 
sudo parted -a opt /dev/sda mkpart primary ext4 0% 100%
``` 

Puede desglosar este comando de la siguiente manera:

* **parted -a opt** se ejecuta dividido, estableciendo el tipo de alineación óptima predeterminada.
* **/dev/sda** es el disco que estás particionando.
* **mkpart primary ext4** crea una partición independiente (es decir, de arranque, no extendida desde otra), utilizando el sistema de archivos ext4.
* **0% 100%** significa que esta partición debe abarcar desde el principio hasta el final del disco.

Para obtener más información, consulte la página del manual de Parted.

Si marca lsblk, debería ver la nueva partición disponible:

``` 
lsblk
Output
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
└─sda1   8:1    0   100G  0 part 
vda    253:0    0    20G  0 disk 
└─vda1 253:1    0    20G  0 part /
``` 

Ahora ha creado una nueva partición, pero aún no se ha inicializado como sistema de archivos. La diferencia entre estos dos pasos es algo arbitraria y exclusiva de la forma en que funcionan los sistemas de archivos de Linux, pero siguen siendo dos pasos en la práctica.


## 5. Cree un sistema de archivos en la nueva partición

Ahora que tiene una partición disponible, puede inicializarla como un sistema de archivos Ext4. Ext4 no es la única opción de sistema de archivos disponible, pero es la opción más sencilla para un volumen de Linux único e independiente. Windows usa sistemas de archivos como NTFS y exFAT, pero tienen soporte limitado en otras plataformas (lo que significa que serán de solo lectura en algunos contextos y no se pueden usar como unidad de arranque para otros sistemas operativos), y macOS usa HFS + y APFS. También hay sistemas de archivos Linux más nuevos que Ext4, como ZFS y BTRFS, pero imponen requisitos diferentes y generalmente se adaptan mejor a matrices de discos múltiples.

Para inicializar un sistema de archivos Ext4, utilice la mkfs.ext4 utilidad. Puede agregar una etiqueta de partición con la -L bandera. Seleccione un nombre que le ayude a identificar esta unidad en particular:

**Nota:** asegúrese de proporcionar la ruta a la partición y no el disco completo. En Linux, los discos tienen nombres como sda,,, etc. Las particiones de estos discos tienen un número sdb añadido hda al final. Entonces querrás usar algo como sda1, no sda.

``` 
sudo mkfs.ext4 -L datapartition /dev/sda1
``` 

Si desea cambiar la etiqueta de la partición más adelante, puede usar el e2label comando:

``` 
sudo e2label /dev/sda1 newlabel
```

Puede ver todas las diferentes formas de identificar su partición lsblk. Debería encontrar el nombre, la etiqueta y el UUID de la partición.

Algunas versiones lsblk imprimirán toda esta información con el --fs argumento:

``` 
sudo lsblk --fs
``` 

También puede especificarlos manualmente lsblk -o seguido de las opciones relevantes:

``` 
sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
``` 

Debería recibir un resultado como este. El resultado resaltado indica diferentes métodos que puede utilizar para hacer referencia al nuevo sistema de archivos:

``` 
Output
NAME   FSTYPE LABEL         UUID                                 MOUNTPOINT
sda                                                              
└─sda1 ext4   datapartition 4b313333-a7b5-48c1-a957-d77d637e4fda 
vda                                                              
└─vda1 ext4   DOROOT        050e1e34-39e6-4072-a03e-ae0bf90ba13a /
``` 

Tome nota de este resultado, ya que lo usará al montar el sistema de archivos en el siguiente paso.


## 6. Montar el nuevo sistema de archivos

Ahora puede montar el sistema de archivos para utilizarlo.

El estándar de jerarquía del sistema de archivos recomienda usar el /mnt directorio o un subdirectorio debajo de él para sistemas de archivos montados temporalmente (como unidades extraíbles). No hace recomendaciones sobre dónde montar almacenamiento más permanente, por lo que puede elegir el esquema que desee. Para este tutorial, montará la unidad en /mnt/data.

Crea ese directorio usando mkdir:

``` 
sudo mkdir -p /mnt/data
``` 

### 6.1 Montar el sistema de archivos temporalmente

Puede montar el sistema de archivos temporalmente escribiendo:

``` 
sudo mount -o defaults /dev/sda1 /mnt/data
``` 

### 6.2 Montar el sistema de archivos automáticamente al arrancar

Para montar el sistema de archivos automáticamente cada vez que se inicia el servidor, agregará una entrada al /etc/fstab archivo. Este archivo contiene información sobre todos los discos permanentes o montados de forma rutinaria de su sistema. Abra el archivo usando nano o su editor de texto favorito:

``` 
sudo nano /etc/fstab
``` 

En el último paso, utilizó el sudo lsblk --fs comando para mostrar identificadores de su sistema de archivos. Puede utilizar cualquiera de estos en este archivo. Este ejemplo usa la etiqueta de partición, pero puedes ver cómo se verían las líneas usando los otros dos identificadores en las líneas comentadas:

``` 
/etc/fstab
. . .
## Use one of the identifiers you found to reference the correct partition
# /dev/sda1 /mnt/data ext4 defaults 0 2
# UUID=4b313333-a7b5-48c1-a957-d77d637e4fda /mnt/data ext4 defaults 0 2
LABEL=datapartition /mnt/data ext4 defaults 0 2
``` 

Más allá del LABEL=datapartition elemento, estas opciones funcionan de la siguiente manera:

* /mnt/data es la ruta donde se está montando el disco.
* ext4 connota que se trata de una partición Ext4.
* defaults significa que este volumen debe montarse con las opciones predeterminadas, como soporte de lectura y escritura.
* 0 2 significa que el sistema de archivos debe ser validado por la máquina local en caso de errores, pero como 2 prioridad, después de su volumen raíz.
  
**Nota:** Puede obtener información sobre los distintos campos del /etc/fstab archivo consultando su página de manual. Para obtener información sobre las opciones de montaje disponibles para un tipo de sistema de archivos específico, marque man [filesystem](como man ext4).

Guarde y cierre el archivo cuando haya terminado. Si está utilizando nano, presione Ctrl+X, luego cuando se le solicite confirmar Y y luego Enter.

Si no montó el sistema de archivos anteriormente, ahora puede montarlo con mount -a:

``` 
sudo mount -a
``` 

### 6.3 Probando el soporte

Después de haber montado el volumen, debemos verificar para asegurarnos de que se pueda acceder al sistema de archivos.

Puede comprobar si el disco está disponible en el resultado del df comando. A veces df se incluirá información innecesaria sobre los sistemas de archivos temporales llamados tmpfs en df la salida, que puede excluir agregando -x tmpfs:

``` 
df -h -x tmpfs
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.3G   18G   7% /
/dev/sda1        99G   60M   94G   1% /mnt/data
``` 

También puede verificar que el disco montado tenga capacidades de lectura y escritura escribiendo en un archivo de prueba:

``` 
echo "success" | sudo tee /mnt/data/test_file
``` 

Vuelva a leer el archivo solo para asegurarse de que la escritura se haya ejecutado correctamente:

``` 
cat /mnt/data/test_file
Output
success
``` 

Puede eliminar el archivo después de haber verificado que el nuevo sistema de archivos funciona correctamente:

``` 
sudo rm /mnt/data/test_file
``` 

## 7. Conclusión

Su nueva unidad ahora debería estar particionada, formateada, montada y lista para usar. Este es el proceso general que puede utilizar para convertir un disco sin formato en un sistema de archivos que Linux puede utilizar para almacenamiento. Existen métodos más complejos de partición, formateo y montaje que pueden ser más apropiados en algunos casos, pero lo anterior es un buen punto de partida para uso general.

