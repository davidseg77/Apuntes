# Introducción a los comandos ad hoc

**¿Por qué utilizar comandos ad hoc?**

Los comandos ad hoc son excelentes para tareas que rara vez repites. Por ejemplo, si desea apagar todas las máquinas de su laboratorio para las vacaciones de Navidad, puede ejecutar una frase rápida en Ansible sin escribir un manual. Un comando ad hoc tiene este aspecto:

``` 
$ ansible [pattern] -m [module] -a "[module options]"
``` 

La -aopción acepta opciones a través de la key=valuesintaxis o una cadena JSON que comienza {y termina con }para una estructura de opciones más compleja. Puede obtener más información sobre patrones y módulos en otras páginas.


## 1. Casos de uso para tareas ad hoc

Las tareas ad hoc se pueden utilizar para reiniciar servidores, copiar archivos, administrar paquetes y usuarios, y mucho más. Puede utilizar cualquier módulo de Ansible en una tarea ad hoc. Las tareas ad hoc, como los manuales, utilizan un modelo declarativo, calculando y ejecutando las acciones necesarias para alcanzar un estado final específico. Logran una forma de idempotencia al verificar el estado actual antes de comenzar y no hacer nada a menos que el estado actual sea diferente del estado final especificado.

### 1.1 Reiniciar servidores

El módulo predeterminado para la ansible utilidad de línea de comandos es el módulo ansible.builtin.command. Puede utilizar una tarea ad hoc para llamar al módulo de comando y reiniciar todos los servidores web en Atlanta, 10 a la vez. Antes de que Ansible pueda hacer esto, debe tener todos los servidores de Atlanta enumerados en un grupo llamado [atlanta] en su inventario y debe tener credenciales SSH que funcionen para cada máquina de ese grupo. Para reiniciar todos los servidores del grupo [atlanta]:

``` 
$ ansible atlanta -a "/sbin/reboot"
``` 

De forma predeterminada, Ansible utiliza sólo cinco procesos simultáneos. Si tiene más hosts que el valor establecido para el recuento de bifurcaciones, puede aumentar el tiempo que le toma a Ansible comunicarse con los hosts. Para reiniciar los servidores [atlanta] con 10 bifurcaciones paralelas:

``` 
$ ansible atlanta -a "/sbin/reboot" -f 10
``` 

/usr/bin/ansible se ejecutará de forma predeterminada desde su cuenta de usuario. Para conectarse como un usuario diferente:

``` 
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username
``` 

El reinicio probablemente requiera una escalada de privilegios. Puede conectarse al servidor username y ejecutar el comando como rootusuario utilizando la palabra clave convert:

``` 
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]
``` 

Si agrega --ask-become-pass o -K, Ansible le solicitará la contraseña que utilizará para la escalada de privilegios (sudo/su/pfexec/doas/etc).

Hasta ahora, todos nuestros ejemplos han utilizado el módulo de 'comando' predeterminado. Para utilizar un módulo diferente, pase -mel nombre del módulo. Por ejemplo, para utilizar el módulo ansible.builtin.shell:

``` 
$ ansible raleigh -m ansible.builtin.shell -a 'echo $TERM'
``` 

Al ejecutar cualquier comando con la CLI ad hoc de Ansible (a diferencia de Playbooks), preste especial atención a las reglas de comillas del shell, de modo que el shell local conserve la variable y la pase a Ansible. Por ejemplo, usar comillas dobles en lugar de simples en el ejemplo anterior evaluaría la variable en el cuadro en el que se encontraba.

### 1.2 Administrar archivos

Una tarea ad hoc puede aprovechar el poder de Ansible y SCP para transferir muchos archivos a varias máquinas en paralelo. Para transferir un archivo directamente a todos los servidores del grupo [atlanta]:

``` 
$ ansible atlanta -m ansible.builtin.copy -a "src=/etc/hosts dest=/tmp/hosts"
``` 

Si planea repetir una tarea como esta, use el módulo ansible.builtin.template en un libro de estrategias.

El módulo ansible.builtin.file permite cambiar la propiedad y los permisos de los archivos. copyEstas mismas opciones también se pueden pasar directamente al módulo:

```
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/a.txt mode=600"
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
``` 

El filemódulo también puede crear directorios, similares a :mkdir -p

``` 
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
``` 

Además de eliminar directorios (recursivamente) y eliminar archivos:

``` 
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c state=absent"
``` 

### 1.3 Gestionar paquetes

También puede utilizar una tarea ad hoc para instalar, actualizar o eliminar paquetes en nodos administrados mediante un módulo de administración de paquetes como yum. Los módulos de administración de paquetes admiten funciones comunes para instalar, eliminar y, en general, administrar paquetes. Es posible que algunas funciones específicas para un administrador de paquetes no estén presentes en el módulo Ansible ya que no forman parte de la administración general de paquetes.

Para garantizar que un paquete se instale sin actualizarlo:

``` 
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=present"
``` 

Para garantizar que esté instalada una versión específica de un paquete:

``` 
$ ansible webservers -m ansible.builtin.yum -a "name=acme-1.5 state=present"
``` 

Para garantizar que un paquete tenga la última versión:

``` 
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=latest"
``` 

Para asegurarse de que un paquete no esté instalado:

```
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=absent"
``` 

Ansible tiene módulos para gestionar paquetes en muchas plataformas. Si no hay ningún módulo para su administrador de paquetes, puede instalar paquetes usando el módulo de comando o crear un módulo para su administrador de paquetes.

### 1.4 Administrar usuarios y grupos

Puede crear, administrar y eliminar cuentas de usuario en sus nodos administrados con tareas ad hoc:

``` 
$ ansible all -m ansible.builtin.user -a "name=foo password=<encrypted password here>"

$ ansible all -m ansible.builtin.user -a "name=foo state=absent"
``` 

### 1.5 Servicios de gestión

Asegúrese de que se inicie un servicio en todos los servidores web:

``` 
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=started"
``` 

Alternativamente, reinicie un servicio en todos los servidores web:

``` 
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=restarted"
``` 

Asegúrese de que un servicio esté detenido:

``` 
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=stopped"
``` 

### 1.6 Recopilando hechos

Los hechos representan variables descubiertas sobre un sistema. Puede utilizar hechos para implementar la ejecución condicional de tareas, pero también solo para obtener información ad hoc sobre sus sistemas. Para ver todos los datos:

``` 
$ ansible all -m ansible.builtin.setup
```

También puede filtrar esta salida para mostrar solo ciertos hechos; consulte la documentación del módulo ansible.builtin.setup para obtener más detalles.

### 1.7 Modo de verificación

En el modo de verificación, Ansible no realiza ningún cambio en los sistemas remotos. Ansible imprime solo los comandos. No ejecuta los comandos.

``` 
$  ansible all -m copy -a "content=foo dest=/root/bar.txt" -C
``` 

Habilitar el modo de verificación ( -Co --check) en el comando anterior significa que Ansible en realidad no crea ni actualiza el /root/bar.txtarchivo en ningún sistema remoto.


