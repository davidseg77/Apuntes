#  Cómo configurar canalizaciones de integración continua en Jenkins en Ubuntu 22.04

## 1. Introducción

Jenkins es un servidor de automatización de código abierto destinado a automatizar tareas técnicas repetitivas involucradas en la integración y entrega continua de software. Con un sólido ecosistema de complementos y un amplio soporte, Jenkins puede manejar un conjunto diverso de cargas de trabajo para crear, probar e implementar aplicaciones.

Para este tutorial, integraremos Jenkins con GitHub para que Jenkins reciba una notificación cuando se envíe código nuevo al repositorio. Cuando se notifica a Jenkins, verificará el código y luego lo probará dentro de los contenedores Docker para aislar el entorno de prueba de la máquina host de Jenkins. Usaremos una aplicación Node.js de ejemplo para mostrar cómo definir el proceso CI/CD para un proyecto.

## 2. Requisitos previos

Para seguir esta guía, necesitará un servidor Ubuntu 22.04 con al menos 1 GB de RAM configurado con una instalación segura de Jenkins. Para proteger adecuadamente la interfaz web, deberá asignar un nombre de dominio al servidor Jenkins. Ademas, habremos de tener:

* Jenkins en Ubuntu 22.04
* Nginx en Ubuntu 22.04
* Proteger Nginx con Let's Encrypt en Ubuntu 22.04
* Configurar Jenkins con SSL usando un proxy inverso Nginx

Para controlar mejor nuestro entorno de pruebas, ejecutaremos las pruebas de nuestra aplicación dentro de contenedores Docker. Una vez que Jenkins esté en funcionamiento, instale Docker en el servidor.

## 3. Configurar Jenkins con SSL usando un proxy inverso Nginx

### 3.1 Configurar Nginx

Abra este archivo para agregar su configuración de proxy inverso:/etc/nginx/sites-available/example.com

``` 
sudo nano /etc/nginx/sites-available/example.com
```

En el server bloque con los ajustes de configuración de SSL, agregue registros de error y acceso específicos de Jenkins:

``` 
. . . 
server {
        . . .
        # SSL Configuration
        #
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        access_log            /var/log/nginx/jenkins.access.log;
        error_log             /var/log/nginx/jenkins.error.log;
        . . .
        }
``` 

A continuación, configure los ajustes del proxy. Dado que todas las solicitudes se envían a Jenkins, comente la try_files línea predeterminada, que de lo contrario devolvería un error 404 antes de que la solicitud llegue a Jenkins:

```
. . .
           location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;        }
. . . 
```

Ahora agreguemos la configuración del proxy, que incluye:

* **proxy_params/etc/nginx/proxy_params:** Nginx proporciona el archivo y garantiza que la información importante, incluido el nombre de host, el protocolo de la solicitud del cliente y la dirección IP del cliente, se conserve y esté disponible en los archivos de registro .
* **proxy_pass:** Esto establece el protocolo y la dirección del servidor proxy, que en este caso será el servidor Jenkins al que se accede a través localhost del puerto 8080.
* **proxy_read_timeout:** Esto permite un aumento del valor predeterminado de 60 segundos de Nginx al valor de 90 segundos recomendado por Jenkins.
* **proxy_redirect:** Esto garantiza que las respuestas se reescriban correctamente para incluir el nombre de host adecuado.
  
Asegúrese de sustituir su nombre de dominio protegido por SSL example.com en la proxy_redirect línea siguiente:

```
Location /  
. . .
           location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                include /etc/nginx/proxy_params;
                proxy_pass          http://localhost:8080;
                proxy_read_timeout  90s;
                # Fix potential "It appears that your reverse proxy setup is broken" error.
                proxy_redirect      http://localhost:8080 https://example.com;
``` 

Una vez que haya realizado estos cambios, guarde el archivo y salga del editor. Esperaremos el reinicio de Nginx hasta que hayamos configurado Jenkins, pero podemos probar nuestra configuración ahora:

``` 
sudo nginx -t
``` 

Si todo está bien, el comando devolverá:

``` 
Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
``` 

De lo contrario, corrija los errores informados hasta que pase la prueba.

### 3.2 Configurar Jenkins

Para que Jenkins funcione con Nginx, deberá actualizar la configuración de Jenkins para que el servidor Jenkins escuche solo en la localhost interfaz en lugar de en todas las interfaces ( 0.0.0.0). Si Jenkins escucha en todas las interfaces, es potencialmente accesible en su puerto original sin cifrar ( 8080).

Modifiquemos el /etc/default/jenkins archivo de configuración para realizar estos ajustes:

``` 
sudo nano /etc/default/jenkins
``` 

Localice la JENKINS_ARGS línea y agréguela --httpListenAddress=127.0.0.1 a los argumentos existentes:

``` 
. . .
JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --httpListenAddress=127.0.0.1"
``` 

