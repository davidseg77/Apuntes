# Información del curso Power BI nivel intermedio de OW

## 1. Introducción

### 1.1 Crear parámetros

Entramos en editar consultas, y vamos a Administrar parámetros y creamos uno. En valor actual añadimos la ruta de donde sacamos los datos.

Si hacemos clic derecho sobre el parámetro, podemos añadirlo a un grupo que llamaremos parámetros. Dentro del grupo podemos crear nuevos parámetros, dentro de los cuales podemos añadir rutas de otros apartados, como bases de datos.

Gracias a los parámetros podemos tener en un sitio la conexión, de manera que no tengamos que estar cambiándolo en todos partes. Es muy útil por ejemplo cuando hacemos una subida al entorno de producción, ya que cambiando la ruta del parámetro en un sitio se cambia la cadena de conexión de todos las tablas que tengan como conexión ese parámetro.

### 1.2 Crear los gráficos del campo importe

En primer lugar, ya tenemos nuestros datos importados, las tablas, pero no las tenemos relacionadas entre si. Para ello, nos vamos dentro del menu superior a la opción Administrar relaciones. Es muy importante tener nuestra tabla de calendarios, para ello vamos arriba a Modelado, Nueva tabla y añadimos la de calendario, poniendo por ejemplo lo siguiente en la barra:

``` 
Tabla = CALENDAR(DATE(2023;1;1);DATE(fecha de fin o to date))
``` 

Vamos a Administrar relaciones y accedemos a una de las tablas y la relacionamos con la deseada. Automáticamente nos indica la cardinalidad. Si después de establecer todas las relaciones vamos al icono del + y Organización automática, veremos de manera muy visual la relación entre nuestras tablas.

Para hacer nuestro primer gráfico vamos a Visualizaciones, escogemos el gráfico y añadimos los campos tanto en el eje como en los valores a mostrar. 
Acto seguido, creamos nuestro marcador. 

### 1.3 Crear marcadores

Una vez sabemos que información vamos a mostrar, la queremos mostrar lo más bonita posible, aquí son muy útiles los marcadores, imaginar poder tener a golpe de click los mismo gráficos vistos en moneda y en porcentaje en una misma página.
Además permite crear un botón de “borrar todos los filtros”.

Para crear el marcador, vamos a Vista y activamos Panel Marcadores y Panel Selección. 

En la pestaña Marcadores, los agregamos. Y en pestaña Formato de imagen, en Acción ponemos el tipo Marcador. 

### 1.4 Sincronizar segmentaciones

Al igual que en excel podemos añadir segmentadores de datos, estos nos permiten filtrar la información por alguna dimensión. Cuando por ejemplo añadimos el mismo segmentador de datos en varias páginas Power BI nos permite poder sincronizarlos, es decir, si aplicamos el filtro en una página, este mismo filtro se aplica al resto de páginas donde hayamos habilitado la sincronización.

En Vista, activamos Sincronizar segmentaciones y ahí añadimos donde sincronizar.

### 1.5 Información sobre la herramienta

Si pasamos el ratón por encima de un gráfico Power BI nos permite añadir información adicional que se muestra solo cuando dejamos el ratón encima del gráfico. Pero no solo no permite agregar nuevos valores, sino que podemos configurar una página entera donde mostrar la información que se desea.


## 2. Aplicar seguridad a un informe

### 2.1 RLS a nivel de dato

**Clase: RLS. Seguridad a nivel de dato, filtro manual**

Sin duda es una de las partes más importantes y delicadas dentro de la organización, hablar que datos puede ver cada usuario y/o departamento.
Power BI nos permite poder crear grupos de seguridad de forma que podemos restringir la información que se muestra en función a criterios de negocio.

**Clase: RLS. Seguridad a nivel de dato, filtro dinámico**

La idea es la misma que la anterior, la diferencia se basa en que podemos tener un solo fichero de configuración de forma que con cambiarlo en un sitio, se cambia en todos los sitios. Es muy útil cuando los trabajadores rotan de departamento a menudo.

Vamos a Modelado, Administrar roles. Ahí podemos crear un nuevo rol en base a lo que queramos, y le agregamos un tipo de filtro. Si vamos a Ver como roles, ya solo podremos ver la parte indicada en el filtro para nuestro rol. 

Sin embargo, la forma más segura es a través de grupos de seguridad.  


## 3. Métricas DAX

### 3.1 Introducción

El lenguaje DAX nos permite poder enriquecer nuestros informes con métricas que no son las básicas (sum, min,max, avg,..) de forma que junto a la métrica de ventas del mes, podemos ver el acumulado y las ventas del año anterior o la variación entre un año y el anterior.


## 4. Servicio Power BI

### 4.1 Gateway

GateWay o puerta de enlace es necesario tenerlo para poder automatizar el refresco de datos cuando tenemos origen de datos local. Sin él, la recarga automática no funciona.

**Instalación y configuración GateWay**

La instalación es muy sencilla de realizar, se debe seguir los pasos de la instalación de Microsoft. Se debe instalar en un servidor que esté siempre encendido. Si se instala en un pc, debe estar enchufado a la hora que se haga la recarga automática de datos.

Una vez instalado debemos crear un origen de datos por cada conexión del informe que tengo datos en local, ejemplo un excel en nuestro pc.

Dentro de la configuración de Power BI Service podemos establecer el Gateway de manera sencilla y programar las actualizaciones sin problemas.

### 4.2 Analizar en excel un informe

Dentro de los informes que tengamos en Power BI Service, tenemos debajo de cada informe la opción Analizar en Excel. Si clicamos, nos descargará un driver de conexión. También podemos hacerlo a través de la flecha superior de descarga, donde también tenemos la opción de Analizar en actualizaciones de Excel. 

Una vez descargado el archivo, en Excel vamos a Datos, Conexiones. Verificamos que la conexión se ha establecido con nuestro Power BI Service. 

### 4.3 Crear un panel

Dentro de una página en Power BI Service, en un objeto tenemos la opción Anclar visualización. Ahí podemos crear un nuevo panel. Ya lo tengo en el apartado Paneles, y si pincho en el panel puedo acceder también al informe u objeto del cual se ha creado dicho panel. 

### 4.4 Crear roles

Dentro de los conjuntos de datos, en el apartado Seguridad nos aparecen los roles ya creados en Desktop. Dentro de ellos puedo asignar a los usuarios en base a su email, asignandoles del mismo modo su función.


## 5. Dataflows

### 5.1 Introducción

Un data flow o flujo de datos, nos permite tener la ETL en una capa diferente a los gráficos, de forma que no penalize el rendimiento y la experiencia de usuario. Además tiene otras ventajas y es que mientras que un conjunto de datos es propio de informe, los flujos de datos se pueden compartir entre varios informes, permitiendo tener la lógica de la ETL en un solo sitio y ser reutilizable por más informes.

**ETL Data Flow**

Nos tenemos que quedar con la idea de que los data flows son iguales que los conjuntos de datos, salvando las distancias.
Son iguales en el sentido de que nos permiten importar datos de diferentes orígenes como cuando le damos a obtener conexión en Power BI Desktop. Una vez importado, podemos transformarlo análogo al desktop y por último cargarlo. Necesita GateWay igual si el origen de datos es en local.

Las diferencias entran en la parte más técnica de todas, y está más enfocado a personal de IT.









