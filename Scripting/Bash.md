# 60 Ejercicios de Bash Shell Script

## Parte 1. echo, read, variables, parámetros, operaciones aritméticas, if…then...else 

1. **Introducir tu nombre en la variable1. Introducir tu dirección de correo electrónico en la variable2.Introducir tu edad en la variable3. Mostrar por pantalla variable1 tiene variable3 años y su correo electrónico es: variable2.**

``` 
#!/bin/bash
clear
echo $1 tiene $2 años y su email es $3
``` 

2. **Pedir el nombre de un animal en la variable1. Pedir el número de patas del animal en la variable2.Mostrar El variable1 tiene variable2 de patas**

``` 
#!/bin/bash
clear
echo El $1 tiene un total de $2 patas.
``` 

3. **Pedir el nombre de 2 equipos de futbol. Pedir el número de goles marcados por el equipo1. Pedir el número de goles marcados por el equipo2. Mostrar el mensaje: Partido: xxxxx - yyyyy Resultado: x-y**

``` 
#!/bin/bash
clear
echo Equipo local
read equipo1
echo Equipo visitante
read equipo2
echo Número de goles del equipo local
read goleslocal
echo Número de goles del equipo visitante
read golesvisitante
echo Partido: $equipo1 - $equipo2
echo Resultado: $goleslocal-$golesvisitante
``` 

4. **Pedir el nombre de 3 amigos, sus emails, números de teléfono y provincias de nacimiento. Mostrar el mensaje:**
   
* El email de mi amigo Lalo es: xxxxx@xxxxxxx.xxx
* El teléfono de Lalo es: 9999999999
* Lalo nació en: xxxxxxx
* El email de mi amigo Lolo es: xxxxx@xxxxxxx.xxx
* El teléfono de Lolo es: 9999999999
* Lolo nació en: xxxxxxx
* El email de mi amigo Lelo es: xxxxx@xxxxxxx.xxx
* El teléfono de Lelo es: 9999999999
* Lelo nació en: xxxxxxx

``` 
#!/bin/bash

clear
echo email Lalo
read emaillalo
echo teléfono Lalo
read telefonolalo
echo Nacimiento Lalo
read ciudadlalo

echo email Lolo
read emaillolo
echo teléfono Lolo
read telefonololo
echo Nacimiento Lolo
read ciudadlolo

echo email Lelo
read emailelo
echo teléfono Lelo
read telefonolelo
echo Nacimiento Lelo
read ciudadlelo

echo El email de mi amigo Lalo es:$emaillalo
echo El teléfono de Lalo es:$telefonolalo
echo La ciudad de nacimiento de Lalo es:$ciudadlalo

echo El email de mi amigo Lolo es:$emaillolo
echo El teléfono de Lolo es:$telefonololo
echo La ciudad de nacimiento de Lolo es:$ciudadlolo
``` 

5. **Introducir primer número: variable1. Introducir segundo número: variable2. Introducir tercer número: variable3. La suma de variable1 + variable2 + variable3 es: resultado. El producto de variable1 x variable2 x variable3 es: resultado**

``` 
#!/bin/bash
clear
echo La suma de $1 + $2 + $3 es:$(($1 + $2 + $3))
echo El producto de $1 x $2 x $3 es:$(($1*$2*$3))
``` 

6. **Pedir 2 números por teclado y mostrar:**

* Los números son variable1 y variable2
* El producto es:	resultado de multiplicación
* La suma es: 		resultado de la suma
* La resta es:     	resultado de la resta
* La división:		resultado de la división

``` 
#!/bin/bash
clear
echo Introduce un número
read num1
echo Introduce otro número
read num2
echo El producto es:$(($num1*$num2))
echo La suma es:$(($num1+$num2))
echo La resta es:$(($num1-$num2))
echo La división es:$(($num1/$num2))
``` 

7. **Mostrar por pantalla todos los valores que se han introducido por PARÁMETROS**

``` 
#!/bin/bash
clear
echo $1 $2 $3
``` 

8. **Mostrar el primer valor pasado por PARÁMETROS al ejecutar el script**

``` 
#!/bin/bash
clear
echo $1
``` 

9. **Mostrar el primer valor, y el segundo pasados por PARÁMETROS al ejecutar el script**

``` 
#!/bin/bash
clear
echo $1 $2
``` 

10. **Mostrar el primer, segundo y tercer valor pasados por PARÁMETROS al ejecutar el script**

``` 
#!/bin/bash
clear
echo $1 $2 $3
``` 

11. **Mostrar el número de valores pasados por PARÁMETROS al ejecutar el script**

``` 
#!/bin/bash
clear
echo El número de valores pasados por parámetros es cinco.
``` 

12. **Pedir un número y si es mayor que 200 mostrar el mensaje “mayor que 200”**

