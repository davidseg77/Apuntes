# Cómo utilizar Ansible para instalar y configurar Docker en Ubuntu 22.04

## 1. Introducción

La automatización de servidores desempeña ahora un papel esencial en la administración de sistemas, debido a la naturaleza desechable de los entornos de aplicaciones modernos. Las herramientas de gestión de configuración, como Ansible , se utilizan normalmente para agilizar el proceso de automatización de la configuración del servidor mediante el establecimiento de procedimientos estándar para nuevos servidores y al mismo tiempo reducir el error humano asociado con las configuraciones manuales.

Ansible ofrece una arquitectura simple que no requiere la instalación de software especial en los nodos. También proporciona un sólido conjunto de funciones y módulos integrados que facilitan la escritura de scripts de automatización.

## 2. Requisitos previos

Para ejecutar la configuración automatizada proporcionada en el manual de esta guía, necesitará:

* Un nodo de control de Ansible: una máquina Ubuntu 22.04 con Ansible instalado y configurado para conectarse a sus hosts de Ansible mediante claves SSH. Asegúrese de que el nodo de control tenga un usuario normal con permisos sudo y un firewall habilitado.
   
* Uno o más Ansible Hosts: uno o más servidores remotos de Ubuntu 22.04 configurados previamente.

Antes de continuar, primero debe asegurarse de que su nodo de control de Ansible pueda conectarse y ejecutar comandos en su(s) host(s) de Ansible. 

## 3. Configuración inicial del servidor con Ubuntu 22.04

### 3.1: iniciar sesión como root

Para iniciar sesión en su servidor, necesitará conocer la dirección IP pública de su servidor. También necesitará la contraseña o la clave privada de la cuenta del usuario root si instaló una clave SSH para la autenticación.

Si no está conectado a su servidor actualmente, inicie sesión como usuario root usando el siguiente comando. Sustituya la parte resaltada your_server_ip del comando con la dirección IP pública de su servidor:

``` 
ssh root@your_server_ip
``` 

Acepte la advertencia sobre la autenticidad del host si aparece. Si su servidor usa autenticación de contraseña, proporcione su contraseña de root para iniciar sesión. Si usa una clave SSH que está protegida con una frase de contraseña, es posible que deba ingresar la frase de contraseña la primera vez que use la clave en cada sesión. Si es la primera vez que inicia sesión en el servidor con una contraseña, es posible que también deba cambiar la contraseña de root . Siga las instrucciones para cambiar la contraseña si recibe un mensaje.

### 3.2: Crear un nuevo usuario

Una vez que inicie sesión como root, podrá agregar la nueva cuenta de usuario. En el futuro, iniciaremos sesión con esta nueva cuenta en lugar de root.

Este ejemplo crea un nuevo usuario llamado sammy, pero debes reemplazarlo con un nombre de usuario que te guste:

``` 
adduser sammy
``` 

Se le harán algunas preguntas, comenzando con la contraseña de la cuenta.

Ingrese una contraseña segura y, opcionalmente, complete cualquier información adicional si lo desea. Esta información no es obligatoria y puede presionar ENTER en cualquier campo que desee omitir.

### 3.3: Otorgar privilegios administrativos

Ahora tiene una nueva cuenta de usuario con privilegios de cuenta habituales. Sin embargo, en ocasiones necesitarás realizar tareas administrativas como usuario root .

Para evitar cerrar la sesión de su usuario habitual y volver a iniciarla como cuenta raíz, puede configurar lo que se conoce como privilegios de superusuario o raíz para la cuenta habitual de su usuario. Estos privilegios permitirán a su usuario normal ejecutar comandos con privilegios administrativos poniendo la palabra sudoantes del comando.

Para agregar estos privilegios a su nuevo usuario, deberá agregar el usuario al grupo del sistema sudo. De forma predeterminada en Ubuntu 22.04, los usuarios que son miembros del grupo sudo pueden usar el sudo comando.

