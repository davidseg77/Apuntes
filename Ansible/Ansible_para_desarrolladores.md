# Curso de Ansible

## 1. Instalación de LXC y LXD en Ubuntu 22.04

Dentro de la terminal:

``` 
sudo snap install lxd
```

Y decimos dentro de las preguntas que si a configurar new storage pool, le damos nombre. Nombre del almacenamiento backend a usar será zfs, creamos el pool ZFS, decimos que no a usar un dispositivo de bloque de almacenamiento. Damos tamaño amplio, por ejemplo 30GiB. Decimos que si a crear nueva red local y damos nombre.

Para las IP address, pasamos adelante, decimos que no a que el servidor LXD actue sobre la red y sí a que cachee imagenes para actualizar automáticamente. Por último, decimos no a que nos genere un yaml con la configuración escogida. 

Con todo esto, ya tendríamos una máquina creada con LXD.

Para saber si tenemos algo arrancado:

```
lxc list
```

Para ver las imagenes que tiene preestablecidas por defecto:

``` 
lxc image list images:
```

Y lanzamos nuestro contenedor:

``` 
lxc launch ubuntu:22.04 + nombre del contenedor que queremos crear
``` 

Comprobamos de nuevo con **lxc list**.

Para detener, arrancar o eliminar el contenedor:

```
lxc stop contenedor
lxc start contenedor
lxc rm contenedor
```

Para acceder al contenedor:

``` 
lxc exec contenedor -- /bin/bash
``` 

Y ahí ya podemos introducir comandos.

## 2. Configuración de ansible

Dentro del servidor, instalamos Ansible:

``` 
sudo apt install ansible
```

Y vemos su versión:

``` 
ansible --version
```

También podriamos instalar la paquetería pip de python:

```
sudo apt install python3-pip
```

E instalar Ansible a través de pip:

``` 
pip3 install ansible
```

## 3. Creación de claves SSH

```
ssh-keygen -t ed25519 -C "email"
```

Cargamos el agente ssh:

``` 
eval "$(ssh-agent -s)"
```

Y la clave privada:

``` 
ssh-add /home/david/.ssh/id_ed25519
```

Esta clave podemos pasarla a AWS, GC, a Github...

## 4. Cómo configurar nodo manejado accesible con clave SSH

Accedemos al contenedor y creamos un usuario:

``` 
adduser david
```

Y lo añadimos al grupo sudo

```
usermod -aG sudo david
```

Ahora tendríamos un usuario con privilegios sudo.

Pasamos a él: **su - david**

Instalamos SSH:

```
apt install openssh-server
```

Ahora vamos al usuario root, y de ahí al archivo de configuración de ssh:

```
nano /etc/ssh/sshd_config
```

Descomentamos PermitRootLogin y lo ponemos a no. Hacemos lo propio con PasswordAuthentication y lo ponemos en yes.

Reiniciamos y habilitamos el servicio ssh:

``` 
systemctl restart ssh
systemctl enable ssh
```

Copiamos las claves públicas:

``` 
ssh-copy-id -i /home/david/.ssh/id_ed25519.pub david@ip
```

Para comprobarlo en ese usuario david:

``` 
cat .ssh/authorized_keys
```

Ahora ponemos de nuevo en el archivo de configuración de ssh el Password Authentication en no. Habría que volver a reiniciar el servicio ssh de nuevo. 

Comprobamos haciendo ssh a ese nodo del usuario david (ssh david@ip)

Si queremos, podemos publicar una imagen de este contenedor con las claves ssh ya creadas:

``` 
lxc publish nombrecontenedor --alias=nombre alias description="Ubuntu 22 04 con clave ssh integrada y servidor ssh"
```

Y creamos dos instancias en base a esta imagen:

``` 
lxc launch alias u1
lxc launch alias u2
```

Comprobamos con **lxc list**.

Y accedemos vía ssh a cualquiera de ellos:

```
ssh -l david -i /home/david/.ssh/id_ed25519 ip_de_contenedor
```

¿Y si quiero acceder a traves de host? Vamos a su archivo:

```
sudo nano /etc/hosts
```

y metemos la ip de cada contenedor con su nombre:

```
ip u1
ip u2
```

Y accedemos por ssh:

``` 
ssh david@u1
```

## 5. Creación de Inventarios

Ejemplo del archivo de inventario (inventory.yaml):

```
all:
  hosts:
    u1:
      ansible_host: 192.168.1.10
    u2:
      ansible_host: 192.168.1.11

ubuntu:
  hosts:
    u1:
    u2:

webservers:
  hosts:
    u1:
    u2:
```

Si queremos probar la sintaxis del fichero podemos usar:

```
ansible-inventory -i inventory --list
```

Dentro de este mismo directorio también tendría el de configuración de ansible (ansible.cfg)

```
[defaults]
inventory = ./inventory-dns
```

Y otro de tipo inventario para dns (inventory-dns):

```
u1
u2
```

Y probamos lanzando nuestro primer comando, comprobando si están presentes los servidores:

``` 
ansible all -i inventory -m ping
```

## 6. Grupos de Máquina en Inventarios y Privilegios

Creamos un directorio donde tenemos los siguientes archivos:

* Archivo de configuración de Ansible (ansible.cfg):

``` 
[defaults]
inventory = ./inventory-dns
[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=True
``` 

Cuando trabajamos en entornos automatizados, estos parámetros no son necesarios. En este caso se incluyen para trabajar de manera manual.

* Un archivo inventory:

```
[webservers]
10.218.144.170
[ubuntu]
10.218.144.233
```

* Un inventory-dns

``` 
[webservers]
u1
u2
```

* inventory-groups

``` 
[all:children]
ubuntu
webservers

[ubuntu]
u1 ansible_host=192.168.1.10
u2 ansible_host=192.168.1.11
[webservers]
u1 ansible_host=192.168.1.10
u2 ansible_host=192.168.1.11
```

* inventory-groups.yaml

```
all:
  children:
    group1:
      hosts:
        u1:
          ansible_host: 192.168.1.10
        u22:
          ansible_host: 192.168.1.11
    group2:
      hosts:
        u3:
          ansible_host: 192.168.1.12
    group3:
      children:
        - group1
        - group2
```

El archivo yaml de inventory sería el mismo que vimos en el punto anterior. Dicho esto, probamos la conectividad bien general o por grupo.

``` 
ansible all -m ping
ansible webservers -m ping
ansible ubuntu -m ping
```

## 7. Uso de Modo Adhoc

Veamoslo a través de este archivo donde se insertan una serie de comandos ad-hoc (adhoc.sh):

```
#!/bin/bash
# mostrando el fichero de config
cat ansible.cfg
# mostrando el intenrory dns
cat inventory-dns
# ejecutando el módulo ping
ansible all -m ping
#ejecutando el módulo copy
ansible all -m copy -a "dest=/tmp/hosts src=/etc/hosts"
# ejercutando el módulo file
ansible all -m file -a "state=absent path=/tmp/hosts"
```

## 8. Uso de playbooks

Se efectuan mediante ficheros YAML. Si queremos comprobar la sintaxis de un fichero YAML podemos hacer uso del comando:

``` 
ansible-playbook -syntax-check playbook.yaml
```

Para ejecutarlo:

``` 
ansible-playbook playbook.yaml
```

O para ejecutarlo paso a paso:

``` 
ansible-playbook --step fichero.yaml
```

O en modo comprobación:

``` 
ansible-playbook -C fichero.yaml
```

**Caso práctico**

En este ejemplo intentaré instalar y mantener un servidor web en una máquina. 

Para ello creo el siguiente fichero yaml (tasks.yaml):

``` 
---
- name: Instalación de Nginx
  hosts: webservers
  #remote_user: root
  tasks:
    - name: Instalando el paquete nginx
      package:
        name: nginx
        state: latest
        update_cache: true
    - name: Servicio Nginx arrancado
      service: #"name=nginx state=started enabled=true"
        name: nginx
        state: started
        enabled: true
...
```

En este caso, como estoy utilizando Ubuntu, voy a usar ufw para gestionar el tráfico. Lo habilito y establezco la política por defecto allow. Por lo tanto, añado las reglas oportunas al yaml:

``` 
---
- name: Instalación de Nginx
  hosts: webservers
  #remote_user: root
  tasks:
    - name: Instalando el paquete nginx
      package:
        name: nginx
        state: latest
        update_cache: yes
    - name: Servicio Nginx arrancado
      service: #"name=nginx state=started enabled=true"
        name: nginx
        state: started
        enabled: true
    - name: Habilitar UFW
      ufw:
        state: enabled
        policy: deny
    - name: Habilitar el log
      ufw:
        logging: 'on'
    - name: abrir el firewall para 22
      ufw:
        rule: allow
        port: "22"
        proto: tcp
    - name: abrir el firewall para 80
      ufw:
        rule: allow
        port: "80"
        proto: tcp
    - name: abrir el firewall para 443
      ufw:
        rule: allow
        port: "443"
        proto: tcp
    - name: Copia el fichero index.html
      copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'
...
```

En el caso de querer desintalar Nginx trás haber detenido el servicio, creamos el siguiente yaml (tasks_uninstall.yaml):

```
---
- name: Instalación de Nginx
  hosts: webservers
  #remote_user: root
  tasks:
    - name: Servicio Nginx apagado 
      service: #"name=nginx state=started enabled=true"
        name: nginx
        state: stopped
        enabled: false
    - name: Desinstalando el paquete nginx
      package:
        name: nginx
        state: absent
...
```

## 9. Despliegue Servidor Web Completo. Frontend

Tan solo habría que lanzar el tasks-completo.yaml. Si yo, por ejemplo, hago cambios de forma manual en el index.html, tendré que realizar el redespliegue del yaml para que esa info pase a todos los nodos de la web.

## 10. Uso de Variables en instalación de MariaDB

Dentro de este directorio tenemos el ansible.cfg:

```
[defaults]
inventory = ./inventory-dns
timeout=60
[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=True
```

Apunta al inventory-dns. Lo vemos:

```
[ubuntu]
u1
u2
u3
[webservers]
u1
u2
[databases]
u3
```

Vamos a instalar MariaDB dentro de nuestro Ubuntu 22.04. En primer lugar, creo el archivo con las variables a utilizar (vars.yaml):

```
mariadb_socket: /run/mysqld/mysqld.sock
mysql_root_password: test
database_name: test
database_user: test
database_password: test
mysql:
  privileges: "ALL"
```

Estos valores pueden codificarse mediante Ansible Vault, algo que se verá en siguientes apartados. 



A posteriori, creo el archivo tasks.yaml:

