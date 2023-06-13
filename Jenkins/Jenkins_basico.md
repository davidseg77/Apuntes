# Apuntes sobre el curso básico de Jenkins

### Instalación de Jenkins en Ubuntu con Docker

**Comando para tirar de Jenkins en Docker Hub**

```
docker pull jenkins/jenkins:lts
```

**Comando para crear el contenedor Jenkins**

```
docker run -d -v $(pwd)/jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 -p 9080:9080 -e JENKINS_OPTS="--httpPort=9080" --name jenkins jenkins/jenkins:lts
```

Desarrollo del comando:

docker run: Lanza el contenedor de Jenkins
-d: en modo desatendido
-v: monta el volumen de JENKINS_HOME en nuestro sistema de archivos (así el home será perdurable)
-p: redirige los puertos de acuerdo a la siguiente sintaxis
http://localhost:9080 —> El Jenkins en si. La primera vez será el asistente de configuración
http://localhost:50000 —> Tendremos información relativa a la instalación
http://localhost:8080 —> Lo emplearemos en el futuro, en prácticas
-e JENKINS_OPTS=”—httpPort=9080”: modificamos el puerto de Jenkins
—name: podremos darle un nombre concreto a nuestro contenedor
jenkins/jenkins:lts: escogeremos la versión “LTS”: Long-term support

En caso de error al iniciar el contenedor, habría que dar permisos al directorio creado de jenkins:

```
sudo chown -R 1000:1000 /home/david/jenkins_home
```

**Comando para ver los registros y dar con la contraseña admin para acceder a Jenkins**

```
docker logs -f jenkins
```

**Para acceder a las funciones CLI de Jenkins**

http://localhost:9080/cli/

### Comandos disponibles

**add-job-to-view**	Adds jobs to view.
  
**build**	Ejecuta una tarea, y opcionalmente espera hasta que acabe.
  
**cancel-quiet-down**	Cancelar el efecto del comando "quiet-down".

**clear-queue**	Limpiar la cola de trabajos

**connect-node**	Reconectarse con un nodo

**console**	Retrieves console output of a build.

**copy-job**	Copia una tarea

**create-credentials-by-xml**	Create Credential by XML

**create-credentials-domain-by-xml**	Create Credentials Domain by XML

**create-job**	Permite crear una nueva tarea leyendo un fichero XML por la entrada estándard.

**create-node**	Creates a new node by reading stdin as a XML configuration.

**create-view**	Creates a new view by reading stdin as a XML configuration.

**declarative-linter**	Validate a Jenkinsfile containing a Declarative Pipeline

**delete-builds**	Permite borrar registros

**delete-credentials**	Delete a Credential

**delete-credentials-domain**	Delete a Credentials Domain

**delete-job**	Borrar una tarea

**delete-node**	Borrar un nodo

**delete-view**	Deletes view(s).

**disable-job**	Desactivar una tarea

**disable-plugin**	Deshabilita uno o más plugins instalados.

**disconnect-node**	Desconectarse de un nodo

**enable-job**	Activar una tarea

**enable-plugin**	Enables one or more installed plugins transitively.

**get-credentials-as-xml**	Get a Credentials as XML (secrets redacted)

**get-credentials-domain-as-xml**	Get a Credentials Domain as XML

**get-gradle**	List available gradle installations

**get-job**	Volcar el fichero XML de la definición de tarea a la salida estándard

**get-node**	Dumps the node definition XML to stdout.

**get-view**	Dumps the view definition XML to stdout.

**groovy**	Ejecuta el script de groovy especificado.

**groovysh**	Ejecuta una shell interactiva de groovy.

**help**	Muestra la lista de todos los comandos disponibles.

**import-credentials-as-xml**	Import credentials as XML. The output of "list-credentials-as-xml" can be used as **input here as is, the only needed change is to set the actual Secrets which are redacted in the output.

**install-plugin**	Instala un plugin desde un fichero, una URL o desde el centro de actualizaciones.

**keep-build**	Marcar la ejecución para ser guardada para siempre.

**list-changes**	Vuelca el registro de cambios del trabajo actual.

**list-credentials**	Lists the Credentials in a specific Store

**list-credentials-as-xml**	Export credentials as XML. The output of this command can be used as input for 

**"import-credentials-as-xml"** as is, the only needed change is to set the actual Secrets which are redacted in the output.