Como root, ejecute este comando para agregar su nuevo usuario al grupo sudo (sustituya el sammy nombre de usuario resaltado con su nuevo usuario):

``` 
usermod -aG sudo sammy
``` 

Ahora puede escribir sudo comandos anteriores para ejecutarlos con privilegios de superusuario cuando inicie sesión como usuario habitual.

### 3.4: configurar un firewall

Los servidores Ubuntu 22.04 pueden usar el firewall UFW para garantizar que solo se permitan conexiones a ciertos servicios. Puede configurar un firewall básico usando esta aplicación.

Las aplicaciones pueden registrar sus perfiles con UFW tras la instalación. Estos perfiles permiten a UFW administrar estas aplicaciones por nombre. OpenSSH, el servicio que te permite conectarte a tu servidor, tiene un perfil registrado en UFW.

Puede examinar la lista de perfiles UFW instalados escribiendo:

``` 
ufw app list
Output
Available applications:
  OpenSSH
``` 

Deberá asegurarse de que el firewall permita conexiones SSH para poder iniciar sesión en su servidor la próxima vez. Permita estas conexiones escribiendo:

``` 
ufw allow OpenSSH
``` 

Ahora habilite el firewall escribiendo:

``` 
ufw enable
``` 
Escribe y y presiona ENTER para continuar. Puedes ver que las conexiones SSH todavía están permitidas escribiendo:

``` 
ufw status
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
``` 

Actualmente, el firewall está bloqueando todas las conexiones excepto SSH. Si instala y configura servicios adicionales, deberá ajustar la configuración del firewall para permitir el nuevo tráfico en su servidor.

### 3.5: Habilitar el acceso externo para su usuario habitual

Ahora que tiene un usuario habitual para uso diario, deberá asegurarse de poder acceder mediante SSH a la cuenta directamente.

La configuración del acceso SSH para su nuevo usuario depende de si la cuenta raíz de su servidor utiliza una contraseña o claves SSH para la autenticación.

* Si la cuenta raíz utiliza autenticación de contraseña

Si inició sesión en su cuenta raíz con una contraseña, la autenticación de contraseña está habilitada para SSH. Puede acceder mediante SSH a su nueva cuenta de usuario abriendo una nueva sesión de terminal y usando SSH con su nuevo nombre de usuario:

``` 
ssh sammy@your_server_ip
``` 

Después de ingresar su contraseña de usuario habitual, iniciará sesión. Recuerde, si necesita ejecutar un comando con privilegios administrativos, escriba sudo antes de esto:

``` 
sudo command_to_run
```

Recibirá un mensaje para su contraseña de usuario habitual cuando la utilice sudo por primera vez en cada sesión (y periódicamente después).

Para mejorar la seguridad de su servidor, recomendamos encarecidamente configurar claves SSH en lugar de utilizar la autenticación con contraseña. 

* Si la cuenta raíz utiliza autenticación de clave SSH
  
Si inició sesión en su cuenta raíz usando claves SSH, la autenticación de contraseña está deshabilitada para SSH. Para iniciar sesión como su usuario habitual con una clave SSH, debe agregar una copia de su clave pública local al ~/.ssh/authorized_keys archivo de su nuevo usuario.

Dado que su clave pública ya está en el archivo de la cuenta raíz ~/.ssh/authorized_keys en el servidor, puede copiar ese archivo y estructura de directorio a su nueva cuenta de usuario usando su sesión actual.

La forma más sencilla de copiar los archivos con la propiedad y los permisos correctos es con el rsync comando. Este comando copiará el directorio del usuario raíz.ssh, preservará los permisos y modificará los propietarios de los archivos, todo en un solo comando. Asegúrese de cambiar las partes resaltadas del siguiente comando para que coincidan con el nombre de su usuario habitual:

``` 
rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy
```

Ahora, abra una nueva sesión de terminal en su máquina local y use SSH con su nuevo nombre de usuario:

```
ssh sammy@your_server_ip
``` 

