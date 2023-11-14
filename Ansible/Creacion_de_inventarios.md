# Cómo construir tu inventario

El inventario más simple es un archivo único con una lista de hosts y grupos. La ubicación predeterminada para este archivo es /etc/ansible/hosts. Puede especificar un archivo de inventario diferente en la línea de comando usando la opción o en la configuración usando .-i <path>inventory

## 1. Conceptos básicos del inventario: formatos, hosts y grupos

Puede crear su archivo de inventario en uno de muchos formatos, según los complementos de inventario que tenga. Los formatos más comunes son INI y YAML. Un INI básico /etc/ansible/hostspodría verse así:

``` 
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
``` 

Los encabezados entre paréntesis son nombres de grupos, que se utilizan para clasificar los hosts y decidir qué hosts se controlan, en qué momentos y con qué propósito. Los nombres de los grupos deben seguir las mismas pautas que la creación de nombres de variables válidos.

Aquí está el mismo archivo de inventario básico en formato YAML:

``` 
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
``` 

### 1.1 Anfitriones en múltiples grupos

Puede (y probablemente lo hará) colocar cada host en más de un grupo. Por ejemplo, un servidor web de producción en un centro de datos en Atlanta podría incluirse en grupos llamados [prod], [atlanta] y [webservers]. Puedes crear grupos que realicen un seguimiento de:

* Qué: una aplicación, pila o microservicio (por ejemplo, servidores de bases de datos, servidores web, etc.).

* Dónde: un centro de datos o una región para comunicarse con el DNS local, el almacenamiento, etc. (por ejemplo, este, oeste).

* Cuándo: la etapa de desarrollo, para evitar realizar pruebas en recursos de producción (por ejemplo, producción, prueba).

Ampliar el inventario YAML anterior para incluir qué, cuándo y dónde se vería así:

``` 
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
test:
  hosts:
    bar.example.com:
    three.example.com:
``` 

Puedes ver que one.example.com existe en los grupos dbservers, east y prod.

### 1.2 Grupos de agrupación: relaciones entre grupos padre/hijo

Puede crear relaciones padre/hijo entre grupos. Los grupos principales también se conocen como grupos anidados o grupos de grupos. Por ejemplo, si todos sus hosts de producción ya están en grupos como atlanta_prody denver_prod, puede crear un productiongrupo que incluya esos grupos más pequeños. Este enfoque reduce el mantenimiento porque puede agregar o eliminar hosts del grupo principal editando los grupos secundarios.

Para crear relaciones padre/hijo para grupos:

- en formato INI, use el :childrensufijo

- en formato YAML, use la children:entrada

Aquí está el mismo inventario que se muestra arriba, simplificado con grupos principales para los grupos prody test. Los dos archivos de inventario le dan los mismos resultados:

```
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  children:
    east:
test:
  children:
    west:
```

Los grupos de niños tienen un par de propiedades a tener en cuenta:

* Cualquier host que sea miembro de un grupo secundario es automáticamente miembro del grupo principal.

* Los grupos pueden tener varios padres e hijos, pero no relaciones circulares.

* Los hosts también pueden estar en varios grupos, pero solo habrá una instancia de un host en tiempo de ejecución. Ansible fusiona los datos de varios grupos.

### 1.3 Agregar rangos de hosts

Si tiene muchos hosts con un patrón similar, puede agregarlos como un rango en lugar de enumerar cada nombre de host por separado:

**En INI:**

``` 
[webservers]
www[01:50].example.com
``` 

**En YAML:**

```
...
  webservers:
    hosts:
      www[01:50].example.com:
``` 

Puede especificar un paso (incrementos entre números de secuencia) al definir un rango numérico de hosts:

**En INI:**

```
[webservers]
www[01:50:2].example.com
```

**En YAML:**

``` 
...
  webservers:
    hosts:
      www[01:50:2].example.com:
``` 

El ejemplo anterior haría que los subdominios www01, www03, www05,…, www49 coincidan, pero no www00, www02, www50 y así sucesivamente, porque el paso (incremento) es de 2 unidades por cada paso.

Para patrones numéricos, se pueden incluir o eliminar ceros a la izquierda, según se desee. Los rangos son inclusivos. También puedes definir rangos alfabéticos:

``` 
[databases]
db-[a:f].example.com
```

### 1.4 Pasar múltiples fuentes de inventario