Guardar y salir del archivo.

Para utilizar los nuevos ajustes de configuración, reinicie Jenkins:

``` 
sudo systemctl restart jenkins
``` 

Como systemctl no muestra resultados, verifique el estado:

``` 
sudo systemctl status jenkins
``` 

Deberías ver el active (exited) estado en la Active línea:

``` 
Output
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 16:35:49 UTC; 2s ago
   Main PID: 89751 (java)
      Tasks: 44 (limit: 4665)
     Memory: 358.9M
        CPU: 20.195s
     CGroup: /system.slice/jenkins.service
             └─89751 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080
``` 

Reinicie Nginx:

``` 
sudo systemctl restart nginx
``` 

Verifique el estado:

``` 
sudo systemctl status nginx
``` 

``` 
Output
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 16:36:17 UTC; 7s ago
       Docs: man:nginx(8)
    Process: 89866 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 89869 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 89870 (nginx)
      Tasks: 3 (limit: 4665)
     Memory: 4.1M
        CPU: 51ms
     CGroup: /system.slice/nginx.service
             ├─89870 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
``` 

Con ambos servidores reiniciados, debería poder visitar el dominio utilizando HTTP o HTTPS. Las solicitudes HTTP se redireccionarán automáticamente a HTTPS y el sitio de Jenkins funcionará de forma segura.

### 3.3 Probar la configuración

Ahora que ha habilitado el cifrado, puede probar la configuración restableciendo la contraseña administrativa. Comencemos visitando el sitio a través de HTTP para verificar que pueda comunicarse con Jenkins y ser redirigido a HTTPS.

En su navegador web, ingrese, sustituyendo su dominio por . Después de presionar , la URL debe comenzar con y la barra de ubicación debe indicar que la conexión es segura. http://example.comexample.com ENTER https

Puede ingresar el nombre de usuario administrativo que creó en la instalación de Jenkins en el campo Usuario y la contraseña que seleccionó en el campo Contraseña.

Una vez que haya iniciado sesión, puede cambiar la contraseña para asegurarse de que sea segura.

Haga clic en su nombre de usuario en la esquina superior derecha de la pantalla. En la página de perfil principal, seleccione Configurar de la lista en el lado izquierdo de la página.

Esto lo llevará a una nueva página, donde podrá ingresar y confirmar una nueva contraseña.

Confirme la nueva contraseña haciendo clic en Guardar . Ahora puede utilizar la interfaz web de Jenkins de forma segura.

## 4. Agregue el usuario de Jenkins al grupo Docker

Después de seguir los requisitos previos, tanto Jenkins como Docker se instalan en su servidor. Sin embargo, de forma predeterminada, el usuario de Linux responsable de ejecutar el proceso de Jenkins no puede acceder a Docker.

Para solucionar este problema, necesitamos agregar el jenkins usuario al docker grupo usando el usermod comando:

``` 
sudo usermod -aG docker jenkins
``` 

Puede enumerar los miembros del docker grupo para confirmar que el jenkins usuario se haya agregado correctamente:

``` 
grep docker /etc/group
Output
docker:x:999:sammy,jenkins
``` 

Para que Jenkins utilice su nueva membresía, debe reiniciar el proceso:

``` 
sudo systemctl restart jenkins
``` 

Si instaló Jenkins con los complementos predeterminados, es posible que deba verificar para asegurarse de que los complementos docker y docker-pipeline también estén habilitados. Para hacerlo, haga clic en Administrar Jenkins en la barra lateral y luego en Administrar complementos en el siguiente menú. Haga clic en la pestaña Disponible del menú de complementos para buscar nuevos complementos y escriba docker en la barra de búsqueda. Si ambos Docker Pipeline y Docker plugin se devuelven como opciones y no están seleccionados, seleccione ambos y, cuando se le solicite, permita que Jenkins se reinicie con los nuevos complementos habilitados.

Esto debería tomar aproximadamente un minuto y la página se actualizará luego.

## 5. Cree un token de acceso personal en GitHub

Para que Jenkins pueda ver sus proyectos de GitHub, deberá crear un token de acceso personal en nuestra cuenta de GitHub.

Comience visitando GitHub e iniciando sesión en su cuenta si aún no lo ha hecho. Luego, haga clic en su icono de usuario en la esquina superior derecha y seleccione Configuración en el menú desplegable.

En la página siguiente, ubique la sección Configuración de desarrollador en el menú de la izquierda y haga clic en Tokens de acceso personal.

Haga clic en el botón Generar nuevo token en la página siguiente.

Serás llevado a una página donde podrás definir el alcance de tu nuevo token.

En el cuadro Descripción del token, agregue una descripción que le permitirá reconocerlo más tarde.

