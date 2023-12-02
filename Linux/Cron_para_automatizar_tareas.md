# Cómo usar Cron para automatizar tareas en Ubuntu

## 1. Introducción

Cron es un demonio de programación de trabajos basado en el tiempo que se encuentra en sistemas operativos tipo Unix, incluidas las distribuciones de Linux. Cron se ejecuta en segundo plano y las operaciones programadas con cron, denominadas "trabajos cron", se ejecutan automáticamente, lo que hace que cron sea útil para automatizar tareas relacionadas con el mantenimiento.

Esta guía proporciona una descripción general de cómo programar tareas utilizando la sintaxis especial de cron. También repasa algunos atajos que puede utilizar para acelerar el proceso de redacción de cronogramas de trabajo y hacerlos más comprensibles.


## 2. Instalación de cron

Casi todas las distribuciones de Linux tienen algún tipo de croninstalado de forma predeterminada. Sin embargo, si está utilizando una máquina Ubuntu en la que cronno está instalado, puede instalarlo usando APT.

Antes de instalar cron en una máquina Ubuntu, actualice el índice del paquete local de la computadora:

``` 
sudo apt update
``` 

Luego instale croncon el siguiente comando:

``` 
sudo apt install cron
``` 

También deberás asegurarte de que esté configurado para ejecutarse en segundo plano:

``` 
sudo systemctl enable cron
Output
Synchronizing state of cron.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable cron
``` 

Después de eso, cronse instalará en su sistema y estará listo para que pueda comenzar a programar trabajos.


## 3. Comprender cómo funciona Cron

Los trabajos cron se registran y administran en un archivo especial conocido como crontab. Cada perfil de usuario en el sistema puede tener su propio perfil crontab donde programar trabajos, que se almacena en /var/spool/cron/crontabs/.

Para programar un trabajo, abra su crontab para editar y agregue una tarea escrita en forma de expresión cron. La sintaxis de las expresiones cron se puede dividir en dos elementos: la programación y el comando a ejecutar.

El comando puede ser prácticamente cualquier comando que normalmente ejecutaría en la línea de comando. El componente de programación de la sintaxis se divide en 5 campos diferentes, que se escriben en el siguiente orden:

* minuto = 0-59
* hora = 0-23
* Día del mes =	1-31
* mes = 1-12 o JAN-DEC
* Día de la semana = 0-6 o SUN-SAT
  
En conjunto, las tareas programadas en un crontab se estructuran de la siguiente manera:

``` 
minute hour day_of_month month day_of_week command_to_run
``` 

A continuación se muestra un ejemplo funcional de una expresión cron. Esta expresión ejecuta el comando curl http://www.google.comtodos los martes a las 5:30 p.m.:

``` 
30 17 * * 2 curl http://www.google.com
``` 

También hay algunos caracteres especiales que puede incluir en el componente de programación de una expresión cron para agilizar la programación de tareas:

* ***:** En expresiones cron, un asterisco es una variable comodín que representa "todos". Por lo tanto, una tarea programada con * * * * * ...se ejecutará cada minuto de cada hora de cada día de cada mes.
* **,:** las comas dividen los valores de programación para formar una lista. Si desea ejecutar una tarea al principio y a la mitad de cada hora, en lugar de escribir dos tareas separadas (por ejemplo, 0 * * * * ...y 30 * * * * ...), puede lograr la misma funcionalidad con una ( 0,30 * * * * ...).
* **-:** Un guión representa un rango de valores en el campo de programación. En lugar de tener 30 tareas programadas separadas para un comando que desea ejecutar durante los primeros 30 minutos de cada hora (como en 0 * * * * ..., 1 * * * * ..., 2 * * * * ...etc.), puede programarlo como 0-29 * * * * ....
* **/:** Puede utilizar una barra diagonal con un asterisco para expresar un valor de paso. Por ejemplo, en lugar de escribir ocho tareas separadas cronpara ejecutar un comando cada tres horas (como en, 0 0 * * * ..., 0 3 * * * ..., 0 6 * * * ...etc.), puede programarlo para que se ejecute de esta manera: 0 */3 * * * ....

Si algo de esto le resulta confuso o si desea ayuda para escribir cronogramas para sus propias crontareas, Cronitor proporciona un práctico croneditor de expresiones de cronograma llamado "Crontab Guru" que puede utilizar para verificar si sus croncronogramas son válidos.


## 4. Administrar crontabs

Una vez que haya establecido un cronograma y sepa el trabajo que desea ejecutar, deberá colocarlo en algún lugar donde su demonio pueda leerlo.

Como se mencionó anteriormente, un crontab es un archivo especial que contiene el cronograma de trabajos cron que se ejecutarán. Sin embargo, estos no están pensados ​​para editarse directamente. En su lugar, se recomienda utilizar el crontab comando. Esto le permite editar su perfil de usuario crontab sin cambiar sus privilegios con sudo. El crontab comando también le permitirá saber si tiene errores de sintaxis en el archivo crontab, mientras que editarlo directamente no lo hará.

Puedes editar tu crontab con el siguiente comando:

``` 
crontab -e
``` 

Si es la primera vez que ejecuta el crontab comando con este perfil de usuario, se le pedirá que seleccione un editor de texto predeterminado para usar al editar su crontab:

``` 
Output
no crontab for sammy - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 
```

Ingrese el número correspondiente al editor de su elección. Alternativamente, puede presionar ENTER para aceptar la opción predeterminada nano.

Después de hacer su selección, accederá a una nueva página crontab que contiene algunas instrucciones comentadas sobre cómo usarlo:

``` 
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
```