``` 
---
- name: Install Databases
  hosts: databases
  vars:
    - repo_software_packages:
        - software-properties-common
        - dirmngr
        - apt-transport-https
    - key_url: "https://mariadb.org/mariadb_release_signing_key.asc"
    - repo_deb: "deb [arch=amd64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_debug: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_max_scale: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/maxscale/latest/apt"
    - repo_deb_tools: "deb [arch=amd64] http://downloads.mariadb.com/Tools/ubuntu"
    - linux_distribution: Ubuntu
    - distribution_release: jammy
    - mariadb_packages:
      - mariadb-server
      - mariadb-common
      - python3-mysqldb
      - python3-openssl
  vars_files: # importación de variables desde fichero
    - vars.yaml

  tasks:
    - name: Update repositories
      apt: update_cache=yes
      ignore_errors: yes
    - name: Install Mariadb Requirements for {{ linux_distribution }} {{ distribution_release }}
      package:
        name: "{{ repo_software_packages }}"
        state: present
    - name: Add MariaDB Repository Key for {{ linux_distribution }} {{ distribution_release }}
      apt_key:
        url: "{{ key_url }}"
        state: present
    - name: Add MariaDB Repository for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb }} {{ distribution_release }}  main"
        state: present
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    - name: Add MariaDB Repository debug for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb_debug }} {{ distribution_release }}  main/debug"
        state: present
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    #- name: Add MariaDB Repository MaxScale for {{ linux_distribution }} {{ distribution_release }}
    #  apt_repository:
    #    repo: "{{ repo_deb_max_scale }} {{ distribution_release }}  main"
    #    state: present
    #    filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    #- name: Add MariaDB Repository Tools for {{ linux_distribution }} {{ distribution_release }}
    #  apt_repository:
    #    repo: "{{ repo_deb_tools }} {{ distribution_release }}  main"
    #    state: present
    #    filename: mariadb
    #    #register: addmariadbrepo
    #    #notify: Update repo cache   
    - name: Update repositories
      apt: update_cache=yes
      ignore_errors: yes
    - name: Install Mariadb server for {{ linux_distribution }} {{ distribution_release }}
      apt:
        name: "{{ mariadb_packages }}"
        state: present
    - name: Servicio arrancado
      service:
        name: mariadb
        state: started
        enabled: true
    - name: Habilitar UFW
      ufw:
        state: enabled
        policy: deny
    - name: Habilitar el log
      ufw:
        logging: 'on'
    - name: abrir el firewall para 22
      ufw:
        rule: allow
        port: "22"
        proto: tcp
    - name: abrir el firewall para 3306
      ufw:
        rule: allow
        port: "3306"
        proto: tcp
    - name: Set MariaDB root password for 127.0.0.1, localhost
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        host: "{{ item }}"
        login_user: root
        login_password: "{{ mysql_root_password }}"
        #login_unix_socket: "{{ mariadb_socket }}"
        state: present
      with_items:
        - 127.0.0.1
        - localhost
      #when: check_passwd_root.rc == 0
      #notify: Flush Priviliges
    - name: Remove all anonymous user
      mysql_user:
        login_user: root
        login_password: "{{ mysql_root_password }}"
        name: 'test'
        host_all: yes
        state: absent
      #notify: Flush Priviliges
    - name: Flush Priviliges
      command: mysql -u root -p{{ mysql_root_password }} -e "FLUSH PRIVILEGES"
    - name: Create Database
      mysql_db:
        name: "{{ database_name }}"
        login_user: root
        login_password: "{{ mysql_root_password }}"
        state: present

    - name: Create mysql User defined database
      mysql_user:
        name: "{{ database_user }}"
        password: "{{ database_password }}"
        priv: '{{ database_name }}.*:{{ mysql.privileges }}'
        login_user: root
        login_password: "{{ mysql_root_password }}"
        state: present
...
```

Para desinstalar la base de datos, vemos el archivo uninstall.yaml:

```
---
- name: Uninstall Databases
  hosts: databases
  vars:
    - repo_software_packages:
        - software-properties-common
        - dirmngr
        - apt-transport-https
    - key_url: "https://mariadb.org/mariadb_release_signing_key.asc"
    - repo_deb: "deb [arch=amd64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_debug: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_max_scale: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/maxscale/latest/apt"
    - repo_deb_tools: "deb [arch=amd64] http://downloads.mariadb.com/Tools/ubuntu"
    - linux_distribution: Ubuntu
    - distribution_release: jammy
    - mariadb_packages:
      - mariadb-server
      - mariadb-common
      - python3-mysqldb
      - python3-openssl
  vars_files: # importación de variables desde fichero
    - vars.yaml

  tasks:
    - name: Habilitar UFW
      ufw:
        state: enabled
        policy: deny
    - name: Habilitar el log
      ufw:
        logging: 'on'
    - name: abrir el firewall para 22
      ufw:
        rule: allow
        port: "22"
        proto: tcp
    - name: cerrar el firewall para 3306
      ufw:
        rule: deny
        port: "3306"
        proto: tcp
    - name: Servicio parado y quitado de arranque
      service:
        name: mariadb
        state: stopped
        enabled: false

    - name: Uninstall Mariadb server for {{ linux_distribution }} {{ distribution_release }}
      apt:
        name: "{{ mariadb_packages }}"
        state: absent
    - name: Uninstall Mariadb Requirements for {{ linux_distribution }} {{ distribution_release }}
      package:
        name: "{{ repo_software_packages }}"
        state: absent
    - name: Remove MariaDB Repository for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb }} {{ distribution_release }}  main"
        state: absent
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    - name: Remove MariaDB Repository debug for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb_debug }} {{ distribution_release }}  main/debug"
        state: absent
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    - name: Remove MariaDB Repository Key for {{ linux_distribution }} {{ distribution_release }}
      apt_key:
        url: "{{ key_url }}"
        state: absent  
  
    - name: Update repositories
      apt: update_cache=yes
      ignore_errors: yes
    
...
```

