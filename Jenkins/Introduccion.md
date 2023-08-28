# Introducción a Jenkins


## 1. Instalación de Jenkins en docker

Tenemos muchos métodos para realizar la instalación de Jenkins.

Nosotros vamos a usar la imagen docker jenkins/jenkins para realizar la instalación de jenkins en un contenedor docker. Podemos ver la documentación de la imagen y realizamos los siguientes pasos:

``` 
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
``` 

Para obtener la la contraseña de administración que nos pregunta al principio ejecutamos:

``` 
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
``` 

Para terminar instalamos los plugins que nos recomiendan y creamos un usuario administrador.


## 2. Introducción a los Pipelines de Jenkins

Un Pipeline es una secuencia de tareas automatizadas que definen el ciclo de vida de la aplicación de nuestro flujo de integración/entrega/despliegue continuo. Podemos decir que un Pipeline es un conjunto de instrucciones del proceso que siga una aplicación desde el repositorio de control de versiones hasta que llega a los usuarios.

* **Disparadores:** Motivo por el cual se comienza la ejecución de tareas automáticas. Puede ser por varios motivos: push en un repositorio github, ejecución cada cierto tiempo, finalización de otra tarea…

* **Stage:** Son las etapas lógicas en las que se dividen los flujos de trabajo de Jenkins. Es una práctica recomendada dividir nuestro flujo de trabajo en etapas ya que nos ayudará a organizar nuestros pipelines en fases. Ejemplos de fases: build, test, deploy…
  
* **Steps:** Son las tareas ó comandos que ejecutados de forma secuencial implementan la lógica de nuestro flujo de trabajo.

* **Node:** Máquina que es parte del entorno de Jenkins y es capaz de ejecutar un Pipeline Jenkins. También llamada agentes de ejecución. Pueden ser la misma máquina donde tenemos instalado Jenkins, o máquinas configuradas para este fin. También podemos usar contenedores docker como agentes de ejecución. Es importante reseñar que el directorio de trabajo (workspace) es compartido por los steps del nodo, de forma que steps de un nodo pueden acceder a ficheros/directorios generados por steps de ese mismo nodo.
  
* **Notificaciones:** Por ejemplo que envíe un correo electrónico al terminar.


### 2.1 Creación de pipelines

Creamos una Nueva Tarea, y le ponemos un nombre y elegimos el tipo Pipeline.

En el apartado Pipeline, escribimos nuestro primer pipeline:

``` 
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Tareas para construir, instalar,...'
            }
        }
        stage('Test') {
            steps {
                echo 'Tareas para realizar test.'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Tareas para desplegar, construir, ...'
            }
        }
    }
}
``` 

Le damos a Guardar y ya podemos Construir ahora para ejecutar el pipeline y construir un Build.
Y si vemos la Console Output vemos la salida del build.


## 3. Instalación de docker como runner de Jenkins

Hasta ahora hemos estado trabajando con una instalación de jenkins sobre docker y hemos comprobado que las tareas que se realizan se hacen sobre el mismo nodo.

En este apartado vamos a configurar jenkins para que use contenedores docker como runner, es decir cuando ejecutemos un pipeline se creará un contenedor docker donde se realizarán las tareas. Esto tiene muchas ventajas, ya que no necesito instalar en la máquina de jenkins las herramientas que necesito para ejecutar las tareas, y además podemos usar cualquier imagen docker para que ejecute dichas tareas.

Para realizar la configuración necesitamos instalar docker en la máquina de jenkins, es por ello que vamos a hacer una nueva instalación usando la paquetería que nos proporciona jenkins:

### 3.1 Instalación de jenkins con apt

Según el manual de instalación. ejecutamos los siguientes comandos:

``` 
apt install openjdk-11-jre
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee   /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]   https://pkg.jenkins.io/debian-stable binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null
apt-get update
apt-get install jenkins

apt install docker.io
usermod -aG docker jenkins
``` 

Accedemos a jenkins y tenemos que obtener la clave de administración:

``` 
cat /var/lib/jenkins/secrets/initialAdminPassword
``` 

### 3.2 Configuración de docker como runner