Deberías estar conectado a tu servidor con la nueva cuenta de usuario sin usar una contraseña. Recuerde, si necesita ejecutar un comando con privilegios administrativos, escriba sudo antes del comando así:

``` 
sudo command_to_run
``` 

Se le solicitará su contraseña de usuario habitual cuando la utilice sudopor primera vez en cada sesión (y periódicamente después).

## 4. Cómo instalar y configurar Ansible en Ubuntu 22.04

### 4.1: instalar Ansible

Para comenzar a utilizar Ansible como medio para administrar la infraestructura de su servidor, debe instalar el software Ansible en la máquina que servirá como nodo de control de Ansible.

Desde su nodo de control, ejecute el siguiente comando para incluir el PPA (archivo de paquete personal) del proyecto oficial en la lista de fuentes de su sistema:

```
sudo apt-add-repository ppa:ansible/ansible
``` 

Presione ENTER cuando se le solicite aceptar la adición del PPA.

A continuación, actualice el índice de paquetes de su sistema para que conozca los paquetes disponibles en el PPA recién incluido:

``` 
sudo apt update
```

Después de esta actualización, puede instalar el software Ansible con:

``` 
sudo apt install ansible
``` 

Su nodo de control de Ansible ahora tiene todo el software necesario para administrar sus hosts. A continuación, veremos cómo agregar sus hosts al archivo de inventario del nodo de control para que pueda controlarlos.

### 4.2: configurar el archivo de inventario

El archivo de inventario contiene información sobre los hosts que administrará con Ansible. Puede incluir desde uno hasta varios cientos de servidores en su archivo de inventario, y los hosts se pueden organizar en grupos y subgrupos. El archivo de inventario también se usa a menudo para establecer variables que serán válidas solo para hosts o grupos específicos, para poder usarse dentro de guías y plantillas. Algunas variables también pueden afectar la forma en que se ejecuta un libro de jugadas, como la ansible_python_interpreter variable que veremos en un momento.

Para editar el contenido de su inventario de Ansible predeterminado, abra el /etc/ansible/hosts archivo usando el editor de texto de su elección, en su nodo de control de Ansible:

``` 
sudo nano /etc/ansible/hosts
```

El archivo de inventario predeterminado proporcionado por la instalación de Ansible contiene una serie de ejemplos que puede utilizar como referencia para configurar su inventario. El siguiente ejemplo define un grupo [servers] con tres servidores diferentes, cada uno identificado por un alias personalizado: servidor1, servidor2 y servidor3. Asegúrese de reemplazar las IP resaltadas con las direcciones IP de sus hosts Ansible.

``` 
/etc/ansible/hosts
[servers]
server1 ansible_host=203.0.113.111
server2 ansible_host=203.0.113.112
server3 ansible_host=203.0.113.113

[all:vars]
ansible_python_interpreter=/usr/bin/python3
``` 

El all:vars subgrupo establece el ansible_python_interpreter parámetro de host que será válido para todos los hosts incluidos en este inventario. Este parámetro garantiza que el servidor remoto utilice el /usr/bin/python3 ejecutable Python 3 en lugar de /usr/bin/python(Python 2.7), que no está presente en las versiones recientes de Ubuntu.

Cuando haya terminado, guarde y cierre el archivo presionando CTRL+X luego Y y ENTER para confirmar sus cambios.

Siempre que quieras comprobar tu inventario, puedes ejecutar:

``` 
ansible-inventory --list -y
``` 

Verá un resultado similar a este, pero que contiene su propia infraestructura de servidor tal como se define en su archivo de inventario:

``` 
Output
all:
  children:
    servers:
      hosts:
        server1:
          ansible_host: 203.0.113.111
          ansible_python_interpreter: /usr/bin/python3
        server2:
          ansible_host: 203.0.113.112
          ansible_python_interpreter: /usr/bin/python3
        server3:
          ansible_host: 203.0.113.113
          ansible_python_interpreter: /usr/bin/python3
    ungrouped: {}
``` 

