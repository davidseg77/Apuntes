# Cómo utilizar Git Hooks para automatizar las tareas de desarrollo e implementación

**Requisitos previos**

Antes de comenzar, debe tenerlo git instalado en su servidor.

## 1. Configurar un repositorio

Para comenzar, creará un repositorio nuevo y vacío en su directorio de inicio. Puedes llamar a esto proj.

``` 
mkdir ~/proj
cd ~/proj
git init
``` 

Ahora estás en el directorio de trabajo vacío de un directorio controlado por git. Antes de hacer cualquier otra cosa, acceda al repositorio que está almacenado en el archivo oculto llamado .git dentro de este directorio:

```
cd .git
ls -F
``` 

Verá una serie de archivos y directorios. El que te interesa es el hooks directorio:

``` 
cd hooks
ls -l
``` 

Puedes ver algunas cosas aquí. Primero, puede ver que cada uno de estos archivos está marcado como ejecutable. Dado que estos scripts solo se llaman por su nombre, deben ser ejecutables y su primera línea debe ser una referencia de número mágico para llamar al intérprete de script correcto. Por lo general, se trata de lenguajes de secuencias de comandos como bash, perl, python, etc.

Lo segundo que puede notar es que todos los archivos terminan en .sample. Esto se debe a que git simplemente mira el nombre del archivo cuando intenta encontrar los archivos de enlace para ejecutar. Desviarse del nombre del script que git está buscando básicamente deshabilita el script. Para habilitar cualquiera de los scripts en este directorio, deberá eliminar el .sample sufijo.

## 2. Primer ejemplo: implementación en un servidor web local con un enlace posterior a la confirmación

Su primer ejemplo utilizará el post-commit enlace para mostrarle cómo implementar en un servidor web local cada vez que se realiza una confirmación. Este no es el gancho que usaría para un entorno de producción, pero nos permite demostrar algunos elementos importantes, apenas documentados, que debe conocer al usar ganchos.

Primero, instalará el servidor web Apache para demostrar:

``` 
sudo apt-get update
sudo apt-get install apache2
``` 

Para que su secuencia de comandos modifique la raíz web en /var/www/html (esta es la raíz del documento en Ubuntu 20.04. Modifique según sea necesario), debe tener permiso de escritura. Primero, otorgue a su usuario normal la propiedad de este directorio. Puedes hacer esto escribiendo:

``` 
sudo chown -R `whoami`:`id -gn` /var/www/html
``` 

Ahora, en el directorio de su proyecto, cree un index.html archivo:

``` 
cd ~/proj
nano index.html
``` 

En el interior, puedes agregar un poco de HTML solo para demostrar la idea. No tiene por qué ser complicado:

``` 
<h1>Here is a title!</h1>

<p>Please deploy me!</p>
``` 

Agregue el nuevo archivo para indicarle a git que rastree el archivo:

``` 
git add .
``` 

Ahora, antes de comprometerse, configurará su post-commit enlace para el repositorio. Cree este archivo dentro del .git/hooks directorio del proyecto:

```
nano .git/hooks/post-commit
``` 

Dado que los ganchos de git son scripts estándar, debes decirle a git que use bash comenzando con un shebang:

``` 
#!/bin/bash
unset GIT_INDEX_FILE
git --work-tree=/var/www/html --git-dir=$HOME/proj/.git checkout -f
``` 

En la siguiente línea, debe observar de cerca las variables ambientales que se establecen cada vez que post-commit se llama al gancho. En particular, GIT_INDEX_FILE está configurado en .git/index.

Esta ruta está en relación con el directorio de trabajo, que en este caso es /var/www/html. Dado que el índice de git no existe en esta ubicación, el script fallará si lo deja como está. Para evitar esta situación, puede desarmar manualmente la variable.

Después de eso, simplemente usará git para descomprimir la versión más reciente del repositorio después de la confirmación, en su directorio web. Querrá forzar esta transacción para asegurarse de que se realice correctamente cada vez.

Cuando haya terminado con estos cambios, guarde y cierre el archivo.

Como se trata de un archivo de script normal, es necesario hacerlo ejecutable:

``` 
chmod +x .git/hooks/post-commit
``` 

Ahora, finalmente está listo para confirmar los cambios que realizó en su repositorio de git. Asegúrese de volver al directorio correcto y luego confirme los cambios:

``` 
cd ~/proj
git commit -m "here we go..."
``` 

