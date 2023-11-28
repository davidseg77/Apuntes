# Conceptos básicos de gestión de paquetes: apt, yum, dnf, pkg

## 1. Sistemas de gestión de paquetes: una breve descripción general

La mayoría de los sistemas de paquetes se basan en colecciones de archivos de paquetes. Un archivo de paquete suele ser un archivo que contiene aplicaciones compiladas y otros recursos utilizados por el software, junto con scripts de instalación. Los paquetes también contienen metadatos valiosos, incluidas sus dependencias, una lista de otros paquetes necesarios para instalarlos y ejecutarlos.

Si bien su funcionalidad y beneficios son muy similares, los formatos y herramientas de empaque varían según la plataforma:

* Para Debian/Ubuntu : .deb paquetes instalados por apt y dpkg
* Para Rocky / Fedora / RHEL : .rpm paquetes instalados por yum
* Para FreeBSD : .txz paquetes instalados por pkg

En Debian y sistemas basados ​​en él, como Ubuntu, Linux Mint y Raspbian, el formato del paquete es el .deb archivo. apt, la herramienta de empaquetado avanzado, proporciona comandos utilizados para las operaciones más comunes: buscar repositorios, instalar colecciones de paquetes y sus dependencias y administrar actualizaciones. aptLos comandos funcionan como una interfaz para la dpkgutilidad de nivel inferior, que maneja la instalación de .deb archivos individuales en el sistema local y, a veces, se invoca directamente.

Los lanzamientos recientes de la mayoría de las distribuciones derivadas de Debian incluyen un único apt comando, que ofrece una interfaz concisa y unificada para operaciones comunes que tradicionalmente han sido manejadas por métodos más específicos apt-get y apt-cache.

Rocky Linux, Fedora y otros miembros de la familia Red Hat usan archivos RPM. Estos solían utilizar un administrador de paquetes llamado yum. En versiones recientes de Fedora y sus derivados, yum ha sido reemplazado por dnf, una bifurcación modernizada que conserva la mayor parte de yum la interfaz.

El sistema de paquetes binarios de FreeBSD se administra con el pkg comando. FreeBSD también ofrece Ports Collection, una estructura de directorio local y herramientas que permiten al usuario buscar, compilar e instalar paquetes directamente desde el código fuente utilizando Make files. Generalmente es mucho más conveniente de usar pkg, pero ocasionalmente no está disponible un paquete precompilado o es posible que necesites cambiar las opciones de tiempo de compilación.


## 2. Actualizar listas de paquetes

La mayoría de los sistemas mantienen una base de datos local de los paquetes disponibles en repositorios remotos. Es mejor actualizar esta base de datos antes de instalar o actualizar paquetes. Como excepción parcial a este patrón, dnf buscará actualizaciones antes de realizar algunas operaciones, pero puede preguntar en cualquier momento si hay actualizaciones disponibles.

* Para Debian/Ubuntu :sudo apt update
* Para Rocky/Fedora/RHEL :dnf check-update
* Para paquetes FreeBSD :sudo pkg update
* Para puertos FreeBSD :sudo portsnap fetch update


## 3. Actualizar paquetes instalados

Asegurarse de que todo el software instalado en una máquina permanezca actualizado sería una tarea enorme sin un sistema de paquetes. Tendría que realizar un seguimiento de los cambios ascendentes y las alertas de seguridad de cientos de paquetes diferentes. Si bien un administrador de paquetes no resuelve todos los problemas que encontrará al actualizar el software, sí le permite mantener la mayoría de los componentes del sistema con unos pocos comandos.

En FreeBSD, la actualización de los puertos instalados puede introducir cambios importantes o requerir pasos de configuración manual. Es mejor leer /usr/ports/UPDATING antes de actualizar con port master.

* Para Debian/Ubuntu :sudo apt upgrade
* Para Rocky/Fedora/RHEL :sudo dnf upgrade
* Para paquetes FreeBSD :sudo pkg upgrade


## 4. Encuentra un paquete

La mayoría de las distribuciones ofrecen una interfaz gráfica o basada en menús para empaquetar colecciones. Éstas pueden ser una buena forma de buscar por categoría y descubrir nuevo software. Sin embargo, a menudo la forma más rápida y eficaz de localizar un paquete es buscar con herramientas de línea de comandos.

* Para Debian/Ubuntu :apt search search_string
* Para Rocky/Fedora/RHEL :dnf search search_string
* Para paquetes FreeBSD :pkg search search_string


## 5. Ver información sobre un paquete específico

Al decidir qué instalar, suele resultar útil leer descripciones detalladas de los paquetes. Junto con el texto legible por humanos, estos suelen incluir metadatos como números de versión y una lista de las dependencias del paquete.

* Para Debian/Ubuntu :apt show package
* Para Rocky/Fedora/RHEL :dnf info package
* Para paquetes FreeBSD :pkg info package
* Para puertos FreeBSD :cd /usr/ports/category/port && cat pkg-descr


## 6. Instalar un paquete desde repositorios

Una vez que conozca el nombre de un paquete, normalmente podrá instalarlo y sus dependencias con un solo comando. En general, puede suministrar varios paquetes para instalarlos a la vez enumerándolos todos.

* Para Debian/Ubuntu :sudo apt install package
* Para Rocky/Fedora/RHEL :sudo dnf install package
* Para paquetes FreeBSD :sudo pkg install package


## 7. Instalar un paquete desde el sistema de archivos local

A veces, aunque el software no esté empaquetado oficialmente para un sistema operativo determinado, un desarrollador o proveedor ofrecerá archivos de paquete para descargar. Por lo general, puede recuperarlos con su navegador web o mediante curl la línea de comando. Una vez que un paquete está en el sistema de destino, a menudo se puede instalar con un solo comando.

En sistemas derivados de Debian, dpkg maneja archivos de paquetes individuales. Si un paquete tiene dependencias no satisfechas, gdebi a menudo se puede utilizar para recuperarlas de los repositorios oficiales.

En Rocky Linux, Fedora o RHEL dnf se utiliza para instalar archivos individuales y también manejará las dependencias necesarias.

* Para Debian/Ubuntu :sudo dpkg -i package.deb
* Para Rocky/Fedora/RHEL :sudo dnf install package.rpm
* Para paquetes FreeBSD :sudo pkg add package.txz


## 8. Eliminar uno o más paquetes instalados

Dado que un administrador de paquetes sabe qué archivos proporciona un paquete determinado, generalmente puede eliminarlos limpiamente de un sistema si el software ya no es necesario.

* Para Debian/Ubuntu :sudo apt remove package
* Para Rocky/Fedora/RHEL :sudo dnf erase package
* Para paquetes FreeBSD :sudo pkg delete package


## 9. Consigue ayuda

Además de la documentación basada en web, tenga en cuenta que las páginas de manual de Unix (generalmente denominadas páginas de manual) están disponibles para la mayoría de los comandos del shell. Para leer una página, utilice man:

``` 
man page
``` 

En man, puedes navegar con las teclas de flecha. Presione / para buscar texto dentro de la página y q para salir.

* Para Debian/Ubuntu :man apt
* Para Rocky/Fedora/RHEL :man dnf
* Para paquetes FreeBSD :man pkg
* Para puertos FreeBSD :man ports