``` 
#!/bin/bash
clear
echo Dime un número
read num1
if [ $num1 -gt 200 ]
then
        echo El $num1 es mayor que 200
else
        echo El $num1 es menor que 200
fi
``` 

13. **Pedir un número y mostrar si ese número es mayor o menor que 100**

``` 
#!/bin/bash
clear
echo Dime un número
read num1
if [ $num1 -gt 100 ]
then
        echo El $num1 es mayor que 100
else
        echo El $num1 es menor que 100
fi
``` 

14. **Pedir dos números y mostrar si son iguales o no**

``` 
#!/bin/bash
clear
echo Dime un número
read num1
echo Dime otro número
read num2
if [ $num1 -eq $num2 ]
then
        echo El $num1 es igual que $num2
else
        echo El $num1 es diferente de $num2
fi
``` 

15. **Pedir un nombre y una edad y si la edad es menor que 18 mostrar “Pepito es menor de edad”**

```  
#!/bin/bash
clear
echo Dime un nombre
read nombre
echo Dime una edad
read edad
if [ $edad -lt 18 ]
then
        echo $nombre es menor de edad
else
        echo $nombre es mayor de edad
fi
``` 

16. **Pedir dos números y comprobar cual es mayor de los dos**

``` 
#!/bin/bash
clear
echo Dime un número
read num1
echo Dime otro número
read num2
if [ $num1 -ge $num2 ]
then
        echo $num1 es mayor o igual que $num2
else
        echo $num1 es menor que $num2
fi
``` 

17. **Pedir dos números y una palabra y si la palabra es “suma” pues sumar los números**

``` 
#!/bin/bash
clear
echo Dime una palabra
read palabra
echo Dime un número
read num1
echo Dime otro número
read num2

if [ $palabra == suma ]
then
        echo El resultado de la suma es: $(($num1 + $num2))
else
        echo La operación no es posible
fi
``` 

18. **Pedir dos números por teclado. Preguntar si desea hacer la suma**
    
- Si la respuesta es sí sumar los números
- Si la respuesta es distinta de sí que mostrar un mensaje “no quiero la suma”

``` 
#!/bin/bash
clear
echo Dime un número
read num1
echo Dime otro número
read num2
echo ¿Quieres hacer la suma?
read respuesta

if [ $respuesta == si ]
then
        echo El resultado de la suma es:$(($num1 + $num2))
else
        echo No quiero la suma
fi
```


19. **Pedir dos números por teclado. Pregunta si quieres ver el resultado de las operaciones:**
    
- Si la respuesta es SUMA, debe hacer la suma de los dos números
- Si la respuesta es RESTA, debe hacer la resta de los dos números
- Si he contestado otra cosa, que haga la multiplicación y la división

``` 
#!/bin/bash
clear
echo Dime un número
read num1
echo Dime otro número
read num2
echo ¿Quieres ver el resultado?
read respuesta

if [ $respuesta == suma ]
then
        echo El resultado de la suma es:$(($num1 + $num2))
else
        if [ $respuesta == resta ]
        then
                echo El resultado de la resta es:$(($num1-$num2))
        else
                echo El resultado de la multiplicación es:$(($num1*$num2)) y el resultado de la división es:$(($num1/$num2))
        fi
fi
``` 

20. **Pedir 3 números por teclado y que escribas suma, resta o multi.**
    
- Si escribo suma:	 sumar el valor de los números y mostrar por pantalla
- Si escribo resta:	 restar los dos primeros valores y mostrar por pantalla
- Si escribo multi:	 multiplicar el primer y tercer valor

``` 
#!/bin/bash
clear
echo Dime un número
read num1
echo Dime un segundo número
read num2
echo Dime un tercer número
read num3
echo ¿Quieres ver el resultado?
read respuesta

if [ $respuesta == suma ]
then
        echo El resultado de la suma es:$(($num1 + $num2 +$num3))
fi

if [ $respuesta == resta ]
then
                echo El resultado de la resta es:$(($num1-$num2))
fi      

if [ $respuesta == multiplicacion ]
then
        echo El resultado de la multiplicación es:$(($num1*$num3))
fi
``` 

21. **Introducir 3 números y `preguntar ¿Qué quieres hacer con los números, sumar, restar o multiplicar?Mostrar: 	El resultado de multiplicar, de restar o de sumar es: 	el que sea**

``` 
#!/bin/bash
clear
echo Introduce un número
read num1
echo Introduce otro número
read num2
echo Introduce un tercer número
read num3
echo ¿Qué quieres hacer con estos números?
read operacion

if [ $operacion == suma ]
then
        echo El resultado de la suma es:$(($num1 + $num2 +$num3))
fi

if [ $operacion == resta ]
then
                echo El resultado de la resta es:$(($num1-$num2-$num3))
fi

if [ $operacion == multiplicacion ]
then
        echo El resultado de la multiplicación es:$(($num1*$num2*$num3))
