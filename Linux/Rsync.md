# Cómo utilizar Rsync para sincronizar directorios locales y remotos

## 1. Introducción

Rsync, que significa sincronización remota, es una herramienta de sincronización de archivos local y remota. Utiliza un algoritmo para minimizar la cantidad de datos copiados moviendo únicamente las partes de los archivos que han cambiado.

En este tutorial, definiremos Rsync, revisaremos la sintaxis al usar rsync, explicaremos cómo usar Rsync para sincronizar con un sistema remoto y otras opciones.


## 2. Comprender la sintaxis de Rsync

La sintaxis de rsyncfunciona de manera similar a la de otras herramientas, como ssh, scp y cp.

Primero, cambie a su directorio de inicio ejecutando el siguiente comando:

``` 
cd ~
``` 

Luego cree un directorio de prueba:

``` 
mkdir dir1
``` 

Cree otro directorio de prueba:

``` 
mkdir dir2
``` 

Ahora agregue algunos archivos de prueba:

``` 
touch dir1/file{1..100}
``` 

Ahora hay un directorio llamado dir1 con 100 archivos vacíos. Confirme enumerando los archivos:

``` 
ls dir1
Output
file1    file18  file27  file36  file45  file54  file63  file72  file81  file90
file10   file19  file28  file37  file46  file55  file64  file73  file82  file91
file100  file2   file29  file38  file47  file56  file65  file74  file83  file92
file11   file20  file3   file39  file48  file57  file66  file75  file84  file93
file12   file21  file30  file4   file49  file58  file67  file76  file85  file94
file13   file22  file31  file40  file5   file59  file68  file77  file86  file95
file14   file23  file32  file41  file50  file6   file69  file78  file87  file96
file15   file24  file33  file42  file51  file60  file7   file79  file88  file97
file16   file25  file34  file43  file52  file61  file70  file8   file89  file98
file17   file26  file35  file44  file53  file62  file71  file80  file9   file99
``` 

También tienes un directorio vacío llamado dir2. Para sincronizar el contenido de dir1to dir2 en el mismo sistema, ejecutará rsyncy utilizará la -rbandera, que significa "recursivo" y es necesaria para la sincronización de directorios:

``` 
rsync -r dir1/ dir2
``` 

Otra opción es utilizar la -a bandera, que es una bandera combinada y significa "archivo". Esta bandera se sincroniza de forma recursiva y conserva enlaces simbólicos, archivos especiales y de dispositivo, horas de modificación, grupos, propietarios y permisos. Se usa con más frecuencia -r y es la bandera recomendada. Ejecute el mismo comando que en el ejemplo anterior, esta vez usando la -a bandera:

``` 
rsync -a dir1/ dir2
``` 

Tenga en cuenta que hay una barra diagonal ( /) al final del primer argumento en la sintaxis de los dos comandos anteriores y resaltada aquí:

``` 
rsync -a dir1/ dir2
``` 

Esta barra diagonal final indica el contenido de dir1. Sin la barra diagonal final, dir1, incluido el directorio, se colocaría dentro de dir2. El resultado crearía una jerarquía como la siguiente:

``` 
~/dir2/dir1/[files]
``` 

Otro consejo es volver a verificar sus argumentos antes de ejecutar un rsync comando. Rsync proporciona un método para hacer esto pasando las opciones -no --dry-run. La -v bandera, que significa "detallada", también es necesaria para obtener el resultado adecuado. Combinará las banderas a, n y ven el siguiente comando:

``` 
rsync -anv dir1/ dir2
Output
sending incremental file list
./
file1
file10
file100
file11
file12
file13
file14
file15
file16
file17
file18
. . .
```

Ahora compare ese resultado con el que recibe al eliminar la barra diagonal, como se muestra a continuación:

``` 
rsync -anv dir1 dir2
Output
sending incremental file list
dir1/
dir1/file1
dir1/file10
dir1/file100
dir1/file11
dir1/file12
dir1/file13
dir1/file14
dir1/file15
dir1/file16
dir1/file17
dir1/file18
. . .
``` 

Este resultado ahora demuestra que se transfirió el directorio en sí, en lugar de solo los archivos dentro del directorio.


## 3. Uso de Rsync para sincronizar con un sistema remoto

Para usar rsync para sincronizar con un sistema remoto, solo necesita el acceso SSH configurado entre sus máquinas locales y remotas, así como rsync instalado en ambos sistemas. Una vez que haya verificado el acceso SSH entre las dos máquinas, puede sincronizar la dir1 carpeta de la sección anterior con una máquina remota utilizando la siguiente sintaxis. Tenga en cuenta que en este caso desea transferir el directorio real, por lo que omitirá la barra diagonal final:

