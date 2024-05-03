# Jenkins - Cultura Devops

Repositorio del curso:

<https://github.com/culturadevops/jenkins>


## 1. Ejemplo tarea en Jenkins

Vemos el código generado para una pipeline Jenkinsfile para un servidor web con Docker:

``` 

pipeline {
  agent any
 parameters {
        string(name: 'name_container', defaultValue: 'proyecto-qa', description: 'nombre del docker')
        string(name: 'name_imagen', defaultValue: 'iproyecto-qa', description: 'nombre de la imagen')
        string(name: 'tag_imagen', defaultValue: 'latest', description: 'etiqueta de la imagen')
        string(name: 'puerto_imagen', defaultValue: '81', description: 'puerto a publicar')
    }
    environment {
        name_final = "${name_container}${tag_imagen}${puerto_imagen}"        
    }
    stages {
          stage('stop/rm') {

            when {
                expression { 
                    DOCKER_EXIST = sh(returnStdout: true, script: 'echo "$(docker ps -q --filter name=${name_final})"').trim()
                    return  DOCKER_EXIST != '' 
                }
            }
            steps {
                script{
                    sh ''' 
                         docker stop ${name_final}
                    '''
                    }
                    
                }                    
                                  
            }
           
        stage('build') {
            steps {
                script{
                    sh ''' 
                    docker build    jobs/dockerweb/ -t ${name_imagen}:${tag_imagen}
                    '''
                    }
                    
                }                    
                                  
            }
            stage('run') {
            steps {
                script{
                    sh ''' 
                        docker run -dp ${puerto_imagen}:80 --name ${name_final} ${name_imagen}:${tag_imagen}
 
                    '''
                    }
                    
                }                    
                                  
            }
            
          
        }   
    }
``` 

Hay que especificar bien la ruta dentro del repositorio. También podría haber problemas con respecto a los permisos del usuario de Jenkins sobre Docker, lo cual se resolverían dandole esos permisos bien mediante chmod o bien agregando el usuario Jenkins al grupo Docker. 


## 2. Creando cuentas de usuarios dentro de instancias EC2 con jenkins

Para ello accedemos al agente con el cual vamos a realizar la tarea. Después, creamos una tarea que vamos a llamar AccesoBastion, por ejemplo. Añadimos la ruta donde tenemos el Jenkinsfile dentro de nuestro repo, el cual tiene la siguiente estructura:

``` 
pipeline {
    agent { label 'bastion'}
    parameters {
        string defaultValue: '', description: 'Nombre del usuario a crear', name: 'USUARIO', trim: true
        string defaultValue: '', description: 'contraseña nueva del usuario a crear', name: 'PASSWORD', trim: true
    }
    stages {
        stage ('Creando Usuario') {
            steps {
                        script {
                           sh  "sudo useradd -d /home/$USUARIO  -m -p \$(echo '$PASSWORD' | openssl passwd -1 -stdin) $USUARIO"
                    }
                
            }
        }
    }
}
```

Construimos con parámetros y manualmente añadimos nombre de usuario y contraseña. Después comprobamos en el bastión. 


## 3. Creando CUENTA DE acceso en una Base De datos RDS MySQL con jenkins

En primer lugar, necesitamos una credencial global. En AWS ya tenemos una instancia RDS creada. Esta nos da ya la ruta de acceso. Las credenciales deben ser las especificadas en Jenkins, han de coincidir.


En el repo, tenemos una carpeta con el Jenkinsfile dentro para esta tarea:

``` 
pipeline {
    agent any
    parameters {
        string defaultValue: '', description: 'Nombre del usuario a crear', name: 'USUARIO', trim: true
        string defaultValue: '', description: 'contraseña nueva del usuario a crear', name: 'PASSWORD', trim: true
        choice choices: ['Lectura','Admin','Desarrollador'], description: '', name: 'TIPOUSUARIO'
        extendedChoice description: '', multiSelectDelimiter: ',', name: 'DATABASE', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_CHECKBOX', 
            value: 'database,otroejemplo', visibleItemCount: 10
    }
    environment {
        ADMINPERMISO = "SHOW DATABASES,INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER "
        DESARROLLADOR= "INSERT, UPDATE, DELETE, CREATE, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE,CREATE VIEW, CREATE ROUTINE, ALTER ROUTINE, TRIGGER"
        PERMISO = "SELECT, SHOW VIEW "
        SUFIJOHOST= "Aquí va el endpoint de la RDS a partir del punto del database."
    }
    stages {
        stage ('Creando Usuario') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mysql', passwordVariable: 'PASSWD_DB', usernameVariable: 'USER_DB')]) {
                        script {
                        def tipousuario="${TIPOUSUARIO}"
                        def browsers ="${DATABASE}".split(",")
                        if (browsers.size()>0){
                            if (tipousuario=="Admin"){
                                PERMISO= PERMISO +","+ ADMINPERMISO
                            }
                            if (tipousuario=="Desarrollador"){
                                PERMISO= PERMISO +","+ DESARROLLADOR
                            }
                            
                            for (int i = 0; i < browsers.size(); ++i) {
                                sh  "mysql -u${USER_DB} -p${PASSWD_DB} -h ${browsers[i]}.${SUFIJOHOST} -e \"CREATE USER IF NOT EXISTS '$USUARIO'@'%' IDENTIFIED BY '$PASSWORD' ;\""
                                sh  "mysql -u${USER_DB} -p${PASSWD_DB} -h ${browsers[i]}.${SUFIJOHOST} -e \"GRANT $PERMISO ON *.* TO '$USUARIO'@'%';\""
                            }
                        }
                    }
                }
            }
        }
    }
}
``` 

Nuestro master debe tener instalado MySQL, lo cual hacemos en su terminal con el siguiente comando:

``` 
sudo apt install mysql-client-core-5.7
``` 

Ejecutamos la tarea, construimos con parámetros y añadimos todos los detalles de la cuenta de acceso a nuestra base de datos, automatizando el proceso.


## 4. Como usar credenciales "secret text"

Para ejecutar tareas donde la info se muestre solo para aquellos con esa potestad. Primero creamos la credencial global de tipo secret. Trás ello, añado la ruta donde tengo este Jenkinsfile:

``` 
pipeline {
    agent { label  "master"}
    parameters {
        string defaultValue: '', description: 'Nombre del usuario a crear', name: 'USUARIO', trim: true
        string defaultValue: '', description: 'contraseña nueva del usuario a crear', name: 'PASSWORD', trim: true
        choice choices: ['Lectura','Admin','Desarrollador','APP'], description: '', name: 'TIPOUSUARIO'
        extendedChoice description: '', multiSelectDelimiter: ',', name: 'DATABASE', quoteValue: false, saveJSONParameterToFile: false, type: 'PT_CHECKBOX', 
            value: 'database,otroejemplo', visibleItemCount: 10
    }
    environment {
        ADMINPERMISO = "SHOW DATABASES,INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER "
        DESARROLLADOR= "SELECT, INSERT, UPDATE, DELETE, CREATE, DROP"
        
        APP= "INSERT, UPDATE, DELETE, EXECUTE"        
        PERMISO = "SELECT, SHOW VIEW "
    }
    stages {
        stage ('Creando Usuario') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mysql', passwordVariable: 'PASSWD_DB', usernameVariable: 'USER_DB')]) {
                        script {
                        def tipousuario="${TIPOUSUARIO}"
                        def browsers ="${DATABASE}".split(",")
                        if (browsers.size()>0){
                            if (tipousuario=="Admin"){
                                PERMISO= PERMISO +","+ ADMINPERMISO
                            }
                            if (tipousuario=="Desarrollador"){
                                PERMISO= PERMISO +","+ DESARROLLADOR
                            }
                            if (tipousuario=="APP"){
                                PERMISO= PERMISO +","+ APP
                            }
                            withCredentials([string(credentialsId: 'basedatos_link', variable: 'DB_LINK')]) {
                                for (int i = 0; i < browsers.size(); ++i) {
                                    sh  "mysql -u${USER_DB} -p${PASSWD_DB} -h ${browsers[i]}.${DB_LINK} -e \"CREATE USER IF NOT EXISTS '$USUARIO'@'%' IDENTIFIED BY '$PASSWORD' ;\""
                                    sh  "mysql -u${USER_DB} -p${PASSWD_DB} -h ${browsers[i]}.${DB_LINK} -e \"GRANT $PERMISO ON *.* TO '$USUARIO'@'%';\""
                                }
                            }
                            
                        }
                    }
                }
            }
        }
    }
}
``` 


## 5. Como instalar Jenkins UBUNTU EN DOCKER en Azure usando Kubernetes AKS

Vamos a ver cómo instalar Jenkins en Kubernetes sobre la nube de Azure.

**Requisitos**

* Un clúster de Kubernetes
* Los siguientes archivos yaml (deployment, service e ingress):

- jenkins-deployment.yaml