## 11. Facts

Muestran información detallada de los nodos, de un host. Para recopilar esta info, se puede usar el módulo **setup**. Para ver la información de un host:

``` 
ansible nombre_host -m setup -a ""
```

O filtrando por un valor:

``` 
ansible nombre_host -m setup -a "filter=distribution"
```

## 12. Handlers

Se utilizan para registrar y manejar eventos en Ansible. Para ello, dentro de un playbook insertaremos la propiedad notify. Y esto incluirá a aquello que entre en el archivo dentro de handlers. Por su parte, handlers va a la misma altura que vars, por ejemplo. Veamos un archivo como muestra donde se incluye el notify Flush Privileges y su correspondiente handler (tasks.yaml):

```
---
- hosts: databases
  vars:
    - repo_software_packages:
        - software-properties-common
        - dirmngr
        - apt-transport-https
    - key_url: "https://mariadb.org/mariadb_release_signing_key.asc"
    - repo_deb: "deb [arch=amd64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_debug: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/mariadb-server/10.9/repo/ubuntu"
    - repo_deb_max_scale: "deb [arch=amd64,arm64] https://dlm.mariadb.com/repo/maxscale/latest/apt"
    - repo_deb_tools: "deb [arch=amd64] http://downloads.mariadb.com/Tools/ubuntu"
    - linux_distribution: Ubuntu
    - distribution_release: jammy
    - mariadb_packages:
      - mariadb-server
      - mariadb-common
      - python3-mysqldb
      - python3-openssl
  vars_files: # importación de variables desde fichero
    - vars.yaml
  handlers:
    - name: Flush Priviliges
      command: mysql -u root -p{{ mysql_root_password }} -e "FLUSH PRIVILEGES"
  tasks:
    - name: Update repositories
      apt: update_cache=yes
      ignore_errors: yes
    - name: Install Mariadb Requirements for {{ linux_distribution }} {{ distribution_release }}
      package:
        name: "{{ repo_software_packages }}"
        state: present
    - name: Add MariaDB Repository Key for {{ linux_distribution }} {{ distribution_release }}
      apt_key:
        url: "{{ key_url }}"
        state: present
    - name: Add MariaDB Repository for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb }} {{ distribution_release }}  main"
        state: present
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    - name: Add MariaDB Repository debug for {{ linux_distribution }} {{ distribution_release }}
      apt_repository:
        repo: "{{ repo_deb_debug }} {{ distribution_release }}  main/debug"
        state: present
        filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    #- name: Add MariaDB Repository MaxScale for {{ linux_distribution }} {{ distribution_release }}
    #  apt_repository:
    #    repo: "{{ repo_deb_max_scale }} {{ distribution_release }}  main"
    #    state: present
    #    filename: mariadb
        #register: addmariadbrepo
        #notify: Update repo cache
    #- name: Add MariaDB Repository Tools for {{ linux_distribution }} {{ distribution_release }}
    #  apt_repository:
    #    repo: "{{ repo_deb_tools }} {{ distribution_release }}  main"
    #    state: present
    #    filename: mariadb
    #    #register: addmariadbrepo
    #    #notify: Update repo cache   
    - name: Update repositories
      apt: update_cache=yes
      ignore_errors: yes
    - name: Install Mariadb server for {{ linux_distribution }} {{ distribution_release }}
      apt:
        name: "{{ mariadb_packages }}"
        state: present
    - name: Servicio arrancado
      service:
        name: mariadb
        state: started
        enabled: true
    - name: Habilitar UFW
      ufw:
        state: enabled
        policy: deny
    - name: Habilitar el log
      ufw:
        logging: 'on'
    - name: abrir el firewall para 22
      ufw:
        rule: allow
        port: "22"
        proto: tcp
    - name: abrir el firewall para 3306
      ufw:
        rule: allow
        port: "3306"
        proto: tcp
    - name: Set MariaDB root password for 127.0.0.1, localhost
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        host: "{{ item }}"
        login_user: root
        login_password: "{{ mysql_root_password }}"
        #login_unix_socket: "{{ mariadb_socket }}"
        state: present
      with_items:
        - 127.0.0.1
        - localhost
      #when: check_passwd_root.rc == 0
      notify: Flush Priviliges
    
    - name: Remove all anonymous user
      mysql_user:
        login_user: root
        login_password: "{{ mysql_root_password }}"
        name: 'test'
        host_all: yes
        state: absent
      notify:
      - Flush Priviliges
    - name: Create Database
      mysql_db:
        name: "{{ database_name }}"
        login_user: root
        login_password: "{{ mysql_root_password }}"
        state: present
    - name: Create mysql User defined database
      mysql_user:
        name: "{{ database_user }}"
        password: "{{ database_password }}"
        priv: '{{ database_name }}.*:{{ mysql.privileges }}'
        login_user: root
        login_password: "{{ mysql_root_password }}"
        state: present
      notify:
      - Flush Priviliges
...
```

