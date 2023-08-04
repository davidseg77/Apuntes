# Información extraida del curso SSH de OW


## 1. Instalación del servicio SSH


``` 
apt install openssh-server
```

Se instalará y me generará las claves públicas y privadas del servicio.

Para comprobar el estado:

``` 
systemctl status ssh
```

Para ver en que proceso se está ejecutando:

``` 
ps aux | grep sshd
```

Para ver desde que puerto trabaja el servicio ssh:

``` 
ss -lntp
```

En el otro equipo, el del cliente, instalamos también el servicio si fuera necesario:

``` 
apt install openssh-client
```


## 2. Utilización de SSH

### 2.1 SSH Agent

Para crear una comunicación autenticada a través de un agente SSH hemos de hacer lo siguiente.
En primer lugar comprobamos mediante las variables de entorno donde se halla el agente de SSH.

``` 
env | grep ssh
```

Y veremos que hay un proceso para SSH agent. 

``` 
ps aux | grep ssh-agent
```

Trás ello, creamos un par de claves 

``` 
ssh-keygen -t rsa
```

Una vez creada con passphrase añado la clave para ssh al agent:

```
ssh-add ~/.ssh/id_rsa
```

Me pide la frase paso. Y una vez introducida compruebo que la clave se ha agregado a ssh-add

```
ssh-add -l
``` 

Y la copiamos en el cliente:

``` 
ssh-copy-id -i ~/.ssh/id_rsa cliente@ipcliente
``` 

Si accedemos por ssh al cliente nos pedirá la passphrase una vez, pero a la segunda ya no lo hará.


### 2.2 Forwarder

Se usa para dar un salto a otra máquina en remoto pasando por una intermedia y que la seguridad se mantenga. Es decir, voy a pasar a una tercera o terceras VM pasando por una intermedia, pero no quiero que mis claves estén circulando libremente por ahí y solo la tenga la máquina intermedia. 

Para ello vamos a reenviar la propiedad del agente SSH a una VM intermedia que va a permitir la conexión con otra máquina sin tener que pasar ese agente ssh. 

Nos vamos con usuario root desde el servidor al archivo /etc/ssh/ssh_config y ponemos ForwardAgent en Yes.

Ya fuera del usuario root, me conecto a la VM intermedia mediante ssh y si hago conexión ssh a la tercera podré efectuar dicha conexión a pesar de que no tengo en esta instalado el agente ssh porque la intermedia hace de enlace. 


## 3. Configuración SSH

### 3.1 Configuración SSH desde el cliente

Nos vamos al archivo /etc/ssh/ssh_config. En el servidor sería sshd_config.

* El caracter de escape para interrumpir una conexión ssh si se queda colgada es mediante la virgulilla:

```
~
```


## 4. Transferencia de archivos a través de SSH

### 4.1 Uso de SCP

Para pasar un archivo desde mi directorio actual hacia una máquina via ssh.

``` 
scp fichero.txt usuario@ipusuario:
```

¿Y al reves?

``` 
scp usuario@ipusuario:fichero.txt .
```

Para pasar una carpeta con varios archivos dentro a otra máquina en remoto:

``` 
scp -R directorio/ usuario@ipusuario:/ruta
```


## 5. Tuneles SSH

Se usan para, por ejemplo, poder enviar información relevante desde un servidor a una máquina remota o cliente a través de un canal (tunel) que consideramos fiable.

Para ello llevamos a cabo el metodo **local-forwarding**. En mi maquina local abro un puerto y establezco desde él un tunel remoto a un servidor ssh que tiene conectividad con el servicio que quiero entablar comunicación.

**Caso práctico**

Me conecto por ssh al cliente e instalo Apache. Vamos a su index.html y le cambio el título y alguna línea más a modo de prueba. 

Ahora vamos a abrir un tunel desde un segundo cliente para que pueda acceder a la web creada por el cliente 1. Desde el servidor hacemos uso del siguiente comando:

``` 
ssh -f -L 1080:ipdelcliente1:80 cliente2@ipdelcliente2 -N
```

Para comprobar que se ha establecido la conexión:

``` 
ps aux | grep ssh
```

Compruebo igualmente que el puerto asignado se ha abierto:

``` 
ss -lntp | grep 1080
```

Para eliminar el uso de ese puerto solo habría que matar el proceso:

```
kill + id del proceso
```

**Abrir un tunel desde el servidor para que lo gestione el cliente 1**

Para ello, volvemos a hacer uso del mismo comando solo que introducimos la función recursiva:

``` 
ssh -f -R 8080:localhost:80 cliente1@ipdelcliente1 -N
```

Para comprobar que se ha establecido la conexión:

``` 
ps aux | grep ssh
```

Compruebo igualmente que el puerto asignado se ha abierto, pero esto no podre verlo desde el servidor, desde local. Esto tendré que hacerlo desde el cliente uno:

``` 
ss -lntp | grep 8080
```



