# Curso Azure AZ-104 III

## Casos prácticos

**Gestión de recursos desde el portal**

Creamos un grupo de recursos en el portal. Podemos añadirle una etiqueta, por ejemplo departamento: IT.

Creamos una red virtual para probar. Dentro de los datos, podemos cambiar el nombre de la subnet. Incluso añadir una subred secundaria, con diferente IP. Por ejemplo, una para el departamento de administración y otra para el de IT.

**Bloqueo de recursos**

Accedemos a los bloqueos en el menu de la izquierda, bajando bastante. 

**Gestionar recursos desde Powershell**

Lo primero es instalarnos el módulo az, para gestionar recursos desde Power Shell

``` 
install-module -name az -allowclobber -force
```

Lo siguiente será asegurarnos de que podamos ejecutar instrucciones que no esten firmadas digitalmente en nuestra consola de Power Shell:

``` 
Set-ExecutionPolicy Unrestricted
```

E indicamos que si queremos hacer cambios.

Lo siguiente será importar el módulo AZ accounts para que nosotros desde Power Shell podamos empezar a ejecutar instrucciones AZ:

```
import-module az.accounts
```

Y ya podemos trabajar con comandos Azure.

Nos conectamos:

``` 
connect-azaccount
```

Para crear un nuevo grupo de recursos:

```
New-AzResourceGroup -Name Testing -Location EastUS -Tag @{Department="IT"}
``` 

Para ver los grupos de recursos:

```
get-AzResourceGroup
```

Para borrar un grupo de recursos:

```
Remove-AzResourceGroup -Name Testing -Force
```

Podemos, mediante una variable, crear un grupo de recursos de forma automática:

```
$RGroupName = Read-Host -Prompt "Escribe el nombre del grupo de recursos"
```

Y podemos ver que se creado:

```
$RGroupName
```

Si yo vuelvo a lanzar toda la instrucción para crear un grupo de recursos, en el campo -Name puedo indicar la variable creada para el grupo de recursos. Esto puede hacerse igualmente para la ubicación o el nombre de una etiqueta.

Podemos hacer esto para crear una red virtual. Comenzamos creando las distintas variables:

```
$RGroupName = "Testing"
$Location = "EastUS"
$TagName = "Department"
$TagValue = "IT"
``` 

Acto seguido, creamos la red virtual añadiendo las variables anteriores a sus respectivos campos.

```
$VNet = New-AZVirtualNetwork -Name "testing-vnet" -ResourceGroupName $RGroupName -Location $Location -AddressPrefix 10.0.0.0/16 -Tag @{$TagName="TagValue"}
``` 

Para crear una nueva IP pública en caso de querer modificar la IP de una red virtual, por ejemplo.

```
$PublicIp = New-AzPublicIpAddress -Name "webserver-ip" -ResourceGroupName $RGroupName -Location $Location -AllocationMethod Static -Tag @{$TagName="TagValue"}
``` 

Vamos a imaginar que queremos crear un grupo de seguridad de red (Network Security Group), donde nosotros vamos a poder crear reglas para, por ejemplo, denegar o permitir el tráfico entrante o saliente hacia o desde nuestros recursos.

Vamos a crear dos reglas de entrada a nuestro grupo de recursos:

* Una para HTTP.

```
$RuleHTTP = New-AzNetworkSecurityRuleConfig -Name "HTTP" -Description "Permitir el acceso por HTTP" -Priority 301 -Protocol Tcp -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 -Direction Inbound -Access Allow
``` 

* Otra para HTTPS.

```
$RuleHTTPS = New-AzNetworkSecurityRuleConfig -Name "HTTPS" -Description "Permitir el acceso por HTTPS" -Priority 305 -Protocol Tcp -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 443 -Direction Inbound -Access Allow
```

Ahora debemos vincular estas reglas con el Network Security Group. 

```
$SecurityGroup = New-AzNetworkSecurityGroup -Name "WebServerSG" -ResourceGroupName $RGroupName -Location $Location -SecurityRules $RuleHTTP,$RuleHTTPS -Tag @{$TagName="$TagValue"}
```

- Mover recursos de un grupo de recursos a otro

Especificamos el grupo de recursos destino y origen:

``` 
$targetRG = "Testing2"
$sourceRG = "Testing"
```

Y el recurso a mover:

``` 
$resource = Get-AzResource -Name WebServerSG
```

Y lo movemos de un grupo de recursos a otro:

``` 
Move-AzResource -DestinationResourceGroupName $targetRG -ResourceId $resource.id -Force
```

Para borrar nuestros grupos de recursos:

```
Remove-AzResourceGroup -Name Testing -Force
```

**Instalar CLI de Azure**

Facil de descargar e instalar. Comprobamos que todo está OK mediante:

```  
az --version (Como cambio sustancial, ahora los parámetros se indican con dos guiones).
```

**Comandos CLI de Azure**

- Para conectarse a Azure

```
az login
```

- Para crear un grupo de recursos

```
az group create --name Testing --location EastUS
``` 

- Para ver el listado de grupo de recursos

``` 
az group list
``` 

- O en formato de tabla:

```  
az group list --output table
```

- Para crear una red virtual

``` 
az network vnet create --name testing-vnet --resourcegroup testing --subnet-name default
``` 

- Para crear una interfaz de red

``` 
az network nic create --resourcegroup testing --vnet-name testing-vnet --subnet MySubnet --name MyNic
``` 

- Para borrar un grupo de recursos

```
az group delete --name Testing
```

**Plantillas ARM**

Facilmente descargables desde el navegador y faciles de instanciar en Azure, pudiendo editarlas de manera sencilla.


















