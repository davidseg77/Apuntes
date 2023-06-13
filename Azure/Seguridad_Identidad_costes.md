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
