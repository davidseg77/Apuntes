# Curso de MySQL

## 1. Instalación de MySQL

Vamos a su web y dentro de downloads bajamos hasta MySQL Community Downloads. Ahí escogemos la opción MySQL Community Server. 

https://dev.mysql.com/downloads/mysql/

Y a partir de ahí todo es muy sencillo de realizar. Yo lo instalaré en modo Custom y escogere el MySQL Server y el Workbench como componentes a instalar. Damos contraseña a root y listo.

## 2. Crear base de datos

Con la siguiente instrucción:

``` 
create database administracion;
```

Para verlas:

``` 
show databases;
```

## 3. Creación de una tabla y mostrar sus campos

Para pasar a usar una base de datos desde terminal:

``` 
use administracion;
```

Para crear una tabla:

``` 
create table usuarios(
    nombre varchar(50),
    edad int(40)
);
```

Para mostrar las tablas:

``` 
show tables;
```

Para eliminar una tabla:

``` 
drop table usuarios;
```

Para describir una tabla:

``` 
describe usuarios;
```

O mostrar columnas de una tabla:

``` 
show columns from usuarios;
```

Para eliminar una tabla siempre y cuando exista:

``` 
drop table if exists usuarios;
```

## 4. Inserciones y consultas a una tabla

Para insertar datos dentro de una tabla:

``` 
insert into usuarios(nombre,edad) values('Frodo',18);
```

Para consultar esos datos:

``` 
select nombre,edad from usuarios;
```

Para crear una tabla con atributos numéricos.

``` 
create table libros(
    autor varchar(50),
    titulo varchar(30),
    precio float,
    cantidad integer
);
```

Y le insertamos valores:

``` 
insert into libros(autor,titulo,precio,cantidad) 
values ("David Segura","Murallas de fuego",12.5,200);
```

## 5. Recuperación de registros específicos

``` 
select titulo from libros
where autor = "David Segura";
```

## 6. Operadores relacionales

Veamoslos:

= igual
<> distinto
> mayor
< menor
>= mayor o igual
<= menor o igual

Veamos algún ejemplo:

``` 
select * from libros
where autor <> "David Segura";
```

O:

``` 
select * from libros
where precio < 12.5;
```

## 7. Eliminar registros de una tabla

Para eliminar todo de una tabla:

``` 
delete from libros;
```

O parcial:

``` 
delete from libros
where cantidad = 200;
```

## 8. Modificar registros de una tabla

Para actualizar registros:

``` 
update libros set autor= "David Segura Tristancho"
where autor="David Segura";
```

O:

``` 
update libros set autor = "David Segura Tristancho", cantidad = 300
where cantidad = 200;
```

Si quisiera poner un mismo valor dentro de un campo a todos:

```
update libros set cantidad = 300;
```

## 9. Clave primaria

Para incluirla dentro de la tabla:

``` 
create table usuarios(
    nombre varchar(30),
    id integer,
    primary key(id)
);
```

Para comprobar nuestra primary key:

```
describe usuarios;
```

Como es obvio, al insertar valores dentro de la tabla no podemos insertar dos id similares, pues la clave primaria es única para cada usuario. 

## 10. Campos autoincrement null, not null y truncate en una tabla

Autoincrement null nos va a asignar, por ejemplo, la id automáticamente sin tener que insertarla de manera ordenada. Not null se usa para definir a un valor que no puede quedar vacío. 

``` 
create table libros(
    codigo int auto_increment not null,
    titulo varchar(40),
    autor varchar(30),
    editorial varchar(15),
    precio decimal(5,2),
    primary key(codigo)
);
```

Y si yo inserto valores no he de poner el código, pues este se asignará por si mismo de manera incremental. 

El **truncate table** se usa para eliminar todos los valores de una tabla.

``` 
truncate libros;
```

## 11. Valores numéricos sin signo (unsigned)

Con unsigned decimos que el valor al que se asocia no puede ser inferior a cero, no puede ser un número negativo. 

Lo incluiríamos así:

``` 
create table libros(
    codigo int auto_increment unsigned not null,
    titulo varchar(40) not null,
    autor varchar(30) not null,
    editorial varchar(15),
    precio decimal(5,2) unsigned,
    primary key(codigo)
);
```

Yo, por ejemplo, no podría poner el precio de un libro de manera negativa. 

## 12. Operadores Lógicos (and - or - not - xor)

* **and** significa y
* **or** significa y/o
* **xor** significa o
* **not** significa no, invierte el resultado

Veamos ejemplos:

``` 
select * from libros
where cantidad <= 200 and cantidad >= 300;
```

Con or podríamos hacer esta misma consulta:

