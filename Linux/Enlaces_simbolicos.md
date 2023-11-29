# Conceptos básicos de la línea de comandos: enlaces simbólicos

## 1. Introducción

Los enlaces simbólicos le permiten vincular archivos y directorios a otros archivos y directorios. Tienen muchos nombres, incluidos enlaces simbólicos, enlaces de shell, enlaces suaves, accesos directos y alias. Desde la perspectiva del usuario, los enlaces simbólicos son muy similares a archivos y directorios normales. Sin embargo, cuando interactúas con ellos, en realidad interactuarán con el objetivo en el otro extremo. Piense en ellos como agujeros de gusano para su sistema de archivos.

Esta guía proporciona una descripción general de qué son los enlaces simbólicos y cómo crearlos desde una línea de comandos de Linux usando el ln comando.


## 2. Configuración de directorios y archivos de ejemplo

Comience creando un par de directorios dentro del /tmp/ directorio. /tmp/es un directorio temporal, lo que significa que todos los archivos y directorios que contenga se eliminarán la próxima vez que se inicie el servidor. Esto será útil para los propósitos de esta guía, ya que puede crear tantos directorios, archivos y enlaces como desee sin tener que preocuparse de que obstruyan su sistema más adelante.

El siguiente mkdir comando crea tres directorios a la vez. Crea un directorio nombrado symlinks/ dentro del /tmp/ directorio y dos directorios (uno con nombre one/ y otro con nombre two/) dentro de symlinks/:

``` 
mkdir -p /tmp/symlinks/{one,two}
```

Navegue al nuevo symlinks/ directorio:

``` 
cd /tmp/symlinks
```

A partir de ahí, cree un par de archivos de muestra, uno para ambos subdirectorios dentro de symlinks/. El siguiente comando crea un archivo nombrado one.txt dentro del one/ subdirectorio cuyo único contenido es una sola línea que dice one:

``` 
echo "one" > ./one/one.txt
``` 

De manera similar, este comando crea un archivo nombrado two.txt dentro del two/ subdirectorio cuyo único contenido es una sola línea que dice two:

``` 
echo "two" > ./two/two.txt
``` 

Si ejecutara tree en este punto para mostrar el contenido de todo el /tmp/symlinks directorio y cualquier subdirectorio anidado, su salida se vería así:

``` 
tree
Output
.
├── one
│   └── one.txt
└── two
    └── two.txt

2 directories, 2 files
``` 

Con estos documentos de muestra en su lugar, estará listo para practicar cómo crear enlaces simbólicos.


## 3. Comprender los enlaces duros

De forma predeterminada, el ln comando creará enlaces físicos en lugar de enlaces simbólicos o suaves.

Digamos que tienes un archivo de texto. Si crea un enlace simbólico a ese archivo, el enlace es sólo un puntero al archivo original. Si elimina el archivo original, el enlace se romperá porque ya no tiene nada a lo que apuntar.

En cambio, un enlace físico es una copia reflejada de un archivo original con exactamente el mismo contenido. Al igual que los enlaces simbólicos, si edita el contenido del archivo original, esos cambios se reflejarán en el enlace físico. Sin embargo, si elimina el archivo original, el enlace físico seguirá funcionando y podrá verlo y editarlo como lo haría con una copia normal del archivo original.

Los enlaces duros cumplen su propósito en el mundo, pero en algunos casos deben evitarse por completo. Por ejemplo, debes evitar el uso de enlaces físicos al vincular dentro de un git repositorio, ya que pueden causar confusión.

Para asegurarse de que está creando enlaces simbólicos, puede pasar la opción -so --symbolical ln comando.


## 4. Trabajar con enlaces simbólicos

Como se mencionó anteriormente, el enlace simbólico es esencialmente como crear un archivo que contiene el nombre de archivo y la ruta del destino. Debido a que un enlace simbólico es solo una referencia al archivo original, cualquier cambio que se realice en el original estará disponible inmediatamente en el enlace simbólico.

Un uso potencial de los enlaces simbólicos es crear directorios locales en el directorio de inicio de un usuario que apunten a archivos que se sincronizan con una aplicación externa, como Dropbox. Otra podría ser crear un enlace simbólico que apunte a la última versión de un proyecto que reside en un directorio con nombre dinámico.

Usando los archivos y directorios de ejemplo de la primera sección, continúe e intente crear un enlace simbólico llamado three que apunte al one directorio que creó anteriormente:

```
ln -s one three
``` 

