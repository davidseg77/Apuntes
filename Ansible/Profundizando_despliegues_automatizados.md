# Info extraida del taller Profundizando en los despliegues automatizados con Ansible

## 1. Apunte sobre variables

Entre otras cuestiones dentro de la estructura de un playbook, vamos a añadir las variables:

* Condiciones: Asigna un valor a una variable y ayuda a trabajar en base a una o una serie de condiciones.

``` 
- name: Install Java
  yum:
    name: jave
  when: install_java == "yes"
```

* Globales: Variables generales para todos los roles y grupos de host. Jerárquicamente al mismo nivel que el main, inventory o roles.

``` 
---

user: david

services_directory: /etc/systemd/system
service_name: webserver.service
storage_dir: /home/openwebinars/web

port: 8000
```

* Defaults: Se definen dentro de cada rol y solo son accesibles desde el rol en que se definan. Su prioridad es mayor que las variables globales.

``` 
---

zeppelin_version: 0.7.3
zeppelin_download_url: http://archive.apache.org/dist/zeppelin/zeppelin-{{ zeppelin_version }}
zeppelin_service_username: zeppelin
zeppelin_service_group: zeppelin
zeppelin_working_directory: /tmp/zeppelin_working_directory
```

## 2. Caso práctico

Dentro de un servidor de Ubuntu, creamos un directorio para almacenar nuestros archivos. Vamos a ese directorio y creamos un main.yml. Del mismo modo, creamos un directorio para inventory, otro para group_vars y otro para roles.

**Hosts**

Dentro de inventory creamos un archivo llamado hosts:

``` 
[webserver]
IP del servidor remoto donde instalaremos el servidor web Nginx

[mongodb]
IP del servidor donde vamos a instalar Mongo.
```

Dentro de roles creamos un directorio webserver_install, otro para webserver_run y dos iguales para mongodb.

**Iniciación del servidor web y de la base de datos**

Vamos a webserver_install y creamos dos directorios: tasks y otro para templates. Vamos a tasks y creamos un main.yml:

``` 
---

- name: Install nginx
  apt: name=nginx state=latest
  become: yes

- name: Copy nginx html template
  template: 
    src: index.nginx-debian.html.j2
    dest: "/var/www/html/index.nginx-debian.html"
    owner: "{{ ssh_user }}"
    group: "{{ ssh_user }}"

- name: Add one line with LineInFile to nginx html file
  lineinfile: dest=/var/www/html/index.nginx-debian.html line="<h2>Bloque de LineInFile</h2>" insertafter="Bienvenidos"
  become: yes

- name: Add one block with BlockInFile to nginx html file
  blockinfile:
    dest: "/var/www/html/index.nginx-debian.html"
    marker: "<!--- {mark} ANSIBLE MANAGED BLOCK NGINX --->
    insertafter: "Bienvenidos"
    block: 
      <h2>Bloque de BlockInFile</h2>
  become: yes
```

- Dentro del directorio template del webserver creamos el archivo index con un simple mensaje en h1.

Ya tenemos la tarea creada para el servidor web. Ahora habria que configurar como arrancar Nginx. Para ello, dentro de webserver_run creamos un directorio tasks y dentro un main.yml:

``` 
---

- name: Start nginx
  service:
    name: nginx
    state: started
  become: yes
```

Con esto arrancaría el servidor web.

Acto seguido, iríamos a mongodb_install, crearíamos un directorio para tasks y dentro un main.yml:

``` 
---

- name: Upgrade all packages to latest version
  apt: 
    name: "*"
    state: latest
  become: yes

- name: Install MongoDB
  apt: name=mongodb state=latest
  become: yes
```

- La configuración de MongoDB se haría de una manera similar que como se hizo con Nginx. 

Quedaría ir a mongodb_run y dentro del directorio tasks crear el main.yml:

``` 
---

- name: Start MongoDB
  service:
    name: mongodb
    state: started
  become: yes
```

**Variables**

Vamos a group_vars y definimos las variables en un archivo (all)

``` 
---

ssh_user: usuario del servidor remoto
```

**Configurar el main principal**

``` 

- hosts: webserver
  remote_user: "{{ ssh_user }}"
  become: True
  serial: "100%"
  roles:
    - role: webserver_install
    - role: webserver_run

- hosts: mongodb
  remote_user: "{{ ssh_user }}"
  become: True
  serial: "100%"
  roles:
    - role: mongodb_install
    - role: mongodb_run   
```

Y lanzamos el despliegue:

``` 
ansible-playbook --ask-become-pass -i inventory/hosts main.yml
```

Damos la contraseña y se desplegará la aplicación. Si vamos al navegador comprobaremos cómo se muestra el mensaje requerido.

Si voy vía ssh al usuario remoto, y accedo al archivo index, podré ver como está creado correctamente. 