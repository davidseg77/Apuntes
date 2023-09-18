# Info extraida del curso Vagrant para desarrolladores


## 1. Instalación

Para asegurarnos de que está instalada después de dar los pasos indicados en la web de descarga de Vagrant, solo hemos de poner lo siguiente:

``` 
vagrant
```


## 2. Flujo básico de trabajo con Vagrant

* Creamos la piedra angular de Vagrant, mediante el fichero Vagrantfile

``` 
vagrant init ubuntu/xenial64
```

* Levantamos la máquina

``` 
vagrant up
```

* Accedemos a la máquina

``` 
vagrant ssh
```

* Apagamos la máquina

``` 
vagrant halt
```

* O la suspendemos

``` 
vagrant suspend
```


### 2.1 Boxes

Las boxes o imagen de SO pueden crearse para distintos providers (Virtualbox, VMware, Parallels...)

* Añadimos boxes con:

``` 
vagrant box add ubuntu/xenial64
```

O

``` 
vagrant box add http://url.a.la.box.com
```

* Listamos boxes con:

``` 
vagrant list
```

* Eliminamos boxes con:

``` 
vagrant box remove ubuntu/xenial64
```

* Actualizamos el entorno vagrant con:

``` 
vagrant box update
```

Cuando actualizamos una box a su versión más reciente, después podemos usarla mediante el siguiente comando:

``` 
vagrant reload
```


### 2.2 Comunicándonos con la máquina virtual por SSH

* Para entrar en la máquina se requiere un cliente ssh. Trás ello, podemos acceder de las siguientes maneras:

``` 
vagrant ssh
```

O 

``` 
vagrant ssh -c `ls -la´ (pasando comandos)
```

O

``` 
vagrant ssh -p (Se deja la autenticación al usuario)
```

La clave privada que se usa está en:

``` 
.vagrant/machines/default/virtualbox/private_key
```


### 2.3 Comunicándonos con la máquina virtual por directorio sincronizado

Se usa para trabajar desde el host influyendo sobre la VM. Para ello, vamos al Vagrantfile y dentro del archivo a la siguiente línea:

``` 
config.vm.synced_folder ".", "/vagrant"
```

Indicamos el directorio requerido y si levanto la VM y hago cualquier cambio en mi directorio en host, este cambio se producirá de inmediato en la VM.

Hay otros tipos de sincronización, como pueden ser el de rsync (unidirecional), NFS o SMB (bidireccionales). Para estos dos últimos se requiere un servidor NFS o SMB en el host y una red privada. 

``` 
config.vm.synced_folder ".", "/vagrant", type: "rsync"
```

``` 
config.vm.synced_folder ".", "/vagrant", type: "nfs"
```


### 2.4 Comunicándonos con la máquina virtual: Acceso por red

* Forwarded ports

``` 
config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
```

* Redes públicas

``` 
config.vm.network "public_network"
```

* Redes privadas

``` 
config.vm.network "private_network", ip: "192.168.33.10"
```

## 3. Provisión automática de la infraestructura

La provisión se configura con 

``` 
vm.config.provision
```

La VM ejecuta la provisión al hacer **vagrant up**, aunque también puede hacerse mediante:

``` 
vagrant provision
```

O

``` 
vagrant up --provision
```

**Provisión con shell**

Es la manera más sencilla y es adecuada para provisionamientos sencillos. Ejemplo:

``` 
config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx
    sed -i -e 's/\\/var\\/www\\/html/\\/vagrant/g /etc/nginx/sites-enabled/default
    systemctl restart nginx
SHELL
```

Si lo que deseamos es subir ficheros de la host a la guest:

``` 
config.vm.provision "file", source:
"/path/to/source", destination: "."
```

### 3.1 Despliegue automático de infraestructura LAMP con Shell Scripts

Para ello dentro del Vagrantfile añadimos lo siguiente:

``` 
Vagrant.configure("2") do |config|
   config.vm.box = "ubuntu/xenial64"
   config.vm.network "private_network", ip: "ip"
   config.vm.synced_folder ".", "/var/www/blog", type: "nfs"
   config.vm.provision "shell", path: "provision.sh"
end  
```

### 3.2 Despliegue automático de infraestructura LAMP con Ansible

Para ello dentro del Vagrantfile añadimos lo siguiente:

``` 
Vagrant.configure("2") do |config|
   config.vm.box = "ubuntu/xenial64"
   config.vm.network "private_network", ip: "ip"
   config.vm.synced_folder ".", "/var/www/blog", type: "nfs"
   config.vm.provision "ansible" do |ansible|
     ansible.playbook = "playbook.yml"
   end
end
``` 


## 4. Networking con Vagrant

### 4.1 Port forwarding

Util para VM con pocos servicios, con pocos puertos que redireccionar.

``` 
config.vm.network "forwarded_port", guest: 80, host: 8080
```

**Red privada**

Solo se puede acceder desde la máquina host. Para ello se asigna una IP privada a la máquina guest, bien mediante DHCP o bien de manera estática.

* DHCP

``` 
config.vm.network "private_network", type: "dhcp"
```

* Estática

``` 
config.vm.network "private_network", ip: "ip"
```

Es útil cuando queremos montar clusters de VM que se comunican entre sí.

**Red pública**

La VM, de cara a la red, es otro equipo más. Se puede obtener IP por DHCP o estática.

* DHCP

``` 
config.vm.network "public_network"
```

* Estática

``` 
config.vm.network "public_network", ip: "ip"
```