Ahora que ha configurado su archivo de inventario, tiene todo lo que necesita para probar la conexión a sus hosts Ansible.

### 4.3: prueba de conexión

Después de configurar el archivo de inventario para incluir sus servidores, es hora de verificar si Ansible puede conectarse a estos servidores y ejecutar comandos a través de SSH.

Para esta guía, usaremos la cuenta raíz de Ubuntu porque suele ser la única cuenta disponible de forma predeterminada en servidores recién creados. Si sus hosts de Ansible ya tienen creado un usuario sudo normal, le recomendamos que utilice esa cuenta en su lugar.

Puede utilizar el -u argumento para especificar el usuario del sistema remoto. Cuando no se proporciona, Ansible intentará conectarse como su usuario actual del sistema en el nodo de control.

Desde su máquina local o nodo de control de Ansible, ejecute:

``` 
ansible all -m ping -u root
``` 

ping. Este comando utilizará el módulo integrado de Ansible para ejecutar una prueba de conectividad en todos los nodos de su inventario predeterminado, conectándose como root. El ping módulo probará:

* si los anfitriones son accesibles;
* si tiene credenciales SSH válidas;
* si los hosts pueden ejecutar módulos de Ansible usando Python.
  
Deberías obtener un resultado similar a este:

``` 
Output
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
server2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
server3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Si es la primera vez que se conecta a estos servidores a través de SSH, se le pedirá que confirme la autenticidad de los hosts a los que se conecta a través de Ansible. Cuando se le solicite, escriba yes y luego presione ENTER para confirmar.

Una vez que reciba una "pong" respuesta de un host, significa que está listo para ejecutar comandos y guías de Ansible en ese servidor.

### 4.4: Ejecutar comandos ad-hoc (opcional)

Después de confirmar que su nodo de control de Ansible puede comunicarse con sus hosts, puede comenzar a ejecutar comandos ad hoc y guías en sus servidores.

Cualquier comando que normalmente ejecutaría en un servidor remoto a través de SSH se puede ejecutar con Ansible en los servidores especificados en su archivo de inventario. Como ejemplo, puede verificar el uso del disco en todos los servidores con:

``` 
ansible all -a "df -h" -u root
``` 

```
Output

server1 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           798M  624K  798M   1% /run
/dev/vda1       155G  2.3G  153G   2% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vda15      105M  3.6M  101M   4% /boot/efi
tmpfs           798M     0  798M   0% /run/user/0

server2 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           395M  608K  394M   1% /run
/dev/vda1        78G  2.2G   76G   3% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/vda15      105M  3.6M  101M   4% /boot/efi
tmpfs           395M     0  395M   0% /run/user/0

...
``` 

El comando resaltado df -h se puede reemplazar por cualquier comando que desee.

También puede ejecutar módulos de Ansible mediante comandos ad-hoc, de manera similar a lo que hemos hecho antes con el ping módulo para probar la conexión. Por ejemplo, así es como podemos usar el apt módulo para instalar la última versión en vimtodos los servidores de su inventario:

``` 
ansible all -m apt -a "name=vim state=latest" -u root
``` 

También puede apuntar a hosts individuales, así como a grupos y subgrupos, al ejecutar comandos de Ansible. Por ejemplo, así es como comprobarías el estado uptime de cada host del servers grupo:

```
ansible servers -a "uptime" -u root
``` 

Podemos especificar múltiples hosts separándolos con dos puntos:

``` 
ansible server1:server2 -m ping -u root
``` 

## 5. Prepara tu libro de jugadas

El playbook.yml archivo es donde se definen todas sus tareas. Una tarea es la unidad de acción más pequeña que puede automatizar utilizando un manual de Ansible. Pero primero, crea tu archivo de libro de jugadas usando tu editor de texto preferido:

``` 
nano playbook.yml
``` 

Esto abrirá un archivo YAML vacío. Antes de sumergirse en agregar tareas a su libro de jugadas, comience agregando lo siguiente:

``` 
libro de jugadas.yml
---
- hosts: all
  become: true
  vars:
    container_count: 4
    default_container_name: docker
    default_container_image: ubuntu
    default_container_command: sleep 1
