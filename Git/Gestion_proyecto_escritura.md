# Cómo utilizar Git para gestionar su proyecto de escritura


## 1. Introducción

El control de versiones no es sólo para el código. Es para cualquier cosa que quieras rastrear, incluido el contenido. Usar Git para administrar su próximo proyecto de escritura le brinda la posibilidad de ver varios borradores al mismo tiempo, ver las diferencias entre esos borradores e incluso volver a una versión anterior. Y si se siente cómodo haciéndolo, puede compartir su trabajo con otras personas en GitHub u otros repositorios centrales de Git.

En este tutorial usarás Git para administrar un pequeño documento de Markdown. Almacenará una versión inicial, la confirmará, realizará cambios, verá la diferencia entre esos cambios y revisará la versión anterior. Cuando haya terminado, tendrá un flujo de trabajo que podrá aplicar a sus propios proyectos de escritura.


## 2. crear un espacio de trabajo para su proyecto de escritura

Para administrar sus cambios, creará un repositorio Git local. Un repositorio Git se encuentra dentro de un directorio existente, así que comience creando un nuevo directorio para su artículo:

``` 
mkdir article
``` 

Cambie al nuevo article directorio:

``` 
cd article
``` 

El git init comando crea un nuevo repositorio Git vacío en el directorio actual. Ejecute ese comando ahora:

``` 
git init
``` 

Verá el siguiente resultado que confirma que se creó su repositorio:

``` 
Output
Initialized empty Git repository in /Users/sammy/article/.git/
``` 

El .gitignore archivo le permite decirle a Git que algunos archivos deben ignorarse. Puede usar esto para ignorar archivos temporales que su editor de texto pueda crear o archivos del sistema operativo. En macOS, por ejemplo, la aplicación Finder crea .DS_Store archivos en directorios. Crea un .gitignore archivo que los ignore:

``` 
nano .gitignore
``` 

Agregue las siguientes líneas al archivo:

``` 
# Ignore Finder files
.DS_store
``` 

La primera línea es un comentario, que le ayudará a identificar lo que ignorará en el futuro. La segunda línea especifica el archivo a ignorar.

Guarde el archivo y salga del editor.

A medida que descubra más archivos que desee ignorar, abra el .gitignore archivo y agregue una nueva línea para cada archivo o directorio que desee ignorar.

Ahora que su repositorio está configurado, puede comenzar a trabajar.


## 3. Guardar su borrador inicial

Git solo conoce los archivos de los que le cuentas. El hecho de que exista un archivo en el directorio que contiene el repositorio no significa que Git rastreará sus cambios. Tienes que agregar un archivo al repositorio y luego confirmar los cambios.

Cree un nuevo archivo Markdown llamado article.md:

``` 
nano article.md
``` 

Agregue algo de texto al archivo:

``` 
# How To Use Git to Manage Your Writing Project

### Introduction

Version control isn't just for code. It's for anything you want to track, including content. Using Git to manage your next writing project gives you the ability to view multiple drafts at the same time,  see differences between those drafts, and even roll back to a previous version. And if you're comfortable doing so, you can then share your work with others on GitHub or other central git repositories.

In this tutorial you'll use Git to manage a small Markdown document. You'll store an initial version, commit it, make changes, view the difference between those changes, and review the previous version. When you're done, you'll have a workflow you can apply to your own writing projects.
```

Guarde los cambios y salga del editor.

El git status comando le mostrará el estado de su repositorio. Le mostrará qué archivos deben agregarse para que Git pueda rastrearlos. Ejecute este comando:

``` 
git status
``` 

Verá este resultado:

``` 
Output
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	article.md

nothing added to commit but untracked files present (use "git add" to track)
``` 

En el resultado, la Untracked file ssección muestra los archivos que Git no está mirando. Estos archivos deben agregarse al repositorio para que Git pueda observar los cambios. Utilice el git add comando para hacer esto:

``` 
git add .gitignore
git add article.md
``` 

Ahora ejecute git status para verificar que esos archivos hayan sido agregados:

``` 
Output
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   .gitignore
	new file:   article.md
``` 

Ambos archivos ahora aparecen listados en la Changes to be committed sección. Git los conoce, pero aún no ha creado una instantánea del trabajo. Utilice el git commit comando para hacer eso.

Cuando crea una nueva confirmación, debe proporcionar un mensaje de confirmación. Un buen mensaje de confirmación indica cuáles son sus cambios. Cuando trabaja con otras personas, cuanto más detallados sean sus mensajes de confirmación, mejor.

