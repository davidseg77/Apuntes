# Cómo establecer cuotas del sistema de archivos en Ubuntu

## 1. Introducción

Las cuotas se utilizan para limitar la cantidad de espacio en disco que un usuario o grupo puede utilizar en un sistema de archivos. Sin tales límites, un usuario podría llenar el disco de la máquina y causar problemas a otros usuarios y servicios.

En este tutorial, instalará herramientas de línea de comandos para crear e inspeccionar cuotas de disco y luego establecerá una cuota para un usuario de ejemplo.


## 2. Instalar las herramientas de cuotas

Para establecer y verificar cuotas, primero debe instalar las herramientas de línea de comando de cuotas usando apt. Primero actualice la lista de paquetes, luego instale el paquete:

``` 
sudo apt update
sudo apt install quota
``` 

Puede verificar que las herramientas estén instaladas ejecutando el quota comando y solicitando información de su versión:

``` 
quota --version
Output
Quota utilities version 4.05.
. . .
``` 

Está bien si su resultado muestra un número de versión ligeramente diferente.

A continuación, asegúrese de tener los módulos del kernel adecuados para monitorear las cuotas.


## 3. Instalar el módulo de kernel de cuota

Si está en un servidor virtual basado en la nube, es posible que su instalación predeterminada de Ubuntu Linux no tenga los módulos del kernel necesarios para admitir la administración de cuotas. Para comprobarlo, utilizará find para buscar los módulos quota_v1 y quota_v2 en el /lib/modules/...directorio:

``` 
find /lib/modules/ -type f -name '*quota_v*.ko*'
Output
/lib/modules/5.4.0-99-generic/kernel/fs/quota/quota_v2.ko
/lib/modules/5.4.0-99-generic/kernel/fs/quota/quota_v1.ko
``` 

Tome nota de la versión de su kernel, resaltada en las rutas de archivo anteriores, ya que la necesitará en un paso posterior. Probablemente será diferente, pero siempre que los dos módulos estén enumerados, estará todo listo y podrá omitir el resto de este paso.

Si no obtiene ningún resultado del comando anterior, instale el linux-image-extra-virtual paquete:

``` 
sudo apt install linux-image-extra-virtual
``` 

Esto proporcionará los módulos del kernel necesarios para implementar cuotas. Ejecute el find comando anterior nuevamente para verificar que la instalación fue exitosa.

A continuación, actualizará mountlas opciones de su sistema de archivos para habilitar cuotas en su sistema de archivos raíz.


## 4. Actualizar las opciones de montaje del sistema de archivos

Para activar cuotas en un sistema de archivos en particular, debe montarlo con algunas opciones relacionadas con las cuotas especificadas. Puede hacer esto actualizando la entrada del sistema de archivos en el /etc/fstab archivo de configuración. Abra ese archivo nano o su editor de texto preferido:

``` 
sudo nano /etc/fstab
``` 

El contenido de este archivo será similar al siguiente:

```
/etc/fstab
LABEL=cloudimg-rootfs   /        ext4   defaults        0 0
LABEL=UEFI      /boot/efi       vfat    defaults        0 0
``` 

Este fstab archivo es de un servidor virtual. Una computadora de escritorio o portátil probablemente tendrá un sistema de archivos ligeramente diferente fstab, pero en la mayoría de los casos tendrá un /sistema de archivos raíz que representa todo su espacio en disco.

La línea resaltada indica el nombre del dispositivo montado, la ubicación donde está montado, el tipo de sistema de archivos y las opciones de montaje utilizadas. El primer cero indica que no se realizarán copias de seguridad y el segundo cero indica que no se realizará ninguna verificación de errores durante el arranque.

Actualice la línea que apunta al sistema de archivos raíz reemplazando la defaults opción con las siguientes opciones resaltadas:

``` 
/etc/fstab
LABEL=cloudimg-rootfs   /        ext4   usrquota,grpquota        0 0
. . .
``` 