Instalamos dos nuevos plugins: Docker y Docker Pipeline. (Administrar Jenkins -> Administrar Plugins). Y reiniciamos Jenkins. (Si no se reinicia desde el panel web podemos ejecutar systemctl restart jenkins).

Configuramos una nueva nube: Administrar Jenkins -> Administrar Nodos -> Configure Clouds.

Y configuramos el acceso a docker. Le ponemos name docker y Docker Host URI (unix:///var/run/docker.sock)

Y ya podemos hacer una prueba, creando un pipeline con las siguientes instrucciones:

```
pipeline {
    agent {
        docker { image 'python:3' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'python --version'
            }
        }
    }
}
```

Al ejecutar el pipeline podemos observar como se ha descargado la imagen, ha creado un contenedor, ha ejecutado la instrucción indicada y finalmente borra el contenedor.


## 4. Creación, testeo y publicación de imágenes docker desde Jenkins

El plugin Docker que instalamos en un ejemplo anterior, además de posibilitar correr nuestros pipelines en contenedores docker, nos ofrece la posibilidad de trabajar con docker: crear imágenes, probarlas, publicarlas en un registro…

En este apartado vamos a crear un pipeline, que va a realizar la siguientes tareas:

- Va a clonar un repositorio donde tenemos un Dockerfile.
- Va a construir la imagen.
- Va a hacer un pequeño test: va a crear un contenedor y va a comprobar que tiene apache2 instalado.
- Va a subir la imagen a DockerHub.
- Y finalmente, va a borrar la imagen generada.
  
El repositorio donde se encuentra el Dockerfile y el Jenkinsfile es https://github.com/josedom24/jenkins_docker.

### 4.1 Credenciales para subir la imagen a DockerHub

Hemos creado unas credenciales del tipo usuario - contraseña:

En Scope ponemos global, indicamos username y password y, por último, en ID ponemos USER_DOCKERHUB.

### 4.2 Pipeline

Una cosa al indicar que lea el pipeline de un Jenkinsfile es que por defecto lo va buscar en la rama master. En los nuevos repositorios hay que cambiarla por main:

Indicamos la URL del repositorio y en rama ponemos main.

El fichero Jenkinsfile tiene el siguiente contenido:

``` 
pipeline {
    environment {
        IMAGEN = "josedom24/myapp"
        USUARIO = 'USER_DOCKERHUB'
    }
    agent any
    stages {
        stage('Clone') {
            steps {
                git branch: "main", url: 'https://github.com/josedom24/jenkins_docker.git'
            }
        }
        stage('Build') {
            steps {
                script {
                    newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("$IMAGEN:$BUILD_NUMBER").inside('-u root') {
                           sh 'apache2ctl -v'
                        }
                    }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry( '', USUARIO ) {
                        newApp.push()
                    }
                }
            }
        }
        stage('Clean Up') {
            steps {
                sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                }
        }
    }
}
``` 

Algunas cosas a tener en cuenta:

* Al clonar el repositorio hemos indicado la rama.
* agent any: Este pipeline se ejecuta en el nodo principal.
* USUARIO = 'USER_DOCKERHUB', las credenciales del tipo username/pasword se leen de esta forma.


## 5. Ejecución de un pipeline en varios runner

Podemos ejecutar un conjunto de stages en un runner, y otro conjunto en otros. Por ejemplo, podemos hacer el build, y el test de la aplicación utilizando un runner que sea un contenedor docker, y el stage deploy, o la construcción de un contenedor docker lo podemos hacer desde la máquina donde tenemos instalado jenkins.

Veamos un ejemplo:

``` 
pipeline {
    agent none
    stages {
        stage("build and test the project") {
            agent {
                docker "python:2"
            }
            stages {
                stage('En el contenedor') {
            
                steps {
                    sh 'python --version'
                    }
                }
            }
        }
        stage("deploy in prodcution") {
            agent any
            stages {
                stage('En la máquina') {
                
                steps {
                    sh 'python3 --version'
                    }
                }
            }
        }
    }
}
``` 

En este caso:

* El stage En el contenedor se ejecuta en un contenedor debian.
* El stage En la máquina se ejecuta en la máquina de jenkins.



