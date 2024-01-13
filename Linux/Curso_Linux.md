# Curso de Linux básico

## 1. Introducción a los comandos

* Para ver la memoria de nuestro equipo:

``` 
free -m
```

* Para ver el listado de un directorio de manera ordenada:

``` 
ls -ls
```

* Para listar aquellos directorios o ficheros que empiezan bien por una letra u por otra:

``` 
ls [af]*
```

* Para listar aquellos directorios o ficheros que terminan con dos digitos:

``` 
ls g*[0-9][0-9]
```

* Para ver una descripción de un archivo, utilizamos el comando **file**:

``` 
file host
```

* Para crear un directorio dentro de otro directorio:

``` 
mkdir -p dir4/prueba
```

* Para copiar el contenido de un archivo en otro existente y queremos que nos pregunte antes:

``` 
cp -i f1.txt f3.txt
```

* Para borrar todos los ficheros de un directorio con extensión .txt, de manera que me pregunte uno a uno si quiero eliminarlos:

``` 
rm -i *txt
```

* Para ver que es cada comando (descripción):

``` 
type + comando a describir
```

* Para ver donde se encuentra bien un comando, ejecutable...

```
which pwd
which firefox
which gedit
```

* Para crear un alias

``` 
alias listadavid='ls -ls /home/david'
```

Y lo ejecuto:

``` 
listadavid
```

* Para ver una fecha determinada del calendario:

``` 
cal -m 6 1987
```

* Para ver cuanto tiempo lleva nuestro equipo encendido, usuario activo...

```
uptime
```

## 2. Enlaces 

* Para crear un enlace duro o físico:

``` 
ln /home/david/fichero1.txt fichero1_enlace.txt
```

Si yo borro ese fichero, al tener un enlace duro el contenido no se borrará realmente. Para ello, tendría que borrar el enlace también.

* Para crear un enlace soft o simbólico:

``` 
ln -s /home/david/fichero1.txt fichero1_enlace.txt
```

## 3. Comando tee

Se utiliza para sacar tanto por la salida estándar como por un fichero los datos a mostrar y conservar. Muestra y redirige al mismo tiempo. Ejemplo:

``` 
ls -ls /home/david | tee listadavid.txt
```

## 4. Comando cut

Si quiero extraer los homes de los usuarios de /etc/passwd:

``` 
cat /etc/passwd | cut -d ':' f6
```

Con -d definimos el delimiter. 

Si quiero extraer el nombre de los usuarios y sus homes:

``` 
cat /etc/passwd | cut -d ':' f1,6
```

## 5. Procesos

Para ver el proceso padre de un proceso hijo:

``` 
ps -ef | grep + pid de ese proceso hijo (que habríamos visto anteriormente con ps -e)
```

Para ver de donde viene un proceso, su estructura jerárquica:

``` 
pstree + pid
```

```
pstree -s + pid
```

Para ver la prioridad de un proceso:

``` 
ps -L
```

También con:

``` 
ps -elf | less
```

Para ver solo el pid, el nice y el comando de un proceso (por. ej un sleep):

``` 
ps -eo pid,ni,comm | grep sleep
```

Para cambiar la prioridad a un proceso:

``` 
renice +10 + pid del proceso
```

## 6. Chown - Chgroup

Para cambiar el propietario de un archivo.

``` 
sudo chown usuario_nuevo_propietario archivo
```

Para cambiar el grupo propietario de un archivo:

``` 
sudo chgroup grupo_nuevo_propietario archivo
```

## 7. Variables

Para ver las de entorno:

``` 
printenv less
```

Para ver las de entorno y las de shell:

``` 
set | less
```

Para ver la variable de entorno del usuario:

``` 
printenv USER
```

Para ver la variable del directorio home:

``` 
echo $HOME
```

Para ver la variable de Shell:

``` 
echo $SHELL
```

Para ver la variable del lenguaje:

``` 
echo $LANG
```

Para ver las rutas donde se encuentran los ejecutables:

``` 
echo $PATH
```

Para introducir un nuevo directorio ejecutable dentro de PATH:

``` 
PATH=$PATH:/home/david/ejecutable
```

Y comprobamos con **echo $PATH**

**PROMPT**

Para cambiar la variable de nuestro prompt:

``` 
PS1="usuario David"
```

O:

``` 
PS1="Usuario \u: "
```

O añadiendo la fecha:

``` 
PS1="Usuario \u \t: "
```

Para volver al prompt normal de usuario:

``` 
PS1="usuario: "
```

Los archivos donde se encuentras estas variables de prompt:

* /etc/profile
* /etc/bash_profile
* /etc/bash.bashrc

## 8. Watch

Se usa para refrescar el uso de comando en función del tiempo que se le diga por comando:

``` 
watch -n 2 free
```

Con -n indicamos el número de segundos que queremos que se tome el comando para refrescar su acción, en este caso free.

Para ver los procesos que más estan consumiendo en memoria:

``` 
watch -n 2 'ps -ao %mem,comm | sort | tail -n 5'
```

## 9. Usuarios

Para ver detalles de un usuario con respecto a su contraseña:

``` 
chage -l david
```

Para ver todos los datos del usuario actual:

```
id
```

Para crear un usuario junto con su directorio home, su grupo y su shell:

``` 
useradd -d /home/david -g 1007 -s /bin/bash -c 'usuario david' david
```

Para cambiar el nombre a un grupo:

``` 
sudo groupmod -n desa desarrollo (nombre antiguo del grupo)
```

Para cambiar el gid a un grupo:

``` 
sudo groupmod 3005 desa
``` 

Para bloquear la contraseña de una cuenta:

``` 
sudo passwd -l cuenta_usuario
```

Para desbloquear la contraseña de una cuenta:

``` 
sudo passwd -u cuenta_usuario
```

Para ver el estado de la contraseña de una cuenta:

``` 
sudo passwd -S cuenta_usuario
```