Ahora debería tener 3 directorios, uno de los cuales apunta a otro. Para obtener una descripción más detallada de la estructura del directorio actual, puede usar el ls comando para imprimir el contenido del directorio de trabajo actual:

``` 
ls
Output
one  three  two
``` 

Ahora hay tres directorios dentro del symlinks/ directorio. Dependiendo de su sistema, puede significar que three en realidad se trata de un enlace simbólico. A veces, esto se hace representando el nombre del enlace en un color diferente o agregándole un @ símbolo.

Para obtener aún más detalles, puede pasar el -l argumento a ls para determinar hacia dónde apunta realmente el enlace simbólico:

``` 
ls -l
Output
total 8
drwxrwxr-x 2 sammy sammy 4096 Oct 30 19:51 one
lrwxrwxrwx 1 sammy sammy    3 Oct 30 19:55 three -> one
drwxrwxr-x 2 sammy sammy 4096 Oct 30 19:51 two
``` 

Observe que el three enlace apunta al one directorio como se esperaba. Además, comienza con l, lo que indica que es un enlace. Los otros dos comienzan con d, lo que significa que son directorios normales.

Los enlaces simbólicos también pueden contener enlaces simbólicos. Como ejemplo, vincule el one.txt archivo desde three al two directorio:

``` 
ln -s three/one.txt two/one.txt
``` 

Ahora debería tener un archivo con nombre one.txt dentro del two directorio. Puedes comprobarlo con el siguiente ls comando:

``` 
ls -l two/
Output
total 4
lrwxrwxrwx 1 sammy sammy 13 Oct 30 19:58 one.txt -> three/one.txt
-rw-rw-r-- 1 sammy sammy  4 Oct 30 19:51 two.txt
``` 

Dependiendo de la configuración de su terminal, el enlace (resaltado en el resultado de este ejemplo) puede aparecer en texto rojo, lo que indica un enlace roto. Aunque se creó el vínculo, la forma en que este ejemplo especificaba la ruta era relativa. El enlace está roto porque el two directorio no contiene un three directorio con el one.txt archivo.

Afortunadamente, puede remediar esta situación indicando ln que se cree el enlace simbólico relativo a la ubicación del enlace utilizando el argumento -ro .--relative

Sin embargo, incluso con la -r bandera, no podrás arreglar el enlace simbólico roto. La razón de esto es que el enlace simbólico ya existe y no podrá sobrescribirlo sin incluir también el argumento -fo :--force

``` 
ln -srf three/one.txt two/one.txt
```

Con eso, ahora tienes two/one.txt cuál estaba vinculado a three/one.txt cuál es un enlace a one/one.txt.

Anidar enlaces simbólicos como este puede resultar confuso rápidamente, pero muchas aplicaciones están equipadas para hacer que dichas estructuras de enlaces sean más comprensibles. Por ejemplo, si ejecutara el tree comando, el destino del enlace que se muestra es en realidad el de la ubicación del archivo original y no el enlace en sí:

``` 
tree
Output
.
├── one
│   └── one.txt
├── three -> one
└── two
    ├── one.txt -> ../one/one.txt
    └── two.txt

3 directories, 3 files
``` 

Ahora que las cosas están bien conectadas, puede comenzar a explorar cómo funcionan los enlaces simbólicos con los archivos alterando el contenido de estos archivos de muestra.

Para tener una idea de lo que contienen sus archivos, ejecute el siguiente cat comando para imprimir el contenido del one.txt archivo en cada uno de los tres directorios que ha creado en esta guía:

``` 
cat {one,two,three}/one.txt
Output
one
one
one
``` 

A continuación, actualice el contenido del one.txt archivo original desde el one/ directorio:

``` 
echo "1. One" > one/one.txt
``` 

Luego verifique el contenido de cada archivo nuevamente:

``` 
cat {one,two,three}/one.txt
Output
1. One
2. One
3. One
``` 

Como indica este resultado, cualquier cambio que realice en el archivo original se reflejará en cualquiera de sus enlaces simbólicos.

Ahora prueba al revés. Ejecute el siguiente comando para cambiar el contenido de uno de los enlaces simbólicos. Este ejemplo cambia el contenido del one.txt archivo dentro del three/ directorio:

``` 
echo "One and done" > three/one.txt
``` 

Luego verifique el contenido de cada archivo una vez más:

``` 
cat {one,two,three}/one.txt
Output
One and done
One and done
One and done
``` 

Debido a que el enlace simbólico que cambió es solo un puntero a otro archivo, cualquier cambio que realice en el enlace se reflejará inmediatamente en el archivo original, así como en cualquiera de sus otros enlaces simbólicos.



