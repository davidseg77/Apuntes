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