Utilice el comando git commit para confirmar sus cambios:

``` 
git commit -m "Add gitignore file and initial version of article"
``` 

El resultado del comando muestra que los archivos fueron confirmados:

``` 
Output
[master (root-commit) 95fed84] Add gitignore file and initial version of article
 2 files changed, 9 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 article.md
``` 

Utilice el git status comando para ver el estado del repositorio:

``` 
git status
``` 

El resultado muestra que no hay cambios que deban agregarse o confirmarse.

``` 
Output
On branch master
nothing to commit, working tree clean
``` 

Ahora veamos cómo trabajar con los cambios.


## 4. Guardar revisiones

Ha agregado su versión inicial del artículo. Ahora agregará más texto para que pueda ver cómo administrar los cambios con Git.

Abra el artículo en su editor:

``` 
nano article.md
``` 

Agregue más texto al final del archivo:

``` 
## Prerequisites

* Git installed on your local computer. The tutorial [How to Contribute to Open Source: Getting Started with Git](https://www.digitalocean.com/community/tutorials/how-to-contribute-to-open-source-getting-started-with-git) walks you through installing Git and covers some background information you may find useful.
```

Guarda el archivo.

Utilice el git status comando para ver dónde están las cosas en su repositorio:

``` 
git status
``` 

El resultado muestra que hay cambios:

``` 
Output
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   article.md

no changes added to commit (use "git add" and/or "git commit -a")
``` 

Como era de esperar, el article.md archivo tiene cambios.

Úsalo git diff para ver cuáles son:

``` 
git diff article.md
``` 

El resultado muestra las líneas que ha agregado:

``` 
diff --git a/article.md b/article.md
index 77b081c..ef6c301 100644
--- a/article.md
+++ b/article.md
@@ -5,3 +5,7 @@
 Version control isn't just for code. It's for anything you want to track, including content. Using Git to manage your next writing project gives you the ability to view multiple drafts at the same time,  see differences between those drafts, and even roll back to a previous version. And if you're comfortable doing so, you can then share your work with others on GitHub or other central git repositories.
 
 In this tutorial you'll use Git to manage a small Markdown document. You'll store an initial version, commit it, make changes, view the difference between those changes, and review the previous version. When you're done, you'll have a workflow you can apply to your own writing projects.
+
+## Prerequisites
+
+* Git installed on your local computer. The tutorial [How to Contribute to Open Source: Getting Started with Git](https://www.digitalocean.com/community/tutorials/how-to-contribute-to-open-source-getting-started-with-git) walks you through installing Git and covers some background information you may find useful.
```

En el resultado, las líneas que comienzan con un signo más (+) son líneas que usted agregó. Las líneas eliminadas aparecerán con un signo menos (-). Las líneas que no se modificaron no tendrían ninguno de estos caracteres al frente.

Usar git diff y git status es una forma útil de ver lo que has cambiado. También puede guardar la diferencia en un archivo para poder verlo más tarde con el siguiente comando:

``` 
git diff article.md > article_diff.diff
``` 

El uso de la .diff extensión ayudará a su editor de texto a aplicar el resaltado de sintaxis adecuado.

Guardar los cambios en su repositorio es un proceso de dos pasos. Primero, agregue el article.md archivo nuevamente y luego confirme. Git quiere que le digas explícitamente qué archivos van en cada confirmación, por lo que aunque agregaste el archivo antes, debes agregarlo nuevamente. Tenga en cuenta que el resultado del git status comando se lo recuerda.

Agregue el archivo y luego confirme los cambios, proporcionando un mensaje de confirmación:

``` 
git add article.md
git commit -m "add prerequisites section"
```

El resultado verifica que la confirmación funcionó:

``` 
Output
[master 1fbfc21] add prerequisites section
 1 file changed, 4 insertions(+)
``` 

Úselo git status para ver el estado de su repositorio. Verás que no hay nada más que hacer.

``` 
git status
Output
On branch master
nothing to commit, working tree clean
```

Continúe este proceso mientras revisa su artículo. Realice cambios, verifíquelos, agregue el archivo y confirme los cambios con un mensaje detallado. Realice sus cambios tan a menudo o tan poco como se sienta cómodo. Puede realizar una confirmación después de terminar cada borrador o justo antes de realizar una revisión importante de la estructura de su artículo.