**list-credentials-context-resolvers**	List Credentials Context Resolvers

**list-credentials-providers**	List Credentials Providers

**list-jobs**	Lists all jobs in a specific view or item group.

**list-plugins**	Outputs a list of installed plugins.

**mail**	Lee de la entrada estándard, y lo envía por email.

**offline-node**	Dejar de utilizar un nodo temporalmente hasta que se ejecute el comando "online-node".

**online-node**	Continuar usando un nodo y candelar el comando "offline-node" mas reciente.

**quiet-down**	Poner Jenkins en modo quieto y estar preparado para un reinicio. No comenzar ninguna ejecución.

**reload-configuration**	Descartar todos los datos presentes en memoria y recargar todo desde el disco duro. Útil cuando se han hecho modificaciones a mano en el disco duro.

**reload-job**	Reload job(s)

**remove-job-from-view**	Removes jobs from view.

**replay-pipeline**	Replay a Pipeline build with edited script taken from standard input

**restart**	Reiniciar Jenkins

**restart-from-stage**	Restart a completed Declarative Pipeline build from a given stage.

**safe-restart**	Reiniciar Jenkins de manera segura

**safe-shutdown**	Poner jenkins en estado de espera hasta que todos los trabajos terminen, después se apagará.

**session-id**	Outputs the session ID, which changes every time Jenkins restarts.

**set-build-description**	Establece la descripción de una ejecución.

**set-build-display-name**	Establece el nombre a mostrar de un trabajo.

**shutdown**	Apagar inmediatamente Jenkins.

**stop-builds**	Stop all running builds for job(s)

**update-credentials-by-xml**	Update Credentials by XML

**update-credentials-domain-by-xml**	Update Credentials Domain by XML

**update-job**	Actualiza el fichero XML de la definición de una tarea desde la entrada estándard. Es lo contrario al comando get-job.

**update-node**	Updates the node definition XML from stdin. The opposite of the get-node command.

**update-view**	Updates the view definition XML from stdin. The opposite of the get-view command.

**version**	Muestra la versión actual.

**wait-node-offline**	Esperando a que el nodo esté desactivado

**wait-node-online**	Esperando hasta que el nodo esté activado

**who-am-i**	Muestra tus credenciales y permisos

### Plugins

Importante instalar el Role-based Authorization Strategy

### Crear job

En el apartado general, podemos especificar todo aquello que queremos insertar en nuestra tarea. Por ejemplo, uso de comandos, o bien parámetros, o combinar todo esto...

**Crear job con Maven**

Creada la tarea, en general, en la sección de ejecutar podemos hacer uso de comandos maven. 

Para instalar maven en nuestro contenedor, hacemos uso del siguiente comando:

```
docker exec -it -u root jenkins /bin/bash -c "apt update; apt install maven -y" 
```

También podemos instalar el plugin Maven Integration. Hecho esto, en global tool configuration abajo del todo tenemos la opción de añadir Maven y darle un nombre (por ej, MAVEN_HOME). Si creamos una tarea, nos aparecerá la opción **crear un proyecto Maven**. 

### Crear jobs anidados

Crear las tareas a anidar. Después, vamos en orden configurando cada tarea. La segunda tarea, vamos a Disparadores de ejecuciones e indicamos que queremos construir detrás de otro proyecto. Dentro de esta segunda tarea, bajamos y añadimos en Ejecutar que queremos ejecutar otros proyectos. 

### Crons

Para nomenclaturas de crontabs, es recomendable visitar la web **crontab guru**.  

Un cron en Jenkins se añade en el campo Ejecutar periodicamente, dentro de Disparador de Ejecuciones. 

### Analizar y recopilar info mediante Jenkins

Creada la tarea con Maven, en Proyecto, además de añadir Clean Package, hacemos lo propio con checkstyle:checkstyle para recopilar la info analizada del repositorio indicado. Abajo del todo, agregamos una acción, guardar los archivos generados. Es decir, vamos a guardar la info recopilada, en este caso todos los archivos de extensión .war 

```
**/*.war
```

### Creación de una pipeline

Creamos tarea en modo pipeline. Bajamos hasta el apartado Pipeline, donde podemos desplegar, por ejemplo, la opción Git + Maven como la escogida para el diseño de nuestra pipeline. 