``` 
select * from libros
where cantidad <= 200 or cantidad >= 300;
```

Para una consulta con valores diferentes:

``` 
select * from libros
where not(precio >= 12);
```

Aquí nos mostrará los libros cuyo valor sea menor de 12. 

## 13. Cláusula order by del select

Para ordenar una consulta en base a un valor dado:

``` 
select * from alumnos order by edad;
```

Para ordenarlo de forma descendente:

``` 
select * from alumnos order by edad desc;
```

## 14. Búsqueda de patrones (like - not like)

``` 
select * from libros
    where nombre like '%David Segura%';
```

Para mostrar los valores que empiezan por una letra:

``` 
select * from libros
    where nombre like 'D%';
```

O terminan por una letra:

```
select * from libros
    where nombre like '%gura';
```

O por número de caracteres. Para ello dejamos esos espacios señalados con barra baja dentro del like:

``` 
select * from libros
    where nombre like '____________';
```

Y me aparecería David Segura al ejecutar esa consulta. 

O para buscar de forma diferente con not like:

``` 
select * from libros
    where nombre not like '%David Segura%';
```

## 15. Operadores relacionales (between - in)

Para busqueda que incluyan varios valores:

``` 
select * from libros
where nombre in('David', 'Santiago', 'Ken');
```

O entre varios valores:

``` 
select * from libros
where cantidad between 220 and 310;
```

O:

``` 
select * from libros
where nombre between 'David' and 'Santiago';
```

## 16. Búsqueda de patrones (regexp)

Son parecidos a like y not like, con algunas salvedades. 

Veamos ejemplos:

* Para buscar por un nombre que empiece por tal letra.

``` 
select * from libros
where autor regexp '^D';
```

* O que termine por tal letra:

``` 
select * from libros
where autor regexp 'a^';
```

* O filtrar por la posición de las letras:

``` 
select * from libros
where autor regexp 'D..i';
```

* O filtrar por número de letras:

``` 
select * from libros
where autor regexp '^.....$';
```

* O que tengan, por ejemplo, dos veces una letra:

``` 
select * from libros
where autor regexp 'a.*a';
```

## 17. Contar registros (count)

``` 
select count(*) from libros
where precio = 12.5;
``` 

## 18. Funciones de agrupamiento (count - max - min - sum - avg)

Para ver el valor máximo de una consulta:

``` 
select max(precio) from libros;
```

El mínimo se haría del mismo modo. 

Para realizar una suma de valores:

``` 
select sum(cantidad) from libros;
```

O una media de valores:

``` 
select avg(cantidad) from libros;
```

## 19. Agrupar registros (GROUP BY)

Veamos un ejemplo:

``` 
select titulo,editorial,count(*) from libros
group by editorial;
```

Para mostrar cantidad de compras agrupadas por fecha:

``` 
select fecha, count(*) from libros
group by fecha;
```

## 20. Selección de un grupo de registros (HAVING)

Con having hacemos un filtro dentro del group by. Having viene a significar teniendo en cuenta que. Veamos un ejemplo:

``` 
select fecha, count(*) from libros
group by fecha having fecha = '2006-31-05';
```

Con esta consulta decimos que nos muestre todos los libros agrupados por fecha, pero teniendo en cuenta que debe mostrarnos solo el agrupamiento de libros con fecha cinco de mayo de 2006. 

Así es como funciona having. 

## 21. Registros duplicados (distinct) 

Para mostrar los distintos valores de una consulta:

``` 
select distinct autor from libros;
```

## 22. Remplazar registros (replace)

Para reemplazar valores:

``` 
replace into alumnos(codigo,nombre) values (1,'Salvador');
```

## 23. Cláusula limit - Rand

Para limitar el número de registros a mostrar en una consulta. Por ejemplo, para mostrar los tres libros más vendidos en 2023:

``` 
select titulo, autor from libros
where año = 2023
order by precio
limit 3;
```

La funcion rand (de random) se usa para mostrar valores al azar. Por ejemplo, para mostrar los nombres y documentos de alumnos tomados al azar de la tabla alumnos:

``` 
select nombre,documento from alumnos
order by rand()
limit 3;
```

## 24. Índices

Son muy utiles cuando queremos analizar una tabla que cuenta con muchos registros.

Creamos una tabla:

``` 
create table medicamentos (
  codigo int unsigned auto_increment,
  nombre varchar(20) not null,
  laboratorio varchar(20),
  precio decimal (6,2) unsigned,
  cantidad int unsigned,
  primary key(codigo),
  index i_laboratorio (laboratorio) (Aquí marco el índice deseado, sino por defecto sería el nombre de la tabla)
);
```