fi
``` 

22. **Pedir nombre y edad**
    
- Si la edad es < 10 años… 			“Pepe eres menor de 10 años”
- Si la edad está entre 11 y 18 años (incluido 18) … “Pepe eres un adolescente”
- Si la edad es mayor de 18 años…. 		“Pepe eres un adulto”

``` 
#!/bin/bash
clear
echo nombre
read nombre
echo edad
read edad

if [ $edad -lt 10 ]
then
        echo Pepe, eres menor de 10 años
fi

if [ $edad -le 18 ] && [ $edad -ge 10 ]
then
        echo Pepe, eres un adolescente
fi

if [ $edad -gt 18 ]
then
        echo Pepe, eres un adulto
fi
``` 

23. **Pedir una palabra. Si la palabra es hola debe:**
    
	* Mostrar el saludo: 		“Hola ¿cómo estás?
	* El directorio actual es:		 el que sea
	* El listado completo de la carpeta de usuario es: 	el que sea    

**Y en caso contrario:**

  * “No has escrito hola”

``` 
#!/bin/bash
clear
echo palabra
read palabra

if [ $palabra == hola ]
then
        echo Hola, ¿cómo estás?
        sleep 1
        pwd
        sleep 1
        ls -l
else
        echo No has escrito hola
fi
``` 

24. **Pedir un nombre y una edad por teclado y mostrar un mensaje diciendo:**
- “Pepe es menor de edad” o “Pepe es mayor de edad”
- Dependiendo de si el valor de la edad es > 18 años

``` 
#!/bin/bash
clear
echo Indica nombre
read nombre
echo Indica edad
read edad

if [ $edad -gt 18 ]
then
        echo $nombre es mayor de edad
else
        echo $nombre es menor de edad
fi
``` 

25. **Pedir una ciudad y si es Sevilla, Huelva, Cádiz, Badajoz, Barcelona o Madrid debe mostrar un mensaje diferente de esas ciudades**

``` 
#!/bin/bash
clear
echo Curiosidades de...
read ciudad

if [ $ciudad == Sevilla ]
then
        echo Capital de Andalucía, antigua Hispalia.
fi

if [ $ciudad == Madrid ]
then
        echo Capital de España, ciudad del chotis
fi

if [ $ciudad == Huelva ]
then
        echo Antigua Onuba
fi

if [ $ciudad == Badajoz ]
then
        echo Ciudad de los pacenses
fi

if [ $ciudad == Cádiz ]
then
        echo Ciudad de la alegria, cuna de Europa
fi
``` 

26. **Pedir una edad y preguntar si se es un bebe, niño, joven, adulto o anciano y dependiendo del valor mostrar un mensaje diferente**

``` 
#!/bin/bash
clear
echo ¿En qué etapa de tu vida estás?
read edad

if [ $edad == bebe ]
then
        echo Es hora de cambiar de pañal
fi

if [ $edad == niño ]
then
        echo Dejate de móvil y sal a la calle a que te de el aire
fi

if [ $edad == joven ]
then
        echo Dejate de porno y sal a la calle a que te de el aire
fi

if [ $edad == adulto ]
then
        echo Living la vida adulta
fi

if [ $edad == anciano ]
then
        echo Es hora de cambiar de pañal
fi
``` 

## Parte 2. Function, case...

27. **Crear un Menú (con case) para hacer una calculadora que sume, reste, multiplique y divida (con y sin funciones)**

``` 
#!/bin/bash
function suma ()
{
        resul=$(($num1+$num2))
}

function resta ()
{
        resul=$(($num1-$num2))
}

function multiplicacion ()
{
        resul=$(($num1*$num2))
}

function division ()
{
        resul=$(($num1/$num2))
}

echo Dime un número
read num1
echo Dime un segundo número
read num2

echo ¿Qué quieres hacer?
read opcion

case $opcion in
        +)
                suma;;
        -)
                resta;;
        x)
                multiplicacion;;
        /)
                division;;
        *)
esac

echo La operación indicada entre $num1 y $num2 da este resultado:$resul
```

## Parte 3. Bucles while, for, until.

28. **Mostrar por pantalla los valores del 1 al 10**

``` 
#!/bin/bash
clear

echo Números del 1 al 10:
num=0
while [ $num -lt 11 ]
do
        echo $num
        num=$(($num+1))
done
``` 

29. **Mostrar por pantalla los números pares entre el 1 y el 100**

``` 
#!/bin/bash
echo Los números pares entre 1 y 100 son los siguientes:
num=0
while test $num -le 100
do
        echo $num
        num=$(($num+2))
done
``` 

30. **Mostrar por pantalla los números impares entre el 1 y el 100**

``` 
#!/bin/bash
echo Los números impares entre 1 y 100 son los siguientes:
num=1
while test $num -le 100
do
        echo $num
        num=$(($num+2))
done
```

31. **Mostrar los cuadrados de los 10 primeros números Naturales**

``` 
#!/bin/bash
clear
num=0
res=0
while test $num -lt 10
do
        res=$(($num*$num))
        num=$(($num+1))
        echo $res
