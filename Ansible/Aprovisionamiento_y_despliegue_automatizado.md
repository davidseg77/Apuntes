# Info extrida del taller Aprovisionamiento y despliegue automatizado con Ansible

## Caso práctico

En primer lugar, accedemos a una máquina Ubuntu Server, donde vamos a instalar Ansible. Para ello hemos de completar tres pasos:

* Añadir repositorio Ansible

```
sudo add-apt-repository ppa:ansible/ansible
```

* Actualizar sistema

``` 
sudo apt update -y
```

* Instalar Ansible

``` 
sudo apt install ansible -y
``` 

**Crear un playbook**

Instalado Ansible en nuestro servidor, vamos a dar forma a nuestro playbook, nuestro primer script de Ansible. Para ello creamos un directorio donde irán aquellos archivos que tengan que ver con el playbook.

Creamos tal directorio, vamos a él e introduciremos dos archivos. Uno será el hosts, donde indicamos los nodos sobre los cuales vamos a realizar los despliegues.

Entre corchetes definimos los grupos donde van a ir esas ips o la del dns:

``` 
[webserver]
192.168.12.11
```

Ahora generamos las claves para conectarnos con esos equipos:

``` 
ssh-keygen
```

Ahora copiamos nuestra clave pública al nodo remoto:

``` 
ssh-copy-id -i ~/.ssh/id_rsa.pub usuarioremoto@ipdelequiporemoto
```

Tendremos que introducir la contraseña del equipo remoto y ya tendremos acceso siempre sin tener que dar la contraseña.

Y ahora sí, creamos nuestro playbook en formato yaml:

``` 
---
- hosts: webserver (nombre del grupo de ips en el archivo hosts)
  remote_user: usuarioremoto
  tasks: 
    - name: Install last version of Nginx
      apt: name=nginx state=latest
      become: yes (Para instalar la app como si fuera root)
    - name: Start/Enable
      service:
        name: nginx
        state: started
        enabled: yes
      become: yes  
```

Guardamos y ya tenemos el playbook creado. Para desplegarlo, hacemos uso del siguiente comando:

``` 
ansible-playbook -i hosts --ask-become-pass archivo.yaml
```

- --ask-become-pass nos solicita la contraseña para que nuestro usuario se convierta en root y pueda usar sudo.

Tendremos que dar nuestra contraseña y listo.

Podemos comprobarlo yendo al navegador o bien haciendo curl + ip del servidor remoto.

**Diferentes módulos playbook**

* Gestor de paquetes: Para, por ejemplo, instalar y administrar paquetes:

``` 
- name: Instalar una lista de paquetes
  yum:
    name: 
      - nginx
      - postgresql
      - postgresql-server
    state: present
```

* Service: Para iniciar o detener paquetes

``` 
- name: Encender el servicio nginx
  service:
    name: nginx
    state: started
```

* Copy: Copia un archivo local o remoto a la máquina remota

```
- name: Copia el fichero "origen.txt" e la ubicacion "destino/destino.txt"
  copy: 
    src: /origen/origen.txt
    dest: /destino/destino.txt
    owner: root
    group: root
    mode: '0644'
    backup: yes
```

* Debug: Imprime declaraciones durante la ejecución

```
- name: Imprimir inventory hostname de la maquina
  debug:
    msg: {{ inventory_hostname }}
```

* File: De los que más se utilizan, sirve para manejar archivos, establecer y eliminar atributos de archivos, crear enlaces simbólicos o directorios

``` 
- name: Crea un directorio si este no existe
  file: 
    path: /etc/some directory
    state: directory (Cuando se crea un directorio)
    mode: '0755'
```

* Command: Ejecución de comandos en el servidor remoto

``` 
- name: Imprimir contenido de some_file
  command: cat /etc/openwebinars_file
```

