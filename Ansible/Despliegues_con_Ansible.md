# Info del curso de Despliegues con Ansible de OW

## 1. Instalación

Dentro de Linux, lo primero es generar el par de claves ssh:

```
ssh-keygen -t rsa -b 4096
``` 

Después copiamos la clave ssh generada a los servidores configurados para trabajar con Ansible:

``` 
ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.126.40.18 (Usuario e ip de ese servidor)
```

Y accedemos al servidor:

``` 
ssh usuario@ip
``` 

Ahora procedemos a instalar Ansible. Primero añadimos el repositorio:

```
sudo apt-add-repository ppa:ansible/ansible
``` 

Actualizamos el sistema:

``` 
sudo apt update -y
``` 

Y una vez actualizado, instalamos Ansible:

``` 
sudo apt install ansible -y
``` 

### 1.1 Comandos ad-hoc

Permite simultanear a todas las máquinas lo que se indica con un comando. Por ejemplo, para reiniciar todas las VM:

``` 
ansible all -a “/sbin/reboot” -i inventory/hosts.ini
``` 

* all hace referencia a todas las máquinas
* Entre comillas se indica el comando, la acción a realizar
* Se indica la ruta donde se aloja el archivo que concierne a las máquinas, a los hosts.

O para crear un fichero en todas las máquinas:

```
ansible all -a 'touch ~/Openwebinars/ansible/testFile' -i inventory/hosts.ini
``` 

## 2. Playbooks

### 2.1 Estructura

Elementos más importantes que la componen:

* Inventario: Conjunto de máquinas con las que va a trabajar Ansible

* Main: Archivo principal, donde se define que se va a ejecutar en cada host.

* Roles Nos permiten separar trabajos de un playbook y agruparlos con (variables, tareas, archivos, templates y módulos).

* Variables: Se utiliza para asignar valores a variables, donde posteriormente las podemos utilizar en nuestro Playbook sin necesidad de hardcodear parámetros

### 2.2 Gestión de ficheros

-    Copy: Copia un archivo desde la máquina local o remota a una ubicación en la máquina remota.
-    Template: Copia un archivo al servidor remoto y permite utilizar variables dentro del archivo.
-    File: Crear/Eliminar archivos, permisos de archivos, directorios o enlaces simbólicos.
-    LineInFile: Añade/reemplaza una línea particular en un archivo.
-    BlockInFile: Inserta, actualiza o elimina un bloque de texto de varias líneas en un archivo en la máquina remota.

## 3. Casos prácticos con Playbooks

### 3.1 Instalar y configurar un servidor web personalizado

En primer lugar, creo el directorio nginx. Dentro del directorio, creo un directorio para inventory y dentro de él el fichero de hosts.ini, que es donde vamos a configurar los servidores. En ese archivo deberemos insertar las ip de los servidores a configurar. 

``` 
[webserver]
ip servidor 1
ip servidor 2
``` 

El siguiente paso consistirá en crear el archivo principal, main.yml.

```
--- (Ha de comenzar con estos tres guiones al principio)

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
```

Después creamos una carpeta donde almacenar todo, llamada roles. Crearemos otra para nginx_install dentro de roles y otra llamada tasks para las tareas a ejecutar. Dentro del tasks creamos otro archivo main.yml con las siguientes funciones:

```
--- 

# Install Nginx 

- name: Update
  apt:
    name: "*"
    state: latest

- name: Install nginx
  apt: 
    name: "nginx"
    state: present

- name: Start and enable service nginx
  systemd:
    name: nginx
    state: started
    enabled: yes
```

Y con esto tendríamos ya la primera parte de nuestro proceso ya completo. Lanzamos el despliegue de nuestra aplicación haciendo uso del siguiente comando desde nuestro directorio nginx:

``` 
ansible-playbook -i inventory/hosts.ini main.yml 
``` 

* hosts.ini hace referencia al archivo donde tenemos los servidores en los cuales queremos desplegar la app.
* main.yml indica el archivo de referencia para el despliegue.

### 3.2 Registrando valores

Editamos el main.yml del directorio principal para añadir un nuevo rol de registro de variables:

```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
```

Para cada rol ha de crearse una nueva carpeta, así que creamos dentro del directorio roles una llamada register_variable. Dentro creamos otra carpeta para tasks y dentro un main.yml.

```
--- 

# Registrar variable

- name: Check if file exists
  stat:
    path: "/root/Openwebinars/ansible/motd"
  register: motd_file
```

Lo que estamos haciendo es cargar el archivo en la variable motd_file. Volvemos al directorio nginx y lanzamos de nuevo el comando de despliegue:

``` 
ansible-playbook -i inventory/hosts.ini main.yml 
```

### 3.3 Añadiendo condiciones

Una condición se utiliza para controlar si una tarea se tiene que ejecutar o se quiere omitir. Para nuestro caso práctico, vamos a añadir un tercer rol en el main.yml principal llamado condiciones:


```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
  - role: condiciones
```

Dentro del directorio roles creamos una carpeta condiciones y dentro una para tasks y dentro de tasks el main relativo a esta condición:

```
--- 

# Register variable

- name: Check if file exists
  stat:
    path: "/root/Openwebinars/ansible/motd"
  register: motd_file

# Debug 

- name: Debug if file exists
  debug: 
    msg: "File exists!"
  when: motd_file.stat.exists == True

- name: Debug if file not exists
  debug: 
    msg: "File not exists!"
  when: motd_file.stat.exists == False
```

