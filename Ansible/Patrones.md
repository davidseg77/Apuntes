# Patrones: segmentación por hosts y grupos

Cuando ejecuta Ansible a través de un comando ad hoc o ejecutando un libro de jugadas, debe elegir contra qué nodos o grupos administrados desea ejecutar. Los patrones le permiten ejecutar comandos y guías contra hosts y/o grupos específicos en su inventario. Un patrón de Ansible puede hacer referencia a un único host, una dirección IP, un grupo de inventario, un conjunto de grupos o todos los hosts de su inventario. Los patrones son muy flexibles: puede excluir o requerir subconjuntos de hosts, usar comodines o expresiones regulares, y más. Ansible se ejecuta en todos los hosts de inventario incluidos en el patrón.

## 1. Usando patrones

Utiliza un patrón casi cada vez que ejecuta un comando ad hoc o un libro de jugadas. El patrón es el único elemento de un comando ad hoc que no tiene bandera. Suele ser el segundo elemento:

``` 
ansible <pattern> -m <module_name> -a "<module options>"
``` 

Por ejemplo:

``` 
ansible webservers -m service -a "name=httpd state=restarted"
``` 

En un libro de jugadas, el patrón es el contenido de la hosts:línea de cada jugada:

``` 
- name: <play_name>
  hosts: <pattern>
``` 

Por ejemplo:

``` 
- name: restart webservers
  hosts: webservers
``` 

Dado que a menudo desea ejecutar un comando o un libro de jugadas en varios hosts a la vez, los patrones a menudo se refieren a grupos de inventario. Tanto el comando ad hoc como el libro de jugadas anterior se ejecutarán en todas las máquinas del webserversgrupo.

### 1.1 Patrones comunes

Una vez que conozcas los patrones básicos, podrás combinarlos. Este ejemplo:

``` 
webservers:dbservers:&staging:!phoenix
``` 

se dirige a todas las máquinas de los grupos 'webservers' y 'dbservers' que también están en el grupo 'staging', excepto las máquinas del grupo 'phoenix'.

Puede utilizar patrones comodín con FQDN o direcciones IP, siempre que los hosts estén nombrados en su inventario por FQDN o dirección IP:

``` 
192.0.*
*.example.com
*.com
``` 

Puedes mezclar patrones y grupos comodín al mismo tiempo:

``` 
one*.com:dbservers
``` 

### 1.2 Limitaciones de patrones

Los patrones dependen del inventario. Si un host o grupo no aparece en su inventario, no puede utilizar un patrón para orientarlo. Si su patrón incluye una dirección IP o un nombre de host que no aparece en su inventario, verá un error como este:

``` 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: Could not match supplied host pattern, ignoring: *.not_in_inventory.com
``` 

Su patrón debe coincidir con la sintaxis de su inventario. Si define un host como un alias :

``` 
atlanta:
  host1:
    http_port: 80
    maxRequestsPerChild: 808
    host: 127.0.0.2
``` 

debes usar el alias en tu patrón. En el ejemplo anterior, debes usarlo host1en tu patrón. Si utiliza la dirección IP, volverá a recibir el error:

``` 
[WARNING]: Could not match supplied host pattern, ignoring: 127.0.0.2
``` 

### 1.3 Orden de procesamiento de patrones

El procesamiento es un poco especial y ocurre en el siguiente orden:

1. :y,

2. &

3. !

Este posicionamiento sólo tiene en cuenta el procesamiento del pedido dentro de cada operación: a:b:&c:!d:!e == &c:a:!d:b:!e == !d:a:!e:&c:b

Todo esto da como resultado lo siguiente:

El host en/es (a o b) Y el host en/es todo(c) Y el host NO en/es todo(d, e).

Ahora a:b:!e:!d:&chay un ligero cambio ya que se !eprocesa antes que el !d, aunque esto no hace mucha diferencia:

El host en/es (a o b) Y el host en/es todo(c) Y el host NO en/es todo(e, d).


## 2. Opciones de patrón avanzadas

Los patrones comunes descritos anteriormente satisfarán la mayoría de sus necesidades, pero Ansible ofrece otras formas de definir los hosts y grupos a los que desea dirigirse.

