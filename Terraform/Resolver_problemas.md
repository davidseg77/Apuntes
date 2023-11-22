# Cómo solucionar problemas de Terraform

## 1. Introducción

Las cosas no siempre salen según lo planeado: las implementaciones pueden fallar, los recursos existentes pueden fallar inesperadamente y usted y su equipo podrían verse obligados a solucionar el problema lo antes posible. Comprender los métodos para abordar la depuración de su proyecto Terraform es crucial cuando necesita dar una respuesta rápida.

De manera similar al desarrollo con otros lenguajes y marcos de programación, configurar niveles de registro en Terraform para obtener información sobre sus flujos de trabajo internos con la detalle necesario es una característica que puede ayudarle a solucionar problemas. Al registrar las acciones internas, puede descubrir errores implícitamente ocultos, como variables que utilizan de forma predeterminada un tipo de datos inadecuado. También es común con los marcos la capacidad de importar y usar módulos (o bibliotecas) de terceros para reducir la duplicación de código entre proyectos.

En este tutorial, verificará que las variables siempre tengan valores razonables y especificará exactamente qué versiones de proveedores y módulos necesita para evitar conflictos. También habilitará varios niveles de detalle del modo de depuración, lo que puede ayudarle a diagnosticar un problema subyacente en Terraform.


## 2. Establecer restricciones de versión

Aunque la capacidad de hacer uso de módulos y proveedores de terceros puede minimizar la duplicación de código y el esfuerzo al escribir su código Terraform, también es posible que los desarrolladores de módulos de terceros lancen nuevas versiones que potencialmente pueden traer cambios importantes a su código específico. Para evitar esto, Terraform le permite especificar límites de versión para garantizar que solo se instalen y utilicen las versiones que desea. Especificar versiones significa que necesitarás las versiones que has probado en tu código, pero también deja la posibilidad de una actualización futura.

Las restricciones de versión se especifican como cadenas y se pasan al version parámetro cuando define los requisitos del módulo o del proveedor. Ejecute el siguiente comando para revisar qué requisitos de versión especifica:

``` 
cat provider.tf
``` 

Encontrarás el código de proveedor de la siguiente manera:

``` 
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}
...
``` 

Las restricciones de versión pueden ser más complejas que simplemente especificar una versión, como es el caso anterior. Pueden contener uno o más grupos de condiciones, separados por una coma ( ,). Cada uno de los grupos define una gama aceptable de versiones y puede incluir operadores, como:

* >, <, >=, <=: para comparaciones, como >=1.0, que requerirían que la versión sea igual o mayor que 1.0.
* !=: para excluir una versión específica: != 1.0 denegaría 1.0 el uso y la solicitud de la versión.
* ~>: para hacer coincidir la versión especificada hasta la parte de la versión más a la derecha, que puede incrementarse ( ~>1.5.10 coincidirá 1.5.10 y 1.5.11, pero no coincidirá 1.5.9).
  
Aquí hay dos ejemplos de restricciones de versión con múltiples grupos:

* >=1.0, <2.0: permite todas las versiones desde la 1.0 serie en adelante, hasta 2.0.
* >1.0, != 1.5: permite versiones mayores, pero no iguales 1.0, con la excepción de 1.5, que también excluye.
  
Para que se seleccione una posible versión disponible, debe superar todas las restricciones especificadas y seguir siendo compatible con otros módulos y proveedores, así como con la versión de Terraform que está utilizando. Si Terraform no considera aceptable ninguna combinación, no podrá realizar ninguna tarea porque las dependencias siguen sin resolverse. Cuando Terraform identifica versiones aceptables que satisfacen las restricciones, utiliza la última disponible.

En esta sección, aprendió a bloquear el rango de versiones de módulos y recursos que puede instalar en su proyecto especificando restricciones de versión. Esto es útil cuando desea estabilidad utilizando únicamente versiones probadas y aprobadas de código de terceros. En la siguiente sección, configurará Terraform para que muestre registros más detallados, que son necesarios para informes de errores y depuración adicional en caso de fallas.

