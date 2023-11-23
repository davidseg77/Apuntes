# Cómo utilizar Git de forma eficaz

## 1. Introducción

Los sistemas de control de versiones como Git son esenciales para las mejores prácticas modernas de desarrollo de software. El control de versiones le permite realizar un seguimiento de su software en el nivel de origen. Puede realizar un seguimiento de los cambios, volver a etapas anteriores y realizar bifurcaciones para crear versiones alternativas de archivos y directorios.

Los archivos de muchos proyectos de software se mantienen en repositorios de Git y plataformas como GitHub, GitLab y Bitbucket ayudan a facilitar el intercambio y la colaboración de proyectos de desarrollo de software.

Después de instalar Git, necesitarás dedicar algo de tiempo a familiarizarte con los comandos principales para mantener un repositorio de proyectos. Este tutorial lo guiará por los primeros pasos para crear y enviar un repositorio Git en la línea de comando.

**Requisitos previos**

La herramienta de control de versiones Git disponible en tu entorno de desarrollo.


## 2. crear tu espacio de trabajo

Si está convirtiendo un proyecto existente en un repositorio Git, puede continuar con el paso 2. De lo contrario, puede comenzar creando un nuevo directorio de trabajo:

``` 
mkdir testing
``` 

A continuación, vaya a ese directorio de trabajo:

``` 
cd testing
``` 

Una vez dentro de ese directorio, necesitarás crear un archivo de muestra para demostrar la funcionalidad de Git. Puedes crear un archivo vacío con el touch comando:

``` 
touch file
``` 

Una vez que todos los archivos de su proyecto estén en su espacio de trabajo, deberá comenzar a rastrear sus archivos con git. El siguiente paso explica ese proceso.

## 3. Convertir un proyecto existente en un entorno de espacio de trabajo

Puede inicializar un repositorio Git en un directorio existente usando el git init comando.

``` 
git init
Output
Initialized empty Git repository in /home/sammy/testing/.git/
``` 

A continuación, deberá utilizar el git add comando para permitir que Git rastree sus archivos existentes. En su mayor parte, Git nunca rastreará archivos nuevos automáticamente, por lo que git add es un paso necesario al agregar contenido nuevo a un repositorio que Git no ha rastreado previamente.

``` 
git add .
``` 

Ahora tienes un repositorio Git con seguimiento activo. De ahora en adelante, cada uno de los pasos de este tutorial será coherente con un flujo de trabajo habitual para actualizar y comprometerse con un repositorio Git existente.


## 4. Crear un mensaje de confirmación

Cada vez que confirmes cambios en un repositorio de Git, deberás proporcionar un mensaje de confirmación. Los mensajes de confirmación resumen los cambios que ha realizado. Los mensajes de confirmación nunca pueden estar vacíos, pero pueden tener cualquier longitud; algunas personas prefieren usar mensajes de confirmación muy largos y descriptivos, aunque algunas plataformas como Github facilitan la lectura de mensajes de confirmación más cortos.

Si está importando un proyecto existente a Git por primera vez, lo habitual es utilizar un mensaje como "Compromiso inicial". Puedes crear una confirmación con el git commit comando:

``` 
git commit -m "Initial Commit" -a
Output
[master (root-commit) 1b830f8] initial commit
 0 files changed
 create mode 100644 file
``` 

Hay dos parámetros importantes del comando anterior. El primero es -m, lo que significa que seguirá su mensaje de confirmación (en este caso, “Commitción inicial”). En segundo lugar, -a significa que su confirmación debe incluir todos los archivos agregados o modificados. Git no trata esto como el comportamiento predeterminado, pero cuando trabaje con Git en el futuro, puede incluir de forma predeterminada todos los archivos actualizados en sus confirmaciones futuras la mayor parte del tiempo.

Para enviar un solo archivo o algunos archivos, podría haber usado:

``` 
git commit -m "Initial Commit" file1 file2
``` 

En el siguiente paso, enviará esta confirmación a un repositorio remoto.


## 5. Enviar cambios a un servidor remoto

Hasta este momento has trabajado exclusivamente en tu propio entorno. De hecho, aún puedes beneficiarte del uso de Git de esta manera, utilizando la funcionalidad avanzada de línea de comandos para rastrear y revertir tus propios cambios. Sin embargo, para poder utilizar sus populares funciones de colaboración en plataformas como Github, deberá enviar los cambios a un servidor remoto.

El primer paso para poder enviar código a un servidor remoto es proporcionar la URL donde se encuentra el repositorio y darle un nombre local. Para configurar un repositorio remoto y ver una lista de todos los remotos (puede tener más de uno), use el git remote comando:

```
git remote add origin ssh://git@git.domain.tld/repository.git 
git remote -v
Output
origin	ssh://git@git.domain.tld/repository.git (fetch)
origin	ssh://git@git.domain.tld/repository.git (push)
``` 

El primer comando agrega un control remoto, llamado "origen", y establece la URL en ssh://git @git .domain.tld/repository.git.

Puedes nombrar tu control remoto como quieras. origines una convención común sobre dónde residirá su copia autorizada y ascendente del código. La URL debe apuntar a un repositorio remoto real. Por ejemplo, si desea enviar código a GitHub, deberá utilizar la URL del repositorio que proporcionan.

Una vez que haya configurado un control remoto, podrá ingresar su código. Puede enviar código a un servidor remoto escribiendo lo siguiente:

``` 
git push origin main

Output
Counting objects: 4, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 266 bytes, done.
Total 3 (delta 1), reused 1 (delta 0)
To ssh://git@git.domain.tld/repository.git
   0e78fdf..e6a8ddc  main -> main
``` 

En el futuro, cuando tenga más confirmaciones para enviar, puede escribir de forma predeterminada git push, que heredará tanto el nombre de la rama como el nombre remoto de su último envío.


## 6. Conclusión

En este tutorial, creó y envió un repositorio Git inicial. Después de enviar y enviar su código a un repositorio como GitHub, puede optar por dedicar más tiempo a colaborar en la interfaz web, pero siempre será importante poder trabajar desde una máquina local en la línea de comandos. Mantener o contribuir a proyectos con múltiples confirmaciones implicará comandos Git más complejos, pero lo que has cubierto en este tutorial es suficiente para trabajar en proyectos personales.