done
``` 

32. **Mostrar los números pares comprendidos entre dos números pasados por PARÁMETROS**

``` 
#!/bin/bash
echo Los números pares entre $1 y $2 son los siguientes:
num=$1
resto=$(($1%2))
if [ $resto -eq 1 ]
then
        num=$(($1+1))
fi
while test $num -le $2
do
                echo $num
                num=$(($num+2))
done
``` 

33. **Mostrar los números del 1 al 100, y el cuadrado, la suma y la multiplicación de cada uno**

``` 
#!/bin/bash
clear
echo Números del 1 al 100, con la suma y multiplicación consigo mismo
num=1
contador=0
while [ $contador -lt 100 ]
do
        contador=$(($contador+1))
        echo $contador
        suma=$(($contador+$contador))
        echo La suma de $contador + $contador es igual a $suma
        multiplicacion=$(($contador*$contador))
        echo La multiplicación de $contador por $contador es igual a $multiplicacion
done
``` 

34. **Mostrar el Factorial de un número N (teclado y PARÁMETRO)**

``` 
#!/bin/bash
clear
echo Vamos a mostrar el factorial de un número a indicar
echo Número a escoger
read num
operando1=1
operando2=1
echo $operando1
echo El resultado del factorial de $num es el siguiente:
while [ $operando1 -lt $num ]
do
        operando1=$(($operando1 +1))
        operando2=$(($operando1*$operando2))
        echo $operando2
done
``` 

35. **Mostrar los N (teclado) primeros números PARES (del 0 a N)**

``` 
#!/bin/bash
clear
echo Números pares desde el 0 al:
echo Indica número
read numero
echo Serían los siguientes:
echo 0
contador=0
resto=$(($numero%2))
if [ $resto -eq 1 ]
then numero=$(($numero-1))
fi
while [ $contador -lt $numero ]
do
        contador=$(($contador+2))
        echo $contador
done
```

36. **Mostrar los N (teclado) primeros números PARES (del N a 0)**

``` 
#!/bin/bash
clear
echo Números pares desde el que sea marcado al 0:
echo Indica número
read numero
contador=0
resto=$(($numero%2))
echo Serían los siguientes:
if [ $resto -eq 1 ]
then    numero=$(($numero-1))
fi
while [ $contador -lt $numero ]
do
        numero=$(($numero-2))
        echo $numero
done
``` 

37. **Mostrar los N (parámetro) primeros números IMPARES (del 1 a N)**

``` 
#!/bin/bash
clear
echo Números impares desde el 1 al:
echo Indica número
read numero
echo Serían los siguientes:
echo 1
contador=1
while [ $contador -lt $numero ]
do
        contador=$(($contador+2))
        echo $contador
done
```

38. **Mostrar los N (parámetro) primeros números IMPARES (del N a 1)**

``` 
#!/bin/bash
clear
echo Números impares desde el que sea marcado al 1:
echo Indica número
read numero
contador=1
resto=$(($numero%2))
echo Serían los siguientes:
if [ $resto -eq 0 ]
then    numero=$(($numero-1))
        echo $numero
fi
while [ $contador -lt $numero ]
do
        numero=$(($numero-2))
        echo $numero
done
``` 

39. **Mostrar los números de 5 en 5 del 1 al 400**

``` 
#!/bin/bash
clear
echo Mostrar numeros de 5 en 5 del 1 al 400
for contador in $(seq 0 5 400)
do
echo $contador
done
``` 

40. **Pedir un valor POR TECLADO y calcule la tabla de multiplicar de ese número**

``` 
#!/bin/bash
clear
echo Tabla de multiplicar de: $num
read num
for contador in $(seq 1 1 10)
do
multiplicacion=$(($num*$contador))
echo $num*$contador=$multiplicacion
done
``` 

41. **Introducir 2 valores por PARÁMETROS y calcular la suma, el producto y la tabla de multiplicar de cada uno**

``` 
#!/bin/bash
clear
contador=1
echo Vamos a realizar una suma y mostrar la tabla de multiplicar de $1 y $2
contador1=1
contador2=1
suma=$(($1+$2))
echo La suma de $1 + $2 da el siguiente resultado=$suma
echo La tabla de multiplicar de $1 es:
echo $1
while [ $contador -lt 10 ]
do
        contador=$(($contador+1))
        multiplicacion=$(($1*$contador))
        echo $1 por $contador es:$multiplicacion
done
echo La tabla de multiplicar de $2 es:
echo $2
until [ $contador2 -ge 10 ]
do
        contador2=$(($contador2+1))
        multip=$(($2*$contador2))
        echo $2 por $contador2 es:$multip
