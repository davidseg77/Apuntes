# Info del curso Aprende a programar con Python 3


## 1. Introducción

### 1.1 Mi primer programa en python3

**Uso del interprete**

Al instalar python3 el ejecutable del interprete lo podemos encontrar en /usr/bin/python3. Este directorio por defecto está en el PATH, por lo tanto lo podemos ejecutar directamente en el terminal. Por lo tanto para entrar en el modo interactivo, donde podemos ejecutar instrucción por instrucción interactivamente, ejecutamos:

``` 
$ python3
Python 3.4.2 (default, Oct  8 2014, 10:45:20) 
[GCC 4.9.1] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
``` 

En el modo interactivo, la última expresión impresa es asignada a la variable _.

``` 
>>> 4 +3
7
>>> 3 + _
10
``` 

Si tenemos nuestro programa en un fichero fuente (suele tener extensión py), por ejemplo programa.py,lo ejecutaríamos de la siguiente manera.

``` 
$ python3 programa.py
``` 

Por defecto la codificación de nuestro código fuente es UTF-8, por lo que no debemos tener ningún problema con los caracteres utilizados en nuestro programaos. Si por cualquier motivo necesitamos cambiar la codificación de los caracteres, en la primera línea de nuestro programa necesitaríamos poner:

``` 
# -*- coding: encoding -
``` 

Por ejemplo:

``` 
# -*- coding: cp-1252 -*-
``` 

**Escribimos un programa**

Un ejemplo de nuestro primer programa, podría ser este "hola mundo" un poco modificado:

``` 
numero = 5
if numero == 5:
	print ("Hola mundo!!!")
``` 

La indentación de la última línea es importante (se puede hacer con espacios o con tabulador), en python se utiliza para indicar bloques de instrucciones definidas por las estructuras de control (if, while, for, ...).

Para ejecutar este programa (guardado en hola.py):

``` 
$ python3 hola.py
$ Hola mundo!!!
``` 

**Ejecución de programas usando shebang**

Podemos ejecutar directamente el fichero utilizando en la primera línea el shebang, donde se indica el ejecutable que vamos a utilizar.

``` 
#!/usr/bin/python3
``` 

También podemos usar el programa env para preguntar al sistema por la ruta el interprete de python:

```
#!/usr/bin/env python
``` 

Por supuesto tenemos que dar permisos de ejecución al fichero.

``` 
$ chmod +x hola.py

$ ./hola.py
$ Hola mundo!!!
``` 

### 1.2 Estructura del programa

Un programa python está formado por instrucciones que acaban en un caracter de "salto de línea".
El punto y coma “;” se puede usar para separar varias sentencias en una misma línea, pero no se aconseja su uso.

Una línea empiza en la primera posición, si tenemos instrucciones dentro de un bloque de una estrucura de control de flujo habra que hacer una identación.
La identación se puede hacer con espacios y tabulaciones pero ambos tipos no se pueden mezclar. Se recomienda usar 4 espacios.

La barra invertida "\" al final de línea se emplea para dividir una línea muy larga en dos o más líneas.
Las expresiones entre paréntesis "()", llaves "{}" y corchetes "[]" separadas por comas "," se pueden escribir ocupando varias líneas.
Cuando el bloque a sangrar sólo ocupa una línea ésta puede escribirse después de los dos punto.
Comentarios

Se utiliza el caracter # para indicar los comentarios.

```
Palabras reservadas
False      class      finally    is         return
None       continue   for        lambda     try
True       def        from       nonlocal   while
and        del        global     not        with
as         elif       if         or         yield
assert     else       import     pass
break      except     in         raise
``` 

**Ejemplo**

``` 
#!/usr/bin/env python

# Sangrado con 4 espacios	

edad = 23
if edad > =18:
   print('Es mayor de edad')  
else:
   print('Es menor de edad')	

# Cada bloque de instrucciones dentro de una estructura de control
# debe estar tabulada	

if num >=0:
	while num<10:
		print (num)
		num = num +1	

# El punto y coma “;” se puede usar para separar varias sentencias 
# en una misma línea, pero no se aconseja su uso.	

edad = 15; print(edad)	

# Cuando el bloque a sangrar sólo ocupa una línea ésta puede
# escribirse después de los dos puntos:   	

if azul: print('Cielo')	

# La barra invertida “\” permite escribir una línea de
# código demasiado extensa en varias líneas:	

if condicion1 and condicion2 and condicion3 and \  
    condicion4 and condicion5:	

# Las expresiones entre paréntesis, llaves o corchetes pueden 
# ocupar varias líneas:	

dias = ['lunes', 'martes', 'miércoles', 'jueves',
        'viernes', 'sábado', 'domingo'] 
``` 

### 1.3 Funciones y constantes predefinidas

**Funciones predefinidas**

Tenemos una serie de funciones predefinidas en python3:

``` 
abs() 		  dict() 		help()		 min() 		setattr()
all() 		  dir() 		hex() 		 next() 	slice()
any() 		  divmod() 		id() 		 object() 	sorted()
ascii() 	  enumerate()		input() 	 oct() 		staticmethod()
bin() 		  eval() 		int() 		 open() 	str()
bool()		  exec() 		isinstance() 	ord() 		sum()
bytearray()	  filter() 		issubclass() 	pow() 		super()
bytes() 	  float() 		iter() 	 	 print() 	tuple()
callable() 	  format() 		len() 		 property() 	type()
chr() 		  frozenset() 		list() 		 range() 	vars()
classmethod()	  getattr() 		locals() 	 repr() 	zip()
compile() 	  globals() 		map() 		 reversed()		__import__()
complex() 	  hasattr() 		max() 		 round() 	 
delattr() 	  hash() 		memoryview() 	set() 
``` 

Todas estas funciones y algunos elmentos comunes del lenguaje están definidas en el módulo builtins.

**Algunos ejemplos de funciones**

La entrada y salida de información se hacen con la función print y la función input:
Tenemos algunas funciones matemáticas como: abs, divmod, hex, max, min,...
Hay funciones que trabajan con caracteres y cadenas: ascii, chr, format, repr,...
Además tenemos funciones que crean o convierten a determinados tipos de datos: int, float, str, bool, range, dict, list, ...

**Constantes predefinidas**

En el módulo builtins se definen las siguientes constantes:

* True y False: Valores booleans
* None especifica que alguna variables u objeto no tiene asignado ningún tipo.

**Ayuda en python**

Un función fundamental cuando queremos obtener información sobre los distintos aspectos del lenguaje es help. Podemos usarla entrar en una sesión interactiva:

``` 
>>> help()	

Welcome to Python 3.4's help utility!	

If this is your first time using Python, you should definitely check out
the tutorial on the Internet at http://docs.python.org/3.4/tutorial/.	

Enter the name of any module, keyword, or topic to get help on writing
Python programs and using Python modules.  To quit this help utility and
return to the interpreter, just type "quit".	

To get a list of available modules, keywords, symbols, or topics, type
"modules", "keywords", "symbols", or "topics".  Each module also comes
with a one-line summary of what it does; to list the modules whose name
or summary contain a given string such as "spam", type "modules spam".

help>
``` 

O pidiendo ayuda de una termino determinado, por ejemplo:

``` 
>>> help(print)
``` 

### 1.4 Datos

***Literales, variables y expresiones***

* Literales

Los literales nos permiten representar valores. Estos valores pueden ser de diferentes tipos, de esta manera tenemos diferentes tipos de literales:

**Literales numéricos**

Para representar números enteros utilizamos cifras enteras (Ejemplos: 3, 12, -23). Si queremos representarlos de forma binaria comenzaremos por la secuencia 0b (Ejemplos: 0b10101, 0b1100). La representación octal la hacemos comenzando por 0o (Ejemplos: 0o377, 0o7) y por último, la representación hexadecimal se comienza por 0x (Ejemplos: 0xdeadbeef, 0xfff).

Para los números reales utilizamos un punto para separar la parte entera de la decimal (12.3, 45.6). Podemos indicar que la parte decimal es 0, por ejemplo 10., o la parte entera es 0, por ejemplo .001, Para la representación de números muy grandes o muy pequeños podemos usar la representación exponencial (Ejemplos: 3.14e-10,1e100).

Por último, también podemos representar números complejos, con una parte real y otra imaginaria (Ejemplo: 1+2j)

**Literales cadenas**

Nos permiten representar cadenas de caracteres. Para delimitar las cadenas podemos usar el carácter ' o el carácter ". También podemos utilizar la combinación ''' cuando la cadena ocupa más de una línea. Ejemplos.

``` 
'hola que tal!'
"Muy bien"
'''Podemos \n
ir al cine'''
``` 

Las cadenas anteriores son del tipo str, si queremos representar una cadena de tipo byte podremos hacerlo de la siguiente manera:

``` 
b'Hola'
B"Muy bien"
``` 

Con el carácter /, podemos escapar algunos caracteres, veamos algunos ejemplos:

``` 
\\ 	Backslash (\) 	 
\' 	Single quote (') 	 
\" 	Double quote (") 	 
\a 	ASCII Bell (BEL) 	 
\b 	ASCII Backspace (BS) 	 
\f 	ASCII Formfeed (FF) 	 
\n 	ASCII Linefeed (LF) 	 
\r 	ASCII Carriage Return (CR) 	 
\t 	ASCII Horizontal Tab (TAB) 	 
\v 	ASCII Vertical Tab (VT)
``` 

**Variables**

Una variables es un identificador que referencia a un valor. Estudiaremos más adelante que python utiliza tipado dinámico, por lo tanto no se usa el concepto de variable como almacén de información. Para que una variable referencie a un valor se utiliza el operador de asignación =.

El nombre de una variable, ha de empezar por una letra o por el carácter guión bajo, seguido de letras, números o guiones bajos. No hay que declarar la variable antes de usarla, el tipo de la variable será el mismo que el del valor al que hace referencia. Por lo tanto su tipo puede cambiar en cualquier momento:

``` 
>>> var = 5
>>> type(var)
<class 'int'>
>>> var = "hola"
>>> type(var)
<class 'str'>
``` 

