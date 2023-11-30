# Cómo editar el archivo Sudoers

## 1. Introducción

La separación de privilegios es uno de los paradigmas de seguridad fundamentales implementados en Linux y sistemas operativos tipo Unix. Los usuarios habituales operan con privilegios limitados para reducir el alcance de su influencia a su propio entorno y no al sistema operativo en general.

Un usuario especial, llamado root, tiene privilegios de superusuario. Esta es una cuenta administrativa sin las restricciones presentes en los usuarios normales. Los usuarios pueden ejecutar comandos con privilegios de superusuario o root de varias maneras diferentes.

En este artículo, analizaremos cómo obtener privilegios de root de forma correcta y segura, con especial atención en la edición del /etc/sudoers archivo.


## 2. Cómo obtener privilegios de root

Hay tres formas básicas de obtener privilegios de root, que varían en su nivel de sofisticación.

### 2.1 Iniciar sesión como root

El método más simple y directo para obtener privilegios de root es iniciar sesión directamente en su servidor como usuario root.

Si está iniciando sesión en una máquina local (o utilizando una función de consola fuera de banda en un servidor virtual), ingrese root su nombre de usuario en el mensaje de inicio de sesión e ingrese la contraseña de root cuando se le solicite.

Si inicia sesión a través de SSH, especifique el usuario raíz antes de la dirección IP o el nombre de dominio en su cadena de conexión SSH:

```
ssh root@server_domain_or_ip
``` 

Si no ha configurado claves SSH para el usuario root, ingrese la contraseña de root cuando se le solicite.

### 2.2 Usar su para convertirse en root

Por lo general, no se recomienda iniciar sesión directamente como root, porque es fácil comenzar a usar el sistema para tareas no administrativas, lo cual es peligroso.

La siguiente forma de obtener privilegios de superusuario le permite convertirse en usuario root en cualquier momento, según lo necesite.

Podemos hacer esto invocando el su comando, que significa "usuario sustituto". Para obtener privilegios de root, escriba:

``` 
su
``` 

Se le solicitará la contraseña del usuario root, después de lo cual se le conectará a una sesión de shell root.

Cuando haya terminado las tareas que requieren privilegios de root, regrese a su shell normal escribiendo:

``` 
exit
``` 

### 2.3 Usar sudo para ejecutar comandos como root

La última forma de obtener privilegios de root que analizaremos es con el sudo comando.

El sudo comando le permite ejecutar comandos únicos con privilegios de root, sin la necesidad de generar un nuevo shell. Se ejecuta así:

``` 
sudo command_to_execute
``` 

A diferencia de su, el sudo comando solicitará la contraseña del usuario actual, no la contraseña de root.

Debido a sus implicaciones de seguridad, sudo el acceso no se otorga a los usuarios de forma predeterminada y debe configurarse antes de que funcione correctamente. 

En la siguiente sección, discutiremos cómo modificar la sudo configuración con mayor detalle.


## 3. ¿Qué es Visudo?

El sudo comando se configura a través de un archivo ubicado en /etc/sudoers.

**Advertencia:** ¡Nunca edites este archivo con un editor de texto normal! ¡Utilice siempre el visudo comando en su lugar!

Debido a que una sintaxis incorrecta en el /etc/sudoers archivo puede dejarlo con un sistema roto donde es imposible obtener privilegios elevados, es importante utilizar el visudo comando para editar el archivo.

El visudo comando abre un editor de texto como de costumbre, pero valida la sintaxis del archivo al guardarlo. Esto evita que los errores de configuración bloqueen sudo las operaciones, que pueden ser su única forma de obtener privilegios de root.

Tradicionalmente, visudo abre el /etc/sudoers archivo con el vi editor de texto. Ubuntu, sin embargo, se ha configurado visudo para utilizar el nano editor de texto.

Si desea volver a cambiarlo a vi, emita el siguiente comando:

``` 
sudo update-alternatives --config editor
Output
There are 4 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /bin/nano            40        auto mode
  1            /bin/ed             -100       manual mode
  2            /bin/nano            40        manual mode
  3            /usr/bin/vim.basic   30        manual mode
  4            /usr/bin/vim.tiny    10        manual mode

Press <enter> to keep the current choice[*], or type selection number:
``` 

Seleccione el número que corresponda con la elección que desea realizar.

Después de haber configurado visudo, ejecute el comando para acceder al /etc/sudoers archivo:

``` 
sudo visudo
```

### 3.1 Cómo modificar el archivo Sudoers

Se le presentará el /etc/sudoers archivo en el editor de texto seleccionado.

Copié y pegué el archivo de Ubuntu 20.04, sin comentarios. 

``` 
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

root    ALL=(ALL:ALL) ALL

%admin ALL=(ALL) ALL
%sudo   ALL=(ALL:ALL) ALL

#includedir /etc/sudoers.d
```

Echemos un vistazo a lo que hacen estas líneas.

### 3.2 Líneas predeterminadas

La primera línea, Defaults env_reset restablece el entorno del terminal para eliminar cualquier variable de usuario. Esta es una medida de seguridad que se utiliza para eliminar variables ambientales potencialmente dañinas de la sudo sesión.