Este cambio nos permitirá habilitar cuotas usrquota basadas en usuarios ( ) y en grupos (grpquota) en el sistema de archivos. Si solo necesita uno u otro, puede omitir la opción no utilizada. Si su fstab línea ya tenía algunas opciones enumeradas en lugar de defaults, debe agregar las nuevas opciones al final de lo que ya esté allí, asegurándose de separar todas las opciones con una coma y sin espacios.

Vuelva a montar el sistema de archivos para que las nuevas opciones surtan efecto:

``` 
sudo mount -o remount /
``` 

Aquí, la -o bandera se utiliza para pasar la remount opción.

**Nota:** asegúrese de que no haya espacios entre las opciones enumeradas en su /etc/fstab archivo. Si pones un espacio después de la ,coma, verás un error como el siguiente:

``` 
Output
mount: /etc/fstab: parse error
``` 

Si ve este mensaje después de ejecutar el mount comando anterior, vuelva a abrir el fstab archivo, corrija los errores y repita el mount comando antes de continuar.

Dicho esto, puede verificar que las nuevas opciones se usaron para montar el sistema de archivos mirando el /proc/mounts archivo. Aquí, use grep para mostrar solo la entrada del sistema de archivos raíz en ese archivo:

``` 
cat /proc/mounts | grep ' / '
Output
/dev/vda1 / ext4 rw,relatime,quota,usrquota,grpquota 0 0
``` 

Tenga en cuenta las dos opciones especificadas. Ahora que ha instalado sus herramientas y actualizado las opciones de su sistema de archivos, puede activar el sistema de cuotas.


## 5. Habilitar cuotas

Antes de activar finalmente el sistema de cuotas, debe ejecutar manualmente el quotacheck comando una vez:

``` 
sudo quotacheck -ugm /
``` 

Este comando crea los archivos /aquota.user y /aquota.group. Estos archivos contienen información sobre los límites y el uso del sistema de archivos y deben existir antes de activar la supervisión de cuotas. Los quotacheck parámetros utilizados son:

* **u:** especifica que se debe crear un archivo de cuota basado en el usuario
* **g:** indica que se debe crear un archivo de cuota basado en grupos
* **m:** deshabilita el remontaje del sistema de archivos como de solo lectura mientras se realiza el recuento inicial de cuotas. Volver a montar el sistema de archivos como de solo lectura dará resultados más precisos en caso de que un usuario esté guardando archivos activamente durante el proceso, pero no es necesario durante esta configuración inicial.

Si no necesita habilitar cuotas basadas en usuarios o grupos, puede dejar de lado la quotacheck opción correspondiente.

Puede verificar que se crearon los archivos apropiados enumerando el directorio raíz:

``` 
ls /
Output
aquota.group  bin   dev  home        initrd.img.old  lib64       media  opt   root  sbin  srv  tmp  var      vmlinuz.old
aquota.user   boot  etc  initrd.img  lib             lost+found  mnt    proc  run   snap  sys  usr vmlinuz
``` 

Si no incluyó las opciones uo gen el quotacheck comando, faltará el archivo correspondiente.

A continuación, deberá agregar los módulos de cuota al kernel de Linux. Alternativamente, puede reiniciar su servidor para realizar la misma tarea. De lo contrario, agréguelos manualmente, reemplazando la versión del kernel resaltada con su versión que se encuentra en el Paso 2:

``` 
sudo modprobe quota_v1 -S 5.4.0-99-generic
sudo modprobe quota_v2 -S 5.4.0-99-generic
```

Ahora estás listo para activar el sistema de cuotas:

```
sudo quotaon -v /
quotaon Output
/dev/vda1 [/]: group quotas turned on
/dev/vda1 [/]: user quotas turned on
``` 

Su servidor ahora está monitoreando y aplicando cuotas, ¡pero aún no hemos establecido ninguna! A continuación, establecerá una cuota de disco para un único usuario.


## 6. Configurar cuotas para un usuario

Hay algunas formas de establecer cuotas para usuarios o grupos. Aquí, repasarás cómo establecer cuotas con los comandos edquota y setquota.

