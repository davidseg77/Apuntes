# Comandos más usados en la creación de MVs en la CLI

## 1. Descripción de las imágenes de máquina virtual

Para ver una lista de las imágenes disponibles, podemos usar el comando:

``` 
az vm image list --output table
``` 

el cual nos mostrará en forma de tabla, una lista con las imágenes más populares en Azure.

Podemos filtrar la búsqueda por los diferentes campos que se muestran, por ejemplo:

``` 
az vm image list --offer Debian --all --output table
``` 

nos muestra una lista offline que Azure ofrece de Debian.


## 2. Búsqueda de los tamaños de máquina virtual disponibles y cambiar el tamaño

Para ver una lista de los tamaños de máquinas virtuales disponibles en una región determinada, podemos usar el comando:

``` 
az vm list-sizes –location westeurope --output table
``` 

Podemos crear la MV directamente con un tamaño específico:

``` 
az vm create –resource-group recurso1 –name mv1 –image debian –size Standard_B1ls –adminusername
adminazure –generate-ssh-keys
``` 

Anteriormente, hemos creado el grupo de recursos, recuro1, para poder crear la MV.
Ahora, podemos cambiar el tamaño del disco en caliente, siempre y cuando el tamaño que queramos se encuentre en el clúster actual. Si no está disponible en nuestro clúster, habría que realizar un ‘deallocate’, posteriormente cambiar el tamaño correspondiente, y volver a arrancar la máquina.

Para ver los tamaños disponibles en nuestro clúster, usamos el siguiente comando:

``` 
az vm lis-vm-resize-options –resource-group recurso1 –name mv1 –query
[].name
``` 

Esto nos devolvería el nombre de los tamaños disponibles.

En nuestro caso, deseamos cambiar el tamaño de Standard_B1ls a Standard_B1ms, dado que los tipos B1 son los que entran dentro de los servicios gratuitos de Azure, además se encuentra entre las opciones de redimensión del clúster. Usamos el comando “az vm resize” para modificar el tamaño:

``` 
az vm resize --resource-group recurso1 --name mv1 --size Standard_B1ms
```

Una vez terminado el proceso, comprobamos el tamaño de la MV para ver si se ha realizado correctamente:

``` 
az vm show --resource-group recurso1 --name mv1 --query hardwareProfile.vmSize 
```

Si el tamaño deseado no se encuentra entre las opciones de redimensión, debemos desasociar nuestra máquina usando el comando “deallocate”, el cual apaga y desasocia la máquina. Luego podremos realizar el “resize” y cuando finalice el trabajo, debemos hacer un “start”.

``` 
az vm deallocate --resource-group recurso1 --name mv1
az vm resize --resource-group recurso1 --name mv1 –size “nuevo tamaño”
az vm start --resource-group recurso1 --name mv1
``` 


## 3. Visualizar la IP de las máquinas virtuales

No sabemos la IP de nuestras máquinas creadas actualmente. Pues con este sencillo comando podemos verla:

```
az vm list-ip-addresses --resource-group recurso1 --output table
``` 

Si queremos ver la IP de una máquina concreta añadimos la opción –name al comando:

```
az vm list-ip-addresses --resource-group recurso1 --name mv2 --output table
```

