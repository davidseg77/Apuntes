# Cómo DESCARGAR e INSTALAR Sonarqube y ANALIZAR proyecto

SonarQube es una herramienta de análisis de código abierto que te permite encontrar problemas de calidad y seguridad en tu código. Con SonarQube, puedes analizar tu código en busca de errores, vulnerabilidades y malas prácticas de programación.

## 1. Descarga e instalación

Comenzamos por descargar Java (openlogic.com, por ejemplo, o de Oracle. Y a partir de la versión 11), en formato .msi. E instalamos facilmente.

Cuando se descargue y extraiga, vamos a su carpeta bin. Esta carpeta hemos de agregarla en las variables de entorno. En Windows vamos a variables de entorno y en path damos en editar y examinar para buscar nuestra carpeta bin de Java. Y agregamos y aceptamos todo. 

Comprobamos con **java --version**


Hacemos lo propio con SonarQube desde su web:

<sonarsource.com/products/sonarqube/downloads/>

Una vez descargado, nos vamos a su directorio y en bin tenemos una carpeta para cada SO. Vamos al que nos venga mejor y ejecutamos su archivo StartSonar.bat

Directamente, se ejecutará en local en el puerto 9000. Nos logueamos (admin y admin), y nos pide cambiar la contraseña. Nos aparece una interfaz, donde podemos añadir un proyecto existente o crear uno manualmente. 

Una vez añadido el proyecto, el código a analizar, se nos pedirá descargar SonarScanner en base al SO determinado. Es recomendable descargarlo en el mismo directorio donde anteriormente se descargo y extrajo SonarQube. 

Cuando se descargue y extraiga, vamos a su carpeta bin. Esta carpeta hemos de agregarla en las variables de entorno. En Windows vamos a variables de entorno y en path damos en editar y examinar para buscar nuestra carpeta bin de SonarScanner. Y agregamos y aceptamos todo. 

Comprobamos con **sonar-scanner --version**

Ahora, a modo de prueba, descargo un proyecto mio de Github y voy a copiarlo en el mismo directorio de Sonar. Entramos dentro de ese directorio, el del proyecto Github, y clic derecho para abrir terminal. 
Acto seguido, copio el comando de ejecución de Sonar Scanner que se nos facilita. Y ejecutamos el escaneo del código de nuestro proyecto Github. 


## 2. Ejecución y análisis de un proyecto propio de forma manual

En primer lugar, vamos a editar unos aspectos de la configuración de Sonar Scanner. Dentro de su carpeta, vamos a conf y ahí tenemos un archivo sonar-scanner. Clic derecho y abrimos con bloc de notas. Dentro tenemos unas propiedades de configuración, y aquí podemos añadir aspectos de nuestro proyecto. 

Por ejemplo, vamos a trabajar con un proyecto basado en python. Pues añadimos lo siguiente al final del archivo:

``` 
#PYTHON-PROJECT
sonar.projectkey = nombreproyecto
sonar.projectName = nombreproyecto (similar al primer campo)
sonar.projectVersion = 1.0
sonar.source = ruta del proyecto (Importante: ruta exacta e importante cambiar la orientación de las barras transversales. Suele dar errores.)
sonar.languages = py (al ser python)
``` 

Ahora vamos al directorio raíz del proyecto y en la barra de la ruta hacemos clic derecho y escribimos cmd para pasar a la terminal directamente desde este directorio.

Hecho esto, pasamos a nuestra interfaz de Sonarqube, con localhost:9000 y en la cuenta del administrador, en Security, generamos un token de acceso, Token User. Lo generamos y copiamos dicho token. 

Volvemos a la terminal desde el directorio raíz del proyecto para insertar el siguiente comando:

``` 
sonar-scanner -Dsonar.login=token
``` 

Y si vamos a la interfaz de SonarQube, en el apartado Projects tendremos nuestro análisis del código del proyecto. 