La segunda línea, Defaults mail_badpass le indica al sistema que envíe por correo avisos de sudo intentos de contraseña incorrecta al mailto usuario configurado. De forma predeterminada, esta es la cuenta raíz.

La tercera línea, que comienza con Defaults secure_path=..., especifica PATH (los lugares en el sistema de archivos donde el sistema operativo buscará aplicaciones) que se utilizarán para sudolas operaciones. Esto evita el uso de rutas de usuario que puedan ser perjudiciales.

### 3.3 Líneas de privilegios de usuario

La cuarta línea, que dicta los privilegios del usuario root sudo, es diferente de las líneas anteriores. Echemos un vistazo a lo que significan los diferentes campos:

* root ALL=(ALL:ALL) ALL El primer campo indica el nombre de usuario al que se aplicará la regla ( root ).

* root ALL=(ALL:ALL) ALL El primer "TODO" indica que esta regla se aplica a todos los hosts.

* root ALL=(ALL:ALL) ALL Este "TODO" indica que el usuario root puede ejecutar comandos como todos los usuarios.

* root ALL=(ALL:ALL) ALL Este "TODO" indica que el usuario root puede ejecutar comandos como todos los grupos.

* root ALL=(ALL:ALL) ALL El último “TODO” indica que estas reglas se aplican a todos los comandos.

Esto significa que nuestro usuario root puede ejecutar cualquier comando usando sudo, siempre que proporcione su contraseña.

### 3.4 Líneas de privilegios grupales

Las dos líneas siguientes son similares a las líneas de privilegios de usuario, pero especifican sudo reglas para grupos.

Los nombres que comienzan con % indican nombres de grupos.

Aquí vemos que el grupo de administración puede ejecutar cualquier comando como cualquier usuario en cualquier host. De manera similar, el grupo sudo tiene los mismos privilegios, pero también puede ejecutarse como cualquier grupo.

### 3.5 Línea incluida /etc/sudoers.d

La última línea podría parecer un comentario a primera vista:

``` 
/etc/sudoers
. . .

#includedir /etc/sudoers.d
```

Comienza con , que normalmente # indica un comentario. Sin embargo, esta línea en realidad indica que los archivos dentro del /etc/sudoers.d directorio también se obtendrán y aplicarán.

Los archivos dentro de ese directorio siguen las mismas reglas que el /etc/sudoers archivo mismo. Cualquier archivo que no termine en ~ y que no tenga . entrada será leído y agregado a la sudo configuración.

Esto está destinado principalmente a que las aplicaciones modifiquen sudo los privilegios durante la instalación. Poner todas las reglas asociadas dentro de un solo archivo en el /etc/sudoers.d directorio puede hacer que sea más fácil ver qué privilegios están asociados con qué cuentas y revertir las credenciales fácilmente sin tener que intentar manipular el /etc/sudoers archivo directamente.

Al igual que con el /etc/sudoers archivo en sí, siempre debes editar los archivos dentro del /etc/sudoers.d directorio con extensión visudo. La sintaxis para editar estos archivos sería:

``` 
sudo visudo -f /etc/sudoers.d/file_to_edit
``` 

## 4. Cómo otorgar privilegios de Sudo a un usuario

La operación más común que los usuarios desean realizar al administrar sudo permisos es otorgar sudo acceso general a un nuevo usuario. Esto es útil si desea otorgarle a una cuenta acceso administrativo completo al sistema.

La forma más sencilla de hacer esto en un sistema configurado con un grupo de administración de propósito general, como el sistema Ubuntu en esta guía, es agregar el usuario en cuestión a ese grupo.

Por ejemplo, en Ubuntu 20.04, el sudo grupo tiene privilegios de administrador completos. Podemos otorgarle a un usuario estos mismos privilegios agregándolo al grupo de esta manera:

``` 
sudo usermod -aG sudo username
``` 

El gpasswd comando también se puede utilizar:

``` 
sudo gpasswd -a username sudo
``` 

Ambos lograrán lo mismo.


## 5. Cómo configurar reglas personalizadas

Ahora que nos hemos familiarizado con la sintaxis general del archivo, creemos algunas reglas nuevas.

### 5.1 Cómo crear alias

El sudoers archivo se puede organizar más fácilmente agrupando elementos con varios tipos de "alias".

Por ejemplo, podemos crear tres grupos diferentes de usuarios, con membresías superpuestas:

``` 
/etc/sudoers
. . .
User_Alias		GROUPONE = abby, brent, carl
User_Alias		GROUPTWO = brent, doris, eric,
User_Alias		GROUPTHREE = doris, felicia, grant
. . .
``` 

Los nombres de los grupos deben comenzar con letra mayúscula. Luego podemos permitir que los miembros GROUPTWO actualicen la apt base de datos creando una regla como esta:

```
/etc/sudoers
. . .
GROUPTWO	ALL = /usr/bin/apt-get update
. . .
``` 

Si no especificamos un usuario/grupo para ejecutar, como se indicó anteriormente, el sudo valor predeterminado es el usuario raíz.