Puede apuntar a múltiples fuentes de inventario (directorios, secuencias de comandos de inventario dinámico o archivos compatibles con complementos de inventario) al mismo tiempo proporcionando múltiples parámetros de inventario desde la línea de comando o configurandoANSIBLE_INVENTORY. Esto puede resultar útil cuando desea centrarse en entornos normalmente separados, como la puesta en escena y la producción, al mismo tiempo para una acción específica.

Para apuntar a dos fuentes de inventario desde la línea de comando:

``` 
ansible-playbook get_logs.yml -i staging -i production
``` 

## 2. Organizar el inventario en un directorio

Puede consolidar múltiples fuentes de inventario en un solo directorio. La versión más simple de esto es un directorio con múltiples archivos en lugar de un único archivo de inventario. Un solo archivo resulta difícil de mantener cuando es demasiado largo. Si tiene varios equipos y varios proyectos de automatización, tener un archivo de inventario por equipo o proyecto permite a todos encontrar fácilmente los hosts y grupos que les interesan.

También puede combinar varios tipos de fuentes de inventario en un directorio de inventario. Esto puede resultar útil para combinar hosts estáticos y dinámicos y administrarlos como un solo inventario. El siguiente directorio de inventario combina una fuente de complemento de inventario, un script de inventario dinámico y un archivo con hosts estáticos:

``` 
inventory/
  openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  dynamic-inventory.py   # add additional hosts with dynamic inventory script
  on-prem                # add static hosts and groups
  parent-groups          # add static hosts and groups
``` 

Puede orientar este directorio de inventario de la siguiente manera:

``` 
ansible-playbook example.yml -i inventory
``` 

También puede configurar el directorio de inventario en su ansible.cfgarchivo. 

### 2.1 Gestionar el orden de carga de inventario

Ansible carga las fuentes del inventario en orden ASCII según los nombres de los archivos. Si define grupos principales en un archivo o directorio y grupos secundarios en otros archivos o directorios, los archivos que definen los grupos secundarios deben cargarse primero. Si los grupos principales se cargan primero, verá el error .Unable to parse /path/to/source_of_parent_groups as an inventory source

Por ejemplo, si tiene un archivo llamado groups-of-groupsque define un productiongrupo con grupos secundarios definidos en un archivo llamado on-prem, Ansible no puede analizar el productiongrupo. Para evitar este problema, puedes controlar el orden de carga agregando prefijos a los archivos:

``` 
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from OpenStack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-on-prem                # add static hosts and groups
  04-groups-of-groups       # add parent groups
``` 

## 3. Asignar una variable a una máquina: variables del lenguaje principal

Puede asignar fácilmente una variable a un solo host y luego usarla más adelante en los manuales. Puede hacer esto directamente en su archivo de inventario.

En INI:

``` 
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
``` 

En YAML:

``` 
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
``` 

Los valores únicos, como los puertos SSH no estándar, funcionan bien como variables del host. Puede agregarlos a su inventario de Ansible agregando el número de puerto después del nombre de host con dos puntos:

``` 
badwolf.example.com:5309
``` 

Las variables de conexión también funcionan bien como variables del lenguaje principal:

``` 
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
``` 

### 3.1 Alias ​​de inventario

También puede definir alias en su inventario utilizando variables del lenguaje principal:

En INI:

``` 
jumper ansible_port=5555 ansible_host=192.0.2.50
``` 

En YAML:

``` 
...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
``` 

En este ejemplo, ejecutar Ansible contra el alias de host "jumper" se conectará a 192.0.2.50 en el puerto 5555. 

### 3.2 Definición de variables en formato INI

Los valores pasados ​​en formato INI usando la key=valuesintaxis se interpretan de manera diferente dependiendo de dónde se declaran:

* Cuando se declaran en línea con el host, los valores INI se interpretan como estructuras literales de Python (cadenas, números, tuplas, listas, dictados, booleanos, Ninguno). Las líneas de host aceptan múltiples key=valueparámetros por línea. Por lo tanto, necesitan una forma de indicar que un espacio es parte de un valor en lugar de un separador. Los valores que contienen espacios en blanco se pueden citar (simples o dobles). Consulte las reglas de análisis de Python shlex para obtener más detalles.

* Cuando se declaran en una :varssección, los valores INI se interpretan como cadenas. Por ejemplo var=FALSEcrearía una cadena igual a 'FALSO'. A diferencia de las líneas de host, :varslas secciones aceptan solo una entrada por línea, por lo que todo lo que sigue a =debe ser el valor de la entrada.

