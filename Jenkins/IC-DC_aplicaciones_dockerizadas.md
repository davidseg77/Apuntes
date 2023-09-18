# Taller de Jenkins para IC/DC de aplicaciones dockerizadas

## 1. Caso práctico

### 1.1 Creación de un Dockerfile


En primer lugar, hemos de tener una cuenta en Docker y Jenkins instalado en un servidor. Hecho esto, vamos a ese servidor de Jenkins y dentro de /opt creamos un directorio para nuestro Dockerfile. Va a ser un Dockerfile sencillo que va a contener una imagen de Nginx. 

### 1.2 Añadir credenciales Docker en Jenkins

A continuación, vamos a añadir nuestras credenciales de Docker en nuestro servidor Jenkins. Si accedemos a la IP del servidor con su puerto correspondiente (8080), añadiríamos /credentials/store/system/domain/_/newCredentials:

``` 
172.16.56.82:8080/credentials/store/system/domain/_/newCredentials
```

Introducimos el usuario y la contraseña de Docker y en ID ponemos dockerhub. Y OK.

Creadas las credenciales, vamos a crear un Jenkinsfile.

### 1.3 Crear Jenkinsfile (Pipeline)

Y vamos a crear tarea, de tipo Pipeline, y nos vamos al script de este Pipeline.

``` 
pipeline {
  agent { node{label 'master'} }

  environment {
    dockerfilePath= "ruta donde está el Dockerfile"
    registry= "Registro de Docker que hemos configurado"
    registryCredential = 'dockerhub' (ID que dimos en credentials)
  }

  stages {
    stage ('Test') {
      steps {
        sh 'echo executing test'     
      }
    }

    stage ('Build Docker') {
      steps {
        script {
          dockerImage = docker.build(registry, "-f ${dockerfilePath}/Dockerfile .")
        }    
      }
    }

    stage ('Push Docker') {
      steps {
        script {
          docker.withRegistry('', registryCredential){
            dockerImage.push()    
          }
        }    
      }
    }

    stage ('Run Docker') {
      steps {
        sh 'docker run -d -p 90:80 ${registry}'    
      }
    }

  }

}
```

### 1.4 Ejecución del Jenkinsfile (Pipeline)

Aplicamos y guardamos. Le damos a construir, y si hubiera algún error durante la ejecución de nuestra pipeline lo consultamos en los logs. Si todo va correctamente, accedemos al navegador con el puerto correspondiente y listo.


