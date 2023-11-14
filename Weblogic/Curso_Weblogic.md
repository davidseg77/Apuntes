# Info extraída del curso de Oracle Weblogic 12c desde cero

## 1. Instalación en Linux

Después de descargarlo de su página oficial, en nuestra terminal Linux descargamos el archivo, por ejemplo con unzip + nombre del archivo comprimido.

Trás ello, nos descargará dos archivos. Para instalar el software:

``` 
java -jar + nombre del fichero de extensión .jar 
```

Previamente, habría que tener instalado java. Hecho esto, se nos abrirá la interfaz gráfica de la instalación de Weblogic. En la izquierda, en el menú de la instalación, en Ubicación de instalación pondo en Directorio raíz de Oracle, por ejemplo, /home/david/weblogic.

Cuando el proceso de instalación se haya completado, vamos al directorio de weblogic:

``` 
cd /home/david/weblogic/
ll
```

Y veremos los directorios alojados en su interior, los instalados. Entre ellos, WL_HOME.

* **WL_HOME:** Esta variable y la ruta del directorio relacionado contienen los archivos instalados necesarios para alojar un servidor WebLogic, por ejemplo.MW_HOME/wlserver_10.3