``` 
rsync -a ~/dir1 username@remote_host:destination_directory
``` 

Este proceso se denomina operación de inserción porque “empuja” un directorio desde el sistema local a un sistema remoto. La operación opuesta es pull y se utiliza para sincronizar un directorio remoto con el sistema local. Si el dir1 directorio estuviera en el sistema remoto en lugar de en su sistema local, la sintaxis sería la siguiente:

``` 
rsync -a username@remote_host:/home/username/dir1 place_to_sync_on_local_machine
``` 

Al igual que cpen herramientas similares, el origen es siempre el primer argumento y el destino es siempre el segundo.


## 4. Usar otras opciones de Rsync

Rsync proporciona muchas opciones para alterar el comportamiento predeterminado de la utilidad, como las opciones de bandera que aprendió en la sección anterior.

Si estás transfiriendo archivos que aún no han sido comprimidos, como archivos de texto, puedes reducir la transferencia de red agregando compresión con la -z opción:

``` 
rsync -az source destination
``` 

La -P bandera también es útil. Combina las banderas --progress y --partial. Este primer indicador proporciona una barra de progreso para las transferencias y el segundo indicador le permite reanudar las transferencias interrumpidas:

``` 
rsync -azP source destination
Output
sending incremental file list
created directory destination
source/
source/file1
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=99/101)
sourcefile10
              0 100%    0.00kB/s    0:00:00 (xfr#2, to-chk=98/101)
source/file100
              0 100%    0.00kB/s    0:00:00 (xfr#3, to-chk=97/101)
source/file11
              0 100%    0.00kB/s    0:00:00 (xfr#4, to-chk=96/101)
source/file12
              0 100%    0.00kB/s    0:00:00 (xfr#5, to-chk=95/101)
. . .
``` 

Si ejecuta el comando nuevamente, recibirá un resultado abreviado ya que no se han realizado cambios. Esto ilustra la capacidad de Rsync para usar tiempos de modificación para determinar si se han realizado cambios:

``` 
rsync -azP source destination
Output
sending incremental file list
sent 818 bytes received 12 bytes 1660.00 bytes/sec
total size is 0 speedup is 0.00
``` 

Supongamos que desea actualizar la hora de modificación en algunos de los archivos con un comando como el siguiente:

``` 
touch dir1/file{1..10}
``` 

Luego, si volviera a ejecutar rsync, -azP notará en el resultado cómo Rsync vuelve a copiar de forma inteligente solo los archivos modificados:

``` 
rsync -azP source destination
Output
sending incremental file list
file1
            0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=99/101)
file10
            0 100%    0.00kB/s    0:00:00 (xfer#2, to-check=98/101)
file2
            0 100%    0.00kB/s    0:00:00 (xfer#3, to-check=87/101)
file3
            0 100%    0.00kB/s    0:00:00 (xfer#4, to-check=76/101)
. . .
``` 

Para mantener dos directorios realmente sincronizados, es necesario eliminar archivos del directorio de destino si se eliminan del origen. De forma predeterminada, rsync no elimina nada del directorio de destino.

Puede cambiar este comportamiento con la --delete opción. Antes de utilizar esta opción, puede utilizar -n la --dry-run opción, para realizar una prueba para evitar la pérdida de datos no deseada:

``` 
rsync -an --delete source destination
``` 

Si prefiere excluir ciertos archivos o directorios ubicados dentro de un directorio que está sincronizando, puede hacerlo especificándolos en una lista separada por comas siguiendo la opción --exclude=:

``` 
rsync -a --exclude=pattern_to_exclude source destination
``` 

Si tiene un patrón específico para excluir, puede anular esa exclusión para archivos que coincidan con un patrón diferente usando la --include=opción:

``` 
rsync -a --exclude=pattern_to_exclude --include=pattern_to_include source destination
``` 

Finalmente, la opción de Rsync --backup se puede utilizar para almacenar copias de seguridad de archivos importantes. Se usa junto con la --backup-dir opción, que especifica el directorio donde se deben almacenar los archivos de respaldo:

```
rsync -a --delete --backup --backup-dir=/path/to/backups /path/to/source destination
``` 

**Conclusión**

Rsync puede agilizar las transferencias de archivos a través de conexiones de red y agregar solidez a la sincronización de directorios locales. La flexibilidad de Rsync lo convierte en una buena opción para muchas operaciones diferentes a nivel de archivos.

