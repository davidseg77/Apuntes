# Manual de Playbooks de Ansible

Los Ansible Playbooks ofrecen un sistema de implementación en múltiples máquinas y gestión de configuración simple, repetible y reutilizable, muy adecuado para implementar aplicaciones complejas. Si necesita ejecutar una tarea con Ansible más de una vez, escriba un libro de jugadas y póngalo bajo control de código fuente. Luego puede usar el libro de jugadas para implementar una nueva configuración o confirmar la configuración de sistemas remotos. 

Los libros de jugadas pueden:

* Declarar configuraciones.

* Orquestar los pasos de cualquier proceso ordenado manualmente, en múltiples conjuntos de máquinas, en un orden definido.

* Lanzar tareas de forma sincrónica o asincrónica.

## 1. Ejecución del libro de jugadas

Un libro de jugadas se ejecuta en orden de arriba a abajo. Dentro de cada juego, las tareas también se ejecutan en orden de arriba a abajo. Los libros de jugadas con múltiples 'jugadas' pueden orquestar implementaciones en múltiples máquinas, ejecutando una jugada en sus servidores web, luego otra en sus servidores de bases de datos, luego una tercera jugada en su infraestructura de red, y así sucesivamente. Como mínimo, cada jugada define dos cosas:

* Los nodos administrados a los que apuntar, usando un patrón.

* Al menos una tarea para ejecutar.

En este ejemplo, la primera reproducción apunta a los servidores web; la segunda jugada apunta a los servidores de bases de datos.

``` 
---
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest

  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest

  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started
``` 

## 2. Ejecutar libros de jugadas

Para ejecutar su libro de jugadas, use el comando ansible-playbook .

``` 
ansible-playbook playbook.yml -f 10
``` 

Utilice la --verbosebandera cuando ejecute su libro de estrategias para ver resultados detallados de los módulos exitosos y de los no exitosos.

### 2.1 Ejecutar libros de jugadas en modo de verificación

El modo de verificación de Ansible le permite ejecutar un libro de jugadas sin aplicar ninguna alteración a sus sistemas. Puede utilizar el modo de verificación para probar los manuales antes de implementarlos en un entorno de producción.

Para ejecutar un libro de jugadas en modo de verificación, puede pasar el indicador -Co --checkal ansible-playbookcomando:

``` 
ansible-playbook --check playbook.yaml
``` 

Al ejecutar este comando, el libro de jugadas se ejecutará normalmente, pero en lugar de implementar modificaciones, Ansible simplemente proporcionará un informe sobre los cambios que habría realizado. Este informe abarca detalles como modificaciones de archivos, ejecución de comandos y llamadas a módulos.

El modo de verificación ofrece un enfoque seguro y práctico para examinar la funcionalidad de sus manuales sin correr el riesgo de realizar cambios no deseados en sus sistemas. Además, es una herramienta valiosa para solucionar problemas de manuales que no funcionan como se esperaba.

## 3. Verificación de manuales

Es posible que desee verificar sus manuales para detectar errores de sintaxis y otros problemas antes de ejecutarlos. El comando ansible-playbook ofrece varias opciones de verificación, incluidas --check, --diff, --list-hosts, --list-tasksy --syntax-check. Las Herramientas para validar playbooks describen otras herramientas para validar y probar playbooks.

**ansible lint**

Puede utilizar ansible-lint para obtener comentarios detallados y específicos de Ansible sobre sus libros de jugadas antes de ejecutarlos. Por ejemplo, si ejecuta ansible-lintel libro de jugadas que se encuentra verify-apache.ymlcerca de la parte superior de esta página, debería obtener los siguientes resultados:

``` 
$ ansible-lint verify-apache.yml
[403] Package installs should not use latest
verify-apache.yml:8
Task/Handler: ensure apache is at the latest version
``` 

La página de reglas predeterminadas de ansible-lint describe cada error. Para [403], la solución recomendada es cambiar a en el libro de jugadas.state: lateststate: present



