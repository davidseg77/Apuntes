# Comandos para la gestión de procesos en Linux

## 1. Diferentes comandos para la gestión de procesos en Linux

Hay dos comandos disponibles en Linux para rastrear los procesos en ejecución. Estos dos comandos son Top y Ps.

### 1.1 El comando Top para administrar procesos de Linux

Para rastrear los procesos en ejecución en su máquina, puede usar el comando superior.

``` 
$ top 
``` 

El comando superior muestra una lista de procesos que se ejecutan en tiempo real junto con su uso de memoria y CPU. Entendamos un poco mejor el resultado:

* **PID:** ID de proceso único proporcionado a cada proceso.
* **Usuario:** Nombre de usuario del propietario del proceso.
* **PR:** Prioridad dada a un proceso durante la programación.
* **NI:** valor 'bonito' de un proceso.
* **VIRT:** Cantidad de memoria virtual utilizada por un proceso.
* **RES:** Cantidad de memoria física utilizada por un proceso.
* **SHR:** Cantidad de memoria compartida con otros procesos.
* **S:** estado del proceso
    - **'D'=** sueño ininterrumpido
    - **'R'=** corriendo
    - * **'S'=** durmiendo
    - * **'T'=** rastreado o detenido
    - * **'Z'=** zombi
  
* **%CPU:** Porcentaje de CPU utilizada por el proceso.
* **%MEM;** Porcentaje de RAM utilizada por el proceso.
* **TIEMPO+:** Tiempo total de CPU consumido por el proceso.
* **Comando:** Comando utilizado para activar el proceso.
  
Puede utilizar las teclas de flecha arriba/abajo para navegar hacia arriba y hacia abajo por la lista. Para salir presione q. Para finalizar un proceso, resalte el proceso con las teclas de flecha arriba/abajo y presione 'k'.

Alternativamente, también puedes usar el comando Kill, que veremos más adelante.


### 1.2 Comando ps

El comando ps es la abreviatura de "Estado del proceso". Muestra los procesos que se están ejecutando actualmente. Sin embargo, a diferencia del comando superior, la salida generada no es en tiempo real.

``` 
$ ps
``` 

La terminología es la siguiente:

**PID**	identificacion de proceso
**TTY**	tipo de terminal
**TIEMPO**	tiempo total que el proceso ha estado ejecutándose
**CMD**	nombre del comando que inicia el proceso

Para obtener más información usando el comando ps, use:

``` 
$ ps -u
```

Aquí:

* El % de CPU representa la cantidad de potencia informática que requiere el proceso.
* %MEM representa la cantidad de memoria que está ocupando el proceso.
* STAT representa el estado del proceso.

Si bien el comando ps solo muestra los procesos que se están ejecutando actualmente, también puede usarlo para enumerar todos los procesos.

``` 
$ ps -A 
``` 

Este comando enumera incluso aquellos procesos que actualmente no se están ejecutando.


### 1.3 Detener un proceso

Para detener un proceso en Linux, use el comando ' kill'. El comando kill envía una señal al proceso.

Hay diferentes tipos de señales que puedes enviar. Sin embargo, el más común es 'kill -9' que es 'SIGKILL'.

Puede enumerar todas las señales usando:

``` 
$ kill -L
``` 

La señal predeterminada es 15, que es SIGTERM. Lo que significa que si simplemente usas el comando kill sin ningún número, envía la señal SIGTERM.

La sintaxis para matar un proceso es:

``` 
$ kill [pid]
``` 

Alternativamente también puedes usar:

``` 
$ kill -9 [pid]
``` 

Este comando enviará una señal 'SIGKILL' al proceso. Esto debe usarse en caso de que el proceso ignore una solicitud de eliminación normal.


### 1.4 Cambiar la prioridad de un proceso

En Linux, puedes priorizar entre procesos. El valor de prioridad de un proceso se denomina valor de "amabilidad". El valor de amabilidad puede oscilar entre -20 y 19. 0 es el valor predeterminado.

La cuarta columna en el resultado del comando superior es la columna del valor de amabilidad.

Para iniciar un proceso y darle un valor agradable distinto al predeterminado, use:

``` 
$ nice -n [value] [process name]
``` 

Para cambiar el valor agradable de un proceso que ya se está ejecutando, utilice:

``` 
renice [value] -p 'PID'
``` 

Que pasa Luis