done
``` 

## Parte 4. Archivos de texto 

42. **Pedir la tabla de multiplicar del número N (teclado) hasta que pulse la letra q. Estas tablas se llevarán a un fichero llamado compratem.txt. Mostrar el contenido del fichero ordenado**

``` 
#!/bin/bash
clear
until [ $opcion = si ]
do
        echo Vamos a conocer la tabla de multiplicar de $num
        read num
        contador=0

        while [ $contador -lt 10 ]
        do
                contador=$(($contador+1))
                multip=$(($num*$contador))
                echo $num por $contador es igual a:$multip>>compratem.txt
        done
                echo ¿Desea salir o mostrar otro número?
                read opcion
                if [ $opcion = si ]
                then
                        sort compratem.txt
                fi
done
``` 

43. **Pedir 5 nombres por teclado y los almacene en un fichero llamado nombres.txt. Ordenar alfabéticamente el fichero llamándolo nombres-ordenado.txt y mostrarlo tanto desordenado como ordenado**

``` 
#!/bin/bash
clear
echo Introduce nombre
read nom1
echo Introduce nombre
read nom2
echo Introduce nombre
read nom3
echo Introduce nombre
read nom4
echo Introduce nombre
read nom5
registro="$nom1
$nom2
$nom3
$nom4
$nom5"
echo $registro >> nombres.txt
cat nombres.txt >> nombresenorden.txt
echo Veamoslo de forma ordenada
sleep 3
sort nombresenorden.txt
echo Ahora los vamos a ver de forma desordenada
sleep 3
cat nombres.txt
``` 

44. **Pedir 5 nombres, los almacene en un fichero mi-agenda-1.txt y mostrar el contenido del fichero**

``` 
#!/bin/bash
clear
echo Introduce nombre
read nom1
echo Introduce nombre
read nom2
echo Introduce nombre
read nom3
echo Introduce nombre
read nom4
echo Introduce nombre
read nom5
registro="$nom1 $nom2 $nom3 $nom4 $nom5"
echo $registro >> mi-agenda-1.txt
cat mi-agenda-1.txt
``` 

45. **Pedir 5 nombres y los almacene en el fichero “mi-agenda-2.txt”. Ordenar el contenido de la agenda (mi-agenda-2-ord.txt) y mostrar el fichero ordenado**

``` 
#!/bin/bash
clear
echo Introduce nombre
read nom1
echo Introduce nombre
read nom2
echo Introduce nombre
read nom3
echo Introduce nombre
read nom4
echo Introduce nombre
read nom5
registro="$nom1, $nom2, $nom3, $nom4, $nom5"
echo $registro >> mi-agenda-2.txt
cat mi-agenda-2.txt >>mi-agenda-2-ord.txt
sort mi-agenda-2-ord.txt
```

46. **Pedir 3 nombres que introduzcamos por línea de comandos. Añadir a un fichero mifichero.txt. Ordenar este fichero en mificheroordenado.txt. Mostrar el contenido de los dos ficheros. Borrar el primer fichero mifichero.txt.**

``` 
#!/bin/bash
clear
echo Introduce nombre
echo Mónica Gordillo
echo Introduce nombre
echo Frodo
echo Introduce nombre
echo Daralea Lujan
registro="Mónica Gordillo, Frodo, Daralea Lujan"
echo $registro >> mifichero.txt
cat mifichero.txt
cat mifichero.txt >>mificheroordenado.txt
sort mificheroordenado.txt
rm mifichero.txt
``` 

47. **Pedir nombres de amigos tuyos hasta que escriba la palabra “salir”. Almacenar estos nombres en un fichero llamado mis-amigos.txt. Mostrar el contenido del fichero “amigos.txt”**

``` 
#!/bin/bash
clear
nombre=x
while [ $nombre != "salir" ]
do
        echo Introduzca nombre y apellidos
        read nombre
        echo $nombre >> misamigos.txt
done
cat misamigos.txt>>amigos.txt
cat amigos.txt
``` 

48. **Pedir nombres de amigos tuyos y sus correos electrónicos hasta que escriba la palabra “salir” Almacenar estos datos en un fichero llamado “amigos- email.txt”. Mostrar el contenido del fichero “amigos-email-ordenado.txt” ordenado**

``` 
#!/bin/bash
clear
echo ¿Desea introducir un nuevo contacto?
read opcion
until [ $opcion = no ]
do
        echo Introduce un nombre
        read nom
        echo Introduce apellidos
        read apellidos
        echo Introduce email
        read email
        registro="$nom, $apellidos, $email"
        echo $registro >> amigos-email.txt
         cat amigos-email.txt

        echo ¿Quieres continuar?
        read opcion
