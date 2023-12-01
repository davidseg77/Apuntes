# Comando ls en Linux/UNIX

## 1. Listado de archivos con el comando ls sin ningún argumento

El comando ls sin opciones enumera archivos y directorios en un formato simple sin mostrar mucha información como tipos de archivos, permisos, fecha y hora de modificación, por mencionar solo algunos. Sintaxis

``` 
$ ls
``` 

## 2. Listado de archivos en orden inverso

Para enumerar los archivos en orden inverso, agregue el indicador -r como se muestra. Sintaxis

```
$ ls -r
``` 

## 3. Listado de permisos de archivos y directorios con la opción -l

usando el indicador -l, puede enumerar los permisos de los archivos y directorios, así como otros atributos como nombres de carpetas, tamaños de archivos y directorios, y fecha y hora de modificación. Sintaxis

``` 
$ ls -l
```

## 4. Ver archivos en un formato legible por humanos

Como habrás notado, los tamaños de archivos y carpetas que se muestran no son fáciles de descifrar y no tienen sentido a primera vista. Para identificar fácilmente los tamaños de archivos como kilobytes (kB), Megabytes (MB) o Gigabytes (GB), agregue el indicador -lh como se muestra .

``` 
$ ls -lh
``` 

## 5. Ver archivos ocultos

Puede ver archivos ocultos agregando el indicador -a. Los archivos ocultos suelen ser archivos del sistema que comienzan con un punto o punto. Sintaxis

``` 
$ ls -a
``` 

## 6. Listado de archivos de forma recursiva

Para mostrar el árbol de directorios de archivos y carpetas, use el ls -R comando como se muestra Sintaxis

``` 
$ ls -R
``` 

## 7. Listado de archivos y directorios con el carácter '/' al final

Si desea seguir adelante y distinguir aún más los archivos de las carpetas, utilice la opción -F para que la carpeta aparezca con un carácter de barra diagonal '/' al final. Sintaxis

``` 
$ ls -F
``` 

## 8. Mostrar el número de inodo de archivos y directorios

Para mostrar el número de inodo de archivos y directorios, agregue la -i bandera al final del comando ls como se muestra Sintaxis

```
$ ls -i
```

## 9. Visualización del UID y GID de archivos y directorios

Si desea mostrar el UID así como el GId de archivos y directorios, agregue el parámetro -n como se muestra Sintaxis

``` 
$ ls -n
``` 

## 10. Definición del comando ls en alias

Los alias son comandos personalizados o modificados en el shell de Linux que se utilizan en lugar de los comandos originales. Podemos crear un alias para el comando ls de esta manera. Sintaxis

``` 
$ alias="ls -l"
``` 

Lo que esto hace es que le dice al sistema que ejecute el ls -l comando en lugar del ls comando. Asegúrese de observar que el resultado que obtenga al ejecutar el comando ls a partir de entonces será como si ejecutara el ls -l comando.

Para eliminar el alias agregado, ejecute

``` 
unalias ls
``` 

## 11. Colorear la salida del comando ls

Para agregar algo de estilo a la visualización de salida según los tipos de archivos, es posible que desee colorear su salida para distinguir fácilmente archivos, carpetas y otros atributos, como permisos de archivos y directorios. Para lograr esto ejecute la sintaxis

``` 
ls --color
``` 

## 12. Mostrando la versión del comando ls

Si tiene un poco de curiosidad sobre qué versión de ls está ejecutando, ejecute el siguiente comando

```
# ls --v
ls (GNU coreutils) 8.22
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Richard M. Stallman and David MacKenzie.
#
``` 

También puede ejecutar el comando ls --version para imprimir la versión del comando ls.


## 13. Mostrando la página de ayuda del comando ls

Para ver más opciones y lo que puedes hacer con ls simplemente ejecuta:

``` 
ls --help
``` 

## 14. Accediendo a las páginas man de ls

Alternativamente, puede ver las páginas de manual para obtener más información sobre su uso ejecutando

``` 
man ls
``` 

