# Introducción a Github

## 1. Creación de cuenta y acceso a los repositorios

Crea una cuenta en GitHub (Si no la tienes!!!). La forma de acceder a los repositorios remotos de GitHub va a ser por SSH, por lo tanto debes copiar tu clave pública RSA a GitHub, para ello:

* Copia el contenido de tu fichero ~/.ssh/id_rsa.pub, para ello: añade una nueva clave SSH en el apartado “SSH keys” de tu perfil en GitHub y pega el contenido de tu clave pública. Si no tienes ese fichero, puedes generar una nueva clave ssh pública.

### 1.1 Generar una clave pública SSH

Tal y como se ha comentado, muchos servidores Git utilizan la autentificación a través de claves públicas SSH. Y, para ello, cada usuario del sistema ha de generarse una, si es que no la tiene ya. El proceso para hacerlo es similar en casi cualquier sistema operativo.

Ante todo, asegurarte que no tengas ya una clave. Por defecto, las claves de cualquier usuario SSH se guardan en la carpeta ~/.ssh de dicho usuario. Puedes verificar si tienes ya unas claves, simplemente situandote sobre dicha carpeta y viendo su contenido:

``` 
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
``` 

Has de buscar un par de archivos con nombres tales como algo y algo.pub; siendo ese "algo" normalmente id_dsa o id_rsa. El archivo terminado en .pub es tu clave pública, y el otro archivo es tu clave privada. Si no tienes esos archivos (o no tienes ni siquiera la carpeta .ssh), has de crearlos; utilizando un programa llamado ssh-keygen, que viene incluido en el paquete SSH de los sistemas Linux/Mac o en el paquete MSysGit en los sistemas Windows:

``` 
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/schacon/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/schacon/.ssh/id_rsa.
Your public key has been saved in /Users/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
43:c5:5b:5f:b1:f1:50:43:ad:20:a6:92:6a:1f:9a:3a schacon@agadorlaptop.local
``` 

Como se vé, este comando primero solicita confirmación de dónde van a a guardarse las claves (.ssh/id_rsa), y luego solicita, dos veces, una contraseña (passphrase), contraseña que puedes dejar en blanco si no deseas tener que teclearla cada vez que uses la clave.

Tras generarla, cada usuario ha de encargarse de enviar su clave pública a quienquiera que administre el servidor Git (en el caso de que este esté configurado con SSH y así lo requiera). Esto se puede realizar simplemente copiando los contenidos del archivo terminado en .pub y enviandoselos por correo electrónico. La clave pública será una serie de números, letras y signos, algo así como esto:

``` 
$ cat ~/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@agadorlaptop.local
``` 

NOTA IMPORTANTE. [^1]

[^1]: En mi caso, la clave la copio en el apartado Settings de mi cuenta de usuario en Github, dentro de **SSH and GPG keys**.


## 2. Creación de un repositorio y configuración de Git

Crea en GitHub un repositorio con el nombre prueba_tu_nombre (inicializa el repositorio con un fichero README) y la descripción Repositorio de prueba 2ASIR.

Instala git en tu ordenador (si no lo tienes instalado!!!).

``` 
 apt-get install git
``` 

**Configuración de git**. 

Lo primero que deberías hacer cuando instalas Git es establecer tu nombre de usuario y dirección de correo electrónico (Asegurate que los datos son correctos y que has puesto tu nombre completo). Esto es importante porque las confirmaciones de cambios (commits) en Git usan esta información, y es introducida de manera inmutable en los commits que envías:

``` 
 git config --global user.name "John Doe"
 git config --global user.email johndoe@example.com
``` 

De nuevo, sólo necesitas hacer esto una vez si especificas la opción --global, ya que Git siempre usará esta información para todo lo que hagas en ese sistema.

**Clonar el repositorio remoto.** 

Copia la url SSH del repositorio (no copies la URL https) y vamos a clonar el repositorio en nuestro ordenador.

``` 
 git clone git@github.com:xxxxxxx/xxxxxxx.git
``` 

Comprueba que dentro del repositorio que hemos creado se encuentra el fichero README.md, en este fichero podemos poner la descripción del proyecto.

Vamos a crear un nuevo fichero, lo vamos a añadir a nuestro repositorio local y luego lo vamos a sincronizar con nuestro repositorio remoto de GitHub. Cada vez que hagamos una modificación en un fichero lo podemos señalar creando un commit. Los mensajes de los commits son fundamentales para explicar la evolución de un proyecto. Un commit debe ser un conjunto pequeño de cambios de los ficheros del proyecto con una cierta coherencia.

``` 
 echo "Esto es una prueba">ejemplo.txt
 git add ejemplo.txt
 git commit -m "He creado el fichero ejemplo.txt"
 git push
``` 

Si modificas un fichero en tu repositorio local, no tienes que volver a añadirlo a tu repositorio (git add). Pero tienes que usar la opción -a al hacer el commit.

``` 
 git commit -am "He modificado el fichero ejemplo.txt"
 git push
``` 

Si quieres cambiar el nombre de un fichero o directorio de tu repositorio:

``` 
 git mv ejemplo.txt ejemplo2.txt
 git commit -am "He cambiado el nombre del fichero"
 git push
``` 

Si quieres borrar un fichero de tu repositorio:

``` 
 git rm ejemplo2.txt
 git commit -am "He borrado el fichero ejemplo2"
 git push
``` 

Puedes clonar tu repositorio de GitHub en varios ordenadores (por ejemplo, si quieres trabajar en tu casa y en el instituto), por lo tanto antes de trabajar en un repositorio local tienes que sincronizar los posibles cambios que se hayan producido en el repositorio remoto, para ello:

``` 
git pull
```

Para comprobar el estado de mi repositorio local:

``` 
git status
``` 


