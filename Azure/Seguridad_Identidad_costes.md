# Curso Azure AZ-900 II

### Seguridad general y de la red

* Azure Security Center
* Azure Sentinel
* Azure Key Vault

### Implementar Azure Key Vault

Accedemos al portal y en el buscador ponemos Almacenes de claves. Creamos un almacen de claves, añadiendo la suscripción y el grupo de recursos donde vamos a crear nuestro almacen de claves. Nos pedirá un nombre, región y plan de tarifa (estandar).

Revisamos y creamos, pasamos la validación de los parámetros puestos y creamos. Accedemos al recurso y será importante la uri del key vault. 

Aquí podemos crear nuestros secretos y certificados.

### Implementar tráfico de red seguro

Dentro de la configuración de una máquina virtual, indicamos en puertos de entrada públicos **ninguno**. En el apartado Redes, decimos en grupo de seguridad de red de NIC **ninguno**. En administración, deshabilitamos el diagnóstico de arranque y creamos. 

Hecho esto, buscamos grupo de seguridad de red en el buscador. Creamos, indicando la suscripción, el grupo de recursos al que queremos asociarlo y damos nombre. Revisamos y creamos. El siguiente paso será asociar este grupo de seguridad de red y la interfaz de red de la VM en cuestión. 

Creado el grupo de seguridad de red, en el menu de la izquierda vamos a Interfaces de red. Ahí podemos asociar con la interfaz de la VM. En el mismo menu, podemos asignar reglas de entrada y salida. Dentro de esa regla, podemos dar la prioridad y un nombre que defina lo que hace la regla. 

### Administrar bloqueos de recursos

Dentro de Azure Portal, creamos un grupo de recursos. Vamos a Grupos de recursos, crear, escogemos suscripción, damos el nombre al grupo de recursos y le decimos la región donde lo queremos crear. Y se crea.

En la parte de configuración, tenemos la opción de bloqueo. Si me voy al bloqueo y le añado un bloqueo de solo lectura, sobre este grupo de recursos queda bloqueado tanto a nivel de añadir o borrar dentro del grupo de recursos, como de borrar el propio grupo de recursos. 

Si quiero crear, por ejemplo, una red virtual dentro de este grupo de recursos, me será imposible. Viene bien para proteger los recursos en ciertos casos. 

Si añadimos un bloqueo de tipo eliminar, si podremos crear, por ejemplo, una red virtual, pero no podremos eliminar el grupo de recursos. De hecho, es recomendable crear este bloqueo por si acaso alguien lo borrará por accidente.

### Creación de una directiva 

En Portal Azure, buscamos Directiva. En el menu de la izquierda, en la sección Creaciones, vamos a Definiciones. En ese apartado aparece un listado con todas las directivas disponibles. 

En la sección creaciones, también tenemos Asignaciones, donde aparecen las directivas asignadas según su ámbito. Aquí en asignaciones, damos en asignar directiva. Escogeremos ámbito, en este caso la suscripción. También podemos excluir ciertos recursos de la directiva. 

La definición **ubicaciones permitidas** hace que los recursos se creen en estas ubicaciones.

Hecho esto, la directiva ha sido creada.

Si yo creo un recurso, por ejemplo una red virtual, y lo hago en una región no permitida por la directiva anterior, pero no se podrá. La directiva no lo permitirá. 

Esta directiva es muy usual en Azure, por que ayuda a la seguridad de los datos y recursos al estar agrupados en una única o pocas regiones.