Hay que tener en cuanta que python distingue entre mayúsculas y minúsculas en el nombre de una variable, pero se recomienda usar sólo minúsculas.

**Definición, borrado y ámbito de variables**

Como hemos comentado anteriormente para crear una variable simplemente tenemos que utilizar un operador de asignación, el más utilizado = para que referencia un valor. Si queremos borrar la variable utilizamos la instrucción del. Por ejemplo:

``` 
>>> a = 5
>>> a
5
>>> del a
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
``` 

Podemos tener también variables que no tengan asignado ningún tipo de datos:

``` 
>>> a = None
>>> type(a)
<class 'NoneType'>
``` 

El ámbito de una variable se refiere a la zona del programa donde se ha definido y existe esa variable. Como primera aproximación las variables creadas dentro de funciones o clases tienen un ámbito local, es decir no existen fuera de la función o clase. Concretaremos cuando estudiamos estos aspectos más profundamente.

**Expresiones**

Una expresión es una combinación de variables, literales, operadores, funciones y expresiones, que tras su evaluación o cálculo nos devuelven un valor de un determinado tipo.

Veamos ejemplos de expresiones:

``` 
a + 7
(a ** 2) + b
``` 

**Operadores. Precedencia de operadores en python**

Los operadores que podemos utilizar se clasifican según el tipo de datos con los que trabajen y podemos poner algunos ejemplos:

``` 
+       -       *       **      /       //      %
<<      >>      &       |       ^       ~
<       >       <=      >=      ==      !=
-=      *=      /=      //=     %=
``` 

Podemos clasificaros en varios tipos:

``` 
Operadores aritméticos
Operadores de cadenas
Operadores de asignación
Operadores de comparación
Operadores lógicos
Operadores a nivel de bit
Operadores de pertenencia
Operadores de identidad
``` 

La procedencia de operadores es la siguiente:

* Los paréntesis rompen la procedencia.
* La potencia (**)
* Operadores unarios (~ + -)
* Multiplicar, dividir, módulo y división entera (* /% // )
* Suma y resta (+ -)
* Desplazamiento nivel de bit (>> <<)
* Operador binario AND (&)
* Operadores binario OR y XOR (^ |)
* Operadores de comparación (<= < > >=)
* Operadores de igualdad (<> == !=)
* Operadores de asignación (= %= /= //= -= += *= **=)
* Operadores de identidad (is, is not)
* Operadores de pertenencia (in, in not)
* Operadores lógicos (not, or, and)

**Función eval()**

La función eval() recibe una expresión como una cadena y la ejecuta.

``` 
>>> x=1
>>> eval("x + 1")
2
``` 

### 1.5 Trabajando con variables

Como hemos indicado anteriormente las variables en python no se declaran, se determina su tipo en tiempo de ejecución empleando una técnica que se lama tipado dinámico.

**¿Qué es el tipado dinámico?**

En python cuando asignamos una variable, se crea una referencia (puntero) al objeto creado, en ese momento se determina el tipo de la variable. Por lo tanto cada vez que asignamos de nuevo la variable puede cambiar el tipo en tempo de ejecución.

``` 
>>> var = 3
>>> type(var)
<class 'int'>
>>> var = "hola"
>>> type(var)
<class 'str'>
``` 

**Objetos inmutables y mutables**

* Objetos inmutables

Python procura no consumir más memoria que la necesaria. Ciertos objetos son inmutables, es decir, no pueden modificar su valor. El número 2 es siempre el núumero 2. Es un objeto inmutable. Python procura almacenar en memoria una sola vez cada valor inmutable. Si dos o más variables contienen ese valor, sus referencias apuntan a la misma zona de memoria.

- Ejemplo

Para comprobar esto, vamos a utilizar la función id, que nos devuelve el identificador de la variable o el objeto en memoria.

Veamos el siguiente código:

``` 
>>> a = 5
``` 

Podemos comprobar que a hace referencia al objeto 5.

``` 
>>> id(5)
10771648
>>> id(a)
10771648
``` 

Esto es muy distinto a otros lenguajes de programación, donde una variable ocupa un espacio de memoria que almacena un valor. Desde este punto cuando asigno otro número a la variable estoy cambiando la referencia.

``` 
>>> a = 6
>>> id(6)
10771680
>>> id(a)
10771680
``` 

Las cadenas también son un objeto inmutable, que lo sean tiene efectos sobre las operaciones que podemos efectuar con ellas. La asignación a un elemento de una cadena, por ejemplo está prohibida:

``` 
>>> a = "Hola"
>>> a[0]="h"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
``` 

De los tipos de datos principales, hay que recordar que son inmutables son los números, las cadenas o las tuplas.

* Objetos mutables
  
El caso contrario lo tenemos por ejemplo en los objetos de tipo listas, en este caso las listas son mutables. Se puede modificar un elemento de una lista.

- Ejemplo

``` 
>>> a = [1,2]
>>> b = a
>>> id(a)
140052934508488
>>> id(b)
140052934508488
``` 

Como anteriormente vemos que dos variables referencia a la misma lista e memoria. Pero aquí viene la diferencia, al poder ser modificada podemos encontrar situaciones como la siguiente:

``` 
>>> a[0] = 5
>>> b
[5, 2]
``` 

Cuando estudiamos las listas abordaremos este compartiendo de manera completa. De los tipos de datos principales, hay que recordar que son mutables son las listas y los diccionarios.

**Operadores de identidad**

Para probar esto de otra forma podemos usar los operadores de identidad:

* is: Devuelve True si dos variables u objetos están referenciando la misma posición de memoria. En caso contrario devuelve False.
* is not: Devuelve True si dos variables u objetos no están referenciando la misma posición de memoria. En caso contrario devuelve False.

- Ejemplo

``` 
>>> a = 5
>>> b = a
>>> a is b
True
>>> b = b +1
>>> a is b
False
>>> b is 6
True
``` 

**Operadores de asignación**

Me permiten asignar una valor a una variable, o mejor dicho: me permiten cambiar la referencia a un nuevo objeto.

El operador principal es =:

``` 
>>> a = 7
>>> a
7
``` 

Podemos hacer diferentes operaciones con la variable y luego asignar, por ejemplo sumar y luego asignar.

``` 
>>> a+=2
>>> a
9
``` 

Otros operadores de asignación: +=, -=, *=, /=, %=, **=, //=

**Asignación múltiple**

En python se permiten asignaciones múltiples de esta manera:

``` 
>>> a, b, c = 1, 2, "hola"
``` 

### 1.6 Entrada y salida estándar

**Función input**

No permite leer por teclado información. Devuelve una cadena de caracteres y puede tener como argumento una cadena que se muestra en pantalla.

- Ejemplos

``` 
>>> nombre=input("Nombre:")
Nombre:jose
>>> nombre
'jose'
>>> edad=int(input("Edad:"))
Edad:23
>>> edad
23
``` 

**Función print**

No permite escribir en la salida estándar. Podemos indicar varios datos a imprimir, que por defecto serán separado por un espacio (se puede indicar el separador) y por defecto se termina con un carácter salto de línea \n (también podemos indicar el carácter final). Podemos también imprimir varias cadenas de texto utilizando la concatenación.

- Ejemplos

``` 
>>> print(1,2,3)
1 2 3
>>> print(1,2,3,sep="-")
1-2-3
>>> print(1,2,3,sep="-",end=".")
1-2-3.>>> 

>>> print("Hola son las",6,"de la tarde")
Hola son las 6 de la tarde
>>> print("Hola son las "+str(6)+" de la tarde")
Hola son las 6 de la tarde
``` 

**Formateando cadenas de caracteres**

Existe dos formas de indicar el formato de impresión de las cadenas. En la documentación encontramos el estilo antiguo y el estilo nuevo.

- Ejemplos del estilo antiguo

``` 
>>> print("%d %f %s" % (2.5,2.5,2.5))
2 2.500000 2.5

>>> print("%s %o %x"%(bin(31),31,31))
0b11111 37 1f

>>> print("El producto %s cantidad=%d precio=%.2f"%("cesta",23,13.456))
El producto cesta cantidad=23 precio=13.46	
``` 

**Función format()**

Para utilizar el nuevo estilo en python3 tenemos una función format y un método format en la clase str. Vamos a ver algunos ejemplos utilizando la función format, cuando estudiemos los métodos de str lo estudiaremos con más detenimiento.

- Ejemplos

``` 
>>> print(format(31,"b"),format(31,"o"),format(31,"x"))
11111 37 1f

>>> print(format(2.345,".2f"))
2.35
``` 

## 2. Tipos de datos numéricos

### 2.1 Datos numéricos

Python3 trabaja con tres tipos numéricos:

* Enteros (int): Representan todos los números enteros (positivos, negativos y 0), sin parte decimal. En python3 este tipo no tiene limitación de espacio.
* Reales (float): Sirve para representar los números reales, tienen una parte decimal y otra decimal. Normalmente se utiliza para su implementación un tipo double de C.
* Complejos (complex): Nos sirven para representar números complejos, con una parte real y otra imaginaria.
Como hemos visto en la unidad anterior son tipos de datos inmutables.

- Ejemplos

``` 
>>> entero = 7
>>> type(entero)
<class 'int'>
>>> real = 7.2
>>> type (real)
<class 'float'
>>> complejo = 1+2j
>>> type(complejo)
<class 'complex'>
``` 

**Operadores aritméticos**

+: Suma dos números
-: Resta dos números
*: Multiplica dos números
/: Divide dos números, el resultado es float.
//: División entera
%: Módulo o resto de la división
**: Potencia
+, -: Operadores unarios positivo y negativo

**Funciones predefinidas que trabajan con números:**

* abs(x): Devuelve al valor absoluto de un número.
* divmod(x,y): Toma como parámetro dos números, y devuelve una tubla con dos valores, la división entera, y el módulo o resto de la división.
* hex(x): Devuelve una cadena con la representación hexadecimal del número que recibe como parámetro.
* oct(x): Devuelve una cadena con la representación octal del número que recibe como parámetro.
* bin(x): Devuelve una cadena con la representación binaria del número que recibe como parámetro.
* pow(x,y): Devuelve la potencia de la base x elevedao al exponete y. Es similar al operador **`.
* round(x,[y]): Devuelve un número real (float) que es el redondeo del número recibido como parámetro, podemos indicar un parámetro opcional que indica el número de decimales en el redondeo.

- Ejemplos

``` 
>>> abs(-7)
7
>>> divmod(7,2)
(3, 1)
>>> hex(255)
'0xff'
>>> oct(255)
'0o377'
>>> pow(2,3)
8
>>> round(7.567,1)
7.6
``` 

**Operadores a nivel de bit**

x | y: x OR y
x ^ y: x XOR y
x & y: a AND y
x << n: Desplazamiento a la izquierda n bits.
x >> n: Desplazamiento a la derecha n bits.
~x: Devuelve los bits invertidos.

**Conversión de tipos**

* int(x): Convierte el valor a entero.
* float(x): Convierte el valor a float.
* complex(x): Convierte el valor a un complejo sin parte imaginaria.
* complex(x,y): Convierta el valor a un complejo, cuya parte real es x y la parte imaginaria y.

Los valores que se reciben también pueden ser cadenas de caracteres (str).

- Ejemplos

``` 
>>> a=int(7.2)
>>> a
7
>>> type(a)
<class 'int'>
>>> a=int("345")
>>> a
345
>>> type(a)
<class 'int'>
>>> b=float(1)
>>> b
1.0
>>> type(b)
<class 'float'>
>>> b=float("1.234")
>>> b
1.234
>>> type(b)
<class 'float'>
``` 

Por último si queremos convertir una cadena a entero, la cadena debe estar formada por caracteres numéricos, sino es así, obtenemos un error:

``` 
a=int("123.3")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: '123.3'
``` 

### 2.2 Tipo de datos booleanos

**Tipo booleano**

El tipo booleano o lógico se considera en python3 como un subtipo del tipo entero. Se puede representar dos valores: verdadero o false (True, False).

**¿Qué valores se interpretan como FALSO?**

Cuando se evalua una expresión, hay determinados valores que se interpretan como False:

None
False
Cualquier número 0. (0, 0.0, 0j)
Cualquier secuencia vacía ([], (), '')
Cualquir diccionario vacío ({})

**Operadores booleanos**

Los operadores booleanos se utilizan para operar sobre expresiones booleanas y se suelen utilizar en las estructuras de control alternativas (if, while):

- x or y: Si x es falso entonces y, sino x. Este operados sólo evalua el segundo argumento si el primero es False.
- x and y: Si x es falso entonces x, sino y. Este operados sólo evalua el segundo argumento si el primero es True.
- not x: Si x es falso entonces True, sino False.

**Operadores de comparación**

== != >= > <= <

**Funciones all() y any()**

* all(iterador): Recibe un iterador, por ejemplo una lista, y devuelve True si todos los elementos son verdaderos o el iterador está vacío.
* any(iterador): Recibe un iterador, por ejemplo una lista, y devuelve True si alguno de sus elemento es verdadero, sino devuelve False.


## 3. Estructura de control

### 3.1 Alternativas

Si al evaluar la expresión lógica obtenermos el resultado True ejecuta un bloque de instrucciones, en otro caso ejecuta otro bloque.

**Alternativas simples**

``` 
if numero<0:
	print("Número es negativo")
``` 

**Alternativas dobles**

``` 
if numero<0:
	print("Número es negativo")	
else:
	print("Número es positivo")
``` 

**Alternativas múltiples**

``` 
if numero>0:
	print("Número es negativo")	
elif numero<0:
	print("Número es positivo")
else:
	print("Número es cero")
``` 

**Expresión reducida del if**

``` 
>>> lang="es"
>>> saludo = 'HOLA' if lang=='es' else 'HI'
>>> saludo
'HOLA'
``` 

### 3.2 Estructura de control repetitiva

**while**

La estructura while nos permite repetir un bloque de instrucciones mientras al evaluar una expresión lógica nos devuelve True. Puede tener una estructura else que se ejecutará al terminar el bucle.

- Ejemplo

``` 
año = 2001 
while año <= 2017: 
	print ("Informes del Año", año) 
	año += 1
else:
	print ("Hemos terminado")
for
``` 

La estructura for nos permite iterar los elementos de una secuencia (lista, rango, tupla, diccionario, cadena de caracteres,...). Puede tener una estructura else que se ejecutará al terminar el bucle.

- Ejemplo

``` 
for i in range(1,10):
    print (i)
else:
    print ("Hemos terminado")
``` 

**Instrucciones break, continue y pass**

* break

Termina la ejecución del bucle, además no ejecuta el bloque de instrucciones indicado por la parte else.

* continue

Deja de ejecutar las restantes instrucciones del bucle y vuelve a iterar.

* pass

Indica una instrucción nula, es decir no se ejecuta nada. Pero no tenemos errores de sintaxis.

**Recorriendo varias secuencias. Función zip()**

Con la instrucción for podemos ejecutar más de una secuencia, utilizando la función zip. Esta función crea una secuencia donde cada elemento es una tupla de los elementos de cada secuencia que toma cómo parámetro.

- Ejemplo

``` 
>>> list(zip(range(1,4),["ana","juan","pepe"]))
[(1, 'ana'), (2, 'juan'), (3, 'pepe')]
``` 

Para recorrerla:

``` 
>>> for x,y in zip(range(1,4),["ana","juan","pepe"]):
...     print(x,y)	
1 ana
2 juan
3 pepe
``` 


## 4. Tipos de datos secuencia

### 4.1 Tipos de datos secuencia

Los tipos de datos secuencia me permiten guardar una sucesión de datos de diferentes tipos. Los tipos de datos secuencias en python son:

* Las listas (list): Me permiten guardar un conjunto de datos que se pueden repetir y que pueden ser de distintos tipos. Es un tipo mutable.
* Las tuplas (tuple): Sirven para los mismo que las listas, pero en este caso es un tipo inmutable.
* Los rangos (range): Es un tipo de secuencias que nos permite crear secuencias de números. Es un tipo inmutable y se suelen utilizar para realizar bucles.
* Las cadenas de caracteres (str): Me permiten guardar secuencias de caracteres. Es un tipo inmutable.
* Las secuencias de bytes (byte): Me permite guardar valores binarios representados por caracteres ASCII. Es un tipo inmutable.
* Las secuencias de bytes (bytearray): En este caso son iguales que las anteriores, pero son de tipo mutables.
* Los conjuntos (set): Me permiten guardar conjuntos de datos, en los que no se existen repeticiones. Es un tipo mutable.
* Los conjuntos (frozenset): Son igales que los anteriores, pero son tipos inmutables.

**Características principales de las secuencias**

Vamos a ver distintos ejemplos partiendo de una lista, que es una secuencia mutable.

``` 
lista = [1,2,3,4,5,6]
``` 

Las secuencias se pueden recorrer

``` 
  >>> for num in lista:
  ...   print(num,end="")
  123456
``` 

* Operadores de pertenencia: Se puede comprobar si un elemento pertenece o no a una secuencia con los operadores in y not in.

``` 
  >>> 2 in lista
  True
  >>> 8 not in lista
  True
``` 

* Concatenación: El operador + me permite unir datos de tipos secuenciales. No se pueden concatenar secuencias de tipo range y de tipo conjunto.

``` 
  >>> lista + [7,8,9]
  [1, 2, 3, 4, 5, 6, 7, 8, 9]
``` 

* Repetición: El operador * me permite repetir un dato de un tipo secuencial. No se pueden repetir secuencias de tipo range y de tipo conjunto.

``` 
  >>> lista * 2
  [1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6]
``` 

* Indexación: Puedo obtener el dato de una secuencia indicando la posición en la secuencia. Los conjuntos no tienen implementado esta característica.

``` 
  >>> lista[3]
  4
``` 

* Slice (rebanada): Puedo obtener una subsecuencia de los datos de una secuencia. En los conjuntos no puedo obtener subconjuntos. Esta característica la estudiaremos más detenidamente en la unidad que estudiemos las listas.

``` 
  >>> lista[2:4]
  [3, 4]
  >>> lista[1:4:2]
  [2, 4]
``` 

Con la función len puedo obtener el tamaño de la secuencia, es decir el número de elementos que contiene.

``` 
  >>> len(lista)
  6
Con las funciones max y min puedo obtener el valor máximo y mínimo de una secuencia.

  >>> max(lista)
  6
  >>> min(lista)
  1
``` 

Además en los tipos mutables puedo realizar las siguientes operaciones:

Puedo modificar un dato de la secuencia indicando su posición.

``` 
  >>> lista[2]=99
  >>> lista
  [1, 2, 99, 4, 5, 6]
``` 

Puedo modificar un subconjunto de elementos de la secuencia haciendo slice.

```
  >>> lista[2:4]=[8,9,10]
  >>> lista
  [1, 2, 8, 9, 10, 5, 6]
``` 

Puedo borrar un subconjunto de elementos con la instrucción del.

``` 
  >>> del lista[5:]
  >>> lista
  [1, 2, 8, 9, 10]
``` 

Puedo actualizar la secuencia con el operador *=

```
  >>> lista*=2
  >>> lista
  [1, 2, 8, 9, 10, 1, 2, 8, 9, 10]
``` 

### 4.2 Tipo de datos secuencia: listas

Las listas (list) me permiten guardar un conjunto de datos que se pueden repetir y que pueden ser de distintos tipos. Es un tipo mutable.

**Construcción de una lista**

Para crear una lista puedo usar varias formas:

Con los caracteres [ y ]:

``` 
  >>> lista1 = []
  >>> lista2 = ["a",1,True]
``` 

Utilizando el constructor list, que toma como parámetro un dato de algún tipo secuencia.

``` 
  >>> lista3 = list()
  >>> lista4 = list("hola")
  >>> lista4
  ['h', 'o', 'l', 'a']
``` 

**Operaciones básicas con listas**

Como veíamos en el apartado "Tipo de datos secuencia" podemos realizar las siguientes operaciones:

Las secuencias se pueden recorrer.

* Operadores de pertenencia: in y not in.

* Concatenación: +

* Repetición: *

* Indexación: Cada elemento tiene un índice, empezamos a contar por el elemento en el índice 0. Si intento acceder a un índice que corresponda a un elemento que no existe obtenemos una excepción IndexError.

```
  >>> lista1[6]
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module
  IndexError: list index out of range	
```

Se pueden utilizar índices negativos:

``` 
  >>> lista[-1]
  6
```

* Slice: Veamos como se puede utilizar

- lista[start:end] # Elementos desde la posición start hasta end-1
- lista[start:] # Elementos desde la posición start hasta el final
- lista[:end] # Elementos desde el principio hata la posición end-1
- lista[:] # Todos Los elementos
- lista[start:end:step] # Igual que el anterior pero dando step saltos.

Se pueden utilizar también índices negativos, por ejemplo: lista[::-1]

**Funciones predefinidas que trabajan con listas**

``` 
>>> lista1 = [20,40,10,40,50]
>>> len(lista1)
5
>>> max(lista1)
50
>>> min(lista1)
10
>>> sum(lista1)
150
>>> sorted(lista1)
[10, 20, 30, 40, 50]
>>> sorted(lista1,reverse=True)
[50, 40, 30, 20, 10]
``` 

Veamos con más detenimiento la función enumerate: que recibe una secuencia y devuelve un objeto enumerado como tuplas:

``` 
>>> seasons = ['Primavera', 'Verano', 'Otoño', 'Invierno']
>>> list(enumerate(seasons))
[(0, 'Primavera'), (1, 'Verano'), (2, 'Otoño'), (3, 'Invierno')]
>>> list(enumerate(seasons, start=1))
[(1, 'Primavera'), (2, 'Verano'), (3, 'Otoño'), (4, 'Invierno')]
``` 

**Las listas son mutables**

Como hemos indicado anteriormente las listas es un tipo de datos mutable. Eso tiene para nostros varias consecuencias, por ejemplo podemos obtener resultados como se los que se muestran a continuación:

``` 
>>> lista1 = [1,2,3]
>>> lista1[2]=4
>>> lista1
[1, 2, 4]
>>> del lista1[2]
>>> lista1
[1, 2]


>>> lista1 = [1,2,3]
>>> lista2 = lista1
>>> lista1[1] = 10
>>> lista2
[1, 10, 3]
``` 

**¿Cómo se copian las listas?**

Por lo tanto si queremos copiar una lista en otra podemos hacerlo de varias formas:

```
>>> lista1 = [1,2,3]
>>> lista2=lista1[:]
>>> lista1[1] = 10
>>> lista2
[1, 2, 3]

>>> lista2 = list(lista1)	

>>> lista2 = lista1.copy()
``` 

**Listas multidimensionales**

A la hora de definir las listas hemos indicado que podemos guardar en ellas datos de cualquier tipo, y evidentemente podemos guardar listas dentro de listas.

``` 
>>> tabla = [[1,2,3],[4,5,6],[7,8,9]]
>>> tabla[1][1]
5

>>> for fila in tabla:
...   for elem in fila:
...      print(elem,end="")
...   print()
 
123
456
789
``` 

### 4.3 Métodos principales de listas

Cuando creamos una lista estamos creando un objeto de la clase list, que tiene definido un conjunto de métodos:

```
lista.append   lista.copy     lista.extend   lista.insert   lista.remove   lista.sort
lista.clear    lista.count    lista.index    lista.pop      lista.reverse
``` 

**Métodos de inserción: append, extend, insert**

``` 
>>> lista = [1,2,3]
>>> lista.append(4)
>>> lista
[1, 2, 3, 4]

>>> lista2 = [5,6]
>>> lista.extend(lista2)
>>> lista
[1, 2, 3, 4, 5, 6]	

>>> lista.insert(1,100)
>>> lista
[1, 100, 2, 3, 4, 5, 6]
``` 

**Métodos de eliminación: pop, remove**

``` 
>>> lista.pop()
6
>>> lista
[1, 100, 2, 3, 4, 5]

>>> lista.pop(1)
100
>>> lista
[1, 2, 3, 4, 5]

>>> lista.remove(3)
>>> lista
[1, 2, 4, 5]
``` 

**Métodos de ordenación: reverse, sort**

```
>>> lista.reverse()
>>> lista
[5, 4, 2, 1]

>>> lista.sort()
>>> lista
[1, 2, 4, 5]

>>> lista.sort(reverse=True)
>>> lista
[5, 4, 2, 1]

>>> lista=["hola","que","tal","Hola","Que","Tal"]
>>> lista.sort()
>>> lista
['Hola', 'Que', 'Tal', 'hola', 'que', 'tal']
>>> lista=["hola","que","tal","Hola","Que","Tal"]
>>> lista.sort(key=str.lower)
>>> lista
['hola', 'Hola', 'que', 'Que', 'tal', 'Tal']
```

**Métodos de búsqueda: count, index**

```
>>> lista.count(5)
1

>>> lista.append(5)
>>> lista
[5, 4, 2, 1, 5]
>>> lista.index(5)
0
>>> lista.index(5,1)
4
>>> lista.index(5,1,4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: 5 is not in list
Método de copia: copy
>>> lista2 = lista1.copy()
``` 

### 4.4 Operaciones avanzadas con secuencias

Las funciones que vamos a estudiar en esta unidad nos acercan al paradigna de la programación funcional que también nos ofrece python. La programación funcional es un paradigma de programación declarativa basado en el uso de funciones matemáticas, en contraste con la programación imperativa, que enfatiza los cambios de estado mediante la mutación de variables.

**Función map**

map(funcion,secuencia): Ejecuta la función enviada por parámetro sobre cada uno de los elementos de la secuencia.

- Ejemplo

```
>>> items = [1, 2, 3, 4, 5]
>>> def sqr(x): return x ** 2
>>> list(map(sqr, items))
[1, 4, 9, 16, 25]
``` 

**Función filter**

* filter(funcion,secuencia): Devuelve una secuencia con los elementos de la secuencia envíada por parámetro que devuelvan True al aplicarle la función envíada también como parámetro.

- Ejemplo

``` 
>>> lista = [1,2,3,4,5]
>>> def par(x): return x % 2==0 
>>> list(filter(par,lista))
``` 

**Función reduce**

* reduce(funcion,secuencia): Devuelve un único valor que es el resultado de aplicar la función á los lementos de la secuencia.

- Ejemplo

``` 
>>> from functools import reduce
>>> lista = [1,2,3,4,5]
>>> def add(x,y): return x + y
>>> reduce(add,lista)
15
``` 

**list comprehension**

list comprehension nos propociona una alternativa para la creación de listas. Es parecida a la función map, pero mientras map ejecuta una función por cada elemento de la secuencia, con esta técnica se aplica una expresión.

- Ejemplo

``` 
>>> [x ** 3 for x in [1,2,3,4,5]]
[1, 8, 27, 64, 125]

>>> [x for x in range(10) if x % 2 == 0]
[0, 2, 4, 6, 8] 

>>> [x + y for x in [1,2,3] for y in [4,5,6]]
[5, 6, 7, 6, 7, 8, 7, 8, 9]
``` 

### 4.5 Tipo de datos secuencia: Rangos

Los rangos (range): Es un tipo de secuencias que nos permite crear secuencias de números. Es un tipo inmutable y se suelen utilizar para realizar bucles.

**Definición de un rango. Constructor range**

Al crear un rango (secuencia de números) obtenemos un objeto que es de la clase range:

``` 
>>> rango = range(0,10,2)
>>> type(rango)
<class 'range'>
``` 

Veamos algunos ejemplos, convirtirndo el rango en lista para ver la secuencia:

``` 
>>> list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> list(range(0, 30, 5))
[0, 5, 10, 15, 20, 25]
>>> list(range(0, 10, 3))
[0, 3, 6, 9]
>>> list(range(0, -10, -1))
[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
``` 

**Recorrido de un rango**

Los rangos se suelen usar para ser recorrido, cuando tengo que crear un bucle cuyo número de iteraciones lo se de antemanos puedo usar una estructura como esta:

``` 
>>> for i in range(11):
...    print(i,end=" ")
0 1 2 3 4 5 6 7 8 9 10  
``` 

**Operaciones básicas con range**

En las tuplas se pueden realizar las siguientes operaciones:

- Los rangos se pueden recorrer.
- Operadores de pertenencia: in y not in.
- Indexación
- Slice
- Entre las funciones definidas podemos usar: len, max, min, sum, sorted.

Ademas un objeto range posee tres atributos que nos almacenan el comienzo, final e intervalo del rango:

``` 
>>> rango = range(1,11,2)
>>> rango.start
1
>>> rango.stop
11
>>> rango.step
2
``` 

### 4.6 Tipo de datos cadenas de caracteres

Las cadenas de caracteres (str): Me permiten guardar secuencias de caracteres. Es un tipo inmutable. Como hemos comentado las cadenas de caracteres en python3 están codificada con Unicode.

**Definición de cadenas. Constructor str**

Podemos definir una cadena de caracteres de distintas formas:

```
>>> cad1 = "Hola"
>>> cad2 = '¿Qué tal?'
>>> cad3 = '''Hola,
	que tal?'''
``` 

También podemos crear cadenas con el constructor str a partir de otros tipos de datos.

``` 
>>> cad1=str(1)
>>> cad2=str(2.45)
>>> cad3=str([1,2,3])
``` 

**Operaciones básicas con cadenas de caracteres**

Como veíamos en el apartado "Tipo de datos secuencia" podemos realizar las siguientes operaciones:

- Las cadenas se pueden recorrer.
- Operadores de pertenencia: in y not in.
- Concatenación: +
- Repetición: *
- Indexación
- Slice
- Entre las funciones definidas podemos usar: len, max, min, sorted.

Las cadenas son inmutables

``` 
>>> cad = "Hola que tal?"
>>> cad[4]="."
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
``` 

**Comparación de cadenas**

Las cadenas se comparan carácter a carácter, en el momento en que dos caracteres no son iguales se compara alfabéticamente (es decir, se convierte a código unicode y se comparan).

- Ejemplos

``` 
>>> "a">"A"
True
>>> ord("a")
97
>>> ord("A")
65

>>> "informatica">"informacion"
True

>>> "abcde">"abcdef"
False
``` 

**Funciones repr, ascii, bin**

* repr(objeto): Devuelve una cadena de caracteres que representa la información de un objeto.

``` 
  >>> repr(range(10))
  'range(0, 10)'
  >>> repr("piña")
  "'piña'"
``` 

La cadena devuelta por repr() debería ser aquella que, pasada a eval(), devuelve el mismo objeto.

``` 
  >>> type(eval(repr(range(10))))
  <class 'range'>
``` 

* ascii(objeto): Devuelve también la representación en cadena de un objeto pero en este caso muestra los caracteres con un código de escape . Por ejemplo en ascii (Latin1) la á se presenta con \xe1.

``` 
  >>> ascii("á")
  "'\\xe1'"
  >>> ascii("piña")
  "'pi\\xf1a'"
``` 

* bin(numero): Devuelve una cadena de caracteres que corresponde a la representación binaria del número recibido.

``` 
  >>> bin(213)
  '0b11010101'	
``` 

### 4.7 Métodos principales de cadenas

Cuando creamos una cadena de caracteres estamos creando un objeto de la clase str, que tiene definido un conjunto de métodos:

``` 
cadena.capitalize    cadena.isalnum       cadena.join          cadena.rsplit
cadena.casefold      cadena.isalpha       cadena.ljust         cadena.rstrip
cadena.center        cadena.isdecimal     cadena.lower         cadena.split
cadena.count         cadena.isdigit       cadena.lstrip        cadena.splitlines
cadena.encode        cadena.isidentifier  cadena.maketrans     cadena.startswith
cadena.endswith      cadena.islower       cadena.partition     cadena.strip
cadena.expandtabs    cadena.isnumeric     cadena.replace       cadena.swapcase
cadena.find          cadena.isprintable   cadena.rfind         cadena.title
cadena.format        cadena.isspace       cadena.rindex        cadena.translate
cadena.format_map    cadena.istitle       cadena.rjust         cadena.upper
cadena.index         cadena.isupper       cadena.rpartition    cadena.zfill
``` 

**Métodos de formato**

``` 
>>> cad = "hola, como estás?"
>>> print(cad.capitalize())
Hola, como estás?

>>> cad = "Hola Mundo" 
>>> print(cad.lower())
hola mundo

>>> cad = "hola mundo"
>>> print(cad.upper())
HOLA MUNDO

>>> cad = "Hola Mundo"
>>> print(cad.swapcase())
hOLA mUNDO

>>> cad = "hola mundo"
>>> print(cad.title())
Hola Mundo

>>> print(cad.center(50))
                    hola mundo                    
>>> print(cad.center(50,"="))
====================hola mundo====================

>>> print(cad.ljust(50,"="))
hola mundo========================================
>>> print(cad.rjust(50,"="))
========================================hola mundo

>>> num = 123
>>> print(str(num).zfill(12))
000000000123
``` 

**Métodos de búsqueda**

``` 
>>> cad = "bienvenido a mi aplicación"
>>> cad.count("a")
3
>>> cad.count("a",16)
2
>>> cad.count("a",10,16)
1

>>> cad.find("mi")
13
>>> cad.find("hola")
-1

>>> cad.rfind("a")
21
```

El método index() y rindex() son similares a los anteriores pero provocan una excepción ValueError cuando no encuentra la subcadena.

**Métodos de validación**

``` 
>>> cad.startswith("b")
True
>>> cad.startswith("m")
False
>>> cad.startswith("m",13)
True

>>> cad.endswith("ción")
True
>>> cad.endswith("ción",0,10)
False
>>> cad.endswith("nido",0,10)
True
``` 

Otras funciones de validación: isalnum(), isalpha(), isdigit(), islower(), isupper(), isspace(), istitle(),...

**Métodos de sustitución**

* format
  
En la unidad "Entrada y salida estándar" ya estuvimos introduciendo el concepto de formateo de la cadenas. Estuvimos viendo que hay dos métodos y vimos algunos ejemplos del nuevo estilo con la función predefinida format().

El uso del estilo nuevo es actualmente el recomendado (puedes obtener más información y ejemplos en algunos de estos enlaces: enlace1 y enlace2) y obtiene toda su potencialidad usando el método format() de las cadenas. Veamos algunos ejemplos:

``` 
>>> '{} {}'.format("a", "b")
'a b'
>>> '{1} {0}'.format("a", "b")
'b a'
>>> 'Coordenadas: {latitude}, {longitude}'.format(latitude='37.24N', longitude='-115.81W')
'Coordenadas: 37.24N, -115.81W'
>>> '{0:b} {1:x} {2:.2f}'.format(123, 223,12.2345)
'1111011 df 12.23'
>>> '{:^10}'.format('test')
'   test   '
``` 

**Otros métodos de sustitución**

``` 
>>> buscar = "nombre apellido"
>>> reemplazar_por = "Juan Pérez" 
>>> print ("Estimado Sr. nombre apellido:".replace(buscar, reemplazar_por)) 
Estimado Sr. Juan Pérez:

>>> cadena = "   www.eugeniabahit.com   " 
>>> print(cadena.strip())
www.eugeniabahit.com
>>> cadena="00000000123000000000"
>>> print(cadena.strip("0"))
123
``` 

De forma similar lstrip(["caracter"]) y rstrip(["caracter"]).

**Métodos de unión y división**

``` 
>>> formato_numero_factura = ("Nº 0000-0", "-0000 (ID: ", ")"
>>> print("275".join(formato_numero_factura))
Nº 0000-0275-0000 (ID: 275)

>>> hora = "12:23"
>>> print(hora.rpartition(":"))
('12', ':', '23')
>>> print(hora.partition(":"))
('12', ':', '23')
>>> hora = "12:23:12"
>>> print(hora.partition(":"))
('12', ':', '23:12')
>>> print(hora.split(":"))
['12', '23', '12']
>>> print(hora.rpartition(":"))
('12:23', ':', '12')
>>> print(hora.rsplit(":",1))
['12:23', '12']


>>> texto = "Linea 1\nLinea 2\nLinea 3" 
>>> print(texto.splitlines())
['Linea 1', 'Linea 2', 'Linea 3']
``` 

## 5. Trabajar con ficheros

### 5.1 Lectura y escritura de ficheros de texto

**Función open()**

La función open() se utiliza normalmente con dos parámetros (fichero con el que vamos a trabajar y modo de acceso) y nos devuelve un objeto de tipo fichero.

``` 
>>> f = open("ejemplo.txt","w")
>>> type(f)
<class '_io.TextIOWrapper'>
>>> f.close()
``` 

**Modos de acceso**

Los modos que podemos indicar son los siguientes:

Añadido en modo binario. Crea si éste no existe

* r	Solo lectura	Al inicio del archivo
* rb	Solo lectura en modo binario	
* r+	Lectura y escritura	Al inicio del archivo
* rb+	Lectura y escritura binario	Al inicio del archivo
* w	Solo escritura. Sobreescribe si existe. Crea el archivo si no existe.	Al inicio del archivo
* wb	Solo escritura en modo binario. Sobreescribe si existe. Crea el archivo si no existe.	Al inicio del archivo
* w+	Escritura y lectura. Sobreescribe si existe. Crea el archivo si no existe.	Al inicio del archivo
* wb+	Escritura y lectura binaria. Sobreescribe si existe. Crea el archivo si no existe.	Al inicio del archivo
* a	Añadido (agregar contenido). Crea el archivo si no existe.	Si el archivo existe, al final de éste. Si el archivo no existe, al comienzo.
* ab		Si el archivo existe, al final de éste. Si el archivo no existe, al comienzo.
* a+	Añadido y lectura. Crea el archivo si no existe.	Si el archivo existe, al final de éste. Si el archivo no existe, al comienzo.
* ab+	Añadido y lectura en binario. Crea el archivo si no existe	Si el archivo existe, al final de éste. Si el archivo no existe, al comienzo.


Como podemos comprobar podemos trabajar con ficheros binarios y con ficheros de textos.

**Codificación de caracteres**

Si trabajamos con fichero de textos podemos indicar también el parámetro encoding que será la codificación de caracteres utilizadas al trabajar con el fichero, por defecto se usa la indicada en el sistema:

``` 
>>> import locale
>>> locale.getpreferredencoding()
'UTF-8'
```

Y por último también podemos indicar el parámetro errors que controla el comportamiento cuando se encuentra con algún error al codificar o decodificar caracteres.

**Objeto fichero**

Al abrir un fichero con un determinado modo de acceso con la función open() se nos devuelve un objeto fichero. El fichero abierto siempre hay que cerrarlo con el método close():

``` 
>>> f = open("ejemplo.txt","w")
>>> type(f)
<class '_io.TextIOWrapper'>
>>> f.close()
``` 

Se pueden acceder a las siguientes propiedades del objeto file:

* closed: retorna True si el archivo se ha cerrado. De lo contrario, False.
* mode: retorna el modo de apertura.
* name: retorna el nombre del archivo
* encoding: retorna la codificación de caracteres de un archivo de texto

Podemos abrirlo y cerrarlo en la misma instrucción con la siguiente estructura:

``` 
>>> with open("ejemplo.txt", "r") as archivo: 
...    contenido = archivo.read()
>>> archivo.closed
True
``` 

**Métodos principales**

- Métodos de lectura

``` 
>>> f = open("ejemplo.txt","r")
>>> f.read()
'Hola que tal\n'

>>> f = open("ejemplo.txt","r")
>>> f.read(4)
'Hola'
>>> f.read(4)
' que'
>>> f.tell()
8
>>> f.seek(0)
>>> f.read()
'Hola que tal\n'

>>> f = open("ejemplo2.txt","r")    
>>> f.readline()
'Línea 1\n'
>>> f.readline()
'Línea 2\n'
>>> f.seek(0)
0
>>> f.readlines()
['Línea 1\n', 'Línea 2\n']
``` 

- Métodos de escritura

```
>>> f = open("ejemplo3.txt","w")
>>> f.write("Prueba 1\n")
9
>>> print("Prueba 2\n",file=f)
>>> f.writelines(["Prueba 3","Prueba 4"])
>>> f.close()
>>> f = open("ejemplo3.txt","r")
>>> f.read()
'Prueba 1\nPrueba 2\n\nPrueba 3Prueba 4'
``` 

- Recorrido de ficheros

``` 
>>> with open("ejemplo3.txt","r") as fichero:
...    for linea in fichero:
...        print(linea)
``` 

### 5.2 Gestionar ficheros CSV

**Módulo CSV**

El módulo cvs nos permite trabajar con ficheros CSV. Un fichero CSV (comma-separated values) son un tipo de documento en formato abierto sencillo para representar datos en forma de tabla, en las que las columnas se separan por comas (o por otro carácter).

**Leer ficheros CSV**

Para leer un fichero CSV utilizamos la función reader():

``` 
>>> import csv
>>> fichero = open("ejemplo1.csv")
>>> contenido = csv.reader(fichero)
>>> list(contenido)
[['4/5/2015 13:34', 'Apples', '73'], ['4/5/2015 3:41', 'Cherries', '85'], ['4/6/2015 12:46', 'Pears', '14'], ['4/8/2015 8:59', 'Oranges', '52'], ['4/10/2015 2:07', 'Apples', '152'], ['4/10/2015 18:10', 'Bananas', '23'], ['4/10/2015 2:40', 'Strawberries', '98']]
>>> list(contenido)
[]
>>> fichero.close()
``` 

Podemos guardar la lista obtenida en una variable y acceder a ella indicando fila y columna.

``` 
...
>>> datos = list(contenido)
>>> datos[0][0]
'4/5/2015 13:34'
>>> datos[1][1]
'Cherries'
>>> datos[2][2]
'14'
``` 

Por supuesto podemos recorrer el resultado:

``` 
...
>>> for row in contenido:
	print("Fila "+str(contenido.line_num)+" "+str(row))	

Fila 1 ['4/5/2015 13:34', 'Apples', '73']
Fila 2 ['4/5/2015 3:41', 'Cherries', '85']
Fila 3 ['4/6/2015 12:46', 'Pears', '14']
Fila 4 ['4/8/2015 8:59', 'Oranges', '52']
Fila 5 ['4/10/2015 2:07', 'Apples', '152']
Fila 6 ['4/10/2015 18:10', 'Bananas', '23']
Fila 7 ['4/10/2015 2:40', 'Strawberries', '98']
``` 

Veamos otro ejemplo un poco más complejo:

``` 
>>> import csv
>>> fichero = open("ejemplo2.csv")
>>> contenido = csv.reader(fichero,quotechar='"')
>>> for row in contenido:
...   print(row)
... 
['Año', 'Marca', 'Modelo', 'Descripción', 'Precio']
['1997', 'Ford', 'E350', 'ac, abs, moon', '3000.00']
['1999', 'Chevy', 'Venture "Extended Edition"', '', '4900.00']
['1999', 'Chevy', 'Venture "Extended Edition, Very Large"', '', '5000.00']
['1996', 'Jeep', 'Grand Cherokee', 'MUST SELL!\nair, moon roof, loaded', '4799.00']
``` 

**Escribir ficheros CSV**

``` 
>>> import csv
>>> fichero = open("ejemplo3.csv","w")
>>> contenido = csv.writer(fichero)
>>> contenido.writerow(['4/5/2015 13:34', 'Apples', '73'])
>>> contenido.writerows(['4/5/2015 3:41', 'Cherries', '85'],['4/6/2015 12:46', 'Pears', '14'])
>>> fichero.close()

$ cat ejemplo3.csv
4/5/2015 13:34,Apples,73
4/5/2015 3:41,Cherries,85
4/6/2015 12:46,Pears,14
``` 

### 5.3 Gestionar ficheros json

El módulo json nos permite gestionar ficheros con formato JSON (JavaScript Object Notation).

**Leer ficheros json**

Desde una cadena de caracteres:

``` 
>>> import json
>>> datos_json='{"nombre":"carlos","edad":23}'
>>> datos = json.loads(datos_json)
>>> type(datos)
<class 'dict'>
>>> print(datos)
{'nombre': 'carlos', 'edad': 23}
``` 

Desde un fichero:

``` 
>>> with open("ejemplo1.json") as fichero:
...   datos=json.load(fichero)
>>> type(datos)
<class 'dict'>
>>> datos
{'bookstore': {'book': [{'_category': 'COOKING', 'price': '30.00', 'author': 'Giada De Laurentiis', 'title': {'__text': 'Everyday Italian', '_lang': 'en'}, 'year': '2005'}, {'_category': 'CHILDREN', 'price': '29.99', 'author': 'J K. Rowling', 'title': {'__text': 'Harry Potter', '_lang': 'en'}, 'year': '2005'}, {'_category': 'WEB', 'price': '49.99', 'author': ['James McGovern', 'Per Bothner', 'Kurt Cagle', 'James Linn', 'Vaidyanathan Nagarajan'], 'title': {'__text': 'XQuery Kick Start', '_lang': 'en'}, 'year': '2003'}, {'_category': 'WEB', 'price': '39.95', 'author': 'Erik T. Ray', 'title': {'__text': 'Learning XML', '_lang': 'en'}, 'year': '2003'}]}}
``` 

**Escribir ficheros json**

``` 
>>> datos = {'isCat': True, 'miceCaught': 0, 'name': 'Zophie','felineIQ': None}
>>> fichero = open("ejemplo2.json","w")
>>> json.dump(datos,fichero)
>>> fichero.close()

cat ejemplo2.json 
{"miceCaught": 0, "name": "Zophie", "felineIQ": null, "isCat": true}
``` 


## 6. Módulos

### 6.1 Módulos y paquetes

* Módulo: Cada uno de los ficheros .py que nosotros creamos se llama módulo. Los elementos creados en un módulo (funciones, clases, ...) se pueden importar para ser utilizados en otro módulo. El nombre que vamos a utilizar para importar un módulo es el nombre del fichero.

* Paquete: Para estructurar nuestros módulos podemos crear paquetes. Un paquete, es una carpeta que contiene archivos .py. Pero, para que una carpeta pueda ser considerada un paquete, debe contener un archivo de inicio llamado __init__.py. Este archivo, no necesita contener ninguna instrucción. Los paquetes, a la vez, también pueden contener otros sub-paquetes.

**Ejecutando módulos como scripts**

Si hemos creado un módulo, donde hemos definido dos funciones y hemos hecho un programa principal donde se utilizan dichas funciones, tenemos dos opciones: ejecutar ese módulo como un script o importar ese módulo desde otro, para utilizar sus funciones. Por ejemplo, si tenemos un fichero llamado potencias.py:

``` 
#!/usr/bin/env python	

def cuadrado(n):
	return(n**2)
def cubo(n):
	return(n**3)
if __name__ == "__main__":
	print(cuadrado(3))
	print(cubo(3))
``` 

En este caso, cuando lo ejecuto como un script:

``` 
$ python3 potencias.py
``` 

El nombre que tiene el módulo es __main__, por lo tanto se ejecutará el programa principal.

Además este módulo se podrá importar (como veremos en el siguiente apartado) y el programa principal no se tendrá en cuenta.

**Importando módulos: import, from**

Para importar un módulo completo tenemos que utilizar las instrucción import. lo podemos importar de la siguiente manera:

``` 
>>> import potencias
>>> potencias.cuadrado(3)
9
>>> potencias.cubo(3)
27
``` 

**Namespace y alias**

Para acceder (desde el módulo donde se realizó la importación), a cualquier elemento del módulo importado, se realiza mediante el namespace, seguido de un punto (.) y el nombre del elemento que se desee obtener. En Python, un namespace, es el nombre que se ha indicado luego de la palabra import, es decir la ruta (namespace) del módulo.

Es posible también, abreviar los namespaces mediante un alias. Para ello, durante la importación, se asigna la palabra clave as seguida del alias con el cuál nos referiremos en el futuro a ese namespace importado:

``` 
>>> import potencias as p
>>> p.cuadrado(3)
9
``` 

**Importando elementos de un módulo: from...import**

Para no utilizar el namespace podemos indicar los elementos concretos que queremos importar de un módulo:

``` 
>>> from potencias import cubo
>>> cubo(3)
27
``` 

Podemos importar varios elementos separándolos con comas:

``` 
>>> from potencias import cubo,cuadrado
``` 

Podemos tener un problema al importar dos elementos de dos módulos que se llamen igual. En este caso tengo que utilizar alias:

``` 
>>> from potencias import cuadrado as pc
>>> from dibujos import cuadrado as dc
>>> pc(3)
9
>>> dc()
Esto es un cuadrado
``` 

**Importando módulos desde paquetes**

Si tenemos un módulo dentro de un paquete la importación se haría de forma similar. tenemos un paquete llamado operaciones:

``` 
$ cd operaciones
$ ls
__init.py__  potencias.py
``` 

Para importarlo:

``` 
>>> import operaciones.potencias
>>> operaciones.potencias.cubo(3)
27

>>> from operaciones.potencias import cubo
>>> cubo(3)
27
``` 

**Función dir()**

La función dir() nos permite averiguar los elementos definidos en un módulo:

``` 
>>> import potencias
>>> dir(potencias)
['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'cuadrado', 'cubo']
``` 

**¿Donde se encuentran los módulos?**

Los módulos estandar (como math o sys por motivos de eficiencia están escrito en C e incorporados en el interprete (builtins)).

Para obtener la lista de módulos builtins:

``` 
>>> import sys
>>> sys.builtin_module_names
('_ast', '_bisect', '_codecs', '_collections', '_datetime', '_elementtree', '_functools', '_heapq', '_imp', '_io', '_locale', '_md5', '_operator', '_pickle', '_posixsubprocess', '_random', '_sha1', '_sha256', '_sha512', '_socket', '_sre', '_stat', '_string', '_struct', '_symtable', '_thread', '_tracemalloc', '_warnings', '_weakref', 'array', 'atexit', 'binascii', 'builtins', 'errno', 'faulthandler', 'fcntl', 'gc', 'grp', 'itertools', 'marshal', 'math', 'posix', 'pwd', 'pyexpat', 'select', 'signal', 'spwd', 'sys', 'syslog', 'time', 'unicodedata', 'xxsubtype', 'zipimport', 'zlib')
``` 

Los demás módulos que podemos importar se encuentran guardados en ficheros, que se encuentra en los path indicados en el interprete:

``` 
>>> sys.path
>>> sys.path
['', '/usr/lib/python3.4', '/usr/lib/python3.4/plat-x86_64-linux-gnu', '/usr/lib/python3.4/lib-dynload', '/usr/local/lib/python3.4/dist-packages', '/usr/lib/python3/dist-packages']
``` 

### 6.2 Módulos estándares: módulos de sistema

Python tiene sus propios módulos, los cuales forman parte de su librería de módulos estándar, que también pueden ser importados. En esta unidad vamos a estudiar las funciones porincipales de módulos realacionados con el sistema operativo.

**Módulo os**

El módulo os nos permite acceder a funcionalidades dependientes del Sistema Operativo. Sobre todo, aquellas que nos refieren información sobre el entorno del mismo y nos permiten manipular la estructura de directorios.

**Descripción y	Método**

* Saber si se puede acceder a un archivo o directorio	- os.access(path, modo_de_acceso)
* Conocer el directorio actual	- os.getcwd()
* Cambiar de directorio de trabajo	- os.chdir(nuevo_path)
* Cambiar al directorio de trabajo raíz	- os.chroot()
* Cambiar los permisos de un archivo o directorio	- os.chmod(path, permisos)
* Cambiar el propietario de un archivo o directorio	- os.chown(path, permisos)
* Crear un directorio	- os.mkdir(path[, modo])
* Crear directorios recursivamente	- os.mkdirs(path[, modo])
* Eliminar un archivo	- os.remove(path)
* Eliminar un directorio	- os.rmdir(path)
* Eliminar directorios recursivamente	- os.removedirs(path)
* Renombrar un archivo	- os.rename(actual, nuevo)
* Crear un enlace simbólico	- os.symlink(path, nombre_destino)

``` 
>>> import os
>>> os.getcwd()
'/home/jose/github/curso_python3/curso/u40'
>>> os.chdir("..")
>>> os.getcwd()
'/home/jose/github/curso_python3/curso'
``` 

El módulo os también nos provee del submódulo path (os.path) el cual nos permite acceder a ciertas funcionalidades relacionadas con los nombres de las rutas de archivos y directorios.


**Descripción	y Método**

* Ruta absoluta	- os.path.abspath(path)
* Directorio base	- os.path.basename(path)
* Saber si un directorio existe	- os.path.exists(path)
* Conocer último acceso a un directorio	- os.path.getatime(path)
* Conocer tamaño del directorio	- os.path.getsize(path)
* Saber si una ruta es absoluta	- os.path.isabs(path)
* Saber si una ruta es un archivo	- os.path.isfile(path)
* Saber si una ruta es un directorio	- os.path.isdir(path)
* Saber si una ruta es un enlace simbólico	- os.path.islink(path)
* Saber si una ruta es un punto de montaje	- os.path.ismount(path)

**Ejecutar comandos del sistema operativo. Módulo subprocess**

Con la función system() del módulo os nos permite ejecutar comandos del sistema operativo.

``` 
>>> os.system("ls")
curso  modelo.odp  README.md
0
``` 

La función nos devuelve un código para indicar si la instrucción se ha ejecutado con éxito.

Tenemos otra forma de ejecutar comandos del sistema operativo que nos da más funcionalidad, por ejemplo nos permite guardar la salida del comando en una variable. Para ello podemos usar el módulo subprocess

``` 
>>> import subprocess
>>> subprocess.call("ls")
curso  modelo.odp  README.md
0

>>> salida=subprocess.check_output("ls")
>>> print(salida.decode())
curso
modelo.odp
README.md

>>> salida=subprocess.check_output(["df","-h"])

>>> salida = subprocess.Popen(["df","-h"], stdout=subprocess.PIPE)
>>> salida.communicate()[0]
``` 

**Módulo shutil**

El módulo shutil de funciones para realizar operaciones de alto nivel con archivos y directorios. Dentro de las operaciones que se pueden realizar está copiar, mover y borrar archivos y directorios; y copiar los permisos y el estado de los archivos.

**Ejecución de scripts con argumentos**

Podemos enviar información (argumentos) a un programa cuando se ejecuta como un script, por ejemplo:

```
#!/usr/bin/env python	
import sys
print("Has instroducido",len(sys.argv),"argumento")
suma=0
for i in range(1,len(sys.argv)):
	suma=suma+int(sys.argv[i])
print("La suma es ",suma)

$  python3 sumar.py 3 4 5
Has introducido 4 argumento
La suma es  12
``` 

### 6.3 Módulos estándares: módulos matemáticos

**Módulo math**

El módulo math nos proporciones distintas funciones y operaciones matemáticas.

``` 
>>> import math
>>> math.factorial(5)
120
>>> math.pow(2,3)
8.0
>>> math.sqrt(7)
2.6457513110645907
>>> math.cos(1)
0.5403023058681398
>>> math.pi
3.141592653589793
>>> math.log(10)
2.302585092994046
``` 

**Módulo fractions**

El módulo fractions nos permite trabajar con fracciones.

``` 
>>> from fractions import Fraction
>>> a=Fraction(2,3)
>>> b=Fraction(1.5)
>>> b
Fraction(3, 2)
>>> c=a+b
>>> c
Fraction(13, 6)
``` 

**Módulo statistics**

El módulo statistics nos proporciona funciones para hacer operaciones estadísticas.

```
>>> import statistics
>>> statistics.mean([1,4,5,2,6])
3.6

>>> statistics.median([1,4,5,2,6])
4
``` 

**Módulo random**

El módulo random nos permite generar datos pseudo-aleatorios.

```
>>> import random
>>> random.randint(10,100)
34

>>> random.choice(["hola","que","tal"])
'que'

>>> frutas=["manzanas","platanos","naranjas"]
>>> random.shuffle(frutas)
>>> frutas
['naranjas', 'manzanas', 'platanos']

>>> lista = [1,3,5,2,7,4,9]
>>> numeros=random.sample(lista,3)
>>> numeros
[1, 2, 4]
``` 

### 6.4 Módulos estándares: módulos de hora y fechas

**Módulo time**

El tiempo es medido como un número real que representa los segundos transcurridos desde el 1 de enero de 1970. Por lo tanto es imposible representar fechas anteriores a esta y fechas a partir de 2038 (tamaño del float en la lubraría C (32 bits)).

``` 
>>> import time
>>> time.time()
1488619835.7858684
``` 

Para convertir la cantidad de segundos a la fecha y hora local:

``` 
>>> tiempo = time.time()
>>> time.localtime(tiempo)
time.struct_time(tm_year=2017, tm_mon=3, tm_mday=4, tm_hour=10, tm_min=37, tm_sec=19, tm_wday=5, tm_yday=63, tm_isdst=0)
``` 

Si queremos obtener la fecha y hora actual:

``` 
>>> time.localtime()
time.struct_time(tm_year=2017, tm_mon=3, tm_mday=4, tm_hour=10, tm_min=37, tm_sec=30, tm_wday=5, tm_yday=63, tm_isdst=0)
``` 

Nos devuelve a una estructura a la que podemos acceder a sus distintos campos.

``` 
>>> tiempo = time.localtime()
>>> tiempo.tm_year
2017
``` 

Podemos representar la fecha y hora como una cadena:

``` 
>>> time.asctime()
'Sat Mar  4 10:41:41 2017'
>>> time.asctime(tiempo)
'Sat Mar  4 10:39:21 2017'
``` 

O con un determinado formato:

``` 
>>> time.strftime('%d/%m/%Y %H:%M:%S')
'04/03/2017 10:44:52'
>>> time.strftime('%d/%m/%Y %H:%M:%S',tiempo)
'04/03/2017 10:39:21'
``` 

**Módulo datetime**

Los módulos datetime y calendar amplían las posibilidades del módulo time que provee funciones para manipular expresiones de tiempo.

``` 
>>> from datetime import datetime
>>> datetime.now()
datetime.datetime(2017, 3, 4, 10, 52, 12, 859564)
>>> datetime.now().day,datetime.now().month,datetime.now().year
(4, 3, 2017)
``` 

Para compara fechas y horas:

```
>>> from datetime import datetime, date, time, timedelta
>>> hora1 = time(10,5,0)
>>> hora2 = time(23,15,0)
>>> hora1>hora2
False

>>> fecha1=date.today()
>>> fecha2=fecha1+timedelta(days=2)
>>> fecha1
datetime.date(2017, 3, 4)
>>> fecha2
datetime.date(2017, 3, 6)
>>> fecha1<fecha2
True
``` 

Podemos imprimir aplicando un formato:

```
>>> fecha1.strftime("%d/%m/%Y")
'04/03/2017'
>>> hora1.strftime("%H:%M:%S")
'10:05:00'
```

Podemos convertir una cadena a un datetime:

``` 
>>> tiempo = datetime.strptime("12/10/2017","%d/%m/%Y")
``` 

Y podemos trabajar con cantidades (segundos, minutos, horas, días, semanas,...) con timedelta:

``` 
>>> hoy = date.today()
>>> ayer = hoy - timedelta(days=1)
>>> diferencia=hoy -ayer
>>> diferencia
datetime.timedelta(1)

>>> fecha1=datetime.now()
>>> fecha2=datetime(1995,10,12,12,23,33)
>>> diferencia=fecha1-fecha2
>>> diferencia
datetime.timedelta(7813, 81981, 333199)
``` 

**Módulo calendar**

Podemos obtener el calendario del mes actual:

``` 
>>> año = date.today().year 
>>> mes = date.today().month
>>> calendario_mes = calendar.month(año, mes)
>>> print(calendario_mes)
     March 2017
Mo Tu We Th Fr Sa Su
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31
```

Y para mostrar todos los meses del año:

``` 
>>> print(calendar.TextCalendar(calendar.MONDAY).formatyear(2017,2, 1, 1, 2))
``` 

### 6.5 Instalación de módulos

Python posse una activa comunidad de desarrolladores y usuarios que desarrollan toanto los módulos estándar de python, como módulos y paquetes desarolados por terceros.

**PyPI y pip**

El Python Package Index o PyPI, es el repositorio de paquetes de software oficial para aplicaciones de terceros en el lenguaje de programación Python.

* pip: Sistema de gestión de paquetes utilizado para instalar y administrar paquetes de software escritos en Python que se encuentran alojados en el repositorio PyPI.

**Instalación de módulos python**

Para instalar un nuevo paquete python tengo varias alternativas:

* Utilizar el que este empaquetado en la distribución que estés usando. Podemos tener problemas si necesitamos una versión determinada.

``` 
 # apt-cache show python3-requests
 ...
 Version: 2.4.3-6
 ...
``` 

* Instalar pip en nuestro equipo, y como superusuario instalar el paquete python que nos interesa. Esta solución nos puede dar muchos problemas, ya que podemos romper las dependencias entre las versiones de nuestros paquetes python instalados en el sistema y algún paquete puede dejar de funcionar.

``` 
 # pip search requests
 ...
 requests (2.13.0)      - Python HTTP for Humans.
 ...
``` 

* Utilizar entornos virtuales: es un mecanismo que me permite gestionar programas y paquetes python sin tener permisos de administración, es decir, cualquier usuario sin privilegios puede tener uno o más "espacios aislados" (ya veremos más adelante que los entornos virtuales se guardan en directorios) donde poder instalar distintas versiones de programas y paquetes python. Para crear los entornos virtuales vamos a usar el programa virtualenv o el módulo venv.

**Creando entornos virtuales con virtualenv**

Podemos utilizar este software para trabajar con cualquier distribución de python, pero evidentemente es obligatorio si estamos trabajando con python 2.x o python 3.x (una versión anterior a la 3.3).

``` 
# apt-get install python-virtualenv
Si queremos crear un entorno virtual con python3:

$ virtualenv -p /usr/bin/python3 entorno2
``` 

La opción -p nos permite indicar el interprete que se va a utilizar en el entorno.

Para activar nuestro entorno virtual:

``` 
$ source entorno2/bin/activate
(entorno2)$ 
``` 

Y para desactivarlo:

``` 
(entorno2)$ deactivate
$
``` 

**Creando entornos virtuales con venv**

A partir de la versión 3.3 de python podemos utilizar el módulo venv para crear el entorno virtual.

Instalamos el siguiente paquete para instalar el módulos:

``` 
# apt-get install python3-venv
``` 

Ahora ya como un usuario sin privilegio podemos crear un entorno virtual con python3:

``` 
$ python3 -m venv entorno3
``` 

La opción -m del interprete nos permite ejecutar un módulo como si fuera un programa.

Para activar y desactivar el entono virtual:

``` 
$ source entorno3/bin/activate
(entorno3)$ deactivate
$ 
``` 

**Instalando paquetes en nuestro entorno virtual**

Independientemente del sistema utilizado para crear nuestro entorno virtual, una vez que lo tenemos activado podemos instalar paquetes python en él utilizando la herramienta pip (que la tenemos instalada automáticamente en nuestro entorno). Partiendo de un entorno activado, podemos, por ejemplo, instalar la última versión de django:

``` 
(entorno3)$ pip install django
``` 

Si queremos ver los paquetes que tenemos instalados con sus dependencias:

``` 
(entorno3)$ pip list
Django (1.10.5)
pip (1.5.6)
setuptools (5.5.1)
``` 

Si necesitamos buscar un paquete podemos utilizar la siguiente opción:

``` 
(entorno3)$ pip search requests
``` 

Si a continuación necesitamos instalar una versión determinada del paquete, que no sea la última, podemos hacerlo de la siguiente manera:

``` 
(entorno3)$ pip install requests=="2.12"
``` 

Si necesitamos borrar un paquete podemos ejecutar:

``` 
(entorno3)$ pip uninstall requests
``` 
Y, por supuesto para instalar la última versión, simplemente:

``` 
(entorno3)$ pip install requests	
``` 

Para terminar de repasar la herramienta pip, vamos a explicar como podemos guardar en un fichero (que se suele llamar requirements.txt) la lista de paquetes instalados, que nos permite de manera sencilla crear otro entorno virtual en otra máquina con los mismos paquetes instalados. Para ello vamos a usar la siguiente opción de pip:

``` 
(entorno3)$ pip freeze
Django==1.10.5
requests==2.13.0
```

Y si queremos guardar esta información en un fichero que podamos distribuir:

``` 
(entorno3)$ pip freeze > requirements.txt
``` 

De tal manera que otro usuario, en otro entorno, teniendo este fichero pude reproducirlo e instalar los mismos paquetes de la siguiente manera:

``` 
(entorno4)$ pip install -r requirements.txt
``` 

## 7. Programación estructurada y modular

### 7.1 Conceptos avanzados sobre funciones

**Tipos de argumentos: posicionales o keyword**

Tenemos dos tipos de parámetros: los posiciónales donde el parámetro real debe coincidir en posición con el parámetro formal:

```
>>> def sumar(n1,n2):
...   return n1+n2
... 
>>> sumar(5,7)
12
>>> sumar(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sumar() missing 1 required positional argument: 'n2'
``` 

Además podemos tener parámetros con valores por defecto:

```
>>> def operar(n1,n2,operador='+',respuesta='El resultado es '):
...   if operador=="+":
...     return respuesta+str(n1+n2)
...   elif operador=="-":
...     return respuesta+str(n1-n2)
...   else:
...     return "Error"
... 
>>> operar(5,7)
'El resultado es 12'
>>> operar(5,7,"-")
'El resultado es -2'
>>> operar(5,7,"-","La resta es ")
'La resta es -2'
``` 

Los parámetros keyword son aquellos donde se indican el nombre del parámetro formal y su valor, por lo tanto no es necesario que tengan la misma posición. Al definir una función o al llamarla, hay que indicar primero los argumentos posicionales y a continuación los argumentos con valor por defecto (keyword).

``` 
>>> operar(5,7)	# dos parámetros posicionales
>>> operar(n1=4,n2=6)	# dos parámetros keyword
>>> operar(4,6,respuesta="La suma es")	# dos parámetros posicionales y uno keyword
>>> operar(4,6,respuesta="La resta es",operador="-")	# dos parámetros posicionales y dos keyword
``` 

**Parámetro ***

Un parámetro * entre los parámetros formales de una función, nos obliga a indicar los parámetros reales posteriores como keyword:

``` 
>>> def sumar(n1,n2,*,op="+"):
...   if op=="+":
...     return n1+n2
...   elif op=="-":
...     return n1-n2
...   else:
...     return "error"
... 
>>> sumar(2,3)
5
>>> sumar(2,3,"-")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sumar() takes 2 positional arguments but 3 were given
>>> sumar(2,3,op="-")
-1
``` 

**Argumentos arbitrarios (*args y **kwargs)**

Para indicar un número indefinido de argumentos posicionales al definir una función, utilizamos el símbolo *:

``` 
>>> def sumar(n,*args):
...   resultado=n
...   for i in args:
...     resultado+=i
...   return resultado
... 
>>> sumar(2)
2
>>> sumar(2,3,4)
9
``` 

Para indicar un número indefinido de argumentos keyword al definir una función, utilizamos el símbolo **:

``` 
>>> def saludar(nombre="pepe",**kwargs):
...   cadena=nombre
...   for valor in kwargs.values():
...    cadena=cadena+" "+valor
...   return "Hola "+cadena
... 
>>> saludar()
'Hola pepe'
>>> saludar("juan")
'Hola juan'
>>> saludar(nombre="juan",nombre2="pepe")
'Hola juan pepe'
>>> saludar(nombre="juan",nombre2="pepe",nombre3="maria")
'Hola juan maria pepe'
``` 

Por lo tanto podríamos tener definiciones de funciones del tipo:

``` 
>>> def f()
>>> def f(a,b=1)
>>> def f(a,*args,b=1)
>>> def f(*args,b=1)
>>> def f(*args,b=1,*kwargs)
>>> def f(*args,*kwargs)
>>> def f(*args)
>>> def f(*kwargs)
``` 

**Desempaquetar argumentos: pasar listas y diccionarios**

En caso contrario es cuando tenemos que pasar parámetros que lo tenemos guardados en una lista o en un diccionario.

Para pasar listas utilizamos el símbolo *:

``` 
>>> lista=[1,2,3]
>>> sumar(*lista)
6
>>> sumar(2,*lista)
8
>>> sumar(2,3,*lista)
11
``` 

Podemos tener parámetros keyword guardados en un diccionario, para enviar un diccionario utilizamos el símbolo **:

``` 
>>> datos={"nombre":"jose","nombre2":"pepe","nombre3":"maria"}
>>> saludar(**datos)
'Hola jose maria pepe'
``` 

**Devolver múltiples resultados**

La instrucción return puede devolver cualquier tipo de resultados, por lo tanto es fácil devolver múltiples datos guardados en una lista o en un diccionario. Veamos un ejemplo en que devolvemos los datos en una tupla:

``` 
>>> def operar(n1,n2):
...   return (n1+n2,n1-n2,n1*n2)	

>>> suma,resta,producto = operar(5,2)
>>> suma
7
>>> resta
3
>>> producto
10
``` 







