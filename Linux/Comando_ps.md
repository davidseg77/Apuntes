# Comando ps de Linux

## 1. Introducción

El ps comando, abreviatura de Estado del proceso, es una utilidad de línea de comandos que se utiliza para mostrar o ver información relacionada con los procesos que se ejecutan en un sistema Linux. Como todos sabemos, Linux es un sistema multitarea y multiprocesamiento. Por lo tanto, se pueden ejecutar varios procesos simultáneamente sin afectarse entre sí. 

El comando ps enumera los procesos en ejecución actuales junto con sus PID y otros atributos. En esta guía, nos centraremos en el uso del comando ps. Recupera información sobre los procesos de archivos virtuales que se encuentran en el sistema de archivos /proc

## 2. Comando ps sin argumentos

El comando ps sin argumentos enumera los procesos en ejecución en el shell actual

``` 
ps
``` 

La salida consta de cuatro columnas:

* **PID** Este es el ID único del proceso 
* **TTY** Este es el tipo de terminal en el que el usuario inició sesión 
* **TIME** Este es el tiempo en minutos y segundos que el proceso ha estado ejecutándose 
* **CMD** El comando que inició el proceso


## 3. Ver todos los procesos en ejecución en diferentes formatos

Para echar un vistazo a todos los procesos en ejecución, ejecute el comando ps -A o ps -e.

## 4. Ver procesos asociados al terminal

Para ver los procesos asociados con el terminal, ejecute ps -T

## 5. Ver procesos no asociados con la terminal

Para ver todos los procesos con excepción de los procesos asociados con la terminal y los líderes de sesión, ejecute ps -a. Un líder de sesión es un proceso que inicia otros procesos.

## 6. Mostrar todos los procesos en ejecución actuales

Para ver todos los procesos actuales, ejecute

``` 
ps -ax
``` 

-a representan todos los procesos -x y mostrarán todos los procesos, incluso aquellos que no están asociados con el tty actual.

## 7. Mostrar todos los procesos en formato BSD

Si desea visualizar procesos en formato BSD, ejecute

```
ps au 
``` 

O

```
ps aux
``` 

## 8. Para realizar un listado en formato completo

Para ver un listado en formato completo, ejecute

``` 
ps -ef 
``` 

O

```
ps -eF
``` 

## 9. Filtrar procesos según el usuario

Si desea enumerar los procesos asociados con un usuario específico, use la -u bandera como se muestra

``` 
ps -u user
``` 

Por ejemplo

``` 
ps -u jamie
``` 

## 10. Filtrar proceso por proceso de subproceso

Si desea conocer el hilo de un proceso en particular, utilice la -L bandera seguida del PID. Por ejemplo

``` 
ps -L 4264
``` 

## 11. Mostrar todos los procesos ejecutándose como root

A veces, es posible que desee revelar todos los procesos ejecutados por el usuario root. Para lograr esta carrera

``` 
ps -U root -u root
``` 

## 12. Mostrar procesos de grupo

Si desea enumerar todos los procesos asociados a un determinado grupo, ejecute

``` 
ps -fG group_name
``` 

O

``` 
ps -fG groupID
``` 

Por ejemplo:

``` 
ps -fG root
``` 

## 13. PID del proceso de búsqueda

Lo más probable es que normalmente no conozca el PID de un proceso. Puede buscar el PID de un proceso ejecutando

``` 
ps -C process_name
``` 

Por ejemplo:

``` 
ps -C bash
``` 

## 14. Listado de procesos por PID

Puede mostrar procesos por su PID como se muestra

``` 
ps -fp PID
``` 

Por ejemplo:

``` 
ps -fp 1294
``` 

## 15. Para mostrar la jerarquía de procesos en un diagrama de árbol

Por lo general, la mayoría de los procesos se bifurcan de los procesos principales. Conocer esta relación entre padres e hijos puede resultar útil. El siguiente comando busca procesos con el nombre apache2

``` 
ps -f --forest -C bash
``` 

## 16. Mostrar procesos secundarios de un proceso padre

Por ejemplo, si desea mostrar todos los procesos bifurcados que pertenecen a Apache, ejecute

``` 
ps -o pid,uname,comm -C bash
``` 

El primer proceso, que es propiedad de root, es el proceso principal de Apache2 y el resto de los procesos se han bifurcado de este proceso principal. Para mostrar todos los procesos secundarios de Apache2 utilizando el pid del proceso principal de Apache2, ejecute

``` 
ps --ppid PID no.
``` 

Por ejemplo:

``` 
ps --ppid 1294
``` 

## 17. Mostrar hilos de proceso

El comando ps se puede utilizar para ver subprocesos junto con los procesos. El siguiente comando muestra todos los subprocesos propiedad del proceso con PID pid_no

``` 
ps -p pid_no -L
``` 

Por ejemplo:

``` 
ps -p 1294 -L 
``` 

## 18. Mostrar una lista seleccionada de columnas

Puede utilizar el comando ps para mostrar solo las columnas que necesita. Por ejemplo:

``` 
ps -e -o pid,uname,pcpu,pmem,comm
``` 

El comando anterior solo mostrará las columnas PID, nombre de usuario, CPU, memoria y comando.

## 19. Cambiar el nombre de las etiquetas de las columnas

Para cambiar el nombre de las etiquetas de las columnas, ejecute el siguiente comando:

``` 
 ps -e -o pid=PID,uname=USERNAME,pcpu=CPU_USAGE,pmem=%MEM,comm=COMMAND
``` 

## 20. Mostrar el tiempo transcurrido de los procesos.

El tiempo transcurrido se refiere a cuánto tiempo ha estado ejecutándose el proceso

``` 
ps -e -o pid,comm,etime
``` 

La opción -o habilita la columna para el tiempo transcurrido

## 21. Usando el comando ps con grep

El comando ps se puede usar con el comando grep para buscar un proceso en particular, por ejemplo:

``` 
ps -ef  | grep systemd
``` 






