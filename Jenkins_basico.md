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
