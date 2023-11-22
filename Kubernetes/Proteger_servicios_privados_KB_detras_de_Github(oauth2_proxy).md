# Cómo proteger los servicios privados de Kubernetes detrás de un inicio de sesión en GitHub con oauth2_proxy

## 1. Introducción

Los ingresos de Kubernetes facilitan la exposición de servicios web a Internet. Sin embargo, cuando se trata de servicios privados, es probable que desees limitar quién puede acceder a ellos. oauth2_proxy puede servir como barrera entre la Internet pública y los servicios privados. oauth2_proxy es un servidor y proxy inverso que proporciona autenticación mediante diferentes proveedores, como GitHub, y valida a los usuarios por su dirección de correo electrónico u otras propiedades.

En este tutorial utilizará oauth2_proxy con GitHub para proteger sus servicios.

## 2. Requisitos previos

Para completar este tutorial, necesitará:

* Un clúster de Kubernetes con dos servicios web ejecutándose con una entrada Nginx y Let's Encrypt. Para ello visita el siguiente enlace: <https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes>
* Una cuenta de GitHub.
* Python instalado en su máquina local. 


## 3. Configurar tus dominios

Después de seguir el tutorial vinculado en la sección Requisitos previos, tendrá dos servicios web ejecutándose en su clúster: echo1 y echo2. También tendrás un ingreso que mapea y a sus servicios correspondientes.echo1.your_domain echo2.your_domain

En este tutorial, usaremos las siguientes convenciones:

Todos los servicios privados estarán bajo el subdominio, como . Agrupar servicios privados en un subdominio es ideal porque la cookie de autenticación se compartirá entre todos los subdominios. .int.your_domain service.int.your_domain *.int.your_domain
El portal de inicio de sesión se ofrecerá en .auth.int.your_domain

Para comenzar, actualice la definición de ingreso existente para mover los servicios echo1 y echo2 a . Abre en tu editor de texto para que puedas cambiar los dominios:.int.your_domain echo_ingress.yaml

``` 
nano echo_ingress.yaml
``` 

Cambie el nombre de todas las instancias de a y reemplace todas las instancias de con:echo1.your_domain echo1.int.your_domain echo2.your_domain echo2.int.your_domain

``` 
echo_ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:  
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - echo1.int.your_domain
    - echo2.int.your_domain
    secretName: letsencrypt-prod
  rules:
  - host: echo1.int.your_domain
    http:
      paths:
      - backend:
          serviceName: echo1
          servicePort: 80
  - host: echo2.int.your_domain
    http:
      paths:
      - backend:
          serviceName: echo2
          servicePort: 80
``` 

Guarde el archivo y aplique los cambios:

``` 
kubectl apply -f echo_ingress.yaml
``` 

Esto también actualizará los certificados TLS para usted echo1 y sus servicios .echo2

Ahora actualice su configuración de DNS para reflejar los cambios que realizó. Primero, busque la dirección IP de su entrada de Nginx ejecutando el siguiente comando para imprimir sus detalles:

``` 
kubectl get svc --namespace=ingress-nginx
``` 

Verá la dirección IP en EXTERNAL-IP el resultado:

```
Output
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx   LoadBalancer   10.245.247.67   203.0.113.0   80:32486/TCP,443:32096/TCP   20h
```

Copie la dirección IP externa a su portapapeles. Busque su servicio de administración de DNS y localice los registros A para que apunten a esa dirección IP externa. 

Elimine los registros de echo1 y echo2. Agregue un nuevo A registro para el nombre de host y apúntelo a la dirección IP externa de la entrada. *.int.your_domain

Ahora, cualquier solicitud a cualquier subdominio se enrutará a la entrada de Nginx, por lo que puede usar estos subdominios dentro de su clúster. *.int.your_domain

A continuación, configurará GitHub como su proveedor de inicio de sesión.


## 4. Crear una aplicación GitHub OAuth

oauth2_proxy admite varios proveedores de inicio de sesión. En este tutorial, utilizará el proveedor GitHub. Para comenzar, cree una nueva aplicación GitHub OAuth.

En la pestaña Aplicaciones OAuth de la página de configuración de desarrollador de su cuenta, haga clic en el botón Nueva aplicación OAuth.

Los campos Nombre de la aplicación y URL de la página de inicio pueden ser los que desee. En el campo URL de devolución de llamada de autorización, ingrese .https://auth.int.your_domain/oauth2/callback

Después de registrar la aplicación, recibirá una ID de cliente y un secreto. Tenga en cuenta los dos, ya que los necesitará en el siguiente paso.

Ahora que ha creado una aplicación GitHub OAuth, puede instalar y configurar oauth2_proxy.

## 5. configurar el portal de inicio de sesión

