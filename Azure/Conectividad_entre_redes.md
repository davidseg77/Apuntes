# Curso Azure AZ-104 V

## Casos prácticos

Creamos grupo de recursos y una red virtual donde le agrego dos subredes.

**Crear subredes**

Dentro de la creación de una red virtual, tengo la obligación de asignarle una subred. Sin embargo, cuando creo la red virtual y accedo a sus recursos, en el menu de la izquierda puedo acceder al apartado subredes y crear una segunda subred. 

Para comprobar el funcionamiento de nuestras redes y subredes, accedemos en el Portal a **Network Watcher**. De ahí pasamos a Topología, seleccionamos el grupo de recursos donde tenemos la red virtual y ahí vemos nuestro escenario. 

**Crear tarjeta de red para una subred**

En el Portal, buscamos el servicio Interfaces de red. Creamos una interfaz de red, donde añado la suscripción, el grupo de recursos, el nombre, la región y en que red virtual quiero crearla. Dentro de dicha red, indico en que subred quiero crearla. También digo si quiero que la asignación de la ip privada sea dinámica o estática. 

Ahora creamos una IP pública, y me voy a la interfaz de red. En su menu, vamos a configuración de ip y hacemos clic sobre la ip que nos aparece. Ahí asociamos la ip pública con la tarjeta de red. 

**Máquinas virtuales con redes y subredes**

Ahora creamos una máquina virtual, damos nombre y decimos que no queremos permitir ninguna entrada de tráfico. Lo asociamos a la red1 y subred1 creadas anteriormente y no le pongo ip pública. Y se crea.

Hacemos lo propio con una segunda máquina virtual para asociarla a la subred2. No obstante, el sistema nos creará las máquinas en la subred indicada, pero no asociadas a sus respectivas tarjetas de red. Automáticamente, se asocian a cada subred a través de una interfaz de red propia de la máquina. 

Si yo quiero asociarlas con las tarjetas de red de cada subred, hago lo siguiente:

1. Vamos a las máquinas y las paramos. 
2. Dentro de la máuina, vamos al apartado Redes y adjuntamos la tarjeta de red deseada, para después desasociar la que traía de serie. 

**Creación de un network security group**

Objeto a través del cual vamos a controlar el tráfico saliente y entrante de nuestras VM. 

Creamos en el Portal un grupo de seguridad de red. Indicamos el grupo de recursos, región... Y se crea. Hecho esto, en el menu de este grupo de seguridad, en Configuración puedo vincularlo a la interfaz de red o subred requerida.  Lo agregamos en este caso a las dos tarjetas de red de nuestro escenario. 

En Configuración, también podemos crear reglas de entrada y salida. Yo puedo acceder desde mis máquinas a los protocolos permitidos en base a las reglas permitidas, desde la función conectar. 


**Mover la VM2 a otra red virtual y hacer el emparejamiento para no perder la conectividad**

Creamos la red virtual con una subred. Ahora vamos a la tarjeta de red de la VM2 y en su menu, en Configuraciones de IP le desasocio la IP pública. Así dejamos de tenerla vinculada a esta tarjeta de red.

Vamos a Interfaces de red y creamos una tarjeta de red para la nueva red virtual. La asociamos con la red virtual y lasubred de esta. Asignamos una IP pública estática y creamos la interfaz de red. 

Dentro de esta tarjeta de red, vamos en su menu a Configuraciones de IP y ahí en la ip que nos da hacemos clic y asociamos la IP pública de la VM2 que acababamos de desvincular de la tarjeta de red secundaria. 

**¿Cómo mover la VM2 a la red virtual 2?**

Se llevará a cabo en dos pasos:

1. Eliminamos la VM2. Se elimina la máquina, pero no los recursos.
2. Creo una nueva máquina a partir del disco de la VM2. Vamos a grupo de recursos y ahí encontraremos al disco de la VM2. Si accedo a ese disco, desde el puedo crear una nueva máquina virtual que voy a llamar, de nuevo, VM2. Eso sí, la vinculamos a la red virtual nueva, le digo que no quiero ninguna IP pública y deshabilito el diagnóstico de arranque.

**Eliminar los recursos que han quedado sueltos**

Vamos a interfaces de red y eliminamos la tarjeta de red sobrante. Vamos a la red virtual 1 y en el apartado subredes elimino la subred que quedaba suelta después de quitar la VM2. 

Hecho esto, ya puedo irme a la nueva máquina virtual y adjuntar la interfaz de red nueva, desasociando la que venía por defecto al crear esta máquina. De hecho, lo siguiente será irnos a esta interfaz de red ya desasociada y la eliminamos. 

**Vincular el Network Security Group con la otra red virtual**

Para ello, vamos al apartado NSG y en el que tenemos, dentro de su menu, nos dirigimos a Subredes. Ahi lo asociamos con la segunda red virtual. Y listo.

Sin embargo, aun no contamos con conectividad entre las máquinas de las dos redes virtuales.

**Emparejamiento de redes virtuales**

Vamos a la red virtual 1 y en su menu, nos dirigimos a Emparejamientos. Agregamos, damos nombre al emparejamiento y también damos nombre al vínculo del emparejamiento, es decir, a como se verá desde la otra red virtual conectada. Probamos la conectividad y tras esto, para mejorar nuestro entramado solo quedará crear una zona DNS privada.

**Creación de una zona DNS privada**

En el Portal vamos a Zonas DNS privadas. Indicamos grupo de recursos, nombre DNS (por ej. david.es)...

Acto seguido, vinculamos nuestra zona DNS con las dos redes virtuales. Para ello, dentro del menu de la zona DNS, vamos a Vínculos de red virtual y ahi creamos dicho vínculo. 

Importante habilitar el registro automático.

Si yo reinicio mis VM, y voy a mi zona DNS, vemos como ya tenemos vinculadas nuestras VM dentro de los registros DNS. 

Si inicio la máquina 1 y me conecto por RDP, habiendo habilitado antes la regla de permiso de este protocolo, descargando el archivo RDP, el Network Security Group estará funcionando correctamente. 

Podemos probar dentro del cmd de esta máquina si el dns marcha según lo previsto haciendo un ping a la máquina 2. 

```  
ping vm2.david.es
```





