# Información extraída del curso Oracle Base de Datos 21c desde cero para principiantes

## 1. Crear base de datos

Vamos al directorio donde se alojan todos los archivos que tienen que ver con Oracle. En este curso:

``` 
cd u01/app/oracle/product/21.0.0/db_1
```

Hacemos un ls para ver todos estos archivos. Uno de los directorios más importantes será **bin**. Accedemos a él, y volvemos a hacer un listado. Dentro del contenido, nos quedamos con dbca (asistente de configuración de base de datos). 

Si hacemos lo siguiente:

``` 
./dbca
```

Se nos abrirá una interfáz gráfica de Oracle para crear la base de datos. El asistente es facil de seguir, y no requiere de grandes complicaciones más allá de los ajustes que deseemos añadir en nuestra base de datos (nombre, contraseña...)

La instalación seguirá su curso y nos indicará donde podemos visionar los logs de la base de datos. 

Una vez realizada la instalación, se nos creará una serie de archivos. Entre ellos cabe destacar el directorio ORADATA y el de fast_recovery, ambos dentro de u01/app/oracle. Ahí encontraremos los logs de la db.

Para ver los procesos pertenecientes a Oracle:

``` 
ps -ef | grep orcl
```

Para ver los segmentos de memoria y su uso por parte de nuestra instancia Oracle:

``` 
ipcs -a
```

## 2. Creación de una segunda base de datos

Volvemos a lanzar el asistente:

``` 
./dbca
```

En esta ocasión, ya nos aparecen casi todas las opciones a poder realizar con una base de datos.

Si vamos a **Configuración avanzada**, podemos crear una base de datos de manera personalizada. Por ejemplo, el tipo (RAC).


## 3. Variables de entorno

Dentro de /etc tenemos un archivo llamado **oratab**. En él, aparecen las bases de datos que tengamos en Oracle junto a la ruta de su directorio. Si de ahi, pasamos a /usr/local/bin, vemos que tenemos un archivo llamado **oraenv**. Es en este archivo donde podemos asignar las variables de entorno, donde puedo decir con que base de datos quiero trabajar en ese momento.

``` 
. oraenv
```

Nos preguntará con que base de datos queremos conectar, lo indicamos y nos dirá su ruta de directorio. 

Para ver estas variables de entorno:

``` 
printenv | grep ORACLE
```

Y para pasar a otra base de datos solo tenemos que ejecutar oraenv e insertar el nombre de la base de datos a la que queremos pasar. 


## 4. Introducción a SQLPlus

Comenzamos yendo a nuestro directorio home de la base de datos:

``` 
cd $ORACLE_HOME
```

De ahí pasamos a bin y si listamos vemos que tenemos un archivo llamado **sqlplus**. 

Si quiero conectarme a mi base de datos mediante sqlplus:

``` 
sqlplus system/contraseña de esa base de datos
```

Si quisiera conectarme mediante sys:

``` 
sqlplus sys/contraseña de esa base de datos sysdba
```

Por ejemplo, para apagar la base de datos:

``` 
shutdown immediate;
```


## 5. Iniciar la base de datos

Para iniciar la base de datos:

``` 
startup
```

Podemos iniciar la base de datos sin abrirla:

``` 
startup nomount
```

Y si después quisiera abrirla, solo tendría que insertar lo siguiente:

``` 
ALTER DATABASE MOUNT;
```

Y:

``` 
ALTER DATABASE OPEN;
```


## 6. Apagar la base de datos

Para terminar esperando a que las transacciones terminen:

``` 
SHUTDOWN TRANSACTIONAL;
```

Para terminar sin esperar transacciones, ni nada:

``` 
SHUTDOWN ABORT;
```

## 7. Recuperar info de la SGA

Vemos las tablas dinámicas relacionadas con SGA (System Global Area):

``` 
select * from v$fixed_table where name like 'v$SGA%';
```

Para ver todo de SGA:

```
SHOW SGA;
```

## 8. Parametros

Para mostrarlos:

``` 
SHOW PARAMETERS;
```

O bien:

``` 
select * from v$parameters;
```

Y si quisiera modificar uno de estos parametros, por ejemplo el lenguaje de mi sistema:

``` 
ALTER SESSION SET NLS_LANGUAGE = 'SPANISH';
```


## 9. Conexiones locales y servicios

Para ver los procesos en local:

``` 
ps -ef | grep LOCAL
```

Para ver los usuarios y la forma de acceso a la base de datos:

``` 
select username, command from v$session;
```

Para que la consulta sea más limpia y facil de analizar:

``` 
col username format A30
r
```

Y voy a cambiar command en lugar de program:

``` 
c .command.program.
r
```

Y en SYS, por ejemplo, vemos que la conexión se ha realizado a través de sqlplus y bajo el protocolo de conexión TNS.

Para ver los servicios de nuestra base de datos:

``` 
select name, value from v$system_parameter where name ='service_names';
```

Para ver el nombre de la db:

``` 
select name, value from v$system_parameter where name ='db_name';
```

O el dominio de la misma:

``` 
select name, value from v$system_parameter where name ='db_domain';
```

## 10. Conexiones remotas

Antes de establecer la conexión, será necesario crear un listener. 

Hay tres ficheros básicos en la configuración de red de Oracle:

* **listener.ora**
* **tnsnames.ora**
* **sqlnet.ora**

El archivo **listener.ora** contiene parámetros de configuración de red del lado del servidor y se encuentra en el directorio $ORACLE_HOME/network/admin del servidor. 

El archivo **tnsnames.ora** y el **sqlnet.ora** contienen parámetros de configuración de red del lado del cliente y se encuentran en el directorio $ORACLE_HOME/network/admin del cliente. 

Para crear un listener nos vamos al directorio ya citado:

``` 
cd $ORACLE_HOME/network/admin
```

Y en el directorio samples, damos con nuestros archivos, entre ellos el listener.ora.

Vamos a este archivo con nano y hallamos las intrucciones para crear un listener. 

Sin embargo, hay un comando que permite hacer todo esto de una manera mucho más sencilla:

```
netca
```

Se nos abre un asistente de configuración de red de Oracle. Ahí podemos crear nuestro listener en base a los parámetros deseados, como por ejemplo el protocolo de conexión. 

### 10.1 EL LSNCTRL

Hay otro comando relacionado con el listener como es el lsnctrl. Es una herramienta encargada de controlar el listener, parecida al sqlplus.

Si accedemos a ese comando podemos ver por ejemplo el estado:

```
status
```

Trás esto, para hacer una conexión remota, voy a sqlplus con su comando e iniciamos la conexión remota.

``` 
system/contraseña@localhost:1521/orcl2
```