``` 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins
          image:  jenkinskubenetes.azurecr.io/jenkins:latest
          env:
            - name: JAVA_OPTS
              value: -Djenkins.install.runSetupWizard=true
          ports:
            - name: http-port
              containerPort: 8080
            - name: jnlp-port
              containerPort: 50000
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
            - name: docker-sock-volume
              mountPath: /var/run/docker.sock
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-pvc
        - name: docker-sock-volume
          hostPath:
           path: /var/run/docker.sock
```

- jenkins-service.yaml

``` 
apiVersion: v1
kind: Service
annotations:
    alb.ingress.kubernetes.io/healthcheck-path: /
metadata:
  name: jenkins-master-svc
  labels:
    app: jenkins
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  - port: 50000
    targetPort: 50000
    protocol: TCP
    name: slave
  selector:
    app: jenkins
status:
  loadBalancer: {}
``` 

- ingress-asks.yaml

``` 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: jenkins.5bba6d77660b4868b18d.eastus.aksapp.io
    http:
      paths:
      - backend:
          serviceName: jenkins-master-svc
          servicePort: 80
        path: /
``` 

**Info previa**

Jenkins usa los puertos 8080 y 50000. Java 8

**Comandos de la instalación**

``` 
sudo docker build -t jenkins .
sudo docker login jenkinskubernetes.azurecr.io/jenkins
sudo docker tag jenkins jenkinskubernetes.azurecr.io/jenkins
sudo docker push jenkinskubernetes.azurecr.io/jenkins
``` 

### 5.1 Creación de instancia

Dentro de la instancia tenemos este Dockerfile:

``` 
FROM jenkins/jenkins:lts
 #https://updates.jenkins.io/download/plugins/
USER root
#RUN /usr/local/bin/install-plugins.sh \  
#   dashboard-view:2.9.10 \  
#   pipeline-stage-view:2.4 \  
#   parameterized-trigger:2.32 \  
#   bitbucket:1.1.5 \  
#   git:3.0.5 \  
#   github:1.26.0 \
#   sonarqube-generic-coverage:1.0 \
#   ssh-slaves:1.31.0 \
#   ec2-fleet:1.17.0 \
#   configuration-as-code-groovy:1.1 \
#   pipeline-maven:3.8.2
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common 
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get install docker-ce=17.12.1~ce-0~debian -y
RUN apt install maven -y
#RUN apt install awscli -y
#RUN apt-get install python3-pip -y
#RUN pip3 install --upgrade awscli
RUN usermod -aG docker jenkins
``` 

Dentro de ese directorio lanzamos la construcción con el nombre jenkins:

``` 
sudo docker build . -t jenkins
``` 

Ahora vamos a la consola de Azure y accedemos al ContainerRegistry creado. En Settings, en Access keys creamos unas credenciales. 

En la instancia lanzamos el segundo de los comandos:

``` 
sudo docker login jenkinskubernetes.azurecr.io/jenkins
``` 

Si tenemos las credenciales creadas, ya no nos la pedirá. Acto seguido, lanzamos los dos siguientes comandos para tagear y pushear. 

``` 
sudo docker tag jenkins jenkinskubernetes.azurecr.io/jenkins
sudo docker push jenkinskubernetes.azurecr.io/jenkins
```

Si refrescamos el container nos aparece ya la imagen Jenkins. Vamos a la consola, arriba a la derecha, y entramos en Kubernetes:

``` 
kubectl get pod
``` 

Y vamos al directorio donde ubicamos los yaml y los ejecutamos cada uno de ellos con kubectl apply -f.

Cuando esté todo desplegado, verificamos y con kubectl get ingress podemos ver la ruta que nos ha asignado. Si la copio y la pego en el navegador, ya accederemos a nuestra app. Solo habremos de insertar la clave de administrador de Jenkins. 

**Recomendación**

Al deployment sería conveniente añadirle un volumen con al menos 2GB de capacidad para no perder la info y que Jenkins pueda trabajar continuamente.


## 6. Instalación y configuración de JENKINS en UBUNTU AWS EC2

**Requisitos**

* VM Linux
* Cuenta en AWS

En primer lugar, hay que instalar Jenkins y Java 8 en la VM.

Trás ello, vamos a AWS y en EC2 creamos una instancia con su grupo de seguridad con todas las reglas activas y un par de claves.

Accedemos a la instancia con la IP dada por AWS y añadimos las credenciales. 

Ya dentro de la terminal, añadimos los comandos necesarios para la instalación de Jenkins y Java 8. 

Y después solo quedará acceder a Jenkins e iniciar la instalación de plugins, clave de administrador...