Cuando lo ejecute crontab -e en el futuro, aparecerá crontab automáticamente en este editor de texto. Una vez en el editor, puede ingresar su cronograma con cada trabajo en una nueva línea. De lo contrario, puede guardar y cerrar el crontab por ahora ( CTRL + X,, Y luego ENTER si seleccionó nano).

**Nota:** En los sistemas Linux, hay otro crontabalmacenado en el /etc/directorio. Este es un sistema para todo el sistema crontab que tiene un campo adicional para el perfil de usuario cron bajo el cual se debe ejecutar cada trabajo. Este tutorial se centra en temas específicos del usuario crontabs, pero si desea editar el sistema en todo el sistema crontab, puede hacerlo con el siguiente comando:

``` 
sudo nano /etc/crontab
``` 

Si desea ver el contenido de su archivo crontab, pero no editarlo, puede usar el siguiente comando:

``` 
crontab -l
``` 

Puedes borrar tu crontab con el siguiente comando:

```
crontab -r
``` 

Este comando eliminará el del usuario crontab inmediatamente. Sin embargo, puede incluir la -i marca para que el comando le solicite que confirme que realmente desea eliminar el usuario crontab:

``` 
crontab -r -i
Output
crontab: really delete sammy's crontab? (y/n)
``` 

Cuando se le solicite, debe ingresar y para eliminar crontab o n cancelar la eliminación.


## 5. Administrar la salida del trabajo cron

Debido a que cron los trabajos se ejecutan en segundo plano, no siempre es evidente que se hayan ejecutado correctamente. Ahora que sabe cómo utilizar el crontab comando y cómo programar un cron trabajo, puede comenzar a experimentar con diferentes formas de redirigir la salida de los cron trabajos para ayudarle a realizar un seguimiento de si se han ejecutado correctamente.

Si tiene un agente de transferencia de correo, como Sendmail, instalado y configurado correctamente en su servidor, puede enviar el resultado de las cron tareas a la dirección de correo electrónico asociada con su perfil de usuario de Linux. También puede especificar manualmente una dirección de correo electrónico proporcionando una MAILTO configuración en la parte superior del archivo crontab.

Por ejemplo, podría agregar las siguientes líneas a un archivo crontab. Estos incluyen una MAILTO declaración seguida de una dirección de correo electrónico de ejemplo, una SHELL directiva que indica el shell que se debe ejecutar (bash en este ejemplo), una HOME directiva que apunta a la ruta en la que buscar el cron binario y una única cron tarea:

``` 
. . .

MAILTO="example@digitalocean.com"
SHELL=/bin/bash
HOME=/

* * * * * echo ‘Run this command every minute’
``` 

Este trabajo en particular devolverá "Ejecute este comando cada minuto" y ese resultado se enviará por correo electrónico cada minuto a la dirección de correo electrónico especificada después de la MAILTO directiva.

También puede redirigir el cron resultado de una tarea a un archivo de registro o a una ubicación vacía para evitar recibir un correo electrónico con el resultado.

Para agregar la salida de un comando programado a un archivo de registro, agréguelo >> al final del comando seguido del nombre y la ubicación de un archivo de registro de su elección, como este:

``` 
* * * * * echo ‘Run this command every minute’ >> /directory/path/file.log
``` 

Supongamos que desea utilizar cron para ejecutar un script pero mantenerlo ejecutándose en segundo plano. Para hacerlo, puede redirigir la salida del script a una ubicación vacía, como /dev/null la cual elimina inmediatamente cualquier dato escrito en él. Por ejemplo, el siguiente cron trabajo ejecuta un script PHP y lo ejecuta en segundo plano:

``` 
* * * * * /usr/bin/php /var/www/domain.com/backup.php > /dev/null 2>&1
``` 

Este cron trabajo también redirige el error estándar, representado por 2, a la salida estándar (>&1). Debido a que la salida estándar ya se está redirigiendo a /dev/null, esto esencialmente permite que el script se ejecute de manera silenciosa. Incluso si crontab contiene una MAILTO declaración, el resultado del comando no se enviará a la dirección de correo electrónico especificada.


## 6. Restringir el acceso

Puede administrar qué usuarios pueden usar el crontab comando con los archivos cron.allow y cron.deny, ambos almacenados en el /etc/directorio. Si el cron.deny archivo existe, cualquier usuario que figure en él no podrá editar su archivo crontab. Si cron.allow existe, solo los usuarios enumerados en él podrán editar sus crontabs. Si ambos archivos existen y el mismo usuario aparece en cada uno, el cron.allow archivo se anulará cron.deny y el usuario podrá editar su archivo crontab.

Por ejemplo, para denegar el acceso a todos los usuarios y luego otorgar acceso al usuario ishmael, puede usar la siguiente secuencia de comandos:

``` 
sudo echo ALL >>/etc/cron.deny
sudo echo ishmael >>/etc/cron.allow
``` 

Primero, bloqueamos a todos los usuarios agregándolos ALL al cron.deny archivo. Luego, al agregar el nombre de usuario al cron.allow archivo, le damos acceso al perfil de usuario de Ismael para ejecutar cron trabajos.

Tenga en cuenta que si un usuario tiene sudo privilegios, puede editar los de otro usuario crontab con el siguiente comando:

``` 
sudo crontab -u user -e
``` 

Sin embargo, si cron.deny existe y usuario aparece en él y no aparecen en cron.allow, recibirá el siguiente error después de ejecutar el comando anterior:

``` 
Output
The user user cannot use this program (crontab)
``` 

De forma predeterminada, la mayoría de cron los demonios asumirán que todos los usuarios tienen acceso cron a menos que exista cron.allow o .cron.deny





