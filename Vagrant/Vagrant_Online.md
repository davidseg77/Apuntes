# Información extraída del curso Vagrant Online de OW

## 1. Introducción

### 1.1 Lanzando nuestra primera máquina

Creamos un directorio y accedemos a él. Será desde este directorio desde donde lanzaremos nuestra primera máquina Vagrant.

``` 
vagrant init + nombre de la imagen
```

Y la levantamos

``` 
vagrant up
```

Trás levantarla, accedemos a ella:

``` 
vagrant ssh
```

### 1.2 Lanzando una máquina configurada

Añado una imagen del tipo de máquina que deseo. Por ejemplo:

``` 
vagrant box add rasmus/php7dev
``` 

Compruebo los boxes, imágenes, que tengo:

``` 
vagrant box list
```

Puedo hacer un git clone del repositorio de esta imagen en el directorio desde el cual voy a lanzar la máquina. A posteriori, lanzo la máquina.


## 2. Uso de Vagrant

### 2.1 Reempaquetar un box

Dentro del directorio donde tengo las boxes (~/.vagrant.d/boxes) existe un directorio por cada box. Puedo acceder a cada uno de ellos y ver su contenido, para por ejemplo copiarme la imagen y reempaquetarla.

``` 
vagrant box repackage centos/7 virtualbox 1703.01
```

### 2.2 Eliminar o añadir una imagen de box

Para eliminar una box:

``` 
vagrant box remove centos/7
```

Para añadir una box:

``` 
vagrant box add package.box --name centos/7
```

### 2.3 Actualización de imágenes

Para comprobar si alguna imagen esta desactualizada:

``` 
vagrant box outdated --global
```

Si tenemos dos imágenes de Ubuntu, por ejemplo, podemos eliminar la más antigua en cuanto a su versión con el siguiente comando:

``` 
vagrant box prune
```

Para actualizar:

``` 
vagrant box update
```

### 2.4 Aprovisionamiento ligero

Se puede configurar del siguiente modo:

``` 
Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "ligero"
    vb.memory = "512"
    vb.cpus = 1
    vb.linked_clone = true (es como VirtualBox especifica el aprovisionamiento ligero)
  end
end
```

### 2.5 Añadir disco a una VM

Se puede configurar del siguiente modo:

``` 
Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "ligero"
    vb.memory = "512"
    vb.cpus = 1
    vb.linked_clone = true (es como VirtualBox especifica el aprovisionamiento ligero)
    file_to_disk = 'tmp/disk.vdi'
    unless File.exist?(file_to_disk)
      vb.customize ['createhd',
                    '--filename', file_to_disk,
                    '--size', 500 * 1024]
    end
    vb.customize ['storageattach', :id,
                  '--storagectl', 'SATA Controller',
                  '--port', 1,
                  '--device', 0,
                  '--type', 'hdd',
                  '--medium', file_to_disk]                 
  end
end
```

### 2.6 Comandos de Vagrant

Después de suspender una VM, podemos volver a ponerla en funcionamiento con:

``` 
vagrant resume
```


## 3. Configuraciones avanzadas

### 3.1 Red pública

Se puede configurar del siguiente modo:

``` 
Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
  config.vm.network "public_network", bridge "wlan0"
end
```

### 3.2 Entorno multimáquinas

Se puede configurar del siguiente modo. Por ejemplo, para un servidor web con una base de datos:

``` 
Vagrant.configure("2") do |config|
  config.vm.define = "web" do |nodo1|
    nodo1.vm.box = "ubuntu/trusty64"
    nodo1.vm.hostname = "web"
    nodo1.vm.network = "public_network", bridge: "wlan0"
    nodo1.vm.network = "private_network", ip: "10.0.100.101"
  end
  config.vm.define = "db" do |nodo2|
    nodo2.vm.box = "ubuntu/trusty64"
    nodo2.vm.hostname = "db"
    nodo1.vm.network = "private_network", ip: "10.0.100.102"
  end
end
```

### 3.3 Vagrant para usar AWS como proveedor

Dentro del directorio Vagrant creo otro que voy a llamar Prueba-aws.

Vemos los plugins que tenemos instalados:

``` 
vagrant plugin list
```

Instalamos el plugin que nos va a permitir aprovisionar AWS.

``` 
vagrant install plugin vagrant-aws
```

Dentro de Prueba-aws me voy a .aws y si hago un **tree** podré ver que hay tengo la configuración y las credenciales. 

Y creamos un Vagrantfile:

``` 
vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  config.ssh.keys_only = false (Con esto digo que voy a usar mi clave ssh, la que tengo en mi equipo)
  config.vm.provider :aws do |aws, override|
    aws.keypair_name = "clave-openstack" (Nombre de la clave ssh subida a AWS)
    aws.ami = "ami-3291be54" (AMI de una Debian/Jessie)
    override.ssh.username = "admin"
  end
end
```

Ahora, agregamos la clave ssh nuestra:

``` 
ssh-add ~/.ssh/clave-openstack.pem
```

La levantamos y comprobamos el proceso en AWS. Y nos conectamos mediante ssh:

``` 
ssh admin@ipAWS
```

Si yo creo un archivo desde el directorio Prueba-aws y en la VM creada no me aparece aún, efectuamos la sincronización mediante:

``` 
vagrant rsync
```

### 3.4 Almacenamiento persistente

Instalamos el plugin para ello:

``` 
vagrant plugin install vagrant-persistent-storage
```

Y dentro del Vagrantfile creamos la siguiente configuración:

```  
vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.persistent_storage.enabled = true
  config.persistent_storage.location = "tmp/disk.vdi"
  config.persistent_storage.size = 500
  config.persistent_storage.mountname = 'mysql'
  config.persistent_storage.filesystem = 'ext4'
  config.persistent_storage.mountpoint = '/var/lib/mysql'
  config.persistent_storage.volgroupname = 'myvolgroup'
end
```