Para mostrar los indices de una tabla:

``` 
show index from medicamentos;
```

O en nuestro caso:

``` 
show index from laboratorio;
```

Para eliminar un índice:

``` 
drop index i_laboratorio on medicamentos;
```

También podemos crear un índice bajo el parámetro unique. 

``` 
create table medicamentos (
  codigo int unsigned auto_increment,
  nombre varchar(20) not null,
  laboratorio varchar(20),
  precio decimal (6,2) unsigned,
  cantidad int unsigned,
  unique i_codigo (codigo),
  unique i_laboratorio (laboratorio)
);
```

## 25. Alterar tablas - (alter table - add - drop - modify - change)

Para alterar una tabla, añadiendo una nueva columna:

``` 
alter table libros add ventas unsigned;
```

Y comprobamos con describe. 

Otro ejemplo:

``` 
alter table libros add editorial varchar(30);
```

Para eliminar un campo existente:

``` 
alter table libros drop ventas;
```

Para modificar un campo:

``` 
alter table libros modify precio decimal(5,2);
```

**Nota**: No podemos modificar un primary key de este modo.

Para cambiar el nombre de un campo:

``` 
alter table libros change precio costo_libro decimal(5,2) not null;
```

## 26. Alterar índices - (alter table - add index - drop index)

Para agregar un índice a una tabla:

``` 
alter table libros add index i_precioventas(precio,ventas);
```

Y vemos el índice:

``` 
show index from libros;
```

Para agregar una clave primaria:

``` 
alter table libros add primary key (titulo);
```

Para eliminar esa clave primaria:

``` 
alter table libros drop primary key;
```

Para eliminar el índice.

``` 
alter table libros drop index i_precioventas;
```

## 27. Renombrar tablas (alter table - rename - rename table)

Para intercambiar el nombre de una tabla:

``` 
rename table libros to articulos;
```

Si hubiera una tabla libros y otra articulos, podríamos intercambiar sus nombres:

``` 
rename table libros to articulos, articulos to libros;
```

## 28. Encriptación de datos (aes_encrypt - aes_decrypt)

Creamos una tabla para los clientes:

``` 
create table clientes (
    nombre varchar(50),
    mail varchar(70),
    tarjetacredito blob, #blob es de tipo imagen, y se usa para tareas de encriptación.
    primary key (nombre)
);
```

E insertamos los valores:

``` 
insert into clientes
values('David Segura','david@gmail.com',aes_encrypt('2510','abcd'));
```

* aes_encrypt tiene dos parámetros (valor,clave).

Si hago una consulta a la table, este registro me aparecera en blob, encriptado. Para poder desencriptar ese valor:

``` 
select nombre,cast(aes_decrypt(tarjetacredito,'abcd') as char) from clientes;
```

## 29. Funciones de control de flujo (if) 

Veamos un ejemplo. Es política de la empresa festejar cada fin de mes, los cumpleaños de todos los empleados que cumplen ese mes. Si los empleados son de sexo femenino, se les regala un ramo de rosas, si son de sexo masculino, una corbata. La secretaria de la Gerencia necesita saber cuántos ramos de rosas y cuántas corbatas debe comprar para el mes de mayo:

``` 
 select sexo,count(sexo),
  if (sexo='f','rosas','corbata') as 'Obsequio'
  from empleados
  where month(fechanacimiento)=5
  group by sexo;
``` 

Además, si el empleado cumple 10,20,30,40... años de servicio, se le regala una placa recordatoria. La secretaria de Gerencia necesita saber la cantidad de años de servicio que cumplen los empleados que ingresaron en el mes de abril para encargar dichas placas:

``` 
 select nombre,fechaingreso,
  year(current_date)-year(fechaingreso) as 'Años de servicio',
  if ( (year(current_date)-year(fechaingreso)) %10=0,'Si','No') as 'Placa'
  from empleados
  where month(fechaingreso)=4;
``` 

Muestre todos los registros y un mensaje si las entradas para una función están agotadas:

``` 
 select sala,fecha,hora,
  if (capacidad=entradasvendidas,'sala llena',capacidad-entradasvendidas) as 'Entradas disponibles'
  from entradas;
``` 

## 30.  Funciones de control de flujo (case)

Veamos unos ejemplos prácticos de como se usa case.

Si el alumno tiene un promedio menor a 4, muestre un mensaje "reprobado", si el promedio es mayor o igual a 4 y menor a 7, muestre "regular", si el promedio es mayor o igual a 7, muestre "promocionado", usando la primer sintaxis de "case":