La condición que hemos pasado es que si el fichero motd existe nos muestre True y de no ser así False. Vamos a modificarlo para que en caso de no existir nos lo cree:

```
--- 

# Register variable

- name: Check if file exists
  stat:
    path: "/root/Openwebinars/ansible/motd"
  register: motd_file

# Debug 

- name: Debug if file exists
  debug: 
    msg: "File exists!"
  when: motd_file.stat.exists == True

- name: Debug if file not exists
  debug: 
    msg: "File not exists!"
  when: motd_file.stat.exists == False

# Create file if not exists
- name: Create file
  file:
    path: "/root/Openwebinars/ansible/motd"
    state: touch
  when: motd_file.stat.exists == False
```

Y si lanzamos el comando de despliegue desde el directorio de nginx nos lo creará si el archivo no existe.

### 3.4 Failed when

Sirve para registrar si un fichero existe o no. En caso de que no existiera, nos dará fallo. Creamos un nuevo rol dentro del main.yml principal:

```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
  - role: condiciones
  - role: forzar_fallo
```

Como se ha hecho con cada rol, vamos a la carpeta roles y creamos una para forzar_fallo. Dentro de ella creamos una para tasks y dentro el main.yml de este rol en concreto:

```
--- 

# Forzar fallo

- name: Check if file exists and fail if not
  stat:
    path: "/root/Openwebinars/ansible/motd"
  register: motd_file
  failed_when: motd_file.stat.exists == False
```

En este caso no fallaría al realizar el despliegue porque el archivo existe, pero viene bien para asegurarnos de que el archivo está bien escrito, por ejemplo.

### 3.5 Ignore errors

Creamos un nuevo rol en el main principal:

```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
  - role: condiciones
  - role: ignorar_error
```

Hemos quitado o podríamos descomentar el rol de forzar fallo porque generaría conflicto. Creamos en roles una carpeta para ignorar_error y dentro la de tasks con el main.yml dentro.

```
--- 

# Ignorar errores

- name: Ignorar error
  command: /bin/false
  ignore_errors: yes
```

Mediante la variable ignore errors los errores serán ignorados en el despliegue. 

### 3.6 Creando templates

Creamos un nuevo rol dentro del main principal:

```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
  - role: condiciones
  - role: ignorar_error
  - role: variable_template
```

Dentro de roles, creamos la carpeta variable_template y dentro una carpeta para tasks y otra para templates. Configuramos el main del tasks:

```
--- 

# Copiar web nginx

- name: Copy nginx template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html

- name: Restart nginx
  systemd:
    name: nginx
    state: restarted
```

Sin embargo, el archivo indicado en ese main, el index.html.j2 aún no lo he definido. Lo creamos dentro del directorio de variable_template, fuera de tasks y template.

``` 
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Hello, Nginx!!</title>
</head>
<body>
  <h1>Hello, Nginx from {{ inventory_hostname }} our variable is: {{ variableOpenwebinars }}</h1>
  <p>We have configured Nginx on server {{ inventory_hostname}} and variable: {{ variableOpenwebinars }}</p>
  </body>
  </html>
```

* inventory_hostname es una variable que Ansible ya tiene por defecto y refiere a lo que he escrito en el archivo de inventory (hosts.ini, por ejemplo). Puede ser una ip, un dominio dns...

Hemos añadido una variable (variableOpenwebinars), que no ha sido declarada. Para ello, vamos a trabajar con las variables en grupo. Dentro de nginx, creamos un directorio llamado group_vars.

Dentro, creamos un archivo all, con lo cual decimos que la variable va a ser para todos los usuarios.

```
---

variableOpenwebinars: valor que queramos darle (por ej. davidseg)
``` 

Lanzamos el despliegue y nginx se instalará mostrandonos el hostname y el valor dado en la variable.


### 3.7 Bucles

Dentro del archivo main original creamos un nuevo rol para este asunto de los bucles.

```
--- 

# Nginx Playbook

- hosts: webserver (Todo lo que pongamos en hosts se ejecuta en webserver)
  remote_user: "root"
  roles:
  - role: nginx_install
  - role: register_variable
  - role: condiciones
  - role: ignorar_error
  - role: variable_template
  - role: bucle_template
```

Y creamos el directorio dentro de roles para bucle_template. Dentro creamos un directorio para tasks y otro para templates. Acto seguido, creamos un main para tasks.

```
--- 

# Copiar web nginx

- name: Copy nginx template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html

- name: Restart nginx
  systemd:
    name: nginx
    state: restarted
```

Sería similar al visto para los templates. El index.html.j2 sería el mismo que vimos en el anterior caso, salvo con alguna excepción enfocada a los bucles.

``` 
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Hello, Nginx!!</title>
</head>
<body>
  <h1>Hello bucle template, Nginx from {{ inventory_hostname }} our variable is: {{ variableOpenwebinars }}</h1>
  <p>The servers inside group [webserver] are {% for host in groups ['webserver'] %} {{ hostvars[host]['ansible_hostname'] }}{%if not loop.last %},{% endif %} {% endfor %}</p>
  </body>
  </html>
```

* Le estamos diciendo que itere sobre el grupo webserver, y que esta iteración se efectue sobre los hosts alojados en este grupo.

* El segundo bucle es para que nos muestre las ips de cada host del grupo webserver.

* El if not loop indica que si el servidor no es el último en mostrarse añada tras él una coma y con el endif culmina sin coma. Este es un detalle más visual que otra cosa.



















