# ¿Cómo restaurar los repositorios predeterminados en Ubuntu?

¿Obtiene errores al actualizar los repositorios del sistema o instalar nuevo software? Es posible que obtenga errores porque su archivo /etc/apt/source.list está dañado y contiene los detalles de los repositorios. En este artículo veremos cómo restaurar los repositorios predeterminados en Ubuntu.

Hay cuatro repositorios estándar en Ubuntu:

* Principal
* Universo
* Restringido
* Multiverso

## 1. Restaurar repositorios predeterminados en Ubuntu

Primero, debemos hacer una copia de seguridad del archivo fuente dañado moviéndolo a otra ubicación. Abra una terminal presionando Ctrl+Alt+T e ingrese el siguiente comando para cambiar el directorio donde se encuentra el archivo fuente:

``` 
cd /etc/apt
``` 

Ahora, mueve el archivo dañado a otra ubicación:

``` 
sudo mv sources.list <location>
sudo mv sources.list /sid/home/Desktop
``` 

Cree un nuevo archivo usando el comando touch:

``` 
sudo touch /etc/apt/sources.list
``` 

Ahora, abra la aplicación **Software & Updates** usando la barra de búsqueda o el cajón de aplicaciones. Cambie el servidor al servidor principal (Main Server) y habilite el repositorio restringido (restricted). También puede habilitar repositorios de universos y multiversos si es necesario.

Para habilitar las actualizaciones, en la pestaña **Updates**, seleccione Todas las actualizaciones o al menos actualizaciones de seguridad (Security Updates Only) en el menú desplegable Suscrito a y haga clic en cerrar.

Haga clic en Recargar. Los repositorios de software se actualizarán.


## 2. Verificar si se han agregado repositorios

Para verificar, abra una terminal presionando Ctrl+Alt+T. Abra el archivo /etc/apt/sources.list ejecutando el siguiente comando:

``` 
sudo vi /etc/apt/sources.list
``` 

Si hay entradas sin el #, se agregaron los repositorios.

Finalmente, actualice los repositorios ejecutando el siguiente comando:

``` 
sudo apt update
``` 

Y listo.