done
cat amigos-email.txt >> amigos-email-ordenado.txt
sort amigos-email-ordenado.txt
``` 

49. **Pedir nombres de amigos tuyos y sus correos electrónicos hasta que escriba la palabra “salir”.Almacenar estos datos en un fichero llamado “amigos-email.txt”. Mostrar aquellos amigos tuyos que tengan email de Gmail**

``` 
#!/bin/bash
clear
echo ¿Desea introducir un nuevo contacto?
read opcion
until [ $opcion = no ]
do
        echo Introduce un nombre
        read nom
        echo Introduce apellidos
        read apellidos
        echo Introduce email
        read email
        registro="$nom, $apellidos, $email"
        echo $registro >> amigos-email.txt

        echo ¿Quieres continuar?
        read opcion
done
cat amigos-email.txt | grep "@gmail"
```

50. **Imprimir en un fichero llamado “numeros-1.txt”**
    
* Impares menores de 100. (con while, until y for)
* Pares entre 100 y 200. (con while, until y for)
* Pares menores de 100 (con while, until y for)
* Tabla de multiplicar del 5 (con while, until y for)

``` 
#!/bin/bash
clear
num1=1
echo Los números impares menores de 100 son:$num1>>numeros1.txt
while [ $num1 -lt 99 ]
do
        num1=$(($num1+2))
        echo $num1>>numeros1.txt
done

num2=100
echo Los números pares entre 100 y 200 son:$num2 >>numeros1.txt
while [ $num2 -lt 200 ]
do
        num2=$(($num2+2))
        echo $num2>>numeros1.txt
done

echo Los números pares menores de 100 son:>>numeros1.txt
for contador in $(seq 0 2 99)
do
        echo $contador>>numeros1.txt
done

num3=5
echo La tabla de multiplicar del cinco da los siguientes resultados:>>numeros1.txt
for contador in $(seq 1 1 10)
do
        multip=$(($num3*$contador))
        echo $multip>>numeros1.txt
done
cat numeros1.txt
``` 

## Parte 5. Usuarios y grupos

51. **Crear un grupo llamado mi-clase con GID 2220**

``` 
#!/bin/bash
clear
sudo groupadd -g 2220 mi-clase
echo El grupo mi-clase ha sido creado
``` 

52. **Crear un usuario llamados smr-A con UID 1111**

``` 
#!/bin/bash
clear
sudo useradd -u 1111 smr-A
echo El usuario smr-A ha sido creado
``` 

53. **Crear un usuario llamados smr-B con UID 1112**

``` 
#!/bin/bash
clear
sudo useradd -u 1112 smr-B
echo El usuario smr-B ha sido creado
``` 

54. **Introduce smr-A en el grupo mi-clase**

``` 
#!/bin/bash
clear
sudo usermod -a -G mi-clase smr-A
echo El usuario smr-A ha sido integrado al grupo mi-clase
``` 

55. **Introduce smr-B en el grupo mi-clase**

``` 
#!/bin/bash
clear
sudo usermod -a -G mi-clase smr-B
echo El usuario smr-B ha sido integrado al grupo mi-clase
```

56. **Crea 5 usuarios llamados Osttos, Eduflow, Yisus, Cr7 y CalvoGeek y 3 grupos llamados Alumnos, Profesores, Comandante**

``` 
#!/bin/bash
clear
sudo useradd Osttos
echo El usuario Osttos ha sido creado
sudo useradd Eduflow
echo El usuario Eduflow ha sido creado
sudo useradd Yisus
echo El usuario Yisus ha sido creado
sudo useradd Cr7
echo El usuario Cr7 ha sido creado
sudo useradd CalvoGeek
echo El usuario CalvoGeek ha sido creado
sudo groupadd Alumnos
echo El grupo Alumnos ha sido creado
sudo groupadd Profesores
echo El grupo Profesores ha sido creado
sudo groupadd Comandante
echo El grupo Comandante ha sido creado
```

57. **Incluir usuarios en grupos:**
    
* Osttos en Profesores y Comandante
* Eduflow en Profesores
* Yisus, Cr7 y CalvoGeek en Alumnos

``` 
#!/bin/bash
clear
sudo usermod Osttos -a -G Profesores
echo El usuario Osttos ha sido agregado al grupo Profesores
sudo usermod Osttos -a -G Comandante
echo El usuario Osttos ha sido agregado al grupo Comandante
sudo usermod Eduflow -a -G Profesores
echo El usuario Eduflow ha sido agregado al grupo Profesores
sudo usermod Yisus -a -G Alumnos
echo El usuario Yisus ha sido agregado al grupo Alumnos
sudo usermod CalvoGeek -a -G Alumnos
echo El usuario CalvoGeek ha sido agregado al grupo Alumnos
sudo usermod Cr7 -a -G Alumnos
echo El usuario Cr7 ha sido agregado al grupo Alumnos
``` 

## Parte 6. Menús

58. **Realizar el siguiente script:**
	
**MENÚ**
1.- Pedir dos números
2.- Sumar los dos números
3.- Multiplicar los dos números
4.- Mostrar la tabla de multiplicar del primer número
5.- Mostrar la tabla de multiplicar del segundo número
6.- Indicar cual es el número mayor
7.- Restar del número mayor – menor
8.- Dividir del número mayor / menor
9.- Salir

``` 
#!/bin/bash
function menu
{
        echo 1.- Pedir dos números
        echo 2.- Sumar los dos números
        echo 3.- Multiplicar los dos números
        echo 4.- Tabla de multiplicar del primer número
        echo 5.- Tabla de multiplicar del segundo número
        echo 6.- Indicar el número mayor
        echo 7.- Restar el número mayor - menor
        echo 8.- Dividir el número mayor/menor
        echo 9.- Salir
}

