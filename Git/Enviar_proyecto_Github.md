# Cómo enviar un proyecto existente a GitHub

## 1. Requisitos previos

Para inicializar el repositorio y enviarlo a GitHub necesitarás:

* Una cuenta de GitHub gratuita
* git instalado en su máquina local


## 2. Crea un nuevo repositorio de GitHub

Inicie sesión en GitHub y cree un nuevo repositorio vacío. Puede optar por inicializar un archivo README o no. Realmente no importa porque de todos modos vamos a anular todo lo que hay en este repositorio remoto.

### 2.1 Inicializa Git en la carpeta del proyecto

Desde su terminal, ejecute los siguientes comandos después de navegar a la carpeta que desea agregar.

Asegúrate de estar en el directorio raíz del proyecto que deseas enviar a GitHub y ejecutar:

**Nota:** Si ya tiene un repositorio Git inicializado, puede omitir este comando.

``` 
git init
``` 

Este paso crea un directorio oculto .git en la carpeta de su proyecto, que el git software reconoce y utiliza para almacenar todos los metadatos y el historial de versiones del proyecto.

Agregue los archivos al índice de Git

``` 
git add -A
``` 

El git add comando se usa para decirle a git qué archivos incluir en una confirmación, y el argumento -A (o --all) significa "incluir todo".

Confirmar archivos agregados

``` 
git commit -m 'Added my project'
``` 

El git commit comando crea una nueva confirmación con todos los archivos que se han "agregado". ( -mo --message) establece el mensaje que se incluirá junto con la confirmación, utilizado como referencia futura para comprender la confirmación. En este caso, el mensaje es: 'Added my project'.

Agregar un nuevo origen remoto

``` 
git remote add origin git@github.com:sammy/my-new-project.git
``` 

**Nota:** Recuerde, deberá reemplazar las partes resaltadas del nombre de usuario y del nombre del repositorio con su propio nombre de usuario y nombre del repositorio.

En Git, un "remoto" se refiere a una versión remota del mismo repositorio, que generalmente se encuentra en algún servidor (en este caso, GitHub) . “origen” es el nombre predeterminado que git le da a un servidor remoto (puede tener varios controles remotos), por lo que git remote add originle indica a git que agregue la URL del servidor remoto predeterminado para este repositorio.

### 2.2 Empujar a GitHub

``` 
git push -u -f origin main
``` 

El indicador -u (o --set-upstream) establece el control remoto origincomo referencia ascendente . Esto le permite ejecutar comandos posteriormente git pushsin git pull tener que especificar un origin ya que en este caso siempre queremos GitHub.

La bandera -f(o --force) significa fuerza. Esto sobrescribirá automáticamente todo lo que esté en el directorio remoto. Lo estamos usando aquí para sobrescribir el archivo README predeterminado que GitHub inicializó automáticamente.

**Nota:** Si no incluyó el archivo README predeterminado al crear el proyecto en GitHub, la -f marca no es realmente necesaria.

Todos juntos

``` 
git init
git add -A
git commit -m 'Added my project'
git remote add origin git@github.com:sammy/my-new-project.git
git push -u -f origin main
``` 


## 3. Conclusión

¡Ahora ya está todo listo para realizar un seguimiento de los cambios de código de forma remota en GitHub! 