``` 
 select legajo,promedio,
  case truncate(promedio,0)
   when 0 then 'reprobado'
   when 1 then 'reprobado'
   when 2 then 'reprobado'
   when 3 then 'reprobado'
   when 4 then 'regular'
   when 5 then 'regular'
   when 6 then 'regular'
   when 7 then 'promocionado'
   when 8 then 'promocionado'
   when 9 then 'promocionado'
   else 'promocionado'
  end as 'estado'
 from alumnos;
```

Muestre el nombre y fecha de ingreso a la página y con un "case" muestre si el nombre del mes corresponde al 1º, 2º o 3º cuatrimestre del año.

``` 
 select nombre,fecha,
  case when (monthname(fecha) in ('January','February','March','April'))
   then '1º cuatrimestre'
  when (monthname(fecha) in ('May','June','July','August'))
   then '2º cuatrimestre'
  else '3º cuatrimestre'
  end as 'mes'
 from visitas;
``` 

## 31. Varias tablas (join)

Para realizar una consulta sobre dos tablas:

``` 
select * from libros join editoriales on libros.codigoeditorial=editoriales.codigo;
```

También puede hacerse mediante alias:

``` 
select * from libros as l join editoriales as e on l.codigoeditorial=e.codigo;
```

## 32. Varias tablas (left join)

Se usa para unir datos de una tabla con otra, en este caso los datos de la tabla izquierda para pasar aquellos datos que queramos a la tabla derecha en la consulta.

Queremos saber de qué provincias no tenemos clientes:

```
select p.codigo,p.nombre from provincias as p
left join clientes as c (Aquí unimos a la tabla clientes)
on c.codigoProvincia=p.codigo (Unimos sobre la clave foranea que vincula ambas tablas)
where c.codigoprovincia is null;
```

Con esta consulta, hemos pasado los datos de la tabla provincias a la de clientes, incluso aquellos datos de provincias que no aparecen en clientes. 

Veamos otro caso. Queremos saber de qué provincias sí tenemos clientes, sin repetir el nombre de la provincia:

``` 
 select distinct p.codigo,p.nombre from provincias as p
  left join clientes as c
  on c.codigoProvincia=p.codigo
  where c.codigoprovincia is not null;
``` 

## 33. Varias tablas (right join)

Se usa para unir datos de una tabla con otra, en este caso los datos de la tabla derecha para pasar aquellos datos que queramos a la tabla izquierda en la consulta.

Por ejemplo, para buscar los nombres de las editoriales que están presentes en libros. Podemos realizar la búsqueda de modo inverso con **right join**.

``` 
select * from libros as l
right join editoriales as e
on e.codigo=l.codigoeditorial;
```

## 34. Subconsultas

Utilizadas para reemplazar una expresión, no dejan de ser una consulta sobre otra consulta. Para ver un ejemplo.

Mostramos el título y precio del libro más costoso:

``` 
 select titulo,autor, precio
  from libros
  where precio=
   (select max(precio) from libros);
``` 

### 34.1 Subconsultas con in

En este apartado, vamos a ver aquellas subconsultas que retornan una lista de valores reemplazando a una expresión en una clausula where que contiene la palabra clave in. Veamos un par de ejemplos:

Necesitamos conocer los nombres de las ciudades de aquellos clientes cuyo domicilio es en calle "San Martin".

``` 
 select nombre
   from ciudades
   where codigo in
     (select codigociudad
       from clientes
       where domicilio like 'San Martin %'); 
``` 

Obtenga los nombre de las ciudades de los clientes cuyo apellido no comienza con una letra específica.

``` 
 select nombre
   from ciudades
   where codigo not in
    (select codigociudad
       from clientes
       where nombre like 'G%');   
``` 

### 34.2 Subconsultas any - some - all

Any y some son sinónimos. Se encargan de verificar si en alguna fila de la lista resultado de una subconsulta se encuentra el valor especificado en la condición.

Veamos unos ejemplos.

Muestre los socios que serán compañeros en tenis y también en natación.

``` 
 select nombre
  from socios
  join inscriptos as i
  on numero=numerosocio
  where deporte='natacion' and 
  numero= any
  (select numerosocio
    from inscriptos as i
    where deporte='tenis');  
``` 

Vea si el socio 1 se ha inscripto en algún deporte en el cual se haya inscripto el socio 2.

``` 
 select deporte
  from inscriptos as i
  where numerosocio=1 and
  deporte= any
   (select deporte
    from inscriptos as i
    where numerosocio=2);
``` 

Muestre los deportes en los cuales el socio 2 pagó más cuotas que TODOS los deportes en que se inscribió el socio 1.