Utilizará Helm para instalar oauth2_proxy en el clúster. Primero, creará un secreto de Kubernetes para guardar el ID de cliente y el secreto de la aplicación GitHub, así como un secreto de cifrado para las cookies del navegador establecidas por oauth2_proxy.

Ejecute el siguiente comando para generar un secreto de cookie seguro:

``` 
python -c 'import os,base64; print base64.b64encode(os.urandom(16))'
``` 

Copia el resultado a tu portapapeles

Luego, cree el secreto de Kubernetes, sustituyendo los valores resaltados por su secreto de cookie, su ID de cliente de GitHub y su clave secreta de GitHub:

``` 
kubectl -n default create secret generic oauth2-proxy-creds \
--from-literal=cookie-secret=YOUR_COOKIE_SECRET \
--from-literal=client-id=YOUR_GITHUB_CLIENT_ID \
--from-literal=client-secret=YOUR_GITHUB_SECRET
``` 

Verá el siguiente resultado:

``` 
Output
secret/oauth2-proxy-creds created
```

A continuación, cree un nuevo archivo llamado oauth2-proxy-config.yaml que contendrá la configuración para oauth2_proxy:

``` 
nano oauth2-proxy-config.yaml
``` 

Los valores que establecerá en este archivo anularán los valores predeterminados del gráfico Helm. Agregue el siguiente código al archivo:

``` 
oauth2-proxy-config.yaml
config:
  existingSecret: oauth2-proxy-creds

extraArgs:
  whitelist-domain: .int.your_domain
  cookie-domain: .int.your_domain
  provider: github

authenticatedEmailsFile:
  enabled: true
  restricted_access: |-
    allowed@user1.com
    allowed@user2.com

ingress:
  enabled: true
  path: /
  hosts:
    - auth.int.your_domain
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
  tls:
    - secretName: oauth2-proxy-https-cert
      hosts:
        - auth.int.your_domain
``` 

Este código hace lo siguiente:

* Indica a oauth2_proxy que utilice el secreto que creó.
* Establece el nombre de dominio y el tipo de proveedor.
* Establece una lista de direcciones de correo electrónico permitidas. Si una cuenta de GitHub está asociada con una de estas direcciones de correo electrónico, se le permitirá el acceso a los servicios privados.
* Configura la entrada que servirá al portal de inicio de sesión con un certificado TLS de Let's Encrypt.auth.int.your_domain
  
Ahora que tiene listo el archivo secreto y de configuración, puede instalar oauth2_proxy. Ejecute el siguiente comando:

``` 
helm repo update \
&& helm upgrade oauth2-proxy --install stable/oauth2-proxy \
--reuse-values \
--values oauth2-proxy-config.yaml
``` 

Es posible que transcurran algunos minutos hasta que se emita e instale el certificado Let's Encrypt.

Para probar que la implementación fue exitosa, vaya a . Verás una página que te solicitará que inicies sesión con GitHub.https://auth.int.your_domain

Con oauth2_proxy configurado y en ejecución, todo lo que queda es solicitar autenticación en sus servicios.

## 6. Proteger los servicios privados

Para proteger un servicio, configure su entrada Nginx para aplicar la autenticación a través de oauth2_proxy. Nginx y nginx-ingress admiten esta configuración de forma nativa, por lo que solo necesita agregar un par de anotaciones a la definición de ingreso.

Protejamos los servicios echo1 y echo2 que configuró en el tutorial de requisitos previos. Abre echo_ingress.yaml en tu editor:

``` 
nano echo_ingress.yaml
``` 

Agregue estas dos anotaciones adicionales al archivo para requerir autenticación:

``` 
echo_ingress.yaml
   annotations:
     kubernetes.io/ingress.class: nginx
     certmanager.k8s.io/cluster-issuer: letsencrypt-prod
     nginx.ingress.kubernetes.io/auth-url: "https://auth.int.your_domain/oauth2/auth"
     nginx.ingress.kubernetes.io/auth-signin: "https://auth.int.your_domain/oauth2/start?rd=https%3A%2F%2F$host$request_uri"
``` 

Guarde el archivo y aplique los cambios:

``` 
kubectl apply -f echo_ingress.yaml
``` 

Ahora, cuando navegue hasta , se le pedirá que inicie sesión con GitHub para poder acceder. Después de iniciar sesión con una cuenta válida, será redirigido nuevamente al servicio. Lo mismo ocurre con .https://echo1.int.your_domainecho1echo2


## 7. Conclusión

En este tutorial, configurará oauth2_proxy en su clúster de Kubernetes y protegerá un servicio privado detrás de un inicio de sesión de GitHub. Para cualquier otro servicio que necesite proteger, simplemente siga las instrucciones descritas en el Paso 4.

oauth2_proxy admite muchos proveedores diferentes además de GitHub.