En este caso, actuará sobre los hosts de base de datos, que en nuestro inventario sería la u3. Después de instalar el servidor MariaDB, hará el flush privileges a modo de evento. 

A modo de resumen, primero se define el handler y luego la llamada a ese handler mediante notify cuando sea necesario. 

## 13. Import e include

Se utilizan para importar, por ejemplo, diferentes playbooks dentro de uno solo. Veamos un ejemplo (main.yaml):

``` 
---
- hosts: all
  tasks:
    - debug:
        msg: play1
- name: Deploy webservers
  import_playbook: tasks/webservers.yaml
- name: Deploy Databases
  import_playbook: tasks/databases.yaml
...
```

## 14. Condicionales

Como su propio nombre, son recursos que se introducen en un playbook cuando las tareas a ejecutar dependen de cierta condición. Se usa mediante la normativa when. Veamos un ejemplo:

```
---
- hosts: localhost
  connection: local
  tasks:
    - name: debug ansible_facts.os_family
      debug:
        msg: "OS Family: {{ ansible_facts.os_family }}"
    - name: ejecuta comando condicialmente
      shell: echo cosa >> fichero.txt
      when: ansible_facts['os_family'] == "Debian"
      register: result
    - name: debug result
      debug:
        var: result
        verbosity: 2
    #- name: debug ansible_facts.os_family
    #  debug:
    #    msg: "{{ ansible_facts }}"
...
```

En este ejemplo, queremos que la acción se lleve a cabo solo sobre aquellas máquinas de distribución Debian.


## 15. Módulos y argumentos

Los módulos son los que nos va a dar tareas que luego podemos integrar dentro de nuestros playbooks. 

Dentro de nuestro control disponemos de varias localizaciones para los módulos:

* **ANSIBLE_LIBRARY**: variable de entorno similar a PATH
* **~/.ansible/plugins/modules**
* **/usr/share/ansible/plugins/modules/**

Disponemos de un Cli que nos dará la info al respecto de cada módulo que es ansible doc:

``` 
ansible-doc nombre_modulo
```

Por ejemplo:

``` 
ansible-doc apt
```

## 16. Módulo File

Es el módulo integrado en Ansible más usado. Ejemplo de uso:

``` 
- name: Create a directory if it does not exist
    ansible.builtin.file:
     path:/var/www/html/wordpress
     state: directory
     mode: '0755'
```

``` 
-name: Update modification and access time of given file
   ansible.builtin.file:
    path:/var/www/html/data.html
    state:file
    modification_time: now
    access_time: now
```

O para cambiar el propietario de un directorio:

```
-name: Recursively change ownership of a directory
   ansible.builtin.file:
    path:/var/www/html
    state: directory
    recurse: yes
    owner: www-data
    group: www-data
```

O para eliminar fichero:

```
-name: Remove file (delete file)
   ansible.builtin.file:
    path:/var/www/html/wordpress/data.html
    state: absent
```

O directorio:

```
-name: Recursively remove directory
   ansible.builtin.file:
    path:/var/www/html/wordpress
    state: absent
```