clear
menu
read num
while [ $num -le 9 ]
do
        clear
        case $num
        in
                1)
                        echo Introduzca el primer número
                        read num1
                        echo Introduzca el segundo número
                        read num2;;
                2)
                        echo La suma de $num1 + $num2 es igual a:$(($num1+$num2));;
                3)
                        echo La multiplicación de $num1 por $num2 es igual a:$(($num1*$num2));;
                4)
                        echo Tabla de multiplicar de $num1
                        contador=0
                        while [ $contador -lt 10 ]
                        do
                                contador=$(($contador+1))
                                multiplicacion=$(($num1*$contador))
                                echo $num1 por $contador es igual a:$multiplicacion
                        done;;
                5)
                        echo Tabla de multiplicar de $num2
                        contador2=0
                        while [ $contador2 -lt 10 ]
                        do
                                contador2=$(($contador2+1))
                                multip=$(($num2*$contador2))
                                echo $num2 por $contador2 es igual a:$multip
                        done;;
                6)
                        if [ $num1 -gt $num2 ]
                        then
                                echo $num1 es mayor que $num2
                        else
                                echo $num2 es mayor que $num1
                        fi;;
                7)
                        if [ $num1 -gt $num2 ]
                        then
                                resta=$(($num1-$num2))
                                echo La resta de $num1 menos $num2 es igual a:$resta
                        else
                                resta2=$(($num2-$num1))
                                echo La resta de $num2 menos $num1 es igual a:$resta2
                        fi;;
                8)
                        if [ $num1 -gt $num2 ]
                        then
                                division=$(($num1/$num2))
                                echo La división de $num1 entre $num2 es igual a:$division
                        else
                                divis=$(($num2/$num1))
                                echo La división de $num2 entre $num1 es igual a:$divis
                        fi;;
                9)
                        break;;
                *)
                        echo Debe ser una opción válida;;
        esac
        menu
        read num
done
``` 

59. **Realizar el siguiente script:**

**MENÚ**

1.- Pedir el nombre de una carpeta.
Comprobar que la carpeta no existe. Nombre de la carpeta ADIOS.
2.- Pedir 5 artículos para comprar y su precio.
Insertar los datos en un archivo llamado micompra.txt dentro de ADIOS
3.- Ordenar micompra.txt y crea micompraorden.txt.
4.- Buscar en mi-compra-orden.txt aquellos artículos que su precio sea de 10 Euros.
5.- Pedir un número y muestra su tabla de multiplicar.
6.- Crear un fichero llamado impares.txt con los números impares del 1 al 100
7.- Crear un fichero llamado pares.txt con los números pares del 0 al 100
8.- Crear un fichero llamado pares-2.txt en ADIOS con los números pares del 1 al 55
9.- Crear una copia, un enlace duro y un enlace simbólico de mi compra-txt
10.- Salir


``` 
#!/bin/bash
function menu
{
        echo 1.- Pedir el nombre de una carpeta
        echo 2.- Pedir artículos para comprar y su precio
        echo 3.- Ordenar micompra.txt y crear micompra-orden.txt
        echo 4.- Buscar en micompra-orden.txt una serie de artículos
        echo 5.- Pedir un número y mostrar su tabla de multiplicar
        echo 6.- Crear un fichero llamado impares.txt
        echo 7.- Crear un fichero llamado pares.txt
        echo 8.- Crear un fichero llamado pares2.txt
        echo 9.- Crear copia, enlace duro y simbólico de micompra.txt
        echo 10.- Salir
}