Podemos crearlo también añadiendo la forma simple de texto plano para crear la misma pipeline. El código sería el siguiente, eso sí, deja bloqueada la fase de ejecución:

```
pipeline {
agent any

tools {
// Install the Maven version configured as "M3" and add it to the path.
maven "MAVEN_HOME"
}

stages {
stage('Construccion') {
steps {
// Get some code from a GitHub repository
git branch: 'main', url: "https://github.com/spring-guides/gs-spring-boot.git"
// git 'https://github.com/spring-guides/gs-spring-boot.git'

// Run Maven on a Unix agent.
echo 'Construimos.....'
sh "mvn clean package -DskipTests=true -f complete/pom.xml"
}

post {
// If Maven was able to run the tests, even if some of the test
// failed, record the test results and archive the jar file.
success {
archiveArtifacts 'complete/target/*.jar'
}
}
}

stage('Pruebas') {
steps {
echo 'Testeo.....'
sh "mvn test -f complete/pom.xml"
}
}

stage('Despliegue') {
steps {
echo 'Despliegue.....'
sh "mvn spring-boot:run -f complete/pom.xml"
}
}
}
}
```

Ahora vamos a crear una pipeline que nos lanza el servidor y cuando la pipeline termina el servidor se detiene. Con esto, evitamos la problemática de la pipeline anterior.

```
pipeline {
agent any

tools {
// Install the Maven version configured as "M3" and add it to the path.
maven "MAVEN_HOME"
}

stages {
stage('Construccion') {
steps {
// Get some code from a GitHub repository
git branch: 'main', url: "https://github.com/spring-guides/gs-spring-boot.git"
// git 'https://github.com/spring-guides/gs-spring-boot.git'

// Run Maven on a Unix agent.
echo 'Construimos.....'
sh "mvn clean package -DskipTests=true -f complete/pom.xml"
}

post {
// If Maven was able to run the tests, even if some of the test
// failed, record the test results and archive the jar file.
success {
archiveArtifacts 'complete/target/*.jar'
}
}
}

stage('Pruebas') {
steps {
echo 'Testeo.....'
sh "mvn test -f complete/pom.xml"
}
}

stage('Despliegue') {
steps {
echo 'Despliegue.....'
script {
    withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
        sh "nohup mvn spring-boot:run -f complete/pom.xml &"
    }
}
}
}
}
}
```

Hecha la pipeline, creamos un job bajo proyecto de estilo libre al que añadimos un script shell.

```
kill $(ps aux | grep '[.m2]/' | awk 'END {print $2}')
```

Otra opcion, en caso de que el contenedor de jenkins ya no contara con el comando ps:

```
kill $(jps -m | grep Launcher | awk '{print $1}')
```

### Herramientas Jenkins

**CatLight**

Monitorea los jobs o pipelines. Facil de instalar y usar.

### Notificación por email

En Configuración, bajamos hasta notificación por correo electrónico.

En servidor de correo saliente: smtp.gmail.com
El sufijo: @gmail.com

En opciones avanzadas activamos la seguridad SSL, añadimos el puerto 465 y la dirección de email deseada.

Puede haber error, para ello debemos ir a la url de google y permitir el acceso no seguro.

Para que el resultado de un job o pipeline nos sea enviado, nos vamos abajo en configuración, al apartado acciones para ejecutar después. Ahí indicamos que nos notifiquen por email.

### Home Backup

Creamos una tarea para hacer lo siguiente:

* En Administrar Jenkins, en Configuración del sistema, tenemos el directorio raíz. Lo copiamos.
* En la tarea creada, en ejecutar por comando, insertamos lo siguiente:
    - export JENKINS_HOME=/var/jenkins_home (directorio raíz)
    - rm -rf $JENKINS_HOME/backups
    - mkdir $JENKINS_HOME/backups
    - tar -zcvf $JENKINS_HOME/backups/backup_jenkins_home.tar.gz exclude='$JENKINS_HOME/backups' $JENKINS_HOME | true

### Herramientas para exprimir nuestro código

**Oxygen**

Para instalarlo

```
sudo apt install doxygen graphviz dot2tex
```

Para buscar los archivos sobre los que actuar dentro del directorio

```
doxygen 
```

Crea dos archivos: html y latex.

Utilizamos el módulo http-server de python para exportar la carpeta
html en el puerto 8888 de la máquina virtual usando un servidor web:

```
python3 -m http.server 8888 --directory html
```

Apunta el navegador a la dirección http://192.169.1.153:8888 (acuérdate de cambiar
la dirección IP y poner la de tu máquina) y podrás navegar por la documentación generada con DOxygen.

**Plugin HTML Publisher**

Es un plugin dentro de Jenkins que permite mostrar la documentación de una pipeline.

Si creo un archivo de tipo pipeline y lo subo a mi repositorio, con Jenkins puedo subirlo como tarea y al lanzarlo me mostrará la documentación generada por este plugin.

**Pipeline Syntax**

Snippet Generator: Dentro podremos escoger la opción archive artifacts. Lo indicamos con un nombre para el artefacto, por ejemplo, documentation.zip, el cual deberemos añadir en uno de los pasos del archivo pipeline con este comando:

```
sh "zip documentation.zip -r html/*"
```

Y en el apartado post del archivo pipeline, añadimos en success lo siguiente:

```
archive 'documentation.zip'
```

Se comitean estos cambios y en Jenkins lo indicamos en la configuración (lo del artefacto) y volvemos a construir.

### Herramientas para analizar nuestro código

**cppcheck**

Para instalarlo:

```
sudo apt install cppcheck
```

Para ejecutarlo:

```
cppcheck *.c --enable=all --inconclusive --language=c *.c <>
```

Para ejecutarlo en Jenkins, primero habremos de generar un informe. 

```
cppcheck *.c --xml --xml-version=2 --enable=all --inconclusive --language=c *.c 2>reports/cppcheck/report.xml
```

Junto al código se facilitan dos tareas de make para ejecutar cpp-check

* make cppcheck
* make cppcheck-xml

Para hacer uso de él en Jenkins, nos vamos al apartado de Plugins y primero instalamos Warnings Next Generation. Hecho esto, dentro de la pipeline en cuestión, en Pipeline Syntax, marcamos la opción recordissues.

En Tool desplegamos cppcheck y añadimos la ruta donde se encuentra el archivo de cppcheck. En este caso del curso sería reports/cppcheck/*.xml

Cuando demos a ejecutar, se nos generará un script pipeline que tendremos que copiar.

Por ende, volvemos a la terminal y editamos el Jenkinsfile para añadir un nuevo paso, el encargado de analizar el código.

Dentro de ese paso añadimos dos líneas:

```
sh 'make cppcheck-xml'
Aquí se copia el script generado anteriormente
```

Se comitean los cambios y si volvemos a Jenkins consola web y construimos la pipeline, nos aparecerá a la izquierda el apartado Warnings. 

Podría haber un error con el directorio report al no encontrarse en el repo. Para ello se crearían con git add -f.

### Herramientas para hacer testing automatizado

**Mocka**

Instalación

```
sudo apt install libcmocka0 libcmocka-dev
```

Ejecución

```
gcc tests/test_is_armstrong_number.c armstrong.o stack.o -lm -lcmocka -o tests/build/test_is_armstrong_number CMOCKA_MESSAGE_OUTPUT=stdout CMOCKA_XML_FILE='' ./tests/build/test_is_armstrong_number
```

Generación de informe XML (junit)

```
gcc tests/test_is_armstrong_number.c armstrong.o stack.o -lm -lcmocka -o tests/build/test_is_armstrong_number CMOCKA_XML_FILE=reports/cmocka/%g.xml CMOCKA_MESSAGE_OUTPUT=xml ./tests/build/test_is_armstrong_number
```

Makefile

Junto al código se facilitan dos tareas de make para ejecutar los tests
y generar el informe:

* make tests
* make tests-xml

**Pipeline Syntax**

Snippet Generator: Dentro podremos escoger la opción junit: Archive junit-formatted test results. En test reports xml, incluimos la ruta: tests/cmocka/*.xml y generamos el pipeline script.

Igualmente, debemos añadir este paso en el script dentro de su correspondiente fase:

```
stage('test'){
    steps {
        sh 'make tests-xml'
        junit 'reports/cmocka/*.xml'
    }
}
```

Podría haber un error al comitear con el directorio build al no encontrarse en el repo. Para ello se crearían con git add -f.

```
git add -f tests/build/.keep
```

Se comitean los cambios y si volvemos a Jenkins consola web y construimos la pipeline, nos aparecerá a la izquierda el apartado Test Result.





















