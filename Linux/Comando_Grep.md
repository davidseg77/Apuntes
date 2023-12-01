# Comando Grep en Linux/UNIX

## 1. Comando Grep en Linux

El comando grep se puede utilizar para buscar una expresión regular o una cadena en un archivo de texto. Para demostrar esto, creemos un archivo de texto bienvenido.txt y agreguemos contenido como se muestra.

``` 
Welcome to Linux !
Linux is a free and opensource Operating system that is mostly used by
developers and in production servers for hosting crucial components such as web
and database servers. Linux has also made a name for itself in PCs.
Beginners looking to experiment with Linux can get started with friendlier linux
distributions such as Ubuntu, Mint, Fedora and Elementary OS.
``` 

¡Excelente! Ahora estamos listos para ejecutar algunos comandos grep y manipular la salida para obtener los resultados deseados. Para buscar una cadena en un archivo, ejecute el siguiente comando Sintaxis

``` 
$ grep "string" file name
``` 

O

``` 
$ filename grep "string"
``` 

Ejemplo :

``` 
$ grep "Linux" welcome.txt
``` 

Como puede ver, grep no solo buscó y encontró coincidencias con la cadena “Linux”, sino que también imprimió las líneas en las que aparece la cadena. Si el archivo se encuentra en una ruta de archivo diferente, asegúrese de especificar la ruta del archivo como se muestra a continuación

``` 
$ grep "string" /path/to/file
``` 


## 2. Colorear los resultados de Grep usando la opción --color

Si está trabajando en un sistema que no muestra la cadena o patrón de búsqueda en un color diferente al resto del texto, utilice --color para que sus resultados se destaquen. Ejemplo

``` 
$ grep --color "free and opensource" welcome.txt 
``` 


## 3. Buscando una cadena de forma recursiva en todos los directorios

Si desea buscar una cadena en su directorio actual y en todos los demás subdirectorios, busque usando la - r bandera como se muestra

``` 
$ grep -r "string-name" *
``` 

Por ejemplo

``` 
$ grep -r "linux" *
``` 


## 4. Ignorar la distinción entre mayúsculas y minúsculas

En el ejemplo anterior, los resultados de nuestra búsqueda nos dieron lo que queríamos porque la cadena "Linux" se especificó en mayúsculas y también existe en el archivo en mayúsculas. Ahora intentemos buscar la cadena en minúsculas.

``` 
$ grep "linux" file name
``` 

Nada de la salida, ¿verdad? Esto se debe a que grepping no pudo encontrar ni hacer coincidir la cadena "linux" ya que la primera letra está en minúscula. Para ignorar la distinción entre mayúsculas y minúsculas, use la -i bandera y ejecute el siguiente comando

``` 
$ grep -i "linux" welcome.txt
``` 

Normalmente -i se utiliza para mostrar cadenas independientemente de su distinción entre mayúsculas y minúsculas.


## 5. Cuente las líneas donde las cadenas coinciden con la opción -c

Para contar el número total de líneas donde aparece o reside el patrón de cadena, ejecute el siguiente comando

``` 
$ grep -c "Linux" welcome.txt
``` 


## 6. Usando Grep para invertir la salida

Para invertir la salida de Grep, use la -v bandera. La -v opción le indica a grep que imprima todas las líneas que no contengan o no coincidan con la expresión. La opción –v le dice a grep que invierta su salida, lo que significa que en lugar de imprimir líneas coincidentes, haga lo contrario e imprima todas las líneas que no coincidan con la expresión. Volviendo a nuestro archivo, mostremos los números de línea como se muestra. Presione ESC en el editor Vim, escriba dos puntos seguidos de

``` 
set nu
``` 

A continuación, presione Enter para mostrar las líneas que no contienen la cadena "Linux" ejecutada

``` 
$ grep -v "Linux" welcome.txt
``` 

Como puede ver, grep ha mostrado las líneas que no contienen el patrón de búsqueda.


## 7. Numere las líneas que contienen el patrón de búsqueda con la opción -n

Para numerar las líneas donde coincide el patrón de cadena, use la -n opción como se muestra

``` 
$ grep -n "Linux" welcome.txt
``` 

## 8. Busque palabras que coincidan exactamente usando la opción -w

Al pasar -w la bandera se buscará la línea que contiene la palabra coincidente exacta como se muestra

``` 
$ grep -w "opensource" welcome.txt
``` 

Sin embargo, si intentas

``` 
$ grep -w "open" welcome.txt
``` 

¡NO se devolverán resultados porque no estamos buscando un patrón sino una palabra exacta!


## 9. Usando tuberías con grep

El comando grep se puede utilizar junto con tuberías para obtener resultados distintos. Por ejemplo, si desea saber si un determinado paquete está instalado en el sistema Ubuntu, ejecute

``` 
$ dpkg -L | grep "package-name"
``` 

Por ejemplo, para saber si OpenSSH se ha instalado en su sistema, ejecute el dpkg -l comando grep como se muestra

``` 
$ dpkg -L | grep -i "openssh"
``` 


## 10. Mostrar el número de líneas antes o después de un patrón de búsqueda Usando tuberías

Puede utilizar -A o -B para mostrar el número de líneas que preceden o vienen después de la cadena de búsqueda. El indicador -A indica las líneas que vienen después de la cadena de búsqueda y -B imprime la salida que aparece antes de la cadena de búsqueda. Por ejemplo

``` 
$ ifconfig | grep -A 4 ens3
``` 

Este comando muestra la línea que contiene la cadena más 4 líneas de texto después de la cadena ens en el ifconfig comando. Por el contrario, en el siguiente ejemplo, el uso del indicador -B mostrará la línea que contiene la cadena de búsqueda más 3 líneas de texto antes de la cadena ether en el ifconfig comando. 

``` 
$ ifconfig | grep -B 4 ether
``` 


## 11. Usando grep con expresiones regulares (REGEX)

El término REGEX es un acrónimo de expresión REG ular EX. Un REGEX es una secuencia de caracteres que se utiliza para hacer coincidir un patrón. Abajo hay algunos ejemplos:

``` 
^      Matches characters at the beginning of a line
$      Matches characters at the end of a line
"."    Matches any character
[a-z]  Matches any characters between A and Z
[^ ..] Matches anything apart from what is contained in the brackets
``` 

Ejemplo Para imprimir líneas que comienzan con un determinado carácter, la sintaxis es;

``` 
grep ^character file_name
```

Por ejemplo, para mostrar las líneas que comienzan con la letra "d" en nuestro archivo bienvenido.txt, ejecutaríamos

``` 
$ grep ^d welcome.txt 
``` 

Para mostrar líneas que terminan con la letra 'x', ejecute

``` 
$ grep x$ welcome.txt
``` 


## 12. Obtener ayuda con más opciones de Grep

Si necesita obtener más información sobre el uso del comando Grep, ejecute el siguiente comando para obtener una vista previa de otras banderas u opciones que puede usar junto con el comando.

``` 
$ grep --help
``` 