En la sección Seleccionar ámbitos, marque las casillas repo:status, repo:public_repo y admin:org_hook. Esto permitirá a Jenkins actualizar los estados de confirmación y crear webhooks para el proyecto. Si está utilizando un repositorio privado, deberá seleccionar el permiso general del repositorio en lugar de los subelementos del repositorio.

Cuando haya terminado, haga clic en Generar token en la parte inferior.

Serás redirigido nuevamente a la página de índice de tokens de acceso personal y se mostrará tu nuevo token.

Copie el token ahora para que podamos hacer referencia a él más adelante. Como indica el mensaje, no hay forma de recuperar el token una vez que abandona esta página.

Ahora que tiene un token de acceso personal para su cuenta de GitHub, podemos configurar Jenkins para que observe el repositorio de su proyecto.

## 6. Agregue el token de acceso personal de GitHub a Jenkins

Ahora que tenemos un token, debemos agregarlo a nuestro servidor Jenkins para que pueda configurar webhooks automáticamente. Inicie sesión en su interfaz web de Jenkins utilizando la cuenta administrativa que configuró durante la instalación.

Haga clic en su nombre de usuario en la esquina superior derecha para acceder a su configuración de usuario y, desde allí, haga clic en Credenciales en el menú de la izquierda.

En la página siguiente, haga clic en la flecha junto a (global) dentro del alcance de Jenkins. En el cuadro que aparece, haga clic en Agregar credenciales.

Se le dirigirá a un formulario para agregar nuevas credenciales.

En el menú desplegable Tipo, seleccione Texto secreto. En el campo Secreto, pegue su token de acceso personal de GitHub. Complete el campo Descripción para que pueda identificar esta entrada en una fecha posterior. Puede dejar el ámbito como Global y el campo ID en blanco.

Haga clic en el botón Aceptar cuando haya terminado.

Ahora podrá hacer referencia a estas credenciales de otras partes de Jenkins para ayudar en la configuración.

## 7. Configurar el acceso de Jenkins a GitHub

De vuelta en el panel principal de Jenkins, haga clic en Administrar Jenkins en el menú de la izquierda.
En la lista de enlaces en la página siguiente, haga clic en Configurar sistema.

Desplácese por las opciones de la página siguiente hasta encontrar la sección GitHub. Haga clic en el botón Agregar servidor GitHub y luego seleccione Servidor GitHub.

La sección se expandirá para solicitar información adicional. En el menú desplegable Credenciales, seleccione su token de acceso personal de GitHub que agregó en la última sección.

Haga clic en el botón Probar conexión. Jenkins realizará una llamada API de prueba a su cuenta y verificará la conectividad. Cuando haya terminado, haga clic en el botón Guardar para implementar sus cambios.

## 8. Configure la aplicación de demostración en su cuenta de GitHub

Para demostrar cómo usar Jenkins para probar una aplicación, usaremos un programa "hola mundo" creado con Hapi.js. Debido a que estamos configurando Jenkins para que reaccione a los envíos al repositorio, debe tener su propia copia del código de demostración.

Visite el repositorio del proyecto y haga clic en el botón Bifurcar en la esquina superior derecha para hacer una copia del repositorio en su cuenta. Se agregará una copia del repositorio a su cuenta.

El repositorio contiene un package.json archivo que define el tiempo de ejecución y las dependencias de desarrollo, así como también cómo ejecutar el conjunto de pruebas incluido. Las dependencias se pueden instalar ejecutando npm install y las pruebas se pueden ejecutar usando npm test.

Jenkinsfile. También agregamos uno al repositorio. Jenkins lee este archivo para determinar las acciones que se ejecutarán en el repositorio para compilarlo, probarlo o implementarlo. Está escrito utilizando la versión declarativa de Jenkins Pipeline DSL.

Lo Jenkinsfile incluido en el hello-hapi repositorio se ve así:

``` 
#!/usr/bin/env groovy

pipeline {

    agent {
        docker {
            image 'node'
            args '-u root'
        }
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'npm test'
            }
        }
    }
}
```

Contiene pipeline la definición completa que Jenkins evaluará. En el interior, tenemos una agent sección que especifica dónde se ejecutarán las acciones en el pipeline. Para aislar nuestros entornos del sistema host, realizaremos pruebas en contenedores Docker, especificados por el docker agente.

Dado que Hapi.js es un marco para Node.js, usaremos la node imagen de Docker como base. Especificamos el root usuario dentro del contenedor para que el usuario pueda escribir simultáneamente tanto en el volumen adjunto que contiene el código extraído como en el volumen en el que el script escribe su salida.

A continuación, el expediente define dos etapas, es decir, divisiones lógicas del trabajo. Al primero lo hemos llamado "Construir" y al segundo "Prueba". El paso de compilación imprime un mensaje de diagnóstico y luego se ejecuta npm install para obtener las dependencias requeridas. El paso de prueba imprime otro mensaje y luego ejecuta las pruebas como se define en el package.json archivo.