### 6.1 Usar edquota para establecer una cuota de usuarios

Utilice el comando edquota para editar cuotas. Editemos la cuota de usuario sammy de su ejemplo:

``` 
sudo edquota -u sammy
```

La -u opción especifica que se trata de una usercuota que editará. Si en su lugar desea editar la cuota de un grupo, use la -g opción en su lugar.

Esto abrirá un archivo en su editor de texto predeterminado, similar a cómo crontab -e abre un archivo temporal para que lo edite. El archivo será similar a este:

``` 
Disk quotas for user sammy (uid 1000):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/vda1                        40          0          0         13        0        0
``` 

Esto enumera el nombre de usuario y uid, los sistemas de archivos que tienen cuotas habilitadas y el uso y los límites basados ​​en bloques e inodos. Establecer una cuota basada en inodos limitaría la cantidad de archivos y directorios que un usuario puede crear, independientemente de la cantidad de espacio en disco que utilice. La mayoría de la gente querrá cuotas basadas en bloques, que limiten específicamente el uso del espacio en disco. Esto es lo que configurarás.

Cada tipo de cuota le permite establecer tanto un límite flexible como un límite estricto. Cuando un usuario excede el límite flexible, supera la cuota, pero no se le impide inmediatamente consumir más espacio o inodos. En cambio, se da cierto margen de maniobra: el usuario tiene, de forma predeterminada, siete días para que el uso de su disco vuelva a estar por debajo del límite flexible. 

Al final del período de gracia de siete días, si el usuario aún supera el límite flexible, se tratará como un límite estricto. Un límite estricto es menos indulgente: toda creación de nuevos bloques o inodos se detiene inmediatamente cuando se alcanza el límite estricto especificado. Esto se comporta como si el disco estuviera completamente sin espacio: las escrituras fallarán, los archivos temporales no se podrán crear y el usuario comenzará a ver advertencias y errores mientras realiza tareas comunes.

Actualicemos su usuario sammy para que tenga una cuota de bloqueo con un límite flexible de 100 MB y un límite estricto de 110 MB:

``` 
Disk quotas for user sammy (uid 1000):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/vda1                        40       100M       110M         13        0        0
``` 

Guarde y cierre el archivo. Para comprobar la nueva cuota puedes utilizar el quota comando:

```
sudo quota -vs sammy
Output
Disk quotas for user sammy (uid 1000):
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
      /dev/vda1     40K    100M    110M              13       0       0
```

El comando genera el estado de su cuota actual y muestra que su cuota es 100M mientras su límite es 110M. Esto corresponde a los límites blando y duro respectivamente.

### 6.2 Usar setquotapara establecer una cuota de usuarios

A diferencia de edquota, setquota actualizará la información de cuota de su usuario con un solo comando, sin un paso de edición interactivo. Especificará el nombre de usuario y los límites flexibles y estrictos para las cuotas basadas en bloques y en inodos y, finalmente, el sistema de archivos al que aplicar la cuota:

```
sudo setquota -u sammy 200M 220M 0 0 /
``` 

El comando anterior duplicará los límites de cuota basados ​​en bloques de sammy a 200 megabytes y 220 megabytes. Los 0 0 límites blandos y estrictos basados ​​en inodos indican que permanecen sin establecer. Esto es necesario incluso si no establece ninguna cuota basada en inodos.

Una vez más, use el quota comando para verificar su trabajo:

``` 
sudo quota -vs sammy
Output
Disk quotas for user sammy (uid 1000):
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
      /dev/vda1     40K    200M    220M              13       0       0
``` 

Ahora que ha establecido algunas cuotas, descubramos cómo generar un informe de cuotas.


## 7. Generar informes de cuotas

Para generar un informe sobre el uso de cuota actual para todos los usuarios en un sistema de archivos en particular, use el repquota comando:

``` 
sudo repquota -s /
Output
*** Report for user quotas on device /dev/vda1
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --   1696M      0K      0K          75018     0     0
daemon    --     64K      0K      0K              4     0     0
man       --   1048K      0K      0K             81     0     0
nobody    --   7664K      0K      0K              3     0     0
syslog    --   2376K      0K      0K             12     0     0
sammy     --     40K    200M    220M             13     0     0
``` 

En este caso, está generando un informe para el sistema de archivos / raíz. El -s comando indica repquota que se utilicen números legibles por humanos cuando sea posible. Hay algunos usuarios del sistema enumerados, que probablemente no tengan cuotas establecidas de forma predeterminada. Su usuario sammy aparece en la parte inferior, con las cantidades utilizadas y los límites estrictos y flexibles.

Tenga en cuenta también la Block grace time:7days leyenda y la grace columna. Si su usuario superó el límite flexible, la grace columna mostraría cuánto tiempo le quedaba para volver a estar por debajo del límite.


## 8. Configurar un período de gracia para excedentes

Puede configurar el período de tiempo en el que un usuario puede flotar por encima del límite flexible. Utilice el setquota comando para hacerlo:

``` 
sudo setquota -t 864000 864000 /
``` 

El comando anterior establece los tiempos de gracia del bloque y del inodo en 864000 segundos o 10 días. Esta configuración se aplica a todos los usuarios y se deben proporcionar ambos valores incluso si no usa ambos tipos de cuota (bloque versus inodo).

Tenga en cuenta que los valores deben especificarse en segundos.

Ejecute repquota nuevamente para verificar que los cambios surtieron efecto:

``` 
sudo repquota -s /
Output
Block grace time: 10days; Inode grace time: 10days
. . .
``` 

Los cambios deben reflejarse inmediatamente en el repquota resultado.


## 9. Conclusión

En este tutorial, instaló las quota herramientas de línea de comandos, verificó que su kernel de Linux puede manejar cuotas de monitoreo, configuró una cuota basada en bloques para un usuario y generó un informe sobre el uso de cuotas de su sistema de archivos.

### 9.1 Apéndice: Mensajes de error comunes relacionados con las cuotas

Los siguientes son algunos errores comunes que puede ver al configurar y manipular cuotas del sistema de archivos.

``` 
quotaon Output
quotaon: cannot find //aquota.group on /dev/vda1 [/]
quotaon: cannot find //aquota.user on /dev/vda1 [/]
```

Este es un error que podría ver si intenta activar las cuotas (usando quotaon) antes de ejecutar el quotacheck comando inicial. El quotacheck comando crea los archivos aquota o quota necesarios para activar el sistema de cuotas. Consulte el Paso 4 para obtener más información.

``` 
quotaon Output
quotaon: using //aquota.group on /dev/vda1 [/]: No such process
quotaon: Quota format not supported in kernel.
quotaon: using //aquota.user on /dev/vda1 [/]: No such process
quotaon: Quota format not supported in kernel.
``` 

Este quotaon error nos indica que su kernel no admite cuotas, o al menos no admite la versión correcta (existe una versión quota_v1 y quota_v2). Esto significa que los módulos del kernel que necesita no están instalados o no se están cargando correctamente. En Ubuntu Server, la causa más probable de esto es el uso de una imagen de instalación reducida en un servidor virtual basado en la nube.

Si este es el caso, se puede solucionar instalando el linux-image-extra-virtual paquete con apt. Consulte el Paso 2 para obtener más detalles.

Si este error persiste después de la instalación, revise el Paso 4. Asegúrese de haber utilizado correctamente los modprobe comandos o de haber reiniciado su servidor si esa es una opción viable para usted.

```
quota Output
quota: Cannot open quotafile //aquota.user: Permission denied
quota: Cannot open quotafile //aquota.user: Permission denied
quota: Cannot open quotafile //quota.user: No such file or directory
``` 

Este es el error que verá si ejecuta quota y su usuario actual no tiene permiso para leer los archivos de cuota para su sistema de archivos. Usted (o el administrador del sistema) deberá ajustar los permisos del archivo de forma adecuada o utilizarlos sudocuando ejecute comandos que requieran acceso al archivo de cuota.