Si un valor de variable establecido en un inventario INI debe ser de un tipo determinado (por ejemplo, una cadena o un valor booleano), especifique siempre el tipo con un filtro en su tarea. No confíe en los tipos establecidos en los inventarios INI cuando consuma variables.

Considere usar el formato YAML para fuentes de inventario para evitar confusión sobre el tipo real de una variable. El complemento de inventario YAML procesa valores variables de manera consistente y correcta.

### 3.3 Asignar una variable a muchas máquinas: variables grupales

Si todos los hosts de un grupo comparten un valor de variable, puede aplicar esa variable a todo un grupo a la vez.

**En INI:**

```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
``` 

**En YAML:**

``` 
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
``` 

Las variables de grupo son una forma conveniente de aplicar variables a varios hosts a la vez. Sin embargo, antes de ejecutar, Ansible siempre aplana las variables, incluidas las variables de inventario, al nivel del host. Si un host es miembro de varios grupos, Ansible lee valores de variables de todos esos grupos. Si asigna diferentes valores a la misma variable en diferentes grupos, Ansible elige qué valor usar según las reglas internas para la fusión.

### 3.4 Heredar valores de variables: variables de grupo para grupos de grupos

Puede aplicar variables a grupos principales (grupos anidados o grupos de grupos), así como a grupos secundarios. La sintaxis es la misma: :varspara formato INI y vars:para formato YAML:

**En INI:**

``` 
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

**En YAML:**

``` 
usa:
  children:
    southeast:
      children:
        atlanta:
          hosts:
            host1:
            host2:
        raleigh:
          hosts:
            host2:
            host3:
      vars:
        some_server: foo.southeast.example.com
        halon_system_timeout: 30
        self_destruct_countdown: 60
        escape_pods: 2
    northeast:
    northwest:
    southwest:
``` 

Las variables de un grupo secundario tendrán mayor prioridad (anularán) las variables de un grupo principal.

### 3.5 Organización de variables de host y grupo

Aunque puede almacenar variables en el archivo de inventario principal, almacenar archivos de variables de host y de grupo separados puede ayudarle a organizar los valores de sus variables más fácilmente. También puede utilizar listas y datos hash en archivos de variables de grupo y host, lo que no puede hacer en su archivo de inventario principal.

Los archivos de variables de grupo y host deben utilizar la sintaxis YAML. Las extensiones de archivo válidas incluyen '.yml', '.yaml', '.json' o ninguna extensión de archivo. Consulte Sintaxis de YAML si es nuevo en YAML.

Ansible carga archivos de variables de grupo y host buscando rutas relativas al archivo de inventario o al archivo del libro de estrategias. Si su archivo de inventario /etc/ansible/hostscontiene un host llamado 'foosball' que pertenece a dos grupos, 'raleigh' y 'webservers', ese host usará variables en archivos YAML en las siguientes ubicaciones:

``` 
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
``` 

Por ejemplo, si agrupa hosts en su inventario por centro de datos y cada centro de datos usa su propio servidor NTP y servidor de base de datos, puede crear un archivo llamado /etc/ansible/group_vars/raleighpara almacenar las variables del raleighgrupo:

``` 
---
ntp_server: acme.example.org
database_server: storage.example.org
```

También puede crear directorios con el nombre de sus grupos o hosts. Ansible leerá todos los archivos de estos directorios en orden lexicográfico. Un ejemplo con el grupo 'raleigh':

``` 
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
``` 

Todos los hosts del grupo 'raleigh' tendrán disponibles las variables definidas en estos archivos. Esto puede resultar muy útil para mantener las variables organizadas cuando un solo archivo se vuelve demasiado grande o cuando desea utilizar Ansible Vault en algunas variables de grupo.

Porque ansible-playbooktambién puede agregar group_vars/directorios host_vars/a su directorio de libro de jugadas. Otros comandos de Ansible (por ejemplo, ansible, ansible-consoleetc.) solo buscarán group_vars/y host_vars/en el directorio de inventario. Si desea que otros comandos carguen variables de grupo y de host desde un directorio del libro de estrategias, debe proporcionar la --playbook-diropción en la línea de comando. Si carga archivos de inventario desde el directorio del libro de estrategias y el directorio de inventario, las variables en el directorio del libro de estrategias anularán las variables establecidas en el directorio de inventario.

Mantener su archivo de inventario y sus variables en un repositorio de git (u otro control de versiones) es una excelente manera de realizar un seguimiento de los cambios en su inventario y sus variables de host.

### 3.6 Gestión de orden de carga variable de inventario

Cuando utilice múltiples fuentes de inventario, tenga en cuenta que cualquier conflicto de variables se resuelve de acuerdo con las reglas descritas en Cómo se combinan las variables y Precedencia de variables: ¿dónde debo colocar una variable? . Puede controlar el orden de combinación de las variables en las fuentes del inventario para obtener el valor de la variable que necesita.

Cuando pasa varias fuentes de inventario en la línea de comando, Ansible fusiona variables en el orden en que pasa esos parámetros. Si [all:vars]en la etapa de preparación se define el inventario y se define el inventario de producción , entonces:myvar = 1myvar = 2

* Pase para ejecutar el libro de jugadas con .-i staging -i productionmyvar = 2

* Pase para ejecutar el libro de jugadas con .-i production -i stagingmyvar = 1

Cuando coloca varias fuentes de inventario en un directorio, Ansible las combina en orden ASCII según los nombres de archivo. Puede controlar el orden de carga agregando prefijos a los archivos:

``` 
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-static-inventory       # add static hosts
  group_vars/
    all.yml                 # assign variables to all hosts