Ahora que tiene un repositorio con un archivo válido Jenkinsfile, podemos configurar Jenkins para que observe este repositorio y ejecute el archivo cuando se introduzcan cambios.

## 9. Crear una nueva tubería en Jenkins

A continuación, podemos configurar Jenkins para que use el token de acceso personal de GitHub para observar nuestro repositorio.

De vuelta en el panel principal de Jenkins, haga clic en Nuevo elemento en el menú de la izquierda.

Ingrese un nombre para su nueva canalización en el campo Ingrese un nombre de elemento. Luego, seleccione Pipeline como tipo de elemento.

Haga clic en el botón Aceptar en la parte inferior para continuar.

En la siguiente pantalla, marque la casilla del proyecto GitHub. En el campo URL del proyecto que aparece, ingrese la URL del repositorio de GitHub de su proyecto.

A continuación, en la sección Activadores de compilación, marque el disparador de enlace de GitHub para la casilla de sondeo GITScm.

En la sección Pipeline, debemos indicarle a Jenkins que ejecute la canalización definida en Jenkinsfile nuestro repositorio. Cambie el tipo de definición a script de canalización desde SCM.

En la nueva sección que aparece, elija Git en el menú SCM. En el campo URL del repositorio que aparece, ingrese nuevamente la URL de su bifurcación del repositorio.

Cuando haya terminado, haga clic en el botón Guardar en la parte inferior de la página.

## 10. Realizar una compilación inicial y configurar los webhooks

Jenkins no configura automáticamente webhooks cuando define la canalización para el repositorio en la interfaz. Para que Jenkins configure los enlaces apropiados, debemos realizar una compilación manual la primera vez.

En la página principal de su canalización, haga clic en Construir ahora en el menú de la izquierda.

Se programará una nueva construcción. En el cuadro Historial de compilación en la esquina inferior izquierda, debería aparecer una nueva compilación en un momento. Además, se comenzará a dibujar una vista de escenario en el área principal de la interfaz. Esto hará un seguimiento del progreso de su ejecución de prueba a medida que se completan las diferentes etapas.

En el cuadro Historial de compilación, haga clic en el número asociado con la compilación para ir a la página de detalles de la compilación. Desde aquí, puede hacer clic en el botón Salida de la consola en el menú de la izquierda para ver los detalles de los pasos que se ejecutaron.

Haga clic en el elemento Volver al proyecto en el menú de la izquierda cuando haya terminado para regresar a la vista principal del proceso.

Ahora que hemos creado el proyecto una vez, podemos hacer que Jenkins cree los webhooks para nuestro proyecto. Haga clic en Configurar en el menú de la izquierda de la canalización.

No son necesarios cambios en esta pantalla, simplemente haga clic en el botón Guardar en la parte inferior. Ahora que Jenkins tiene información sobre el proyecto desde el proceso de compilación inicial, registrará un webhook con nuestro proyecto GitHub cuando guarde la página.

Puede verificar esto yendo a su repositorio de GitHub y haciendo clic en el botón Configuración. En la página siguiente, haga clic en Webhooks en el menú lateral. Deberías ver el webhook de tu servidor Jenkins en la interfaz principal.

Si por alguna razón Jenkins no pudo registrar el enlace (por ejemplo, debido a cambios en la API ascendente o interrupciones entre Jenkins y Github), puede agregar uno rápidamente usted mismo haciendo clic en Agregar webhook y asegurándose de que la URL de carga útil esté configurada en y el tipo de contenido . está configurado en y luego haga clic en Agregar webhook nuevamente en la parte inferior del mensaje. https://my-jenkins-server:8080/github-webhook application/json

Ahora, cuando envíe nuevos cambios a su repositorio, se notificará a Jenkins. Luego extraerá el nuevo código y lo volverá a probar utilizando el mismo procedimiento.

Para aproximar esto, en nuestra página de repositorio en GitHub, puede hacer clic en el botón Crear nuevo archivo a la izquierda del botón verde Clonar o descargar.

En la página siguiente, elija un nombre de archivo y algunos contenidos ficticios.

Haga clic en el botón Confirmar nuevo archivo en la parte inferior cuando haya terminado.

Si regresa a su interfaz de Jenkins, verá que se inicia automáticamente una nueva compilación.

Puede iniciar compilaciones adicionales realizando confirmaciones en una copia local del repositorio y enviándola nuevamente a GitHub.

## 11. Conclusión

En esta guía, configuramos Jenkins para observar un proyecto de GitHub y probar automáticamente cualquier cambio nuevo que se confirme. Jenkins extrae el código del repositorio y luego ejecuta los procedimientos de compilación y prueba desde contenedores Docker aislados. El código resultante se puede implementar o almacenar agregando instrucciones adicionales al mismo Jenkinsfile.





