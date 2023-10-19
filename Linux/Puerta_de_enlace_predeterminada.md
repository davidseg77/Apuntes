# Como cambiar y comprobar la puerta de enlace predeterminada

A continuación, se explica cómo puedes poner una puerta de enlace como predeterminada en Linux, o cambiar la puerta de enlace predeterminada siguiendo estos pasos.

Listamos la puerta de enlace actual: 

``` 
route -n
``` 

Y si quiero cambiar o asignar puerta de enlace en Linux:

``` 
sudo route add default gw "IP" "adaptador
``` 

Por ejemplo:

``` 
sudo route add default gw 192.168.12.1 enp0s3
``` 

Añadimos la ip deseada, la que pasará a ser la de la puerta de enlace, ponemos contraseña y volvemos a comprobar para asegurarnos al 100%

```
route -n
```


