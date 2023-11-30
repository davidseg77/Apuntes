# El comando dpkg en Linux

## 1. Usando el comando dpkg

Exploremos los usos comunes del comando dpkg. Como el comando funciona igual tanto para los sistemas Debian como para Ubuntu, de ahora en adelante solo mencionaremos Ubuntu.

### 1.1 Instalar un paquete

El uso más básico del comando dpkg en Ubuntu es la instalación de un paquete. Podemos instalar un paquete deb en Ubuntu o Debian usando la dpkg -i opción comando.

Así es como instalarías un paquete.

``` 
sudo dpkg -i [package name]
``` 

También puede instalar varios paquetes al mismo tiempo especificando los nombres de los paquetes separados por espacios.

### 1.2 Quitar un paquete

Cuando ya no necesita un programa o servicio en su sistema, no sirve de nada conservarlo.

El comando dpkg también nos cubre aquí.

Podemos desinstalar un programa o servicio de nuestro sistema usando la dpkg -r opción.

Eliminemos el reproductor VLC que instalamos para esta demostración.

``` 
sudo dpkg -r [package name]
``` 

### 1.3 Actualizando tus repositorios

El repositorio dpkg almacena todos los paquetes disponibles para su instalación en su distribución Ubuntu o Debian Linux.

Sin embargo, como estos paquetes se almacenan localmente, a menudo puedes terminar teniendo versiones antiguas de paquetes para un programa cuando ya se han lanzado versiones más nuevas. Esto provoca la necesidad de un método para actualizar sus repositorios.

¿Adivina qué? La dpkg --update-avail opción lo tiene cubierto.

Comprueba los repositorios en línea y descarga todos los paquetes actualizados a su repositorio local.

Actualicemos nuestros repositorios locales a la última versión:

``` 
sudo dpkg --update-avail
``` 

