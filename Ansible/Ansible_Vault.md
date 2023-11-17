# Cómo utilizar Ansible Vault para proteger datos confidenciales del libro de jugadas

## 1. ¿Qué es Ansible Vault?

Ansible Vault es un mecanismo que permite incorporar contenido cifrado de forma transparente en los flujos de trabajo de Ansible. Una utilidad llamada ansible-vaultprotege los datos confidenciales, llamados secretos , cifrándolos en el disco. Para integrar estos secretos con los datos normales de Ansible, tanto los comandos ansiblecomo ansible-playbookpara ejecutar tareas ad hoc y el libro de jugadas estructurado, respectivamente, tienen soporte para descifrar contenido cifrado en bóveda en tiempo de ejecución.

Vault se implementa con granularidad a nivel de archivo, lo que significa que los archivos individuales están cifrados o no. Utiliza el AES256algoritmo para proporcionar cifrado simétrico vinculado a una contraseña proporcionada por el usuario. Esto significa que se utiliza la misma contraseña para cifrar y descifrar el contenido, lo que resulta útil desde el punto de vista de la usabilidad. Ansible es capaz de identificar y descifrar cualquier archivo cifrado en bóveda que encuentre mientras ejecuta un libro de estrategias o una tarea.

Ahora que comprende un poco qué es Vault, podemos comenzar a analizar las herramientas que proporciona Ansible y cómo utilizar Vault con los flujos de trabajo existentes.

## 2. Configuración del editor de Ansible Vault

Antes de utilizar el ansible-vault comando, es una buena idea especificar su editor de texto preferido. Algunos de los comandos de Vault implican abrir un editor para manipular el contenido de un archivo cifrado. Ansible mirará la EDITOR variable de entorno para encontrar su editor preferido. Si esto no está configurado, el valor predeterminado será vi.

Si no desea editar con vi el editor, debe configurar la EDITOR variable en su entorno.

Para configurar el editor para un comando individual, anteponga el comando con la asignación de variable de entorno, así:

``` 
EDITOR=nano ansible-vault . . .
``` 

Para que esto sea persistente, abra su ~/.bashrc archivo:

``` 
nano ~/.bashrc
``` 

Especifique su editor preferido agregando una EDITOR asignación al final del archivo:

``` 
~/.bashrc
export EDITOR=nano
``` 

Guarde y cierre el archivo cuando haya terminado.

Obtenga el archivo nuevamente para leer el cambio en la sesión actual:

``` 
. ~/.bashrc
``` 

Muestre la EDITORvariable para comprobar que se aplicó su configuración:

``` 
echo $EDITOR
``` 

Ahora que ha establecido su editor preferido, podemos analizar las operaciones disponibles con el ansible-vault comando.

## 3. Cómo administrar archivos confidenciales con ansible-vault

El ansible-vault comando es la interfaz principal para administrar contenido cifrado dentro de Ansible. Este comando se utiliza inicialmente para cifrar archivos y posteriormente para ver, editar o descifrar los datos.

### 3.1 Crear nuevos archivos cifrados

Para crear un nuevo archivo cifrado con Vault, utilice el ansible-vault create comando. Pase el nombre del archivo que desea crear. Por ejemplo, para crear un archivo YAML cifrado llamado vault.yml para almacenar variables confidenciales, puede escribir:

``` 
ansible-vault create vault.yml
``` 

Se le pedirá que ingrese y confirme una contraseña:

``` 
Output
New Vault password: 
Confirm New Vault password:
``` 

Cuando haya confirmado su contraseña, Ansible abrirá inmediatamente una ventana de edición donde podrá ingresar el contenido que desee.

Para probar la función de cifrado, ingrese algún texto de prueba:

``` 
Secret information
``` 

Ansible cifrará el contenido cuando cierre el archivo. Si revisas el archivo, en lugar de ver las palabras que escribiste, verás un bloque cifrado:

``` 
cat vault.yml
```

Podemos ver información del encabezado que Ansible usa para saber cómo manejar el archivo, seguida del contenido cifrado, que se muestra como números.


### 3.2 Cifrar archivos existentes

Si ya tiene un archivo que desea cifrar con Vault, utilice el ansible-vault encryptcomando en su lugar.

Para realizar pruebas, podemos crear un archivo de ejemplo escribiendo:

``` 
echo 'unencrypted stuff' > encrypt_me.txt
``` 

Ahora puedes cifrar el archivo existente escribiendo:

``` 
ansible-vault encrypt encrypt_me.txt
``` 

Nuevamente, se le pedirá que proporcione y confirme una contraseña. Posteriormente, un mensaje confirmará el cifrado:

``` 
Output
New Vault password: 
Confirm New Vault password:
Encryption successful
``` 

En lugar de abrir una ventana de edición, ansible-vaultcifrará el contenido del archivo y lo volverá a escribir en el disco, reemplazando la versión no cifrada.

Si revisamos el archivo, deberíamos ver un patrón cifrado similar:

``` 
cat encrypt_me.txt
``` 

Como puede ver, Ansible cifra el contenido existente de la misma manera que cifra los archivos nuevos.

### 3.3 Ver archivos cifrados

A veces, es posible que necesite hacer referencia al contenido de un archivo cifrado en una bóveda sin necesidad de editarlo o escribirlo en el sistema de archivos sin cifrar. El ansible-vault view comando envía el contenido de un archivo a formato estándar. Por defecto, esto significa que los contenidos se muestran en el terminal.

Pase el archivo cifrado de la bóveda al comando:

``` 
ansible-vault view vault.yml
``` 

Se le pedirá la contraseña del archivo. Después de ingresarlo exitosamente, se mostrará el contenido:

``` 
Output
Vault password:
Secret information
``` 

Como puede ver, la solicitud de contraseña se mezcla con la salida del contenido del archivo. Tenga esto en cuenta cuando lo utilice ansible-vault view en procesos automatizados.

### 3.4 Editar archivos cifrados

Cuando necesite editar un archivo cifrado, use el ansible-vault edit comando:

``` 
ansible-vault edit vault.yml
``` 

Se le pedirá la contraseña del archivo. Después de ingresarlo, Ansible abrirá el archivo en una ventana de edición, donde podrá realizar los cambios necesarios.

Al guardar, el nuevo contenido se cifrará nuevamente utilizando la contraseña de cifrado del archivo y se escribirá en el disco.

### 3.5 Descifrar manualmente archivos cifrados

Para descifrar un archivo cifrado de bóveda, utilice el ansible-vault decrypt comando.

**Nota:** Debido a la mayor probabilidad de enviar accidentalmente datos confidenciales al repositorio de su proyecto, el ansible-vault decrypt comando solo se sugiere cuando desea eliminar el cifrado de un archivo de forma permanente. Si necesita ver o editar un archivo cifrado de bóveda, normalmente es mejor utilizar los comandos ansible-vault view o ansible-vault edit, respectivamente.

Pase el nombre del archivo cifrado:

``` 
ansible-vault decrypt vault.yml
``` 

Se le solicitará la contraseña de cifrado del archivo. Una vez que ingrese la contraseña correcta, el archivo será descifrado:

``` 
Output
Vault password:
Decryption successful
``` 

Si vuelve a ver el archivo, en lugar del cifrado de la bóveda, debería ver el contenido real del archivo:

``` 
cat vault.yml
``` 

Su archivo ahora no está cifrado en el disco. Asegúrese de eliminar cualquier información confidencial o volver a cifrar el archivo cuando haya terminado.

### 3.6 Cambiar la contraseña de archivos cifrados

Si necesita cambiar la contraseña de un archivo cifrado, use el ansible-vault rekey comando:

``` 
ansible-vault rekey encrypt_me.txt
``` 

Cuando ingrese el comando, primero se le solicitará la contraseña actual del archivo:

``` 
Output
Vault password:
``` 

Después de ingresarla, se le pedirá que seleccione y confirme una nueva contraseña de la bóveda:

``` 
Output
Vault password:
New Vault password:
Confirm New Vault password:
``` 

Cuando haya confirmado con éxito una nueva contraseña, recibirá un mensaje indicando el éxito del proceso de nuevo cifrado:

``` 
Output
Rekey successful
``` 

Ahora debería poder accederse al archivo con la nueva contraseña. La contraseña anterior ya no funcionará.


## 4. Ejecución de Ansible con archivos cifrados en Vault

