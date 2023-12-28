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

## 11. Borrar la base de datos

Para ello: 

``` 
dbca
```

Se nos abre el asistente. Damos en Suprimir base de datos. 


## 12. Instalar SQL Developer

En primer lugar, hemos de tener en cuenta que igualmente tenemos que instalar Java. Cuando hayamos descargado e instalado tanto Java como SQL Developer, accedemos al directorio de este y nos quedamos con el archivo **sqldeveloper.sh**. Lo ejecutamos:

``` 
./sqldeveloper.sh
```

## 13. Tablespaces

Ya dentro de la interfáz gráfica, podemos ver los archivos asociados con los tablespaces de almacenamiento:

``` 
select * from dba_data_files;
```

O las propias tablespaces:

``` 
select * from dba_tablespaces;
```

O el archivo de los tablespaces de temp:

``` 
select * from dba_temp_files;
```

Para hacer estas consulta de manera más dinámica, y a veces más eficaz:

``` 
select * from v$datafile;
select * from v$tempfile;
```

Para ver los espacios libres que tienen nuestras tablespaces:

``` 
select * from dba_free_space;
```

## 14. Cómo crear tablespaces

Vamos al siguiente directorio:

``` 
cd /home/oracle/u01/app/oracle/oradata/ORCL
```

Ahora vamos al directorio de sqldeveloper y ejecutamos su archivo de inicio:

``` 
./sqldeveloper.sh
```

Hecho esto, vamos al directorio home y creamos una carpeta (datos). Y cambiamos el propietario de esa carpeta, para que sea el usuario de nuestra base de datos.

```
sudo chown oracle:dba datos
```

Y creamos la tablespace en la interfáz gráfica:

``` 
create tablespace RRHH datafile '/home/oracle/datos/RRHH.dbf' size 10M;
```

Y comprobamos tanto en el propio directorio como en Oracle:

``` 
select * from v$datafile;
```

## 15. Cómo crear un datafile

Lo hacemos con el siguiente comando:

``` 
alter tablespace RRHH add datafile '/home/oracle/datos/tl.dbf' size 50M;
```

Y compruebo:

``` 
select * from dba_data_files;
```


## 16. Cómo cambiar datafile de estado

Por ejemplo, queremos que nuestro tablespace pase a estar offline. Y dentro de offline podemos ponerlo en modo Normal, Temporary o Immediate, este último se utiliza cuando hay un error importante en nuestra base de datos y queremos cerrarla de inmediato y para ello hemos de cerrar nuestra tablespace.

```
alter tablespace "VENTAS" offline normal;
```

Y compruebo:

``` 
select * from dba_data_files;
```

Y para volver a ponerla en online:

``` 
alter tablespace "VENTAS" online;
```

O para poner la ts en modo solo lectura:

``` 
alter tablespace "VENTAS" read only;
```

## 17. Autoextender Datafile

Lo podemos hacer con el siguiente comando:

``` 
create tablespace TBS3 datafile '/home/oracle/datos/TBS3.dbf' size 1M autoextend on next 1M maxsize 250M;
```

## 18. Cambiar nombre y mover datafile

Para cambiar el nombre y mover el datafila:

``` 
alter database move datafile '/home/oracle/datos/t1.dbf' to '/home/oracle/datos/t1_prueba.dbf';
```

Incluso para moverlo a otro directorio nuevo:

``` 
alter database move datafile '/home/oracle/datos/t1.prueba.dbf' to '/home/oracle/datos/bk_datos/tbs4.dat';
```

Esto nos dará un error al no tener los permisos en esa carpeta. Lo resolvemos:

``` 
sudo chown oracle:dba bk_datos
```

## 19. Crear tablespaces temporales

Para consultar las propiedades de la base de datos en base a TEMP.

``` 
select * from database_properties where property_name like '%TEMP%';
```

Para ver los temp files de nuestra base de datos:

``` 
select * from dba_temp_files;
```

Para crear el tablespace temporal:

``` 
create temporary tablespace TEMP1 tempfile '/home/oracle/datos/temp1.dbf' size 100M;
```

Y, ¿Cómo hago que esta tablespace temporal sea la de defecto de nuestra base de datos? Con la siguiente instrucción:

``` 
alter database default temporary tablespace TEMP1;
```

Por último, para hacer que pueda autoextenderse:

``` 
alter database tempfile '/home/oracle/datos/temp1.dbf' autoextend on next 10M;
```

## 20. Cómo borrar un tablespace

``` 
drop tablespace tbs3 including contents;
```

Pero es más eficaz si eliminamos igualmente los datafiles asociados:

```
drop tablespace rrhh including contents and datafiles;
```

## 21. Archivos de control

Para ver donde se ubican:

``` 
select * from v$controlfile;
```

Aparecen dos, pues el archivo de control se duplica. Uno está activo y el otro es una copia.


## 22. Crear tablespace Undo

``` 
create undo tablespace UNDOTBS datafile '/home/oracle/datos/undotbs2.dbf' size 100M autoextend on next 10M maxsize 250M;
```

Comprobamos:

``` 
select * from dba_tablespaces where tablespace_name like '%UNDO%';
```

Y: 

``` 
show parameter undo
```

Sin embargo, nuestra tablespace no aparece como la principal en el sistema. Cambiamos eso:

``` 
alter system set undo_tablespace = undotbs2 scope=both;
```

Si quisiera cambiar el tiempo de retención de archivos en este tablespace:

``` 
alter system set undo_retention = 1800;
```

Se añade el tiempo, obviamente, en segundos.

Para garantizar esa retención:

```
alter tablespace undotbs2 retention guarantee;
```

## 23. Crear miembros grupos Redo Logs

Vemos los redo logs:

```
select * from v$log;
```

Hay tres, y siempre uno va a estar en activo mientras los demas están inactivos. Para ver donde están los ficheros:

``` 
select * from v$logfile;
```

Para realizar el cambio en anillo y que pase a estar activo el siguiente redo log archivo:

``` 
alter system switch logfile;
```

Para añadir un miembro a un grupo de redo log:

``` 
alter database add logfile member '/home/oracle/datos/grupo1-log2.log' to group 1;
```

Y comprobamos:

``` 
select * from v$log;
select * from v$logfile;
```

Mínimo debe haber dos miembros por cada grupo, y recomendable es tener tres.


## 24. Crear grupos de redo logs

``` 
alter database add logfile group 5 ('/home/oracle/datos/grupo5-log1.log','/home/oracle/datos/grupo5-log2.log') size 200M;
```

## 25. Borrar miembros en redo logs

En primer lugar, si quiero eliminar miembros del redo log activo, he de cambiar al siguiente redo log. 

``` 
alter system switch logfile;
```

Y ahora sí podemos eliminar este miembro:

```
alter database drop logfile member '/home/oracle/datos/grupo3-log2.log';
```

Comprobamos:

``` 
select * from v$log;
select * from v$logfile;
```

## 26. Fast Recovery Area

La fast recovery area es un componente crucial que permite recuperar datos en caso de fallos o errores en el sistema para su uso y mantenimiento.

 Aquí veremos cómo configurar correctamente los valores de la fast recovery area en tu base de datos Oracle. 

 Para verlo:

 ``` 
 show parameter recovery
 ```

 Vamos a la terminal y nos dirigimos al directorio de fast_recovery_area:

 ``` 
 cd u01/app/oracle/fast_recovery_area
 ```

 Volvemos al home con cd y accedemos a sqlplus insertando dicho comando. Accedemos con usuario y clave:

 ``` 
 sys/contraseña as sysdba
 ```

Ahora damos tamaño al archivo de recovery y especificamos su ámbito:

``` 
alter system set db_recovery_file_dest_size=50G scope=both sid='*';
```

Compruebo:

``` 
show parameter recovery
```

Y ahora establecemos nuestro database recovery file dest. Con esto decimos que los ficheros deben dejarse en la ruta que se indicará.

``` 
alter system set db_recovery_file_dest='/home/oracle/u01/app/oracle/fast_recovery_area' scope=both;
```

Para visionar como los datos se van logueando:

``` 
show parameter log_archive_des
```

Y para ver en que formato se van registrando los datos:

``` 
show parameter log_archive_format
```

Y ahora nos preguntamos, ¿Cómo sé si mi base de datos tiene activado mi sistema de archive log o no? Lo comprobamos:

``` 
select * from v$database;
```

En el campo log_mode nos dirá si archive log está activo o no.


## 27. Configurar ubicación Archive logs

``` 
alter system set log_archive_dest_1='location=/home/oracle/datos/archivelogs';
```

Veamos otro ejemplo:

``` 
alter system set log_archive_dest_2='location=USE_DB_RECOVERY_FILE_DEST';
```

Para ver en que formato quiero que se vean mis archive logs:

``` 
alter system set log_archive_format = 'arch_%t_%s_%r.arc' scope=spfile;
```

## 28. Activación de Archive log.

Accedemos a sqlplus e ingresamos con nuestro usuario sys:

``` 
sys/clave as sysdba
```

Apagamos la base de datos con shutdown immediate, e inicimos de nuevo con startup mount. Hacemos un listado de los archive log:

```
ARCHIVE LOG LIST
```

Modificamos la base de datos:

``` 
ALTER DATABASE ARCHIVELOG;
```

Y si vuelvo a hacer el listado de archive log nos dirá que está archivado. 