``` 

Casi todos los manuales con los que te encuentres comenzarán con declaraciones similares a esta. **hosts** declara a qué servidores se dirigirá el nodo de control de Ansible con este manual. **become** indica si todos los comandos se realizarán con privilegios de root escalados. **vars** permite almacenar datos en variables. Si decide cambiarlos en el futuro, solo tendrá que editar estas líneas individuales en su archivo. Aquí hay una breve explicación de cada variable:

* container_count: El número de contenedores a crear.
* default_container_name: Nombre del contenedor predeterminado.
* default_container_image: Imagen de Docker predeterminada que se utilizará al crear contenedores.
* default_container_command: Comando predeterminado para ejecutar en contenedores nuevos.

## 6. Agregar tareas de instalación de paquetes a su Playbook

De forma predeterminada, Ansible ejecuta las tareas de forma sincrónica en orden de arriba a abajo en su libro de estrategias. Esto significa que el orden de las tareas es importante y puede asumir con seguridad que una tarea terminará de ejecutarse antes de que comience la siguiente.

Todas las tareas de este manual pueden ser independientes y reutilizarse en sus otros manuales.

Agregue sus primeras tareas de instalación aptitude, una herramienta para interactuar con el administrador de paquetes de Linux, e instalar los paquetes del sistema necesarios. Ansible se asegurará de que estos paquetes estén siempre instalados en su servidor:

``` 
libro de jugadas.yml
  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true
``` 

Aquí, está utilizando el módulo apt integrado de Ansible para indicarle a Ansible que instale sus paquetes. Los módulos en Ansible son atajos para ejecutar operaciones que de otro modo tendría que ejecutar como comandos bash sin formato. Ansible recurre de forma segura a la instalación de paquetes si no está disponible, pero Ansible históricamente ha preferido .apt aptitude aptitude

Puedes agregar o quitar paquetes a tu gusto. Esto garantizará que todos los paquetes no solo estén presentes, sino que también tengan la última versión y se realicen después de aptllamar a una actualización.

## 7. Agregar tareas de instalación de Docker a su Playbook

Su tarea instalará la última versión de Docker desde el repositorio oficial. Se agrega la clave Docker GPG para verificar la descarga, se agrega el repositorio oficial como una nueva fuente de paquete y se instalará Docker. Además, también se instalará el módulo Docker para Python:

``` 
libro de jugadas.yml
    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu jammy stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Install Docker Module for Python
      pip:
        name: docker
``` 

Verá que los apt_key módulos apt_repository Ansible integrados primero apuntan a las URL correctas y luego se les asigna la tarea de garantizar que estén presentes. Esto permite la instalación de la última versión de Docker, además de pip instalar el módulo para Python.

## 8. Agregar tareas de contenedores e imágenes de Docker a su Playbook

La creación real de sus contenedores Docker comienza aquí con la extracción de la imagen Docker deseada. De forma predeterminada, estas imágenes provienen del Docker Hub oficial . Usando esta imagen, los contenedores se crearán de acuerdo con las especificaciones establecidas por las variables declaradas en la parte superior de su manual:

``` 
libro de jugadas.yml
    - name: Pull default Docker image
      community.docker.docker_image:
        name: "{{ default_container_image }}"
        source: pull

    - name: Create default containers
      community.docker.docker_container:
        name: "{{ default_container_name }}{{ item }}"
        image: "{{ default_container_image }}"
        command: "{{ default_container_command }}"
        state: present
      with_sequence: count={{ container_count }}