``` 

Si 01-openstack.ymldefine para el grupo , define y define , el libro de jugadas se ejecutará con .myvar = 1all02-dynamic-inventory.pymyvar = 203-static-inventorymyvar = 3myvar = 3


## 4. Conexión a hosts: parámetros de inventario de comportamiento

Como se describió anteriormente, configurar las siguientes variables controla cómo Ansible interactúa con hosts remotos.

* conexión_ansible
  
Tipo de conexión al host. Este puede ser el nombre de cualquier complemento de conexión de Ansible. Los tipos de protocolo SSH son smart, ssho paramiko. El valor predeterminado es inteligente. Los tipos no basados ​​en SSH se describen en la siguiente sección.

**General para todas las conexiones:**

* ansible_host
  
El nombre del host al que conectarse, si es diferente del alias que desea darle.

* puerto_ansible
  
El número de puerto de conexión, si no es el predeterminado (22 para ssh)

* usuario_ansible
  
El nombre de usuario que se utilizará al conectarse al host

* contraseña_ansible
  
La contraseña que se utilizará para autenticarse en el host (nunca almacene esta variable en texto sin formato; utilice siempre una bóveda. Consulte Mantener visibles de forma segura las variables protegidas ).

**Específico para la conexión SSH:**

* ansible_ssh_private_key_file
  
Archivo de clave privada utilizado por SSH. Útil si utiliza varias claves y no desea utilizar el agente SSH.

* ansible_ssh_common_args
  
Esta configuración siempre se agrega a la línea de comando predeterminada para sftp , scp y ssh . Útil para configurar ProxyCommandun determinado host (o grupo).

* ansible_sftp_extra_args
  
Esta configuración siempre se agrega a la línea de comando sftp predeterminada.

* ansible_scp_extra_args
  
Esta configuración siempre se agrega a la línea de comando scp predeterminada .

* ansible_ssh_extra_args
  
Esta configuración siempre se agrega a la línea de comando ssh predeterminada .

* ansible_ssh_pipelining
  
Determina si se utilizará o no la canalización SSH. Esto puede anular la pipeliningconfiguración en ansible.cfg.

* ansible_ssh_executable (agregado en la versión 2.2)
  
Esta configuración anula el comportamiento predeterminado para usar el sistema ssh . Esto puede anular la ssh_executableconfiguración en ansible.cfg.

**Escalada de privilegios:**

* ansible_convertirse

Equivalente a ansible_sudoo ansible_su, permite forzar la escalada de privilegios

* ansible_become_method

Permite configurar el método de escalada de privilegios

* ansible_convertirse_usuario

Equivalente a ansible_sudo_usero ansible_su_user, permite configurar el usuario en el que se convierte a través de la escalada de privilegios.

* ansible_convertirse_contraseña

Equivale a ansible_sudo_passwordo ansible_su_password, le permite establecer la contraseña de escalada de privilegios (nunca almacene esta variable en texto sin formato; utilice siempre una bóveda. Consulte Mantener visibles de forma segura las variables protegidas ) .

* ansible_become_exe

Equivalente a ansible_sudo_exeo ansible_su_exe, le permite configurar el ejecutable para el método de escalamiento seleccionado

* ansible_become_flags

Equivalente a ansible_sudo_flagso ansible_su_flags, le permite configurar las banderas pasadas al método de escalamiento seleccionado. Esto también se puede configurar globalmente en ansible.cfgla sudo_flagsopción

**Parámetros del entorno del host remoto:**

* tipo_shell_ansible

El tipo de shell del sistema de destino. No debe utilizar esta configuración a menos que haya configurado ansible_shell_executable en un shell no compatible con Bourne (sh). De forma predeterminada, los comandos se formatean utilizando shla sintaxis de estilo. Establecer esto en csho fishhará que los comandos ejecutados en los sistemas de destino sigan la sintaxis de esos shells.

* ansible_python_interpreter

La ruta de Python del host de destino. Esto es útil para sistemas con más de un Python o que no están ubicados en /usr/bin/python como *BSD, o donde /usr/bin/python no es un Python de la serie 2.X. No utilizamos el mecanismo /usr/bin/env ya que requiere que la ruta del usuario remoto esté configurada correctamente y también supone que el ejecutable de Python se llama python, donde el ejecutable podría tener un nombre similar a python2.6 .

* ansible_*_intérprete

Funciona para cualquier cosa como Ruby o Perl y funciona igual que ansible_python_interpreter . Esto reemplaza una gran cantidad de módulos que se ejecutarán en ese host.

Nuevo en la versión 2.1.

* ansible_shell_executable

Esto establece el shell que usará el controlador ansible en la máquina de destino, anulando executable el ansible.cfgvalor predeterminado /bin/sh . En realidad, solo debería cambiarlo si no es posible usar /bin/sh (en otras palabras, si /bin/sh no está instalado en la máquina de destino o no se puede ejecutar desde sudo).

Ejemplos de un archivo host Ansible-INI:

``` 
some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3
``` 

## 5. Ejemplos de configuración de inventario

### 5.1 Ejemplo: un inventario por entorno

Si necesita administrar múltiples entornos, a veces es prudente tener solo hosts de un único entorno definidos por inventario. De esta manera, es más difícil, por ejemplo, cambiar accidentalmente el estado de los nodos dentro del entorno de "prueba" cuando en realidad desea actualizar algunos servidores "de prueba".

Para el ejemplo mencionado anteriormente, podría tener un inventory_testarchivo:

``` 
[dbservers]
db01.test.example.com
db02.test.example.com