Una vez que haya cifrado su información confidencial con Vault, puede comenzar a usar los archivos con las herramientas convencionales de Ansible. Los comandos ansible y ansible-playbook saben cómo descifrar archivos protegidos en bóveda con la contraseña correcta. Hay algunas formas diferentes de proporcionar contraseñas para estos comandos según sus necesidades.

Para seguir adelante, necesitará un archivo cifrado en una bóveda. Puedes crear uno escribiendo:

``` 
ansible-vault create secret_key
```

Seleccione y confirme una contraseña. Complete el contenido ficticio que desee:

``` 
secret_key
confidential data
``` 

Guarde y cierre el archivo.

También podemos crear un hosts archivo temporal a modo de inventario:

``` 
nano hosts
``` 

Le agregaremos solo el host local de Ansible. Para prepararnos para un paso posterior, lo colocaremos en el [database] grupo:

``` 
Hosts
[database]
localhost ansible_connection=local
``` 

Guarde y cierre el archivo cuando haya terminado.

A continuación, cree un ansible.cfg archivo en el directorio actual si aún no existe ninguno:

``` 
nano ansible.cfg
```

Por ahora, simplemente agregue una [defaults] sección y apunte Ansible al inventario que acabamos de crear:

``` 
ansible.cfg
[defaults]
inventory = ./hosts
``` 

Cuando estés listo, continúa.

### 4.1 Usando un mensaje interactivo

La forma más sencilla de descifrar contenido en tiempo de ejecución es hacer que Ansible le solicite las credenciales adecuadas. Puede hacer esto agregando el comando --ask-vault-pass a cualquiera ansible o ansible-playbook. Ansible le solicitará una contraseña que utilizará para intentar descifrar cualquier contenido protegido por bóveda que encuentre.

Por ejemplo, si necesitáramos copiar el contenido de un archivo cifrado en un almacén a un host, podríamos hacerlo con el copy módulo y la --ask-vault-pass bandera. Si el archivo realmente contiene datos confidenciales, lo más probable es que desee bloquear el acceso al host remoto con restricciones de permiso y propiedad.

**Nota:** Usamos localhost como host de destino en este ejemplo para minimizar la cantidad de servidores necesarios, pero los resultados deberían ser los mismos que si el host fuera realmente remoto:

``` 
ansible --ask-vault-pass -bK -m copy -a 'src=secret_key dest=/tmp/secret_key mode=0600 owner=root group=root' localhost
``` 

Nuestra tarea especifica que la propiedad del archivo debe cambiarse a root, por lo que se requieren privilegios administrativos. La -bK bandera le indica a Ansible que solicite la sudo contraseña del host de destino, por lo que se le solicitará su sudo contraseña. Luego se le pedirá la contraseña de Vault:

``` 
Output
BECOME password:
Vault password:
``` 

Cuando se proporciona la contraseña, Ansible intentará ejecutar la tarea, utilizando la contraseña de Vault para cualquier archivo cifrado que encuentre. Tenga en cuenta que todos los archivos a los que se haga referencia durante la ejecución deben utilizar la misma contraseña:

``` 
Output
localhost | SUCCESS => {
    "changed": true, 
    "checksum": "7a2eb5528c44877da9b0250710cba321bc6dac2d", 
    "dest": "/tmp/secret_key", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "270ac7da333dd1db7d5f7d8307bd6b41", 
    "mode": "0600", 
    "owner": "root", 
    "size": 18, 
    "src": "/home/sammy/.ansible/tmp/ansible-tmp-1480978964.81-196645606972905/source", 
    "state": "file", 
    "uid": 0
}
``` 

Solicitar una contraseña es seguro, pero puede resultar tedioso, especialmente en ejecuciones repetidas, y también dificulta la automatización. Afortunadamente, existen algunas alternativas para estas situaciones.


### 4.2 Uso de Ansible Vault con un archivo de contraseña

Si no desea escribir la contraseña de Vault cada vez que ejecuta una tarea, puede agregar su contraseña de Vault a un archivo y hacer referencia al archivo durante la ejecución.

Por ejemplo, podrías poner tu contraseña en un .vault_pass archivo como este:

``` 
echo 'my_vault_password' > .vault_pass
``` 

Si está utilizando el control de versiones, asegúrese de agregar el archivo de contraseña al archivo de ignorar de su software de control de versiones para evitar cometerlo accidentalmente:

``` 
echo '.vault_pass' >> .gitignore
``` 

