# LINUX 🖥️ SAMBA 🔒 ACTIVE DIRECTORY DOMAIN CONTROLLER

En primer lugar, configuramos una red interna en VirtualBox con máquinas Ubuntu Desktop, Ubuntu Server y Windows 10. De esta manera, configuraremos las direcciones IP de los equipos conectados en la red LAN - LOCAL AREA NETWORK - para que tengan conectividad entre ellos.

A continuación, dejo un enlace donde puede verse como crearse esta red interna de manera sencilla:

<https://www.youtube.com/watch?v=XhDeZOIMw48&list=PLHjuPxrwcdsb8tf7MQIO7gYNNcJziMDrb&index=1>

Y la red interna quedaría estructurada del siguiente modo:

![Red](https://github.com/davidseg77/Apuntes/blob/main/Linux/Active_Directory/Red.jpg)

## 1. DOMAIN CONTROLLER EN UBUNTU SERVER + UNION WINDOWS 10


En esta práctica aprehenderás a configurar un servidor Ubuntu como un Controlador de Dominio Active Directory utilizando **Samba 4**. Comenzaremos instalando y configurando Samba 4 en nuestro servidor Ubuntu, convirtiéndolo en un AD DC totalmente funcional. 

Luego, aprenderemos cómo instalar y configurar **NTPDATE** para sincronizar el tiempo en el dominio, garantizando una gestión eficaz del tiempo en todos los equipos del entorno. Además, demostraremos cómo unir un equipo Windows 10 al dominio implementado en el AD DC de Samba, permitiendo una autenticación centralizada y un control de acceso más granular en la red.

### 1.1 CONFIGURACIÓN SERVIDOR

Cambiar hostname:

``` 
sudo hostnamectl set-hostname dc
``` 

Modificar fichero hosts:

``` 
sudo nano /etc/hos
192.168.1.8 dc.clockwork.local dc
``` 

Verificar el FQDN. Un FQDN es un nombre de dominio completo que incluye el nombre de la computadora y el nombre de dominio asociado a ese equipo.​​ Por ejemplo, dada la computadora llamada «serv1» y el nombre de dominio «bar.com.», el FQDN será «serv1.bar.com.»:

``` 
hostname -f
``` 

Verificar si el FQDN es capaz de resolver la dirección Ip del Samba:

``` 
ping -c2 dc.clockwork.local
``` 

Desactivar servicio systemd-resolved:

``` 
sudo systemctl disable --now systemd-resolved
```

Eliminar enlace simbólico al archivo /etc/resolv.conf:

``` 
sudo unlink /etc/resolv.conf
``` 

Creamos de nuevo el archivo /etc/resolv.conf:

``` 
sudo nano /etc/resolv.conf
```

Añadimos las siguientes líneas:

``` 
nameserver 192.168.1.8
nameserver 8.8.8.8
search clockwork.local
``` 

Hacemos inmutable al archivo /etc/resolv.conf para que no pueda cambiar:

``` 
sudo chattr +i /etc/resolv.conf
``` 

### 1.2 INSTALACIÓN SAMBA

Actualizar el índice de paquetes:

``` 
sudo apt update
``` 

Instalar samba con sus paquetes y dependencias:

``` 
sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools
``` 

CLOCKWORK.LOCAL
dc.clockwork.local
dc.clockwork.local

Detener y deshabilitar los servicios que el servidor de Active Directory de Samba no requiere (smbd, nmbd y winbind)

``` 
sudo systemctl disable --now smbd nmbd winbind
``` 

El servidor solo necesita samba-ac-dc para funcionar como Active Directory y controlador de dominio:

``` 
sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc
```

### 1.3 CONFIGURACIÓN SAMBA ACTIVE DIRECTORY

Crear una copia de seguridad del archivo /etc/samba/smb.conf:

``` 
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.orig
``` 

Ejecutar el comando samba-tool para comenzar a aprovisionar Samba Active Directory:

``` 
sudo samba-tool domain provision

Realm: CLOCKWORK.LOCAL
Domain: CLOCKWORK
Server Role: dc
DNS backend: SAMBA_INTERNAL
DNS forwarder IP address: 8.8.8.8
``` 

Crear copia de seguridad de la configuración predeterminada de Kerberos:

``` 
sudo mv /etc/krb5.conf /etc/krb5.conf.orig
```

Reemplazar con el archivo /var/lib/samba/private/krb5.conf:

``` 
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

Iniciar servicio Samba Active Directory samba-ad-dc:

``` 
sudo systemctl start samba-ad-dc
```

Comprobar servicio:

``` 
sudo systemctl status samba-ad-dc
```

### 1.4 CONFIGURAR SINCRONIZACIÓN DE TIEMPO

**Samba Active Directory** depende del protocolo **Kerberos**, y el protocolo Kerberos requiere que los tiempos del servidor AD y de la estación de trabajo estén sincronizados. Para garantizar una sincronización de tiempo adecuada, también deberemos configurar un servidor de Protocolo de tiempo de red (NTP) en Samba.
Los beneficios de la sincronización de tiempo de AD incluyen la prevención de ataques de repetición y la resolución de conflictos de replicación de AD.

Cambiar el permiso y la propiedad predeterminados del directorio /var/lib/samba/ntp_signd/ntp_signed. El usuario/grupo chrony debe tener permiso de lectura en el directorio ntp_signed.

``` 
sudo chown root:_chrony /var/lib/samba/ntp_signd/
sudo chmod 750 /var/lib/samba/ntp_signd/
``` 

Modificar el archivo de configuración /etc/chrony/chrony.conf para habilitar el servidor NTP de chrony y apuntar a la ubicación del socket NTP a /var/lib/samba/ntp_signd:

``` 
sudo nano /etc/chrony/chrony.conf

bindcmdaddress 192.168.1.8
allow 192.168.1.0/24
ntpsigndsocket /var/lib/samba/ntp_signd
```

Reiniciar y verificar el servicio chronyd en el servidor Samba AD:

``` 
sudo systemctl restart chronyd
sudo systemctl status chronyd
``` 

### 1.5 VERIFICAR SAMBA ACTIVE DIRECTORY

Verificar nombres de dominio:

``` 
host -t A clockwork.local
host -t A dc.clockwork.local
```

Verificar que los registros de servicio kerberos y ldap apunten al FQDN de su servidor Samba Active Directory:

``` 
host -t SRV _kerberos._udp.clockwork.local
host -t SRV _ldap._tcp.clockwork.local
``` 

Verificar los recursos predeterminados disponibles en Samba Active Directory:

``` 
smbclient -L clockwork.local -N
``` 

Comprobar autenticación en el servidor de Kerberos mediante el administrador de usuarios:

``` 
kinit administrator@CLOCKWORK.LOCAL
klist
``` 

Iniciar sesión en el servidor a través de smb:

``` 
sudo smbclient //localhost/netlogon -U 'administrator'
``` 

Cambiar contraseña usuario administrator:

``` 
sudo samba-tool user setpassword administrator
``` 

Verificar la integridad del archivo de configuración de Samba:

``` 
testparm
``` 

Verificar funcionamiento WINDOWS AD DC 2008:

``` 
sudo samba-tool domain level show
``` 

Crear usuario SAMBA AD:

``` 
sudo samba-tool user create clockworker
``` 

Listar usuarios SAMBA AD:

``` 
sudo samba-tool user list
``` 

Eliminar un usuario:

``` 
samba-tool user delete <nombre_del_usuario>
``` 

Listar equipos SAMBA AD:

``` 
sudo samba-tool computer list
``` 

Eliminar equipo SAMBA AD:

``` 
sudo samba-tool computer delete <nombre_del_equipo>
``` 

Crear grupo:

``` 
samba-tool group add <nombre_del_grupo>
``` 

Listar grupos:

``` 
samba-tool group list
``` 

Listar miembros de un grupo:

``` 
samba-tool group listmembers 'Domain Admins'
``` 

Agregar un miembro a un grupo:

``` 
samba-tool group addmembers <nombre_del_grupo> <nombre_del_usuario>
``` 

Eliminar un miembro de un grupo:

``` 
samba-tool group removemembers <nombre_del_grupo> <nombre_del_usuario>
``` 

## 2. INTEGRAR UBUNTU DESKTOP EN DOMINIO UBUNTU SERVER

Cambiar hostname:

``` 
sudo hostnamectl set-hostname ud101
hostname -f
``` 

Modificar fichero hosts:

``` 
sudo nano /etc/hosts
192.168.1.8 clockwork.local clockwork
192.168.1.8 dc.clockwork.local dc
``` 

Comprobamos conexión:

``` 
ping -c2 clockwork.local
``` 

Es importante y necesario que resuelva.

Ahora, procedemos a instalar NTPDATE, que va a servirnos para sincronizar los relojes de nuestras máquinas.

``` 
sudo apt-get install ntpdate
sudo nptdate -q clockwork.local
sudo ntpdate clockwork.local
``` 

Instalar samba con sus paquetes y dependencias:

``` 
sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools
``` 

Los nombres de dominio que vamos a colocar son:

CLOCKWORK.LOCAL
dc.clockwork.local
dc.clockwork.local

Comprobar autenticación en el servidor de Kerberos mediante el administrador de usuarios:

``` 
kinit administrator@CLOCKWORK.LOCAL
klist
``` 

Movemos archivo smb.conf y creamos copia de seguridad:

``` 
mv /etc/samba/smb.conf /etc/samba/smb.conf.initial
``` 

Creamos archivo smb.conf vacío:

``` 
[global]
        workgroup = CLOCKWORK
        realm = CLOCKWORK.LOCAL
        netbios name = ud101
        security = ADS
        dns forwarder = 192.168.1.8

idmap config * : backend = tdb
idmap config *:range = 50000-1000000

    template homedir = /home/%D/%U
    template shell = /bin/bash
    winbind use default domain = true
    winbind offline logon = false
    winbind nss info = rfc2307
    winbind enum users = yes
    winbind enum groups = yes

vfs objects = acl_xattr
map acl inherit = Yes
store dos attributes = Yes
``` 

Reiniciamos los daemons de samba:

``` 
sudo systemctl restart smbd nmbd
``` 

Detenemos los servicios innecesarios:

``` 
sudo systemctl stop samba-ad-dc
``` 

Habilitamos los servicios de Samba:

``` 
sudo systemctl enable smbd nmbd
```

### 2.1 Unir Ubuntu Desktop a SAMBA AD DC

``` 
sudo net ads join -U administrator
```

Comprobamos en el servidor:

``` 
sudo samba-tool computer list
``` 

Y ya nos aparecerá el equipo de Ubuntu Desktop en el listado.

### 2.2 Configurar autenticación de cuentas AD

Editamos el archivo de configuración del conmutador de servicio de nombres:

``` 
sudo nano /etc/nsswitch.conf
```

``` 
passwd:     compat winbind
group:      compat winbind
shadow:     compat winbind
gshadow:    files

hosts:      files dns
networks:   files

protocols:  db files
services:   db files
ethers:     db files
rpc:        db files

netgroup:   nis
``` 

Reiniciamos servicio winbind:

```
sudo systemctl restart winbind
```

Listamos usuarios y grupos del dominio:

``` 
wbinfo -u
wbinfo -g
``` 

Verificamos el módulo winbind nsswitch con el comando getent:

``` 
sudo getent passwd| grep administrator
sudo getent group|grep 'domain admins'
id administrator
``` 

Configuramos pam-auth-update para autenticarnos con cuentas de dominio y que se creen automáticamente los directorios:

```
sudo pam-auth-update
``` 

Y activamos el apartado **Create home directory on login**

Editamos el archivo /etc/pam.d/common-account para que cree automáticamente los directorios:

``` 
nano /etc/pam.d/common-account
``` 

Añadimos al final del archivo:

``` 
session required    pam_mkhomedir.so    skel=/etc/skel/     umask=0022
``` 

y ahora podemos autenticarnos con la cuenta de administrator:

```
su administrator
``` 

Añadimos cuenta de dominio con privilegios root:

``` 
sudo usermod -aG sudo administrator
``` 

Si ahora cierro sesión en Ubuntu Desktop y me logueo por ejemplo con clockworker@clockwork.local, pongo la contraseña y me creará un directorio dentro del dominio.

Y con esto ya tenemos implementado servidor Samba con AD y acceso integrado con Ubuntu Desktop.
