[appservers]
app01.test.example.com
app02.test.example.com
app03.test.example.com
``` 

Ese archivo sólo incluye hosts que forman parte del entorno de "prueba". Defina las máquinas "preparadas" en otro archivo llamado inventory_staging:

``` 
[dbservers]
db01.staging.example.com
db02.staging.example.com

[appservers]
app01.staging.example.com
app02.staging.example.com
app03.staging.example.com
``` 

Para aplicar un libro de jugadas llamado site.yml a todos los servidores de aplicaciones en el entorno de prueba, use el siguiente comando:

``` 
ansible-playbook -i inventory_test -l appservers site.yml
``` 

### 5.2 Ejemplo: agrupar por función

En la sección anterior ya vio un ejemplo de uso de grupos para agrupar hosts que tienen la misma función. Esto le permite, por ejemplo, definir reglas de firewall dentro de un playbook o rol que afecta solo a los servidores de bases de datos:

``` 
- hosts: dbservers
  tasks:
  - name: Allow access from 10.0.0.1
    ansible.builtin.iptables:
      chain: INPUT
      jump: ACCEPT
      source: 10.0.0.1
``` 

### 5.3 Ejemplo: agrupar por ubicación

Otras tareas pueden centrarse en dónde se encuentra un determinado host. Digamos que db01.test.example.comy app01.test.example.comestán ubicados en DC1 mientras que db02.test.example.comestá en DC2:

``` 
[dc1]
db01.test.example.com
app01.test.example.com

[dc2]
db02.test.example.com
``` 

En la práctica, incluso podría terminar mezclando todas estas configuraciones, ya que es posible que un día necesite actualizar todos los nodos en un centro de datos específico y, otro día, actualizar todos los servidores de aplicaciones sin importar su ubicación.