clear
menu
read num
while [ $num -le 10 ]
do
        clear
        case $num
        in
                1)
                        echo Dime el nombre de una carpeta
                        read nom
                        cat $nom
                        if ! [ -e $nom ]
                        then
                                echo El directorio $nom no existe
                                mkdir $nom
                                else
                                echo El directorio $nom existe
                        fi;;
                2)
                        if ! [ -e ADIOS/micompra.txt ]
                        then
                                touch ADIOS/micompra.txt
                        fi
                        echo Opcion
                        read opcion
                        until [ $opcion = si ]
                        do
                                echo Dame un artículo
                                read articulo
                                echo Dame un precio 
                                read precio
                                echo $articulo:$precio>>ADIOS/micompra.txt
                                echo ¿Deseas salir? Si o no
                                read $opcion
                        done;;
                3)
                        if ! [ -e ADIOS/micompra-orden.txt ]
                        then
                                touch ADIOS/micompra-orden.txt
                        fi
                        sort ADIOS/micompra.txt>>ADIOS/micompra-orden.txt;;
                4)
                        cat micompra-orden.txt | grep "10 Euros";;
                5)
                        echo Dime un número
                        read num1
                        contador=0
                        echo La tabla de multiplicar de $num1 es:
                        while [ $contador -lt 10 ]
                        do
                                contador=$(($contador+1))
                                multip=$(($num1*$contador))
                                echo $num1 por $contador=$multip
                        done;;
                6)
                        if ! [ -e impares.txt ]
                        then
                                touch impares.txt
                        fi
                        for numeroimpar in $(seq 1 2 99)
                        do
                                echo $numeroimpar>>impares.txt
                        done;;
                7)
                        if ! [ -e pares.txt ]
                        then
                                touch pares.txt
                        fi
                        for numeropar in $(seq 0 2 100)
                        do
                                echo $numeropar>>pares.txt
                        done;;
                8)
                        if ! [ -e ADIOS/pares2.txt ]
                        then
                                touch ADIOS/pares2.txt
                        fi
                        for contadorpar in $(seq 0 2 54)
                        do
                                echo $contadorpar>> ADIOS/pares2.txt
                        done;;
                9)
                        cp ADIOS/micompra.txt micompra-copia.txt
                        echo Nombre de la copia: micompra-copia.txt
                        ln ADIOS/micompra.txt micompra-duro.txt
                        echo Nombre del enlace duro: micompra-duro.txt
                        ln -s ADIOS/micompra.txt micompra-simb.txt
                        echo Nombre del enlace simbólico: micompra-simb.txt;;
                10)
                        break;;
        esac
        menu
        read num
done
``` 

60. **Realizar el siguiente script:**

**MENÚ**

1.- Crear una carpeta en tu home llamada HOLA
2.- Situarte en la carpeta HOLA
3.- Pedir nombre de 3 amigos y sus teléfonos y guárdalos en un archivo agenda.txt dentro de HOLA
Introduce el nombre de un amigo y su teléfono Pedro - 77777777
4.- Ordenar agenda.txt y guárdala como agenda-ordenada.txt en HOLA
5.- Borrar agenda.txt
6.- Crear un enlace duro de agenda llamada agenda-duro.txt
7.- Crear un enlace simbólico de agenda llamada agenda-simb.txt
8.- Visualizar la estructura de la carpeta HOLA
9.- Borrar la pantalla
10.- Salir

``` 
#!/bin/bash
function menu
{
        echo 1.- Crear una carpeta en mi home 
        echo 2.- Situarte en la carpeta HOLA
        echo 3.- Crear registros y almacenarlos en agenda.txt
        echo 4.- Ordenar agenda.txt y guardar como agenda-ordenada.txt
        echo 5.- Borrar agenda.txt
        echo 6.- Crear un enlace duro llamado agenda-duro.txt
        echo 7.- Crear un enlace simbólico llamado agenda-simb.txt
        echo 8.- Visualizar estructura de HOLA
        echo 9.- Borrar la pantalla
        echo 10.- Salir
}

clear
menu
read num
while [ $num -le 10 ]
do
        clear
        case $num
        in
                1)
                        if ! [ -e HOLA ]
                        then
                                 mkdir HOLA
                        fi;;
                2)
                        cd HOLA;;
                3)
                        if ! [ -e HOLA/agenda.txt ]
                        then
                                touch HOLA/agenda.txt
                        fi
                        echo Opcion
                        read opcion
                        until [ $opcion = si ]
                        do
                                echo Indica nombre de un amigo
                                read nom
                                echo Inserta un número de tlf
                                read tlf
                                echo $nom:$tlf>>HOLA/agenda.txt
                                echo ¿Desea salir? Si o no
                                read $opcion
                        done;;
                4)
                        sort HOLA/agenda.txt>>HOLA/agenda-ordenada.txt;;
                5)
                        rm HOLA/agenda.txt;;
                6)
                        ln HOLA/agenda.txt HOLA/agenda-duro.txt;;
                7)
                        ln -s HOLA/agenda.txt HOLA/agenda-simb.txt;;
                8)
                        tree HOLA;;
                9)
                        clear;;
                10)
                        break;;
        esac
        menu
        read num
done
``` 

## Ejercicio adicional de texto

61. **Meter registros de 1 en 1 hasta que queramos:**

``` 
#!/bin/bash
clear
echo opcion
read opcion
until [ $opcion = no ]
do
        echo Introduce un nombre
        read nom
        echo Introduce apellidos
        read apellidos
        echo Introduce teléfono
        read telf
        registro="$nom, $apellidos, $telf"
        echo $registro >> agenda.txt
         cat agenda.txt

        echo ¿Quieres continuar?
        read opcion
done
``` 






