### 2.1 Usar variables en patrones

Puede utilizar variables para permitir el paso de especificadores de grupo con el -eargumento de ansible-playbook:

``` 
webservers:!{{ excluded }}:&{{ required }}
``` 

### 2.2 Usar la posición del grupo en patrones

Puede definir un host o un subconjunto de hosts según su posición en un grupo. Por ejemplo, dado el siguiente grupo:

```
[webservers]
cobweb
webbing
weber
``` 

puede utilizar subíndices para seleccionar hosts o rangos individuales dentro del grupo de servidores web.

### 2.3 Cortar en artículos específicos

Operación: s[i]

Resultado: i-th elemento de sdónde está el origen de indexación0

Si i es negativo, el índice es relativo al final de la secuencia s : se sustituye. Sin embargo lo es .len(s) + i-00

``` 
webservers[0]       # == cobweb
webservers[-1]      # == weber
``` 

### 2.4 Cortar con puntos de inicio y fin

Operación: s[i:j]

Resultado: porción de sdesde iaj

La porción de s de i a j se define como la secuencia de elementos con índice k tal que . Si se omite i , utilice . Si se omite j , utilice . El segmento que omite tanto i como j da como resultado un patrón de host no válido. Si i es mayor que j , el sector está vacío. Si i es igual a j , se sustituye s[i] .i <= k <= j0len(s)

``` 
webservers[0:2]     # == webservers[0],webservers[1],webservers[2]
                    # == cobweb,webbing,weber
webservers[1:2]     # == webservers[1],webservers[2]
                    # == webbing,weber
webservers[1:]      # == webbing,weber
webservers[:3]      # == cobweb,webbing,weber
``` 

### 2.5 Usar expresiones regulares en patrones

Puede especificar un patrón como expresión regular comenzando el patrón con ~:

``` 
~(web|db).*\.example\.com
``` 

## 3. Patrones y comandos ad-hoc

Puede cambiar el comportamiento de los patrones definidos en comandos ad-hoc usando opciones de línea de comandos. También puedes limitar los hosts a los que apuntas en una carrera en particular con la --limitbandera.

**Límite a un host**

``` 
$ ansible all -m <module> -a "<module options>" --limit "host1"
``` 

**Limitar a múltiples hosts**

``` 
$ ansible all -m <module> -a "<module options>" --limit "host1,host2"
``` 

**Límite negado. Tenga en cuenta que DEBEN usarse comillas simples para evitar la interpolación de bash.**

``` 
$ ansible all -m <module> -a "<module options>" --limit 'all:!host1'
``` 

**Limitar al grupo anfitrión**

``` 
$ ansible all -m <module> -a "<module options>" --limit 'group1'
``` 

## 4. Patrones y banderas del libro de jugadas ansible

Puede cambiar el comportamiento de los patrones definidos en los libros de jugadas utilizando opciones de línea de comandos. Por ejemplo, puede ejecutar un libro de jugadas que se defina en un solo host especificando (tenga en cuenta la coma final). Esto funciona incluso si el host al que se dirige no está definido en su inventario, pero este método NO leerá su inventario en busca de variables vinculadas a este host y cualquier variable requerida por el libro de jugadas deberá especificarse manualmente en la línea de comando. También puedes limitar los hosts a los que te diriges en una ejecución particular con la bandera, que hará referencia a tu inventario:hosts: all-i 127.0.0.2,--limit

``` 
ansible-playbook site.yml --limit datacenter2
``` 

Finalmente, puede utilizar --limitpara leer la lista de hosts de un archivo anteponiendo el nombre del archivo con @:

```
ansible-playbook site.yml --limit @retry_hosts.txt
``` 

Si RETRY_FILES_ENABLED está configurado en True, .retryse creará un archivo después de la ansible-playbookejecución que contiene una lista de hosts fallidos de todas las reproducciones. Este archivo se sobrescribe cada vez que ansible-playbooktermina de ejecutarse.

``` 
ansible-playbook site.yml --limit @site.retry
``` 