Ahora, en su lugar, puede hacer referencia al archivo. La --vault-password-file bandera está disponible en la línea de comando. Podríamos completar la misma tarea de la última sección escribiendo:

``` 
ansible --vault-password-file=.vault_pass -bK -m copy -a 'src=secret_key dest=/tmp/secret_key mode=0600 owner=root group=root' localhost
``` 

Esta vez no se le solicitará la contraseña de Vault.

``` 
Output
localhost | SUCCESS => {
    "changed": false,
    "checksum": "52d7a243aea83e6b0e478db55a2554a8530358b0",
    "dest": "/tmp/secret_key",
    "gid": 80,
    "group": "root",
    "mode": "0600",
    "owner": "root",
    "path": "/tmp/secret_key",
    "size": 8,
    "state": "file",
    "uid": 0
}
``` 

### 4.3 Leer el archivo de contraseña automáticamente

Para evitar tener que proporcionar una marca, puede configurar la ANSIBLE_VAULT_PASSWORD_FILE variable de entorno con la ruta al archivo de contraseña:

``` 
export ANSIBLE_VAULT_PASSWORD_FILE=./.vault_pass
``` 

Ahora debería poder ejecutar el comando sin la --vault-password-file bandera para la sesión actual:

``` 
ansible -bK -m copy -a 'src=secret_key dest=/tmp/secret_key mode=0600 owner=root group=root' localhost
``` 

Para que Ansible conozca la ubicación del archivo de contraseña entre sesiones, puede editar su ansible.cfgarchivo.

Abra el ansible.cfg archivo local que creamos anteriormente:

``` 
nano ansible.cfg
``` 

En la [defaults] sección, establezca la vault_password_file configuración. Señale la ubicación de su archivo de contraseña. Puede ser una ruta relativa o absoluta, según cuál le resulte más útil:

``` 
[defaults]
. . .
vault_password_file = ./.vault_pass
``` 

Ahora, cuando ejecute comandos que requieran descifrado, ya no se le solicitará la contraseña de la bóveda. Como beneficio adicional, ansible-vault no solo usará la contraseña del archivo para descifrar cualquier archivo, sino que también aplicará la contraseña al crear nuevos archivos con ansible-vault create y ansible-vault encrypt.

### 4.4 Leer la contraseña de una variable de entorno

Es posible que le preocupe enviar accidentalmente su archivo de contraseña a su repositorio. Si bien Ansible tiene una variable de entorno para señalar la ubicación de un archivo de contraseña, no tiene una para configurar la contraseña.

Sin embargo, si su archivo de contraseña es ejecutable, Ansible lo ejecutará como un script y utilizará el resultado resultante como contraseña. En una edición de GitHub , Brian Schwind sugiere que se puede utilizar el siguiente script para extraer la contraseña de una variable de entorno.

Abre tu .vault_passarchivo en tu editor:

``` 
nano .vault_pass
``` 

Reemplace el contenido con el siguiente script:

``` 
#!/usr/bin/env python3

import os
print os.environ['VAULT_PASSWORD']
``` 

Haga que el archivo sea ejecutable escribiendo:

``` 
chmod +x .vault_pass
``` 

Luego puede configurar y exportar la VAULT_PASSWORD variable de entorno, que estará disponible para su sesión actual:

``` 
export VAULT_PASSWORD=my_vault_password
``` 

Tendrá que hacer esto al comienzo de cada sesión de Ansible, lo que puede parecer inconveniente. Sin embargo, esto protege eficazmente contra la confirmación accidental de su contraseña de cifrado de Vault, lo que podría tener serios inconvenientes.


## 5. Uso de variables cifradas en Vault con variables regulares

Si bien Ansible Vault se puede utilizar con archivos arbitrarios, se utiliza con mayor frecuencia para proteger variables confidenciales. Trabajaremos con un ejemplo para mostrarle cómo transformar un archivo de variables normal en una configuración que equilibre la seguridad y la usabilidad.

### 5.1 Configurando el ejemplo

Para este ejemplo, simularemos que estamos configurando un servidor de base de datos, sin necesidad de instalar una base de datos como requisito previo.

Cuando creó el hosts archivo anteriormente, colocó la localhost entrada en un grupo llamado database para prepararse para este paso. Las bases de datos normalmente requieren una combinación de variables sensibles y no sensibles. Estos se pueden asignar en un group_vars directorio en un archivo con el nombre del grupo:

``` 
mkdir -p group_vars
nano group_vars/database
``` 

Dentro del group_vars/database archivo, agregaremos algunas variables típicas. Algunas variables, como el número de puerto de MySQL, no son secretas y se pueden compartir libremente. Otras variables, como la contraseña de la base de datos, serán confidenciales:

``` 
---
# nonsensitive data
mysql_port: 3306
mysql_host: 10.0.0.3
mysql_user: fred

# sensitive data
mysql_password: supersecretpassword
``` 

Podemos probar que todas las variables están disponibles para nuestro host con el debug módulo de Ansible y la hostvars variable:

``` 
ansible -m debug -a 'var=hostvars[inventory_hostname]' database
``` 

``` 
Output
localhost | SUCCESS => {
    "hostvars[inventory_hostname]": {
        "ansible_check_mode": false, 
        "ansible_version": {
            "full": "2.2.0.0", 
            "major": 2, 
            "minor": 2, 
            "revision": 0, 
            "string": "2.2.0.0"
        }, 
        "group_names": [
            "database"
        ], 
        "groups": {
            "all": [
                "localhost"
            ], 
            "database": [
                "localhost"
            ], 
            "ungrouped": []
        }, 
        "inventory_dir": "/home/sammy", 
        "inventory_file": "hosts", 
        "inventory_hostname": "localhost", 
        "inventory_hostname_short": "localhost", 
        "mysql_host": "10.0.0.3",
        "mysql_password": "supersecretpassword",
        "mysql_port": 3306,
        "mysql_user": "fred",
        "omit": "__omit_place_holder__1c934a5a224ca1d235ff05eb9bda22044a6fb400", 
        "playbook_dir": "."
    }
}
``` 

El resultado confirma que todas las variables que configuramos se aplican al host. Sin embargo, nuestro group_vars/database archivo actualmente contiene todas nuestras variables. Esto significa que podemos dejarlo sin cifrar, lo cual es un problema de seguridad debido a la variable de contraseña de la base de datos, o cifrar todas las variables, lo que crea problemas de usabilidad y colaboración.


### 5.2 Mover variables sensibles a Ansible Vault

Para resolver este problema, debemos hacer una distinción entre variables sensibles y no sensibles. Deberíamos poder cifrar valores confidenciales y al mismo tiempo compartir fácilmente nuestras variables no sensibles. Para hacerlo, dividiremos nuestras variables entre dos archivos.

Es posible utilizar un directorio de variables en lugar de un archivo de variables de Ansible para aplicar variables de más de un archivo. Podemos refactorizar nuestra configuración para aprovechar esa capacidad. Primero, cambie el nombre del archivo existente de databasea vars. Este será nuestro archivo de variables sin cifrar:

``` 
mv group_vars/database group_vars/vars
``` 

A continuación, cree un directorio con el mismo nombre que el archivo de variables anterior. Mueva el vars archivo dentro:

``` 
mkdir group_vars/database
mv group_vars/vars group_vars/database/
``` 

Ahora tenemos un directorio de variables para el database grupo en lugar de un único archivo y tenemos un único archivo variable sin cifrar. Dado que cifraremos nuestras variables confidenciales, debemos eliminarlas de nuestro archivo no cifrado. Edite el group_vars/database/vars archivo para eliminar los datos confidenciales:

``` 
nano group_vars/database/vars
``` 

En este caso, queremos eliminar la mysql_password variable. El archivo ahora debería verse así:

``` 
---
# nonsensitive data
mysql_port: 3306
mysql_host: 10.0.0.3
mysql_user: fred
``` 

A continuación, cree un archivo cifrado en la bóveda dentro del directorio que convivirá junto al vars archivo no cifrado:

```
ansible-vault create group_vars/database/vault
``` 

En este archivo, defina las variables sensibles que solían estar en el vars archivo. Utilice los mismos nombres de variables, pero anteponga la cadena vault_para indicar que estas variables están definidas en el archivo protegido por almacén:

``` 
group_vars/base de datos/bóveda
---
vault_mysql_password: supersecretpassword
``` 

Guarde y cierre el archivo cuando haya terminado.

La estructura de directorio resultante tiene este aspecto:

``` 
.
├── . . .
├── group_vars/
│   └── database/
│       ├── vars
│       └── vault
└── . . .
``` 

En este punto, las variables están separadas y sólo se cifran los datos confidenciales. Esto es seguro, pero nuestra implementación ha afectado nuestra usabilidad. Si bien nuestro objetivo era proteger los valores confidenciales, también hemos reducido involuntariamente la visibilidad de los nombres reales de las variables. No está claro qué variables se asignan sin hacer referencia a más de un archivo y, si bien es posible que desee restringir el acceso a datos confidenciales mientras colabora, probablemente desee compartir los nombres de las variables.

Para abordar esto, el proyecto Ansible generalmente recomienda un enfoque ligeramente diferente.


### 5.3 Hacer referencia a variables de Vault desde variables no cifradas

Cuando movimos nuestros datos confidenciales al archivo protegido por bóveda, antepusimos los nombres de las variables con vault_( mysql_password se convirtió en vault_mysql_password). Podemos agregar los nombres de las variables originales ( mysql_password) nuevamente al archivo no cifrado. En lugar de establecerlos directamente en valores confidenciales, podemos usar declaraciones de plantillas de Jinja2 para hacer referencia a los nombres de las variables cifradas desde nuestro archivo de variables no cifradas. De esta manera, puede ver todas las variables definidas haciendo referencia a un solo archivo, pero los valores confidenciales permanecerán en el archivo cifrado.

Para demostrarlo, abra nuevamente el archivo de variables no cifradas:

``` 
nano group_vars/database/vars
``` 

Agregue la mysql_password variable nuevamente. Esta vez, use la plantilla Jinja2 para hacer referencia a la variable definida en el archivo protegido por bóveda:

``` 
---
# nonsensitive data
mysql_port: 3306
mysql_host: 10.0.0.3
mysql_user: fred

# sensitive data
mysql_password: "{{ vault_mysql_password }}"
``` 

La mysql_password variable se establecerá en el valor de la vault_mysql_password variable, que está definido en el archivo de almacén.

Con este método, puede comprender todas las variables que se aplicarán a los hosts del database grupo al ver el group_vars/database/vars archivo. Las partes sensibles quedarán oscurecidas por la plantilla Jinja2. Solo group_vars/database/vault es necesario abrirlo cuando es necesario ver o cambiar los valores mismos.

Puede verificar para asegurarse de que todas las mysql_* variables aún se apliquen correctamente utilizando el mismo método que la última vez.

**Nota:** Si su contraseña de Vault no se aplica automáticamente con un archivo de contraseña, agregue la --ask-vault-pass marca al siguiente comando.

``` 
ansible -m debug -a 'var=hostvars[inventory_hostname]' database
``` 

``` 
Output
localhost | SUCCESS => {
    "hostvars[inventory_hostname]": {
        "ansible_check_mode": false, 
        "ansible_version": {
            "full": "2.2.0.0", 
            "major": 2, 
            "minor": 2, 
            "revision": 0, 
            "string": "2.2.0.0"
        }, 
        "group_names": [
            "database"
        ], 
        "groups": {
            "all": [
                "localhost"
            ], 
            "database": [
                "localhost"
            ], 
            "ungrouped": []
        }, 
        "inventory_dir": "/home/sammy/vault", 
        "inventory_file": "./hosts", 
        "inventory_hostname": "localhost", 
        "inventory_hostname_short": "localhost", 
        "mysql_host": "10.0.0.3",
        "mysql_password": "supersecretpassword",
        "mysql_port": 3306,
        "mysql_user": "fred",
        "omit": "__omit_place_holder__6dd15dda7eddafe98b6226226c7298934f666fc8", 
        "playbook_dir": ".", 
        "vault_mysql_password": "supersecretpassword"
    }
}
``` 

Tanto el vault_mysql_password como el mysql_password son accesibles. Esta duplicación es inofensiva y no afectará el uso que usted haga de este sistema.


**Conclusión**

Sus proyectos deben tener toda la información necesaria para instalar y configurar sistemas complejos con éxito. Sin embargo, algunos datos de configuración son, por definición, confidenciales y no deben exponerse públicamente. En esta guía, demostramos cómo Ansible Vault puede cifrar información confidencial para que pueda mantener todos sus datos de configuración en un solo lugar sin comprometer la seguridad.








