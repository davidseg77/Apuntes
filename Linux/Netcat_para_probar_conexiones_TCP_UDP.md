# Cómo utilizar Netcat para establecer y probar conexiones TCP y UDP

## 1. Introducción

Linux es conocido por tener una gran cantidad de utilidades de línea de comandos útiles y maduras disponibles listas para usar en la mayoría de las distribuciones. A menudo, los administradores de sistemas pueden realizar gran parte de su trabajo utilizando las herramientas integradas sin tener que instalar software adicional.

En esta guía, analizaremos cómo utilizar la utilidad netcat. Este comando versátil puede ayudarlo a monitorear, probar y enviar datos a través de conexiones de red.

Netcat debería estar disponible en casi cualquier distribución moderna de Linux. Ubuntu viene con la variante BSD de netcat, y esto es lo que usaremos en esta guía. Otras versiones pueden funcionar de manera diferente o ofrecer otras opciones.


## 2. Sintaxis general

De forma predeterminada, netcat funciona iniciando una conexión TCP a un host remoto.

La sintaxis más básica es:

``` 
netcat [options] host port
``` 

Esto intentará iniciar una conexión TCP al host definido en el número de puerto especificado. Esto funciona de manera similar al antiguo telnet comando de Linux. Tenga en cuenta que su conexión no está cifrada en absoluto.

Si desea enviar un paquete UDP en lugar de iniciar una conexión TCP, puede utilizar la -u opción:

``` 
netcat -u host port
``` 

Puede especificar un rango de puertos colocando un guión entre el primero y el último:

``` 
netcat host startport-endport
``` 

Esto generalmente se usa con algunas banderas adicionales.

En la mayoría de los sistemas, podemos usar cualquiera de ellos netcat o ncindistintamente. Son alias del mismo comando.


## 3. Cómo utilizar Netcat para escanear puertos

Uno de los usos más comunes de netcat es como escáner de puertos.

Aunque netcat probablemente no sea la herramienta más sofisticada para el trabajo (nmap es una mejor opción en la mayoría de los casos), puede realizar escaneos de puertos simples para identificar fácilmente los puertos abiertos.

Hacemos esto especificando un rango de puertos para escanear, como hicimos anteriormente, junto con la -z opción de realizar un escaneo en lugar de intentar iniciar una conexión.

Por ejemplo, podemos escanear todos los puertos hasta el 1000 emitiendo este comando:

``` 
netcat -z -v domain.com 1-1000
``` 

Junto con la -z opción, también hemos especificado la -v opción para decirle a netcat que proporcione información más detallada.

La salida se verá así:

``` 
Output
nc: connect to domain.com port 1 (tcp) failed: Connection refused
nc: connect to domain.com port 2 (tcp) failed: Connection refused
nc: connect to domain.com port 3 (tcp) failed: Connection refused
nc: connect to domain.com port 4 (tcp) failed: Connection refused
nc: connect to domain.com port 5 (tcp) failed: Connection refused
nc: connect to domain.com port 6 (tcp) failed: Connection refused
nc: connect to domain.com port 7 (tcp) failed: Connection refused
. . .
Connection to domain.com 22 port [tcp/ssh] succeeded!
. . .
``` 

Como puede ver, esto proporciona mucha información y le indicará para cada puerto si el escaneo fue exitoso o no.

Si realmente está utilizando un nombre de dominio, este es el formulario que deberá utilizar.

Sin embargo, su escaneo será mucho más rápido si conoce la dirección IP que necesita. Luego puede usar la -n bandera para especificar que no necesita resolver la dirección IP usando DNS:

``` 
netcat -z -n -v 198.51.100.0 1-1000
``` 

Los mensajes devueltos en realidad se envían a un error estándar (consulte nuestro artículo sobre redirección de E/S para obtener más información). Podemos enviar los mensajes de error estándar a la salida estándar, lo que nos permitirá filtrar los resultados más fácilmente.

Redireccionaremos el error estándar a la salida estándar usando la 2>&1 sintaxis bash. Luego filtraremos los resultados con grep:

``` 
netcat -z -n -v 198.51.100.0 1-1000 2>&1 | grep succeeded
Output
Connection to 198.51.100.0 22 port [tcp/*] succeeded!
``` 

Aquí podemos ver que el único puerto abierto en el rango de 1 a 1000 en la computadora remota es el puerto 22, el puerto SSH tradicional.


## 4. Cómo comunicarse a través de Netcat

Netcat no se limita a enviar paquetes TCP y UDP. También puede escuchar en un puerto conexiones y paquetes. Esto nos da la oportunidad de conectar dos instancias de netcat en una relación cliente-servidor.

Qué ordenador es el servidor y cuál el cliente es sólo una distinción relevante durante la configuración inicial. Una vez establecida la conexión, la comunicación es exactamente igual en ambas direcciones.

En una máquina, puede decirle a netcat que escuche un puerto específico para las conexiones. Podemos hacer esto proporcionando el -l parámetro y eligiendo un puerto:

``` 
netcat -l 4444
``` 

Esto le indicará a netcat que escuche las conexiones TCP en el puerto 4444. Como usuario normal (no root), no podrá abrir ningún puerto inferior a 1000, como medida de seguridad.

En un segundo servidor, podemos conectarnos a la primera máquina en el número de puerto que elegimos. Hacemos esto de la misma manera que hemos estado estableciendo conexiones anteriormente:

``` 
netcat domain.com 4444
``` 