``` 
 select deporte
  from inscriptos as i
  where numerosocio=2 and
  cuotas>all
   (select cuotas
    from inscriptos
    where numerosocio=1);
``` 

### 34.3 Subconsultas (Exists y No Exists)

Los operadores exists y no exists se emplean para determinar si hay o no datos en una lista de valores. 

Veamos algunos ejemplos.

Emplee una subconsulta con el operador "exists" para devolver la lista de socios que se inscribieron en 'natacion'.

``` 
 select nombre
  from socios as s
  where exists
   (select * from inscriptos as i
     where s.numero=i.numerosocio
     and i.deporte='natacion');
``` 

Busque los socios que NO se han inscripto en 'natacion' empleando "not exists".

``` 
 select nombre
  from socios as s
  where not exists
   (select * from inscriptos as i
     where s.numero=i.numerosocio
     and i.deporte='natacion');  
``` 

## 35. Vistas

Una vista es como una tabla virtual que almacena una consulta, para poder ser usada posteriormente las veces convenidas. 

Para crear una vista:

``` 
create view nombrevista as
select * from libros;
```

Usos muy aconsejables de vistas a la hora de administrar una base de datos:

* **Ocultar info**: Permitiendo el acceso a algunos datos y manteniendo oculto el resto de la info que no se incluye en la vista.

* **Simplificar la administración de los permisos de usuario**: Se pueden dar permisos al usuario para que solo pueda acceder a los datos a través de vistas, en lugar de concederle permisos para acceder a ciertos campos, así se protegen las tablas base de cambios en su estructura.

* **Mejorar el rendimiento**: Se puede evitar tipear instrucciones repetidamente almacenando en una vista el resultado de una consulta compleja que incluya info de varias tablas.

Para eliminar una vista:

``` 
drop view nombrevista
```

## 36. Procedimientos almacenados

Son conjuntos de instrucciones (comandos SQL) a los que se les da un nombre, que se almacenan en el servidor. Permiten encapsular tareas repetitivas. 

Por ejemplo:

``` 
delimiter //
create procedure libros_limite_stock()
begin
  select * from libros
  where stock<=10;
end //
delimiter ;
```

Para crear un procedimiento. Cree un procedimiento almacenado llamado "pa_empleados_sueldo" que seleccione los nombres, apellidos y sueldos de los empleados.

``` 
 delimiter //
 create procedure pa_empleados_sueldo()
 begin
   select nombre,apellido,sueldo
     from empleados;
 end //
 delimiter ;
``` 

Para ejecutar este procedimiento:

``` 
call pa_empleados_sueldo();
```

Para eliminarlo:

``` 
drop procedure pa_empleados_sueldo();
```

### 36.1 Procedimientos almacenados (parámetros de entrada)

Dentro del procedimiento creamos un parámetro de entrada que habremos de insertar en la consulta cuando el procedimiento sea llamado. Veamos ejemplo, aunque es sencillo de entender:

``` 
 delimiter //
 create procedure pa_libros_autor(in p_autor varchar(30))
 begin
   select titulo, editorial,precio
     from libros
     where autor= p_autor;
 end //
 delimiter ;
```

Cuando lo ejecute, se haría de este modo:

``` 
 
 call pa_libros_autor('David Segura');
```

Y nos mostrará el título, editorial y precio del libro de David Segura. 

Igualmente, podemos insertar más de un parámetro de entrada.

``` 
 delimiter //
 create procedure pa_libros_autor_editorial(
   in p_autor varchar(30),
   in p_editorial varchar(20))
 begin
   select titulo, precio
     from libros
     where autor= p_autor and
           editorial=p_editorial;
 end //
 delimiter ;
```

Y lo llamaría del siguiente modo:

``` 
 call pa_libros_autor_editorial('Richard Bach','Planeta');
```

### 36.2 Procedimientos almacenados (parámetros de salida)

Pueden insertarse dentro de un procedimiento combinándolos junto con los parámetros de salida.

Cree un procedimiento almacenado llamado "pa_seccion" al cual le enviamos el nombre de una sección y que nos retorne el promedio de sueldos de todos los empleados de esa sección y el valor mayor de sueldo (de esa sección)

``` 
delimiter //
 create procedure pa_seccion(
   in p_seccion varchar(20),
   out promedio float,
   out mayor float)
 begin
   select avg(sueldo) into promedio
     from empleados
     where seccion=p_seccion;
   select max(sueldo) into mayor
   from empleados
    where seccion=p_seccion; 
  end //  
 delimiter ;  
``` 

Ejecute el procedimiento creado anteriormente con distintos valores.

``` 
 call pa_seccion('Contaduria', @promedio, @mayor);
 select @promedio,@mayor;
``` 
 

