``` 

**docker_image** se utiliza para extraer la imagen de Docker que desea utilizar como base para sus contenedores. **docker_container** le permite especificar los detalles de los contenedores que crea, junto con el comando que desea pasarles.

**with_sequence** es la forma Ansible de crear un bucle y, en este caso, repetirá la creación de sus contenedores de acuerdo con el recuento que especificó. Este es un ciclo de conteo básico, por lo que la item variable aquí proporciona un número que representa la iteración del ciclo actual. Este número se utiliza aquí para nombrar sus contenedores.

## 9. Revisar su manual completo

Su manual debería verse más o menos como el siguiente, con pequeñas diferencias dependiendo de sus personalizaciones:

``` 
libro de jugadas.yml
---
- hosts: all
  become: true
  vars:
    container_count: 4
    default_container_name: docker
    default_container_image: ubuntu
    default_container_command: sleep 1d

  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu jammy stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Install Docker Module for Python
      pip:
        name: docker

    - name: Pull default Docker image
      community.docker.docker_image:
        name: "{{ default_container_image }}"
        source: pull

    - name: Create default containers
      community.docker.docker_container:
        name: "{{ default_container_name }}{{ item }}"
        image: "{{ default_container_image }}"
        command: "{{ default_container_command }}"
        state: present
      with_sequence: count={{ container_count }}
``` 

No dude en modificar este manual para adaptarlo mejor a sus necesidades individuales dentro de su propio flujo de trabajo. Por ejemplo, puede utilizar el módulo docker_image para enviar imágenes a Docker Hub o el módulo docker_container para configurar redes de contenedores.

Una vez que esté satisfecho con su libro de jugadas, puede salir de su editor de texto y guardar.

## 10. Ejecutar su libro de jugadas

Ahora está listo para ejecutar este manual en uno o más servidores. La mayoría de los playbooks están configurados para ejecutarse en todos los servidores de su inventario de forma predeterminada, pero esta vez especificará su servidor.

Para ejecutar el libro de jugadas solo en server1, conectándose como sammy, puede usar el siguiente comando:

``` 
ansible-playbook playbook.yml -l server1 -u sammy
``` 

La -l bandera especifica su servidor y la -u bandera especifica en qué usuario iniciar sesión en el servidor remoto. Obtendrá un resultado similar a este:

``` 
Output
. . .
changed: [server1]

TASK [Create default containers] *****************************************************************************************************************
changed: [server1] => (item=1)
changed: [server1] => (item=2)
changed: [server1] => (item=3)
changed: [server1] => (item=4)

PLAY RECAP ***************************************************************************************************************************************
server1              	: ok=9	changed=8	unreachable=0	failed=0	skipped=0	rescued=0	ignored=0  
``` 

¡Esto indica que la configuración de su servidor está completa! Su resultado no tiene que ser exactamente el mismo, pero es importante que no tenga fallas.

Cuando el libro de estrategias termine de ejecutarse, inicie sesión mediante SSH en el servidor proporcionado por Ansible para verificar si los contenedores se crearon correctamente.

Inicie sesión en el servidor remoto con:

``` 
ssh sammy@your_remote_server_ip
``` 

Y enumere sus contenedores Docker en el servidor remoto:

``` 
sudo docker ps -a
``` 

Deberías ver un resultado similar a este:

``` 
Output
CONTAINER ID    	IMAGE           	COMMAND         	CREATED         	STATUS          	PORTS           	NAMES
a3fe9bfb89cf    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker4
8799c16cde1e    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker3
ad0c2123b183    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker2
b9350916ffd8    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker1
``` 

Esto significa que los contenedores definidos en el libro de estrategias se crearon correctamente. Dado que esta fue la última tarea del libro de jugadas, también confirma que el libro de jugadas se ejecutó por completo en este servidor.

## 11. Conclusión

Automatizar la configuración de su infraestructura no solo puede ahorrarle tiempo, sino que también ayuda a garantizar que sus servidores sigan una configuración estándar que pueda personalizarse según sus necesidades. 

Con la naturaleza distribuida de las aplicaciones modernas y la necesidad de coherencia entre diferentes entornos de preparación, una automatización como esta se ha convertido en un componente central en los procesos de desarrollo de muchos equipos.