Ahora, si visita el nombre de dominio o la dirección IP de su servidor en su navegador, debería ver el index.html archivo que creó:

``` 
http://server_domain_or_IP
``` 

Como puede ver, sus cambios más recientes se enviaron automáticamente a la raíz de documentos de su servidor web al confirmarse. Puede realizar algunos cambios adicionales para mostrar que funciona en cada confirmación:

``` 
echo "<p>Here is a change.</p>" >> index.html
git add .
git commit -m "First change"
``` 

Cuando actualice su navegador, debería ver inmediatamente los nuevos cambios que aplicó.

Como puede ver, este tipo de configuración puede facilitar las pruebas de cambios localmente. Sin embargo, casi nunca querrás publicar una confirmación en un entorno de producción. Es mucho más seguro enviarlo después de haber probado el código y estar seguro de que está listo.

## 3. Uso de Git Hooks para implementar en un servidor de producción independiente

En el siguiente ejemplo, demostrará una mejor manera de actualizar un servidor de producción. Puede hacer esto utilizando el modelo push-to-deploy para actualizar su servidor web cada vez que ingresa a un repositorio git simple. Puede utilizar el mismo servidor que ha configurado como su máquina de desarrollo.

En su máquina de producción, configurará otro servidor web, un repositorio de git básico al que enviará los cambios y un gancho de git que se ejecutará cada vez que se reciba un envío.

Complete los pasos a continuación como usuario normal con privilegios sudo.


### 3.1 Configurar el enlace posterior a la recepción del servidor de producción

En el servidor de producción, comience instalando el servidor web:

``` 
sudo apt-get update
sudo apt-get install apache2
``` 

Nuevamente, debes otorgar la propiedad de la raíz del documento al usuario con el que estás operando:

``` 
sudo chown -R `whoami`:`id -gn` /var/www/html
``` 

También necesitas instalar git en esta máquina:

``` 
sudo apt-get install git
``` 

Ahora, puede crear un directorio dentro del directorio de inicio de su usuario para contener el repositorio. Luego puede ingresar a ese directorio e inicializar un repositorio básico. Un repositorio básico no tiene un directorio de trabajo y es mejor para servidores con los que no trabajará mucho directamente:

```
mkdir ~/proj
cd ~/proj
git init --bare
``` 

Dado que se trata de un repositorio básico, no hay un directorio de trabajo y todos los archivos que normalmente se encuentran .git ahora se encuentran en el directorio principal.

A continuación, necesitas crear otro gancho de git. Esta vez, lo que le interesa es el post-receive gancho, que se ejecuta en el servidor que recibe un archivo git push. Abra este archivo en su editor:

```
nano hooks/post-receive
``` 

Nuevamente, debes comenzar identificando el tipo de guión que estás escribiendo. Después de eso, puede escribir el mismo comando de pago que usó en su post-commit archivo, modificado para usar las rutas de esta máquina:

``` 
#!/bin/bash
while read oldrev newrev ref
do
if [[ $ref =~ .*/master$ ]];
then
echo "Master ref received.  Deploying master branch to production..."
git --work-tree=/var/www/html --git-dir=$HOME/proj checkout -f
else
echo "Ref $ref successfully received.  Doing nothing: only the master branch may be deployed on this server."
fi
done
``` 

Dado que se trata de un repositorio básico, --git-dir debería apuntar al directorio de nivel superior de ese repositorio.

Sin embargo, es necesario agregar algo de lógica adicional a este script. Si accidentalmente envía una test-feature rama a este servidor, no querrá que se implemente. Desea asegurarse de que solo implementará la master sucursal.

Primero, necesitas leer la entrada estándar. Para cada referencia que se envíe, las tres piezas de información (revisión anterior, nueva revisión, referencia) se enviarán al script, separadas por espacios en blanco, como entrada estándar. Puede leer esto con un while bucle para rodear el git comando.

Entonces, ahora tendrá tres variables configuradas en función de lo que se está impulsando. Para una inserción de rama maestra, el ref objeto contendrá algo similar a refs/heads/master. Puede verificar si la referencia que recibe el servidor tiene este formato utilizando una if construcción.

Finalmente, agregue algo de texto que describa qué situación se detectó y qué acción se tomó. Debe agregar un else bloque para notificar al usuario cuando se recibió correctamente una rama no maestra, aunque la acción no desencadenará una implementación.

Cuando haya terminado, guarde y cierre el archivo. Pero recuerda, debes hacer que el script sea ejecutable para que el gancho funcione:

```
chmod +x hooks/post-receive
``` 

Ahora puede configurar el acceso a este servidor remoto en su cliente.


### 3.2 Configure el servidor remoto en su máquina cliente

De vuelta en su máquina cliente (de desarrollo), regrese al directorio de trabajo de su proyecto:

``` 
cd ~/proj
``` 

Dentro, agregue el servidor remoto como un control remoto llamado production. El comando que escriba debería verse así:

``` 
git remote add production sammy@remote_server_domain_or_IP:proj
``` 

Ahora envíe su rama maestra actual a su servidor de producción:

``` 
git push production master
``` 

Si no tiene claves SSH configuradas, es posible que deba ingresar la contraseña de su usuario del servidor de producción. Deberías ver algo parecido a esto:

``` 
Output
Counting objects: 8, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 473 bytes | 0 bytes/s, done.
Total 4 (delta 0), reused 0 (delta 0)
remote: Master ref received.  Deploying master branch...
To sammy@107.170.14.32:proj
   009183f..f1b9027  master -> master
``` 

Como puede ver, el texto de su post-receive enlace está en la salida del comando. Si visita el nombre de dominio o la dirección IP de su servidor de producción en su navegador web, debería ver la versión actual de su proyecto.

Parece que el gancho ha llevado exitosamente su código a producción una vez que recibió la información.

Ahora es el momento de probar algún código nuevo. De vuelta en la máquina de desarrollo, creará una nueva rama para guardar sus cambios. Haga una nueva sucursal llamada test_feature y verifique la nueva sucursal escribiendo:

```
git checkout -b test_feature
``` 

Ahora estás trabajando en la test_feature sucursal. Intente realizar un cambio que quizás desee trasladar a producción. Lo enviarás a esta rama:

``` 
echo "<h2>New Feature Here</h2>" >> index.html
git add .
git commit -m "Trying out new feature"
```

En este punto, si va a la dirección IP o al nombre de dominio de su máquina de desarrollo, debería ver los cambios mostrados.

Esto se debe a que su máquina de desarrollo todavía se está volviendo a implementar en cada confirmación. Este flujo de trabajo es excelente para probar los cambios antes de pasarlos a producción.

Puede enviar su test_feature sucursal a su servidor de producción remoto:

``` 
git push production test_feature
``` 

Deberías ver el otro mensaje de tu post-receive gancho en el resultado:

``` 
Output
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 301 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Ref refs/heads/test_feature successfully received.  Doing nothing: only the master branch may be deployed on this server
To sammy@107.170.14.32:proj
   83e9dc4..5617b50  test_feature -> test_feature
```

Si vuelve a comprobar el servidor de producción en su navegador, debería ver que nada ha cambiado. Esto es lo que esperaba, ya que el cambio que impulsó no estaba en la rama maestra.

Ahora que ha probado los cambios en su máquina de desarrollo, está seguro de que desea incorporar esta característica en su rama maestra. Puede verificar su master rama y fusionarse en su test_feature rama en su máquina de desarrollo:

``` 
git checkout master
git merge test_feature
``` 

Ahora, ha fusionado la nueva función en la rama maestra. Al enviar al servidor de producción se implementarán los cambios:

``` 
git push production master
``` 

Si verifica el nombre de dominio o la dirección IP de su servidor de producción, verá los cambios.

Con este flujo de trabajo, puede tener una máquina de desarrollo que mostrará inmediatamente cualquier cambio confirmado. La máquina de producción se actualizará cada vez que presione la rama maestra.

## 4. Conclusión

Si ha seguido hasta aquí, debería poder ver las diferentes formas en que git hooks puede ayudar a automatizar algunas de sus tareas. Pueden ayudarle a implementar su código o ayudarle a mantener los estándares de calidad rechazando cambios no conformes o mensajes de confirmación.

Si bien la utilidad de los git hooks es difícil de discutir, la implementación real puede ser bastante difícil de comprender y frustrante de solucionar. Practicar la implementación de varias configuraciones, experimentar con el análisis de argumentos y entradas estándar y realizar un seguimiento de cómo git construye el entorno de los ganchos será de gran ayuda para enseñarle cómo escribir ganchos efectivos. A largo plazo, la inversión de tiempo suele valer la pena, ya que puede ahorrarle a usted y a su equipo mucho trabajo manual a lo largo de la vida de su proyecto.