## 29. Sesiones de usuario

Para verlas:

``` 
select * from v$session;
```

O de un usuario en concreto:

``` 
select * from v$session where username='HR';
```

## 30. Creación de un usuario

Puede hacerse con el siguiente comando:

``` 
create user david identified by acceso1;
```

Para corregir un error habitual con este proceso alteramos la sesión de este modo:

```
alter session set "_ORACLE_SCRIPT" = true;
```

Para ver los usuarios con una lista detallada:

```
select * from all_users;
```

En caso de no poder crear sesión con este usuario le damos ese permiso:

``` 
grant connect to david;
grant resource to david;
```

Le he dado tambien permiso para poder crear objetos en la base de datos. Aún así, si yo creo, por ejemplo, una tabla, el sistema no me lo permitirá hacer hasta que no vuelva a salir y loguearme de nuevo para ahora sí tener los permisos de creación requeridos.

## 31. Asignar tablespace a usuario

Lo hacemos, por ejemplo, al crear el propio usuario:

``` 
create user usuario02 identified by acceso2
default tablespace + nombre de ese tablespace;
```

Incluso podemos añadirle un tablespace temporal:

``` 
create user usuario02 identified by acceso2
default tablespace + nombre de ese tablespace
temporary tablespace temp;
```

Le damos los permisos oportunos para poder conectarse y crear objetos en la base de datos:

``` 
grant connect to usuario02;
grant resource to usuario02;
```

## 32. Modificaciones en un usuario

Por ejemplo, para modificar la contraseña:

``` 
alter user david identified by + nueva contraseña;
```

Para cambiarlo de tablespace:

``` 
alter user david default tablespace + nombre de tablespace;
```

Veamos como podemos asignar quotas a los usuarios para limitar su espacio de uso en nuestra base.

``` 
alter user david quota 10m on + nombre de tablespace;
```

O con cuota ilimitada:

``` 
alter user david quota unlimited on + nombre de tablespace;
```

Para saber que espacio de cuota tiene el usuario en nuestra base de datos:

``` 
select * from dba_ts_quotas;
```

O:

``` 
select * from dba_ts_quotas where username = 'david';
```

## 33. Bloquear un usuario

```
alter user david account lock;
```

Y para desbloquear igual pero finalizando con **unlock**.

Si quiero que la contraseña de un usuario expire, lo hacemos del siguiente modo:

``` 
alter user david password expire;
```

Ese usuario, cuando vuelva a conectarse, habrá de insertar una nueva contraseña.

## 34. Borrar un usuario

```
drop user david;
```

Para eliminar un usuario junto con sus objetos:

``` 
drop user david cascade;
```

## 35. Asignar privilegios a un usuario

* Permisos de conexión

``` 
grant connect to david;
```

* Permisos de creación de tabla

``` 
grant create table to david;
```

* Permisos de creación de sinónimos
  
``` 
grant create synonym to david;
```

* Permiso para actuar sobre cualquier tabla

``` 
grant create any table to david;
```

## 36. Heredar y quitar privilegios a un usuario

En primer lugar, damos permiso de crear procedimiento:

```
grant create procedure to david with admin option;
```

Y ahora sí, este usuario puede conceder privilegios a otros:

``` 
grant create procedure to usuario02;
```

Para quitar privilegios:

```
revoke create procedure from david;
```

O:

``` 
revoke create table from david;
```

## 37. Privilegios de objetos

Si desde el usuario david quiero que usuario02 pueda hacer consultas y actualizar datos en una tabla:

```
grant select, update on productos to usuario02;
```

O para que pueda hacer cualquier cosa con mi tabla:

```
grant all on productos to usuario02;
```

O para una columna en concreto:

``` 
grant update (apellido) on alumnos to usuario02;
```

## 38. Vistas de privilegios

Para ver los privilegios de sistema:

``` 
select * from dba_sys_privs;
```

O en base a un usuario concreto:

```
select * from dba_sys_privs where grantee = 'david';
```

Para ver los privilegios que tenemos sobre las tablas de nuestra base de datos:

```
select * from dba_tab_privs;
```

O por un usuario en concreto:

``` 
select table_name, privilege, grantable, grantor
from dba_tab_privs where grantee='david';
```

Para ver los privilegios sobre columnas:

```
select * from dba_col_privs;
```

O por un usuario en concreto:

```
select table_name, privilege, grantable, grantor, column_name
from dba_col_privs where grantee='david';
```

Para ver los privilegios que el sistema asigna a los diferentes usuarios para ocasiones especiales:

``` 
select * from user_tab_privs_recd;
```

Y para ver los privilegios que los usuarios realizan dentro del sistema:

```
select * from user_tab_privs_made;
```














