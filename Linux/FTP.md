# Como hacer un servidor FTP


## 1. Instalación y configuración en el servidor

En primer lugar, para poder crear nuestro servidor en Linux hacemos uso del siguiente comando:

``` 
sudo apt install vsftpd
```

A continuación, habrá que ir al archivo de configuración. 

``` 
sudo nano /etc/vsftpd.conf
```

Descomentamos en dicho archivo la línea siguiente:

``` 
write_enable=YES
```

Con esto especificamos que los usuarios puedan escribir o modificar los archivos que estén dentro de nuestro servidor FTP.

Y hecho este cambio, reinicio el servicio:

``` 
sudo service vsftpd restart
```

Y comprobamos su estado.

``` 
sudo service vsftpd status
```

Comprobamos nuestra ip. Puede ser que tengamos que instalar las net tools. Si fuera necesario, lo hacemos con el siguiente comando.

``` 
sudo apt install net-tools
```

## 2. Filezilla para el servidor y el cliente

Es una forma más recomendable. Lo descargamos facilmente desde su página web. Desde el cliente instalamos el Filezilla para cliente y una vez dentro, solo tenemos que especificar la IP del servidor, usuario y contraseña. El puerto sería el 21 (el propio de FTP), y clicamos en Conexión rápida. 

En ocasiones, puede haber problemas por culpa del Firewall. Habría que crear reglas de entrada para permitir la conexión FTP. Para ello, vamos al Firewall, reglas de entrada, crear. Y creamos una en base a un puerto, en este caso al puerto 21. 

Ahora ya podemos acceder, pero puede que no a las carpetas y archivos del servidor. Para ello, habrá de crearse otra regla en el Firewall. Antes hay que ir al Filezilla del servidor, en Edit, Settings, Passive mode Settings y activamos la casilla Use custom port range. Asignamos ese rango de puertos o uno en cuestión (2000).

Y ahora sí, vamos al Firewall y creamos la regla de entrada para el puerto 2000 o el rango asignado. Y ya si podemos realizar la transferencia de archivos entre servidor y cliente y viceversa.



