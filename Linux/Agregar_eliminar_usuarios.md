# Cómo agregar y eliminar usuarios en Ubuntu

## 1. Agregar un usuario

Si ha iniciado sesión como usuario root, puede crear un nuevo usuario en cualquier momento ejecutando lo siguiente:

``` 
adduser newuser
``` 

Si ha iniciado sesión como un usuario no root al que se le han otorgado sudo privilegios, puede agregar un nuevo usuario con el siguiente comando:

``` 
sudo adduser newuser
``` 

De cualquier manera, se le pedirá que responda una serie de preguntas:

* Asigne y confirme una contraseña para el nuevo usuario.
* Ingrese cualquier información adicional sobre el nuevo usuario. Esto es opcional y puede omitirse presionando ENTER si no desea utilizar estos campos.
* Finalmente, se le pedirá que confirme que la información que proporcionó fue correcta. Presione Y para continuar.
  
Su nuevo usuario ahora está listo para usar y puede iniciar sesión con la contraseña que ingresó.

Si necesita que su nuevo usuario tenga privilegios administrativos, continúe con la siguiente sección.


## 2. Otorgar privilegios Sudo a un usuario

Si su nuevo usuario debe tener la capacidad de ejecutar comandos con privilegios de root (administrativo), deberá otorgarle acceso a sudo. Examinemos dos enfoques para esta tarea: primero, agregar el usuario a un grupo de usuarios sudo predefinido y, segundo, especificar privilegios por usuario en la configuración de sudo

### 2.1 Agregar el nuevo usuario al grupo Sudo

De forma predeterminada, sudo en los sistemas Ubuntu  está configurado para extender privilegios completos a cualquier usuario del grupo sudo.

Puedes ver en qué grupos se encuentra tu nuevo usuario con el groups comando:

``` 
groups newuser
Output
newuser : newuser
``` 

De forma predeterminada, un nuevo usuario solo está en su propio grupo porque add user lo crea además del perfil de usuario. Un usuario y su propio grupo comparten el mismo nombre. Para agregar el usuario a un nuevo grupo, puede usar el usermod comando:

``` 
usermod -aG sudo newuser
``` 

La -aG opción indica usermod que se agregue el usuario a los grupos enumerados.

Tenga en cuenta que el usermod comando en sí requiere sudo privilegios. Esto significa que solo puede agregar usuarios al sudo grupo si ha iniciado sesión como usuario raíz o como otro usuario que ya se haya agregado como miembro del sudo grupo. En el último caso, tendrás que anteponer este comando a sudo, como en este ejemplo:

``` 
sudo usermod -aG sudo newuser
``` 

### 2.2 Especificación de privilegios de usuario explícitos en/etc/sudoers

Como alternativa a colocar a su usuario en el grupo sudo, puede usar el visudo comando, que abre un archivo de configuración llamado /etc/sudoers en el editor predeterminado del sistema y especifica explícitamente los privilegios por usuario.

Usar visudo es la única forma recomendada de realizar cambios /etc/sudoers porque bloquea el archivo contra múltiples ediciones simultáneas y realiza una verificación de validación de su contenido antes de sobrescribir el archivo. Esto ayuda a evitar una situación en la que realice una configuración incorrecta sudo y no pueda solucionar el problema porque ha perdido sudo privilegios.

Si actualmente ha iniciado sesión como root, ejecute lo siguiente:

``` 
visudo
``` 

Si ha iniciado sesión como usuario no root con sudo privilegios, ejecute el mismo comando con el sudo prefijo:

``` 
sudo visudo
``` 

Tradicionalmente, visudo se abre /etc/sudoers en el vi editor, lo que puede resultar confuso para usuarios inexpertos. De forma predeterminada, en las nuevas instalaciones de Ubuntu, visudo se utilizará el nano editor de texto, que proporciona una experiencia de edición de texto más conveniente y accesible. Utilice las teclas de flecha para mover el cursor y busque la línea que dice lo siguiente:

``` 
/etc/sudoers
root    ALL=(ALL:ALL) ALL
``` 

Debajo de esta línea, agregue la siguiente línea resaltada. Asegúrese de cambiar newuser el nombre del perfil de usuario al que desea otorgar sudo privilegios:

``` 
/etc/sudoers
root    ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
``` 

Agregue una nueva línea como esta para cada usuario al que se le deben otorgar sudo privilegios completos. Cuando haya terminado, guarde y cierre el archivo presionando CTRL + X, seguido de Y y luego ENTER para confirmar.


## 3. Probar los privilegios Sudo de su usuario

Ahora su nuevo usuario puede ejecutar comandos con privilegios administrativos.

Cuando inicie sesión como nuevo usuario, puede ejecutar comandos como su usuario habitual escribiendo comandos normalmente:

``` 
some_command
``` 

Puede ejecutar el mismo comando con privilegios administrativos escribiendo sudo delante del comando:

``` 
sudo some_command
``` 

Al hacer esto, se le pedirá que ingrese la contraseña de la cuenta de usuario habitual con la que inició sesión.


## 4. Eliminar un usuario

En el caso de que ya no necesites un usuario, lo mejor es eliminar la cuenta anterior.

Puedes eliminar al propio usuario, sin eliminar ninguno de sus archivos, ejecutando el siguiente comando como root:

``` 
deluser newuser
``` 

Si ha iniciado sesión como otro usuario no root sudo con privilegios, deberá utilizar lo siguiente:

``` 
sudo deluser newuser
``` 

Si, en cambio, desea eliminar el directorio de inicio del usuario cuando se elimina el usuario, puede ejecutar el siguiente comando como root :

``` 
deluser --remove-home newuser
``` 

Si está ejecutando esto como usuario no root con sudo privilegios, ejecutará el mismo comando con el sudo prefijo:

``` 
sudo deluser --remove-home newuser
``` 

Si previamente configuró sudo privilegios para el usuario que eliminó, es posible que desee eliminar la línea relevante nuevamente:

``` 
visudo
``` 

O utilice el siguiente comando si no es un usuario root con sudo privilegios:

``` 
sudo visudo
/etc/sudoers
root    ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL   # DELETE THIS LINE
``` 

Esto evitará que un nuevo usuario creado con el mismo nombre reciba sudo privilegios accidentalmente.

