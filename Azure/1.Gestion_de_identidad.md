# Curso Azure AZ-104 I

### Gestión de usuarios con Power Shell

* **New-AzureAduser**: Para crear un nuevo usuario
* **Get-AzureAduser**: Para mostrar los usuarios existentes
* **Set-AzureAduser**: Para cambiar la propiedad de un usuario
* **Remove-AzureAduser**: Para borrar un usuario
* **Set-AzureADUserPassword**: Para cambiar el password de un usuario

### Gestión de grupos con Power Shell

* **New-AzureAdgroup**: Para crear un nuevo grupo
* **Get-AzureAdgroup**: Para mostrar los grupos existentes
* **Set-AzureAdgroup**: Para cambiar la propiedad de un grupo
* **Remove-AzureAdgroup**: Para borrar un grupo
* **Add-AzureADGroupMember**: Para añadir un usuario a un grupo
* **Get-AzureADGroupMember**: Para mostrar los miembros del grupo
* **Remove-AzureADGroupMember**: Para quitar un usuario del grupo


## Caso práctico

Lo primero que se ha de hacer para trabajar con nuestros usuarios es entrar al portal de Azure y en el buscador poner Azure Active Directory, que se llama Iscateam. 

Dentro de la creación del usuario, puedo asignar sus grupos y roles. 

**Crear usuarios desde Power Shell**

Nos vamos a la Power Shell en modo administrador. Y comienzo creando el módulo de Azure AD mediante comando:

```
install-module -name azuread
```

Ahora lo conectamos con el tenant de Azure.

```
connect-azuread
```

Daremos nuestro nombre de usuario de Azure y su respectiva contraseña. Y tendremos conectada nuestro power shell con Azure.

Para crear un usuario, existe el problema de la contraseña. Para crear una contraseña a dicho usuario, primero habra que crear un objeto de la siguiente forma:

```
$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile

$PasswordProfile.password = "David77"

New-AzureADUser -DisplayName "david" -PasswordProfile $PasswordProfile -UserPrincipalName "david@iscateam.onmicrosoft.com" -AccountEnabled $true -MailNickName "david"
```

Para ver los usuarios creados

```
get-azureaduser
```

**Cambiar propiedad de un usuario**

```
$user = Get-AzureADUser -ObjectId david@iscateam.onmicrosoft.com

$user.Displayname = 'dasetris'

Set-AzureADUser -ObjectId david@iscateam.onmicrosoft.com -DisplayName $user.Displayname
```

Y comprobamos:

```
get-azureaduser
```

**Eliminar usuario**

```
remove-azureaduser -objectid + Ese object id
```

**Cambiar contraseña de un usuario**

```
$pass = ConvertTo-SecureString -String "P@ssW0rD!" -Force -AsPlainText 
```
Extraemos el object id del usuario en cuestión:

``` 
get-azureaduser
```

```
Set-AzureADUserPassword -objectid + "Ese object id" -password $pass
```

**Crear grupo desde PowerShell**

``` 
New-AzureADGroup -DisplayName "grupo2" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
``` 

Y se nos creará el grupo.

**Cambiar propiedades al grupo**

Por ejemplo el display name

```
set-AzureADGroup -ObjectId ObjectId -description "Administradores de Red"
```

Para ver los grupos

```
get-azureadgroup
```

**Borrar un grupo**

```  
Remove-AzureADGroup -ObjectId ObjectID del grupo a borrar
```

**Añadir un usuario dentro de un grupo**

```
Add-AzureADGroupMember -ObjectId ObjectId del grupo -RefObjectId ObjectId del usuario
```

Para comprobarlo:

```
Get-AzureADGroupMember -ObjectId ObjectId del grupo
```

**Quitar un usuario de un grupo**

``` 
remove-AzureADGroupMember -ObjectId ObjectId del grupo -memberId ObjectId del usuario
```

**AD Connect**

Lo instalamos desde afuera del dominio. A la hora de instalarlo, escogemos la opción de sincronización. 
Especificamos el directorio activo al cual queremos conectarlo e indicamos el dominio y el usuario administrador. 

En opciones de sincronizar, podemos escoger entre todo o los directorios a requerir.




