Si envía un borrador de un documento a otra persona y esta le realiza cambios, tome su copia y reemplace su archivo con el de ellos. Luego use git diff para ver los cambios que hicieron rápidamente. Git verá los cambios ya sea que los haya escrito directamente o haya reemplazado el archivo con uno que descargó de la web, del correo electrónico o de otro lugar.

Ahora veamos cómo administrar las versiones de su artículo.


## 5. Gestionar los cambios

A veces resulta útil consultar una versión anterior de un documento. Siempre que ha utilizado git commit, ha proporcionado un mensaje útil que resume lo que ha hecho.

El git log comando le muestra el historial de confirmaciones de su repositorio. Cada cambio que ha realizado tiene una entrada en el registro.

``` 
git log
Output
commit 1fbfc2173f3cec0741e0a6b21803fbd0be511bc4
Author: Sammy Shark <sammy@digitalocean>
Date:   Thu Sep 19 16:35:41 2019 -0500

    add prerequisites section

commit 95fed849b0205c49eda994fff91ec03642d59c79
Author: Sammy Shark <sammy@digitalocean>
Date:   Thu Sep 19 16:32:34 2019 -0500

    Add gitignore file and initial version of article
``` 

Cada confirmación tiene un identificador específico. Utilice este número para hacer referencia a los cambios de una confirmación específica. Sin embargo, sólo necesitas los primeros caracteres del identificador. El git log --oneline comando le brinda una versión condensada del registro con identificadores más cortos:

``` 
git log --oneline
Output
1fbfc21 add prerequisites section
95fed84 Add gitignore file and initial version of article
``` 

Para ver la versión inicial de su archivo, use git show y el identificador de confirmación. Los identificadores en su repositorio serán diferentes a los de estos ejemplos.

``` 
git show 95fed84 article.md
``` 

El resultado muestra los detalles de la confirmación, así como los cambios que ocurrieron durante esa confirmación:

``` 
Output
commit 95fed849b0205c49eda994fff91ec03642d59c79
Author: Sammy Shark <sammy@digitalocean>
Date:   Thu Sep 19 16:32:34 2019 -0500

    Add gitignore file and initial version of article

diff --git a/article.md b/article.md
new file mode 100644
index 0000000..77b081c
--- /dev/null
+++ b/article.md
@@ -0,0 +1,7 @@
+# How To Use Git to Manage Your Writing Project
+
+### Introduction
+
+Version control isn't just for code. It's for anything you want to track, including content. Using Git to manage your next writing project gives you the ability to view multiple drafts at the same time,  see differences between those drafts, and even roll back to a previous version. And if you're comfortable doing so, you can then share your work with others on GitHub or other central git repositories.
+
+In this tutorial you'll use Git to manage a small Markdown document. You'll store an initial version, commit it, make changes, view the difference between those changes, and review the previous version. When you're done, you'll have a workflow you can apply to your own writing projects.
``` 

Para ver el archivo en sí, modifique ligeramente el comando. En lugar de un espacio entre el identificador de confirmación y el archivo, reemplácelo por :./algo así:

``` 
git show 95fed84:./article.md
``` 

Verá el contenido de ese archivo, en esa revisión:

``` 
Output
# How To Use Git to Manage Your Writing Project

### Introduction

Version control isn't just for code. It's for anything you want to track, including content. Using Git to manage your next writing project gives you the ability to view multiple drafts at the same time,  see differences between those drafts, and even roll back to a previous version. And if you're comfortable doing so, you can then share your work with others on GitHub or other central git repositories.

In this tutorial you'll use Git to manage a small Markdown document. You'll store an initial version, commit it, make changes, view the difference between those changes, and review the previous version. When you're done, you'll have a workflow you can apply to your own writing projects.
``` 

Puede guardar esa salida en un archivo si la necesita para otra cosa:

``` 
git show 95fed84:./article.md > old_article.md
``` 

A medida que realice más cambios, su registro crecerá y podrá revisar todos los cambios que haya realizado en su artículo a lo largo del tiempo.


**Conclusión**

En este tutorial, utilizó un repositorio Git local para realizar un seguimiento de los cambios en su proyecto de escritura. Puede utilizar este enfoque para gestionar artículos individuales, todas las publicaciones de su blog o incluso su próxima novela. Y si envías tu repositorio a GitHub, puedes invitar a otras personas para que te ayuden a editar tu trabajo.