## 3. Habilitar el modo de depuración

Podría haber un error o una entrada con formato incorrecto en su flujo de trabajo, lo que puede provocar que sus recursos no se aprovisionen según lo previsto. En casos tan raros, es importante saber cómo acceder a registros detallados que describen lo que está haciendo Terraform. Pueden ayudar a identificar la causa del error, indicarle si fue creado por el usuario o solicitarle que informe el problema a los desarrolladores de Terraform si se trata de un error interno.

Terraform expone la TF_LOG variable de entorno para establecer el nivel de detalle del registro. Hay cinco niveles:

**TRACE:** la verbosidad más elaborada, ya que muestra cada paso dado por Terraform y produce resultados enormes con registros internos.

**DEBUG:** describe lo que sucede internamente de una manera más concisa en comparación con TRACE.

**ERROR:** muestra errores que impiden que Terraform continúe.

**WARN:** registra advertencias, que pueden indicar una mala configuración o errores, pero no son críticas para la ejecución.

**INFO:** muestra mensajes generales de alto nivel sobre el proceso de ejecución.

Para especificar el nivel de registro deseado, deberá establecer la variable de entorno en el valor apropiado:

```
export TF_LOG=log_level
``` 

Si TF_LOG está definido, pero el valor no es uno de los cinco niveles de detalle enumerados, Terraform utilizará de forma predeterminada TRACE.

Ahora definirá un recurso Droplet e intentará implementarlo con diferentes niveles de registro. Almacenará la definición de Droplet en un archivo llamado droplets.tf, así que créelo y ábralo para editarlo:

``` 
nano droplets.tf
``` 

Agregue las siguientes líneas:

``` 
resource "digitalocean_droplet" "test-droplet" {
  image  = "ubuntu-18-04-x64"
  name   = "test-droplet"
  region = "fra1"
  size   = "s-1vcpu-1gb"
}
``` 

Este Droplet ejecutará Ubuntu 18.04 con un núcleo de CPU y 1 GB de RAM en la fra1 región; lo llamarás test-droplet. Eso es todo lo que necesita definir, así que guarde y cierre el archivo.

Antes de implementar el Droplet, establezca el nivel de registro DEBUG ejecutando:

``` 
export TF_LOG=DEBUG
```

Luego, planifique el aprovisionamiento de Droplet:

```
terraform plan -var "do_token=${DO_PAT}"
``` 

El resultado será muy largo y puede inspeccionarlo más de cerca para descubrir que cada línea comienza con el nivel de detalle (importancia) entre paréntesis dobles. Verás que la mayoría de las líneas comienzan con [DEBUG].

[WARN] y [INFO] también están presentes; eso se debe a que TF_LOG establece el nivel de registro más bajo. Esto significa que tendría que configurar TF_LOG para TRACE mostrar TRACE y todos los demás niveles de registro al mismo tiempo.

Si se produce un error interno, Terraform mostrará el seguimiento de la pila y saldrá, deteniendo la ejecución. Desde allí, podrá localizar en qué parte del código fuente se produjo el error y, si se trata de un error, informarlo a los desarrolladores de Terraform. De lo contrario, si se trata de un error en su código, Terraform se lo señalará para que pueda corregirlo en su proyecto.

Así es como sería la salida del registro cuando el backend de DigitalOcean no pueda verificar su token API. Genera un error de usuario debido a una entrada incorrecta:

``` 
Output...
digitalocean_droplet.test-droplet: Creating...
2021/01/20 06:54:35 [ERROR] eval: *terraform.EvalApplyPost, err: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 401 Unable to authenticate you
2021/01/20 06:54:35 [ERROR] eval: *terraform.EvalSequence, err: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 401 Unable to authenticate you

Error: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 401 Unable to authenticate you

  on droplets.tf line 1, in resource "digitalocean_droplet" "test-droplet":
   1: resource "digitalocean_droplet" "test-droplet" {
   ...
```

