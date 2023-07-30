# Taller de Configuración de un servicio SSH en Linux

## Caso práctico

En primer lugar, hemos de cerciorarnos de que hay conexión entre los equipos que van a tener este servicio SSH. Hacemos, por lo tanto, ping al equipo al cual queremos acceder vía SSH.

Y hacemos ese ssh:

```
ssh usuario@192.168.12.12
```

Dentro del directorio .ssh hay un archivo llamado known_hosts. En este archivo se encuentran las fingerprints que permiten el acceso ssh.

En este archivo se almacenan las credenciales de acceso de los equipos a los cuales ya hemos accedido. Si yo vuelvo a realizar la conexión ssh anterior, ya no me pedirá la contraseña pues está ya ha sido registrada en known_hosts.

**Acceso ssh mediante claves**

Para ello creamos esas claves:

``` 
ssh-keygen -t rsa
``` 

Las claves se almacenan dentro de .ssh en el archivo id_rsa.pub

Ahora necesitamos llevar esa clave pública a la máquina que deseamos acceder sin password.

``` 
ssh-copy-id usuario@ip (usuario e ip de la máquina a la cual queremos hacer el ssh)
``` 

Y si hacemos de nuevo la conexión ssh, no nos pedirá contraseña.




