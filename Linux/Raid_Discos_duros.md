# Como CREAR un RAID de discos duros en LINUX

## 1. Escenario 

Para mostrar toda la info de la distribución que estamos usando.

``` 
lsb_release -a
```

## 2. Añadir los discos duros

Primero, apagamos la máquina. Vamos a propiedades, dentro de la VM en VirtualBox y en Almacenamiento, en Controlador Sata añadimos disco. Creamos uno de 1GB. Creamos 6, uno para el Raid y uno para reserva.
Y trás esto, damos en añadir.

Volvemos a encender la máquina y comprobamos identificador de los discos.

``` 
lsblk
```

## 3. Crear las particiones

Creamos la tabla de particiones del primer disco:

``` 
sudo fdisk /dev/sdb
```

Primero insertamos la m para ver la ayuda del comando fdisk. Con la n añadimos una nueva partición. Le decimos que queremos que sea primaria (p), y damos al número 1, de 1 sola partición. Después, con la p podemos ver la tabla de particiones creada. 

Con l listamos todas las particiones. Damos a la t para cambiar el tipo de partición, y ponemos el code FD (tipo RAID). Si ponemos la p volvemos a ver la tabla de particiones creada de tipo RAID. Con la w guardamos los cambios. Nos dirá que se están sincronizando los discos.

Con los discos restantes, los cinco restantes, utilizamos la siguiente herramienta para copiar la tabla de particiones a todos ellos.

``` 
sfdisk -d /dev/sdb | sfdisk /dev/sdc
sfdisk -d /dev/sdb | sfdisk /dev/sdd
sfdisk -d /dev/sdb | sfdisk /dev/sde
sfdisk -d /dev/sdb | sfdisk /dev/sdf
sfdisk -d /dev/sdb | sfdisk /dev/sdg
```

Con lsblk comprobamos que las particiones se han hecho completamente para cada disco.

## 4. Crear el RAID de discos

Instalar el paquete de gestión RAID

``` 
sudo apt-get update
sudo apt-get install mdadm
```

## 5. Crear el RAID 6

``` 
sudo mdadm --create --verbose /dev/md0 --level=6 --raid-device=5 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1 /dev/sdf1
```

* **create** para crear un raid
* **verbose** para mostrar más info
* **/dev/md0** dispositivo del RAID
* **level6** tipo de RAID
* **raid-devices=5** nº de discos en el RAID
  
El proceso tarda un tiempo en sincronizar. Para ver dicho proceso:

``` 
sudo cat /proc/mdstat
```

Para actualizar solo el comando:

```
sudo watch -n1 sudo cat /proc/mdstat
```

Para ver detalles del RAID:

``` 
sudo mdadm --detail /dev/md0
```

Para examinar todos los discos:

``` 
sudo mdadm -E /dev/sd[b-f]1
```

## 6. Crear y montar el filesystem

Ahora mismo tenemos creado el dispositivo pero no el Filesystem. 

``` 
sudo mkfs.ext4 /dev/md0
```

Creamos el directorio de montaje:

``` 
sudo mkdir /mnt/Raid6
```

Montamos el RAID sobre el directorio:

```
sudo mount /dev/md0 mnt/Raid6
```

Comprobamos la info de los discos

``` 
lsblk
```

Verificamos el tamaño del RAID:

``` 
sudo fdisk -l /dev/md0

o

df -h
```

## 7. Comprobar el funcionamiento del RAID

Ya podemos guardar archivos en /mnt/Raid6

```
cd /mnt/Raid6
```

Creamos un fichero

``` 
sudo touch david.txt
ls -l
``` 

Lo editamos:

``` 
sudo nano david.txt
```

Y comprobamos el fichero con la nueva info:

```
cat david.txt
```

## 8. Guardar configuración para mantener RAID trás el inicio

El problema consiste en que si reiniciamos ahora el dispositivo md0 montado en Raid6 se perdería. 

Para evitarlo, escaneamos el RAID activo y lo añadimos a mdadm.conf:

``` 
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

Actualizamos el fichero de sistema de la RAM inicial:

``` 
sudo update-initramfs -u
```

Editamos el fichero:

``` 
sudo nano /etc/fstab
```

Añadimos la siguiente línea al final:

``` 
echo '/dev/md0 /mnt/Raid6 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

* /dev/md0 = Sistema de archivos
* /mnt/Raid6 = Directorio del punto de montaje del Raid
* ext4 = Tipo del sistema de archivos
* defaults, nofail, discard = Opciones
* 0 = Volcado
* 0 = Pass

## 9. Añadir disco de reserva al Raid

Para ampliar el Raid, añadimos un disco. 

Comprobamos la info de los discos:

``` 
lsblk
```

Detalles del RAID:

``` 
sudo mdadm --detail /dev/md0
```

Añadimos partición al RAID:

``` 
sudo mdadm --add /dev/md0 /dev/sdg1
```

Detalles del RAID de nuevo:

``` 
sudo mdadm --detail /dev/md0 
```

Como vemos, se queda el disco en reserva.

Ahora generamos un fallo

``` 
sudo mdadm --manage --fail /dev/md0 /dev/sdd1
```

Y verificamos el estado de los discos del RAID:

``` 
sudo mdadm --detail /dev/md0 
```

Quitamos el disco del RAID que ha fallado:

``` 
sudo mdadm --manage /dev/md0 --remove /dev/sdd1
```

Verificamos estado:

``` 
sudo mdadm -D /dev/md0
```

Añadimos de nuevo el disco:

``` 
sudo mdadm --manage /dev/md0 --add /dev/sdd1
```

Y verificamos estado:

``` 
sudo mdadm -D /dev/md0
```