Para deshabilitar el modo de depuración y restablecer la detalle del registro a su nivel predeterminado, borre la TF_LOG variable de entorno ejecutando:

``` 
unset TF_LOG
``` 

Ahora ha aprendido a habilitar modos de registro más detallados. Son muy útiles para diagnosticar fallos y comportamientos inesperados de Terraform. En la siguiente sección, revisará la verificación de variables y la prevención de casos extremos.

## 4. Validación de variables

En esta sección, se asegurará de que las variables siempre tengan valores sensatos y apropiados según su tipo y parámetros de validación.

En HCL (lenguaje de configuración de HashiCorp), al definir una variable no es necesario especificar nada excepto su nombre. Declararías una variable de ejemplo llamada test_ipasí:

``` 
variable "test_ip" { }
``` 

Luego puede usar este valor a través del código, pasando su valor cuando ejecute Terraform.

Si bien eso funcionará, esta definición tiene dos desventajas: primero, no se puede pasar un valor en tiempo de ejecución; y segundo, puede ser de cualquier tipo ( bool, string, etc.), que puede no ser adecuado para su propósito. Para remediar esto, siempre debes especificar su valor y tipo predeterminados:

``` 
variable "test_ip" {
  type    = string
  default = "8.8.8.8"
}
``` 

Al establecer un default valor, se asegura de que el código que hace referencia a la variable permanezca operativo en caso de que no se proporcione un valor más específico. Cuando especifica un tipo, Terraform puede validar el nuevo valor en el que se debe establecer la variable y mostrar un error si no se ajusta al tipo. Un ejemplo de este comportamiento sería intentar encajar a string en un archivo number.

Puede proporcionar una rutina de validación para variables que pueden generar un mensaje de error si la validación falla. Ejemplos de validación serían verificar la longitud del nuevo valor si es una cadena o buscar al menos una coincidencia con una expresión RegEx en el caso de datos estructurados.

Para agregar validación de entrada a su variable, defina un validation bloque:

``` 
variable "test_ip" {
  type    = string
  default = "8.8.8.8"

  validation {
    condition     = can(regex("\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}", var.test_ip))
    error_message = "The provided value is not a valid IP address."
  }
}
``` 

En validation, puede especificar dos parámetros entre llaves:

* A **condition** que acepte un bool calculará, lo que indicará si pasa la validación.
* An **error_message** que especifica el mensaje de error en caso de que la validación no pase.

En este ejemplo, se calcula condition buscando una regex coincidencia en el valor de la variable. Pasas eso a la can función. La can función devuelve true si la función que se pasa como parámetro se ejecutó sin errores, por lo que es útil para comprobar cuándo una función se completó correctamente o devolvió resultados.

La regex función que estamos usando aquí acepta una expresión regular (RegEx), la aplica a una cadena determinada y devuelve las subcadenas coincidentes. La expresión regular coincide con cuatro pares de números de tres dígitos, separados por puntos entre ellos. 

Ahora sabe cómo especificar un valor predeterminado para una variable, cómo configurar su tipo y cómo habilitar la validación de entrada mediante expresiones RegEx.

## 5. Conclusión

En este tutorial, solucionó problemas de Terraform habilitando el modo de depuración y configurando la detalle del registro en los niveles apropiados. También ha aprendido sobre algunas de las características avanzadas de las variables, como declarar procedimientos de validación y establecer buenos valores predeterminados. Omitir los valores predeterminados es un error común que puede causar problemas extraños más adelante en el desarrollo de su proyecto.

Para la estabilidad de su proyecto, se recomienda bloquear las versiones de proveedores y módulos de terceros, ya que le deja con un sistema estable y una ruta de actualización cuando sea necesario.

La verificación de los valores de entrada de las variables no se limita a hacer coincidir con regex. Para conocer más funciones integradas que puede utilizar, visite los documentos oficiales.

