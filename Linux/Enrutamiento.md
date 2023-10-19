# Como crear un Router usando Ubuntu Server

## 1. Diagrama

* Ubuntu Server (Que hará de router y se conectará por adaptador puente al router físico.)
* PC (Máquina virtual que se conectará al servidor por red interna)

## 2. Requerimientos mínimos

Para el servidor.

- CPU:   1 gigahertz o más
- RAM:   1 gigabyte o más
- Disco: un mínimo de 2.5 gigabytes

## 3. Configurar interfaces de red

En el servidor, vamos al archivo yaml del netplan.

``` 
sudo nano /etc/netplan/00-installer-config.yaml
``` 

Y ponemos lo siguiente:

``` 
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses: [192.168.222.1/24]
      #gateway4: 192.168.222.1 
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```


## 4. Revisar las rutas de enrutamiento

Asignamos los cambios, con sudo netplan apply. Comprobamos los adaptadores con ip ad y la ruta con ip route.

La ruta por defecto tiene que ser enp0s3


## 5. Si queremos ver los caracteres de una manera más estética y eficiente (opcional)


• Instalar el paquete SSH:

``` 
sudo apt-get install ssh
``` 

•  Abrir cliente SSH (Putty) y conectar 

 https://www.putty.org/ 

 Abrimos putty y ponemos la ip del servidor con el puerto si fuera necesario (22). Establecemos conexión SSH.

 Con clic derecho sobre el panel de Putty, en Settings podemos ajustar el tamaño de los caracteres para hacer el texto más grande.


 ## 6. Habilitar el reenvio de paquetes entre interfaces

En Linux, la configuración por defecto impide que los paquetes, la info, pase de una interfaz a otra. Por ende, hay que habilitar la opción que sí lo permite. 

Vamos a este archivo:

```
cat /proc/sys/net/ipv4/ip_forward
```

Este archivo solo contiene un bit que permite el paso de la info. Esta en 0, y queremos que esté en 1. Para ello, tenemos que configurarlo para que pase a 1 y se quede así cada vez que se inicie el equipo. 

Pues vamos a este archivo:

``` 
nano /etc/sysctl.conf
``` 

Y descomentamos la siguiente línea

```
net.ipv4.ip_forward=1
```

Aplicamos los cambios con el comando 

``` 
sudo sysctl -p /etc/sysctl.conf
``` 

Y compruebo

``` 
cat /proc/sys/net/ipv4/ip_forward
```

Ya tendríamos habilitado el reenvio de forma permanente, lo cual es un punto clave dentro de este caso de enrutamiento.


## 7. Configurar Source NAT (SNAT) en IpTables

Revisamos que la política esté por defecto 

``` 
iptables -L
``` 

Si esta a DROP modificar a ACCEPT
 
``` 
iptables -P FORWARD ACCEPT
``` 

Revisamos que la tabla nat esta vacía

``` 
iptables -L -nv -t nat
``` 

Activamos SNAT con emmascaramiento IP (Masquerade) poniendo el interfaz enp0s3(DHCP) para los paquetes que van hacia fuera.

``` 
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
``` 

Y revisamos que la tabla nat esta configurada

``` 
iptables -L -nv -t nat
``` 

Si hubiera algún error al introducir estas reglas, las limpiamos:

``` 
iptables -F
``` 

O bien

``` 
iptables -t nat -F
``` 

## 8. Guardar la configuración IpTables de manera permanente

Instalamos el paquete encargado de esta persistencia:

``` 
apt-get install iptables-persistent
``` 

Y nos preguntará automáticamente si queremos guardar las reglas que ya tenemos. Si editamos esas reglas, podemos guardar esas nuevas reglas haciendo uso del siguiente comando:

``` 
netfilter-persistent save
``` 

Y listo.