Parecerá como si nada hubiera pasado. Sin embargo, ahora puedes enviar mensajes en cualquier lado de la conexión y se verán en ambos extremos.

Escribe un mensaje y presiona ENTER. Aparecerá tanto en la pantalla local como en la remota. Esto también funciona en la dirección opuesta.

Cuando haya terminado de pasar mensajes, puede presionar CTRL-D para cerrar la conexión TCP.


## 5. Cómo enviar archivos a través de Netcat

A partir del ejemplo anterior, podemos realizar tareas más útiles.

Como estamos estableciendo una conexión TCP normal, podemos transmitir casi cualquier tipo de información a través de esa conexión. No se limita a los mensajes de chat que escribe un usuario. Podemos utilizar este conocimiento para convertir netcat en un programa de transferencia de archivos.

Una vez más, debemos elegir un extremo de la conexión para escuchar las conexiones. Sin embargo, en lugar de imprimir información en la pantalla, como hicimos en el último ejemplo, colocaremos toda la información directamente en un archivo:

``` 
netcat -l 4444 > received_file
``` 

En > este comando redirige toda la salida netcat al nombre de archivo especificado.

En la segunda computadora, cree un archivo de texto simple escribiendo:

``` 
echo "Hello, this is a file" > original_file
``` 

Ahora podemos usar este archivo como entrada para la conexión netcat que estableceremos con la computadora que escucha. El archivo se transmitirá tal como si lo hubiéramos tecleado de forma interactiva:

``` 
netcat domain.com 4444 < original_file
``` 

Podemos ver en el ordenador que estaba esperando conexión, que ahora tenemos un nuevo archivo llamado received_file con el contenido del archivo que escribimos en el otro ordenador:

``` 
cat received_file
Output
Hello, this is a file
``` 

Como puede ver, al canalizar cosas, podemos aprovechar fácilmente esta conexión para transferir todo tipo de cosas.

Por ejemplo, podemos transferir el contenido de un directorio completo creando un archivo tar sin nombre sobre la marcha, transfiriéndolo al sistema remoto y descomprimiéndolo en el directorio remoto.

En el lado receptor, podemos anticipar que llegará un archivo que deberá descomprimirse y extraerse escribiendo:

``` 
netcat -l 4444 | tar xzvf -
``` 

El guión final (-) significa que tar funcionará en la entrada estándar, que se canaliza desde netcat a través de la red cuando se realiza una conexión.

Del lado del contenido del directorio que queremos transferir, podemos empaquetarlo en un tarball y luego enviarlo a la computadora remota a través de netcat:

``` 
tar -czf - * | netcat domain.com 4444
``` 

Esta vez, el guión en el comando tar significa tar y comprimir el contenido del directorio actual (como lo especifica el comodín *) y escribir el resultado en la salida estándar.

Luego, esto se escribe directamente en la conexión TCP, que luego se recibe en el otro extremo y se descomprime en el directorio actual de la computadora remota.

Este es sólo un ejemplo de cómo transferir datos más complejos de una computadora a otra. Otra idea común es utilizar el dd comando para crear una imagen de un disco en un lado y transferirlo a una computadora remota. Sin embargo, no cubriremos esto aquí.


## 6. Cómo utilizar Netcat como servidor web sencillo

Hemos estado configurando netcat para escuchar conexiones con el fin de comunicarnos y transferir archivos. Podemos utilizar este mismo concepto para operar netcat como un servidor web muy simple. Esto puede resultar útil para probar las páginas que está creando.

Primero, creemos un archivo HTML simple en un servidor:

``` 
nano index.html
``` 

Aquí hay algo de HTML simple que puede usar en su archivo:

``` 
<html>
        <head>
                <title>Test Page</title>
        </head>
        <body>
                <h1>Level 1 header</h1>
                <h2>Subheading</h2>
                <p>Normal text here</p>
        </body>
</html>
``` 

Guarde y cierre el archivo.

Sin privilegios de root, no puede publicar este archivo en el puerto web predeterminado, el puerto 80. Podemos elegir el puerto 8888 como usuario normal.

Si solo desea mostrar esta página una vez para comprobar cómo se representa, puede ejecutar el siguiente comando:

``` 
printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | netcat -l 8888
``` 

Ahora, en tu navegador, puedes acceder al contenido visitando:

``` 
http://server_IP:8888
``` 

Esto servirá a la página y luego se cerrará la conexión netcat. Si intenta actualizar la página, desaparecerá.

Podemos hacer que netcat sirva la página indefinidamente envolviendo el último comando en un bucle infinito, como este:

while true; do printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | netcat -l 8888; done
Esto le permitirá continuar recibiendo conexiones después de que se cierre la primera conexión.

Podemos detener el ciclo escribiendo CTRL-Cen el servidor.

Esto le permite ver cómo se representa una página en un navegador, pero no proporciona mucha más funcionalidad. Nunca debes usar esto para servir sitios web reales. No hay seguridad y cosas simples como los enlaces ni siquiera funcionan correctamente.


## 7. Conclusión

Ahora debería tener una idea bastante clara de para qué se puede utilizar netcat. Es una herramienta versátil que puede resultar útil para diagnosticar problemas y verificar que la funcionalidad de nivel básico esté funcionando correctamente con conexiones TCP/UDP.

Con netcat, puede comunicarse entre diferentes computadoras muy fácilmente para lograr interacciones rápidas. Netcat intenta hacer que las interacciones de red sean transparentes entre computadoras eliminando la complejidad de formar conexiones.

