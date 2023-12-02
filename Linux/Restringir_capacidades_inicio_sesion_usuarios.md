# Cómo restringir las capacidades de inicio de sesión de los usuarios en Ubuntu

## 1. Cómo restringir el acceso con /etc/passwd

Un método para restringir las capacidades de inicio de sesión es configurar el shell de inicio de sesión de la cuenta en un valor especial.

Un ejemplo de esto es el messagebus usuario, que puedes usar grep para buscar dentro del /etc/passwd archivo:

``` 
less /etc/passwd | grep messagebus
Output
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
``` 

El valor final es el shell o el comando que se ejecuta cuando el inicio de sesión es exitoso. En este caso, el valor se establece en /usr/sbin/nologin.

Si intenta cambiar al messagebus usuario usando sudo su, fallará:

``` 
sudo su messagebus
Output
This account is currently not available.
``` 

Recibe este mensaje porque el shell para messagebus está configurado en /usr/sbin/nologin.

Puede usar la usermod herramienta para cambiar el shell de inicio de sesión predeterminado de un usuario (generalmente /bin/bash) a un shell inexistente, como nologin cuando necesita evitar que inicien sesión.

``` 
sudo usermod -s /usr/sbin/nologin username
``` 


## 2. Cómo restringir el acceso con /etc/shadow

Otro método similar para restringir el acceso es utilizar el /etc/shadow archivo. Este archivo contiene los valores de contraseña hash de cada usuario del sistema.

Puede utilizar less para ver el archivo completo:

``` 
sudo less /etc/shadow
Output
. . .
uuidd:*:19105:0:99999:7:::
tcpdump:*:19105:0:99999:7:::
sshd:*:19105:0:99999:7:::
pollinate:*:19105:0:99999:7:::
landscape:*:19105:0:99999:7:::
lxd:!:19180::::::
sammy:$y$j9T$4gyOQ5ieEWdx1ZdggX3Nj1$AbEA9FsG03aTsQhl.ZVMXatwCAvnxFbE/GHUKpjf9u6:19276:0:99999:7:::
``` 

El segundo campo (el que comienza $y$j9T$4gyO…en la última línea) contiene el valor de la contraseña con hash.

Las cuentas del sistema tienen un asterisco (*) en lugar de un valor hash complejo. Las cuentas con un asterisco en el segundo campo no tienen una contraseña establecida y no pueden autenticarse mediante contraseña sin cambiarla.

Puede deshabilitar un valor de contraseña (de hecho, crear una cuenta con una contraseña equivalente al valor de asterisco) precediendo el valor hash con un signo de exclamación (!).

Dos herramientas pueden hacer esto "bloqueando" la cuenta especificada.

El passwd comando se puede bloquear con la -l bandera y desbloquear con la -u bandera:

``` 
sudo passwd -l sammy
sudo less /etc/shadow | grep sammy
Output
sammy:!$y$j9T$4gyOQ5ieEWdx1ZdggX3Nj1$AbEA9FsG03aTsQhl.ZVMXatwCAvnxFbE/GHUKpjf9u6:19276:0:99999:7::::::
``` 

Como puede ver, la contraseña hash se conserva, pero se invalida al colocar un !delante de ella.

La cuenta se puede desbloquear nuevamente escribiendo:

``` 
sudo passwd -u sammy
``` 

Las operaciones equivalentes están disponibles usando el usermod comando. Las banderas correspondientes son -L para bloquear y -U desbloquear:

``` 
sudo usermod -L sammy
sudo usermod -U sammy
``` 

**Nota:** Si bien este método de restringir el acceso funcionará correctamente para todos los inicios de sesión basados ​​en contraseña, los métodos para iniciar sesión sin contraseña (por ejemplo, con claves ssh) seguirán estando disponibles.


## 3. Cómo restringir el acceso con /etc/nologin

Puede haber algunas situaciones en las que necesite deshabilitar todos los inicios de sesión de cuentas, excepto la raíz.

Esto podría deberse a un mantenimiento exhaustivo o a que una o más de sus cuentas de usuario se han visto comprometidas.

En cualquier caso, esto se puede lograr creando un archivo en /etc/nologin:

``` 
sudo touch /etc/nologin
``` 

Esto evitará que cualquier cuenta que no tenga privilegios de superusuario pueda iniciar sesión.

Con un /etc/nologin archivo vacío, esto simplemente devuelve al usuario a su shell local sin ninguna explicación.

Lo que realmente sucede es que el contenido del archivo se devuelve al usuario. Si agrega un mensaje, los usuarios recibirán una explicación del error de inicio de sesión:

``` 
sudo sh -c 'echo "Planned maintenance. Log in capabilities will be restored at 1545 UTC" > /etc/nologin'
``` 

Ahora, cuando intente iniciar sesión con una contraseña, recibirá este mensaje:

``` 
ssh sammy@host
Output
sammy@host's password:
Planned maintenance. Log in capabilities will be restored at 1545 UTC

Connection closed by host
``` 

Los usuarios root aún pueden iniciar sesión normalmente. Elimine el archivo “/etc/nologin” para revertir la restricción de inicio de sesión:

``` 
sudo rm /etc/nologin
``` 


**Conclusión**

La autenticación de usuarios en Linux es un área relativamente flexible de la gestión del sistema. Hay muchas maneras de lograr el mismo objetivo con herramientas ampliamente disponibles.

Ahora debería saber cómo restringir el uso mediante varios métodos.