Podemos permitir que los miembros GROUPTHREE apaguen y reinicien la máquina creando un "alias de comando" y usándolo en una regla para GROUPTHREE:

``` 
/etc/sudoers
. . .
Cmnd_Alias		POWER = /sbin/shutdown, /sbin/halt, /sbin/reboot, /sbin/restart
GROUPTHREE	ALL = POWER
. . .
``` 

Creamos un alias de comando llamado POWER que contiene comandos para apagar y reiniciar la máquina. Luego permitimos que los miembros de GROUPTHREE ejecuten estos comandos.

También podemos crear alias de "Ejecutar como", que pueden reemplazar la parte de la regla que especifica que el usuario ejecutará el comando como:

``` 
/etc/sudoers
. . .
Runas_Alias		WEB = www-data, apache
GROUPONE	ALL = (WEB) ALL
. . .
``` 

Esto permitirá que cualquier persona que sea miembro GROUPONE ejecute comandos como el www-data usuario o el apache usuario.

Sólo tenga en cuenta que las reglas posteriores anularán las reglas anteriores cuando haya un conflicto entre las dos.

### 5.2 Cómo bloquear las reglas

Hay varias formas de lograr un mayor control sobre cómo sudo reacciona ante una llamada.

El updatedb comando asociado con el mlocate paquete es relativamente inofensivo en un sistema de usuario único. Si queremos permitir que los usuarios lo ejecuten con privilegios de root sin tener que escribir una contraseña, podemos hacer una regla como esta:

``` 
/etc/sudoers
. . .
GROUPONE	ALL = NOPASSWD: /usr/bin/updatedb
. . .
``` 

NOPASSWD es una “etiqueta” que significa que no se solicitará ninguna contraseña. Tiene un comando complementario llamado PASSWD, que es el comportamiento predeterminado. Una etiqueta es relevante para el resto de la regla a menos que sea anulada por su etiqueta "gemela" más adelante.

Por ejemplo, podemos tener una línea como esta:

```
/etc/sudoers
. . .
GROUPTWO	ALL = NOPASSWD: /usr/bin/updatedb, PASSWD: /bin/kill
. . .
``` 

Otra etiqueta útil es NOEXEC, que puede usarse para evitar comportamientos peligrosos en ciertos programas.

Por ejemplo, algunos programas, como less, pueden generar otros comandos escribiendo esto desde su interfaz:

``` 
!command_to_run
``` 

Básicamente, esto ejecuta cualquier comando que el usuario le dé con los mismos permisos con los que less se ejecuta, lo que puede ser bastante peligroso.

Para restringir esto, podríamos usar una línea como esta:

``` 
/etc/sudoers
. . .
username	ALL = NOEXEC: /usr/bin/less
. . .
``` 


## 6. Información Variada

Hay algunos datos más que pueden resultar útiles a la hora de tratar con sudo.

Si especificó un usuario o grupo para "ejecutar como" en el archivo de configuración, puede ejecutar comandos como esos usuarios usando las banderas -u y -g, respectivamente:

``` 
sudo -u run_as_user command
sudo -g run_as_group command
``` 

Para mayor comodidad, de forma predeterminada, sudo se guardarán sus datos de autenticación durante un cierto período de tiempo en un terminal. Esto significa que no tendrá que volver a escribir su contraseña hasta que se acabe el tiempo.

Por motivos de seguridad, si desea borrar este temporizador cuando haya terminado de ejecutar comandos administrativos, puede ejecutar:

``` 
sudo -k
``` 

Si, por el contrario, desea "preparar" el sudo comando para que no se le solicite más tarde, o renovar su sudo contrato de arrendamiento, siempre puede escribir:

``` 
sudo -v
``` 

Se le solicitará su contraseña, que se almacenará en caché para sudo usos posteriores hasta que sudo expire el plazo.

Si simplemente se pregunta qué tipo de privilegios están definidos para su nombre de usuario, puede escribir:

``` 
sudo -l
``` 

Esto enumerará todas las reglas del /etc/sudoers archivo que se aplican a su usuario. Esto le da una buena idea de lo que se le permitirá o no hacer sudo como cualquier usuario.

Hay muchas ocasiones en las que ejecuta un comando y falla porque olvidó anteponerlo sudo. Para evitar tener que volver a escribir el comando, puedes aprovechar una funcionalidad bash que significa "repetir el último comando":

``` 
sudo !!
``` 

El doble signo de exclamación repetirá el último comando. Lo precedimos con sudo cambiar rápidamente el comando sin privilegios a un comando con privilegios.

Para divertirte, puedes agregar la siguiente línea a tu /etc/sudoers archivo con visudo:

``` 
sudo visudo
/etc/sudoers
. . .
Defaults	insults
. . .
``` 

Esto provocará sudo que se devuelva un insulto tonto cuando un usuario escriba una contraseña incorrecta para sudo. Podemos usar sudo -k para borrar la sudo contraseña anterior almacenada en caché para probarla:

``` 
sudo -k
sudo ls
Output
[sudo] password for demo:    # enter an incorrect password here to see the results
Your mind just hasn't been the same since the electro-shock, has it?
[sudo] password for demo:
My mind is going. I can feel it.
``` 

