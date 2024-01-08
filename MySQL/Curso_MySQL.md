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















