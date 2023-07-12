# Curso Azure AZ-104 VI

## Casos prácticos

Creamos un grupo de recursos y tres redes virtuales con una subred cada una.

Creamos una máquina virtual para cada red, donde permitimos la conexión por RDP (3389).

**Vincular el Network Security Group con la otra red virtual**

Para ello, vamos al apartado NSG y en el que tenemos, dentro de su menu, nos dirigimos a Subredes. Ahi lo asociamos con la primera y la segunda red virtual. Y listo.

Sin embargo, aun no contamos con conectividad entre las máquinas de las dos primeras redes virtuales.

**Emparejamiento de redes virtuales**

Vamos a la red virtual 1 y en su menu, nos dirigimos a Emparejamientos. Agregamos, damos nombre al emparejamiento y también damos nombre al vínculo del emparejamiento, es decir, a como se verá desde la otra red virtual conectada. 

Vamos a Bloquear el tráfico que se origina fuera de esta red virtual, y agregamos el emparejamiento, que va a ser bidireccional. 

Acto seguido, emparejamos la red virtual 1 con la 3. Bloqueamos el tráfico como hicimos en el emparejamiento anterior y agregamos. 

**Problemas de conexión en Network Watcher**

Dentro de Network Watcher, en el apartado Solución de problemas de conexión, podemos ver el estado de la conectividad entre distintas redes virtuales y VM. 

Marcamos en origen la VM1 y en destino la ip de la VM2. La especificamos manualmente con su ip y lo probamos con el protocolo TCP y el puerto 3389 y activamos. La herramienta nos dirá que el estado es accesible. El emparejamiento ha funcionado.

Si lo hicieramos desde la VM2 a la VM3 no tendríamos acceso, puesto que no están emparejadas. 

**Cambiar rutas para que haya conectividad entre la red 2 y la 3. Rutas personalizadas**

Vamos a Interfaces de red, concretamente a la de la red 1 y en su menu a Configuraciones de IP y habilito el reenvio ip. Guardamos los cambios. Ahora esta VM puede reenviar paquetes. 

El siguiente paso será conectarnos a esa VM, descargando el archivo RDP. Queremos que esta VM funcione como un router, para ello lo hacemos desde Power Shell. En primer lugar, tenemos que instalarnos la característica de acceso remoto. 

```
Install-WindowsFeature RemoteAccess -IncludeManagementTools 
``` 

A continuación, instalamos el enroutamiento:

```
Install-WindowsFeature -Name Routing -IncludeManagementTools -IncludeAllSubFeature
``` 

Instalamos la característica de acceso remoto RSAT:

```
Install-WindowsFeature -Name "RSAT-RemoteAccess-PowerShell"
```

Especificamos que queremos que esta máquina solamente enrute:

```
Install-RemoteAccess -VpnType -RoutingOnly 
```

Y, por último, le decimos a nuestra interfaz de red que la característica que tiene de forwarding esté habilitada.

``` 
Get-NetAdapter | Set-NetIPInterface -Forwarding Enabled
```

Con todo esto, la VM1 tiene la capacidad de pasar cosas de una VM a otra, haciendo el rol de un router.

**Crear tablas de rutas**

En el portal, buscamos Tablas de rutas. Creamos, escogemos el grupo de recursos, región, damos nombre (Rutas2-3, pues serán las rutas que nos llevarán de la red 2 a la 3) y no propagamos rutas de puerta de enlace. Creamos. 

Dentro de la tabla de rutas, en el Menu vamos a rutas y agregamos una. Damos nombre, el prefijo de dirección ip que queremos alcanzar, por lo que ponemos la dirección de red de la 3.

En Tipo del próximo salto, marcamos dispositivo virtual y en Dirección del primer salto la ip de la VM1. Y aceptar.

Se nos está creando una ruta dentro de nuestra tabla de rutas.

Queda vincular la tabla de rutas con la subred correspondiente. Dentro de esta ruta, vamos a subredes y la asociamos con la subred de la red virtual 2.

Creamos otra ruta que llevará de la red 3 a la 2, pasando por la 1. Y la asociamos con la subred de la red virtual 3.

Y ahora tendremos conectividad entre todas las máquinas y redes de nuestra topología.








