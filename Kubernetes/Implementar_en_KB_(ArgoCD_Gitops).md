# Cómo implementar en Kubernetes usando Argo CD y GitOps


## 1. Introducción

El uso de Kubernetes para implementar su aplicación puede proporcionar importantes ventajas de infraestructura, como escalamiento flexible, administración de componentes distribuidos y control sobre diferentes versiones de su aplicación. Sin embargo, ese mayor control conlleva una mayor complejidad. Los sistemas de integración continua e implementación continua (CI/CD) suelen funcionar con un alto nivel de abstracción para proporcionar control de versiones, registro de cambios y funcionalidad de reversión. Un enfoque popular para esta capa de abstracción se llama GitOps.

Existen varias herramientas que utilizan Git como punto focal para los procesos de DevOps en Kubernetes. En este tutorial, aprenderá a utilizar Argo CD, una herramienta declarativa de entrega continua. Argo CD proporciona herramientas de entrega continua que sincronizan e implementan automáticamente su aplicación cada vez que se realiza un cambio en su repositorio de GitHub. Al gestionar la implementación y el ciclo de vida de una aplicación, proporciona soluciones para el control de versiones, configuraciones y definiciones de aplicaciones en entornos Kubernetes, organizando datos complejos con una interfaz de usuario fácil de entender. Puede manejar varios tipos de manifiestos de Kubernetes, incluidos Jsonnet, aplicaciones Kustomize, gráficos Helm y archivos YAML/json, y admite notificaciones de webhooks de GitHub, GitLab y Bitbucket.

En este artículo, utilizará Argo CD para sincronizar e implementar una aplicación desde un repositorio de GitHub.

## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un par de claves SSH en su máquina Linux/MacOS/BSD local. 

* Un clúster de Kubernetes existente que comprende al menos un nodo trabajador. kubectl debe estar instalado en su entorno de trabajo y poder conectarse a su clúster.

* Familiaridad con los conceptos de Kubernetes. 


## 3. Instalar Argo CD en su clúster

Para instalar Argo CD, primero debe tener una configuración válida de Kubernetes configurada con kubectl, desde la cual puede hacer ping a sus nodos trabajadores. Puedes probar esto ejecutando kubectl get nodes:

``` 
kubectl get nodes
``` 

Este comando debería devolver una lista de nodos con el Ready estado:

``` 
Output
NAME                   STATUS   ROLES    AGE   VERSION
pool-uqv8a47h0-ul5a7   Ready    <none>   22m   v1.21.5
pool-uqv8a47h0-ul5am   Ready    <none>   21m   v1.21.5
pool-uqv8a47h0-ul5aq   Ready    <none>   21m   v1.21.5
```

Si kubectl no devuelve un conjunto de nodos con el Ready estado, debe revisar la configuración de su clúster y la documentación de Kubernetes.

A continuación, cree el argocd espacio de nombres en su clúster, que contendrá Argo CD y sus servicios asociados:

``` 
kubectl create namespace argocd
``` 

Después de eso, puede ejecutar el script de instalación del CD Argo proporcionado por los mantenedores del proyecto.

``` 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Una vez que la instalación se complete exitosamente, puede usar el watch comando para verificar el estado de sus pods de Kubernetes:

``` 
watch kubectl get pods -n argocd
``` 

De forma predeterminada, debería haber cinco pods que eventualmente reciban el Running estado como parte de una instalación estándar de Argo CD.

``` 
Output
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0       1/1     Running   0          2m28s
argocd-dex-server-66f865ffb4-chwwg    1/1     Running   0          2m30s
argocd-redis-5b6967fdfc-q4klp         1/1     Running   0          2m30s
argocd-repo-server-656c76778f-vsn7l   1/1     Running   0          2m29s
argocd-server-cd68f46f8-zg7hq         1/1     Running   0          2m28s
``` 

Puede presionar Ctrl+C para salir de la watch interfaz. ¡Ahora tiene Argo CD ejecutándose en su clúster de Kubernetes! Sin embargo, debido a la forma en que Kubernetes crea abstracciones alrededor de sus interfaces de red, no podrá acceder directamente sin reenviar puertos desde dentro de su clúster. Aprenderá cómo manejar eso en el siguiente paso.


## 4. Reenvío de puertos para acceder a Argo CD

Debido a que Kubernetes implementa servicios en direcciones de red arbitrarias dentro de su clúster, deberá reenviar los puertos relevantes para poder acceder a ellos desde su máquina local. Argo CD configura argocd-server internamente un servicio denominado en el puerto 443. Debido a que el puerto 443 es el puerto HTTPS predeterminado y es posible que esté ejecutando otros servicios HTTP/HTTPS, es una práctica común reenviarlos a otros puertos elegidos arbitrariamente, como, así 8080:

``` 
kubectl port-forward svc/argocd-server -n argocd 8080:443
``` 

El reenvío de puertos bloqueará el terminal en el que se está ejecutando mientras esté activo, por lo que probablemente querrás ejecutar esto en una nueva ventana de terminal mientras continúas trabajando. Puede presionar Ctrl+C para salir elegantemente de un proceso de bloqueo como este cuando desee dejar de reenviar el puerto.

Mientras tanto, debería poder acceder a Argo CD en un navegador web navegando a localhost:8080. Sin embargo, se le solicitará una contraseña de inicio de sesión que deberá utilizar la línea de comando para recuperar en el siguiente paso. Probablemente necesitarás hacer clic en una advertencia de seguridad porque Argo CD aún no se ha configurado con un certificado SSL válido.


## 5. Trabajar con Argo CD desde la línea de comandos

Para los siguientes pasos, querrá tener el argocd comando instalado localmente para interactuar y cambiar la configuración en su instancia de Argo CD. La documentación oficial de Argo CD recomienda instalarlo a través del administrador de paquetes Homebrew. Homebrew es muy popular para administrar herramientas de línea de comandos en MacOS y, más recientemente, se ha portado a Linux para facilitar el mantenimiento de herramientas como esta.

Si aún no tienes Homebrew instalado, puedes recuperarlo e instalarlo con un comando de una línea:

```
​​/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Es posible que se le solicite su contraseña durante el proceso de instalación. Luego, deberías tener el brew comando disponible en tu terminal. Puedes usarlo para instalar Argo CD:

``` 
brew install argocd
``` 

Esto a su vez proporciona el argocd comando. Antes de usarlo, querrá usarlo kubectl nuevamente para recuperar la contraseña de administrador que se generó automáticamente durante su instalación, de modo que pueda usarla para iniciar sesión. Le pasará una ruta a un archivo JSON particular que se almacena usando Kubernetes. secretos y extraer el valor relevante:

``` 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

``` 
Output
fbP20pvw-o-D5uxH
``` 

Luego puede iniciar sesión en su panel de Argo CD volviendo a localhost:8080 un navegador e iniciando sesión como admin usuario con su propia contraseña.

Una vez que todo esté funcionando, puede usar las mismas credenciales para iniciar sesión en Argo CD a través de la línea de comando, ejecutando argocd login. Esto será necesario para implementar desde la línea de comando más adelante:

``` 
argocd login localhost:8080
```

Recibirá nuevamente la advertencia de certificado equivalente en la línea de comando aquí y deberá ingresar y para continuar cuando se le solicite. Si lo desea, puede cambiar su contraseña por algo más seguro o más fácil de recordar ejecutando argocd account update-password. Después de eso, tendrá una configuración de Argo CD completamente funcional. En los pasos finales de este tutorial, aprenderá cómo usarlo para implementar algunas aplicaciones de ejemplo.


## 6. Manejo de múltiples clústeres (opcional)

Antes de implementar una aplicación, debe revisar dónde realmente desea implementarla. De forma predeterminada, Argo CD implementará aplicaciones en el mismo clúster en el que se ejecuta Argo CD, lo cual está bien para una demostración, pero probablemente no sea lo que querrás en producción. Para enumerar todos los clústeres conocidos por su máquina actual, puede utilizar kubectl config:

``` 
kubectl config get-contexts -o name
Output
test-deploy-cluster
test-target-cluster
``` 

Suponiendo que instaló Argo CD test-deploy-cluster y desea usarlo para implementar aplicaciones test-target-cluster, puede registrarse test-target-cluster en Argo CD ejecutando argocd cluster add:

``` 
argocd cluster add target-k8s
``` 

Esto agregará los detalles de inicio de sesión del clúster adicional a Argo CD y permitirá que Argo CD implemente servicios en el clúster.


## 7.  Implementación de una aplicación de ejemplo (opcional)

Ahora que tiene Argo CD ejecutándose y comprende cómo implementar aplicaciones en diferentes clústeres de Kubernetes, es hora de ponerlo en práctica. El proyecto Argo CD mantiene un repositorio de aplicaciones de ejemplo que han sido diseñadas para mostrar los fundamentos de GitOps. Muchos de estos ejemplos son adaptaciones de la misma guestbook aplicación de demostración a diferentes tipos de manifiestos de Kubernetes, como Jsonnet. En este caso, implementará el helm-guestbook ejemplo, que utiliza un gráfico Helm, una de las soluciones de administración de Kubernetes más duraderas.

Para hacer eso, usará el argocd app create comando, proporcionando la ruta al repositorio de Git, el helm-guestbook ejemplo específico y pasando su destino y espacio de nombres predeterminados:

``` 
argocd app create helm-guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path helm-guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
```

Después de "crear" la aplicación dentro de Argo CD, puede verificar su estado con argocd app get:

``` 
argocd app get helm-guestbook
Output
Name:               helm-guestbook
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:8080/applications/helm-guestbook
Repo:               https://github.com/argoproj/argocd-example-apps.git
Target:
Path:               helm-guestbook
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (53e28ff)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME            STATUS     HEALTH   HOOK  MESSAGE
       Service     default    helm-guestbook  OutOfSync  Missing
apps   Deployment  default    helm-guestbook  OutOfSync  Missing
``` 

El OutOfSync estado de la solicitud es normal. Obtuviste el gráfico de timón de la aplicación de Github y creaste una entrada para él en Argo CD, pero todavía no has generado ningún recurso de Kubernetes para ello. Para implementar realmente la aplicación, ejecutará argocd app sync:

``` 
argocd app sync helm-guestbook
``` 

sync es sinónimo de implementación aquí de acuerdo con los principios de GitOps: el objetivo al usar Argo CD es que su aplicación siempre realice un seguimiento 1:1 con su configuración ascendente.

``` 
Output
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2022-01-19T11:01:48-08:00            Service     default        helm-guestbook  OutOfSync  Missing
2022-01-19T11:01:48-08:00   apps  Deployment     default        helm-guestbook  OutOfSync  Missing
2022-01-19T11:01:48-08:00            Service     default        helm-guestbook    Synced  Healthy
2022-01-19T11:01:48-08:00            Service     default        helm-guestbook    Synced   Healthy              service/helm-guestbook created
2022-01-19T11:01:48-08:00   apps  Deployment     default        helm-guestbook  OutOfSync  Missing              deployment.apps/helm-guestbook created
2022-01-19T11:01:49-08:00   apps  Deployment     default        helm-guestbook    Synced  Progressing              deployment.apps/helm-guestbook created

Name:               helm-guestbook
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:8080/applications/helm-guestbook
Repo:               https://github.com/argoproj/argocd-example-apps.git
Target:
Path:               helm-guestbook
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (53e28ff)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      53e28ff20cc530b9ada2173fbbd64d48338583ba
Phase:              Succeeded
Start:              2022-01-19 11:01:49 -0800 PST
Finished:           2022-01-19 11:01:50 -0800 PST
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME            STATUS  HEALTH       HOOK  MESSAGE
       Service     default    helm-guestbook  Synced  Healthy            service/helm-guestbook created
apps   Deployment  default    helm-guestbook  Synced  Progressing        deployment.apps/helm-guestbook created
``` 

¡Ahora ha implementado exitosamente una aplicación usando Argo CD! Es posible lograr lo mismo desde la interfaz web de Argo CD, pero generalmente es más rápido y reproducible implementarlo a través de la línea de comando. Sin embargo, es muy útil consultar el panel web de Argo CD después de la implementación para verificar que sus aplicaciones se estén ejecutando correctamente. Puedes verlo abriendo localhost:8080 en un navegador.

En este punto, lo último que debe hacer es asegurarse de poder acceder a su nueva implementación en un navegador. Para hacer eso, reenviará otro puerto, como lo hizo con Argo CD. Internamente, la helm-guestbook aplicación se ejecuta en el puerto HTTP normal 80 y, para evitar conflictos con cualquier cosa que pueda estar ejecutándose en su propio puerto 80 o en el puerto 8080 que está usando para Argo CD, puede reenviarla al puerto 9090:

``` 
kubectl port-forward svc/helm-guestbook 9090:80
``` 

Como antes, probablemente querrás hacer esto en otra terminal, porque bloqueará esa terminal hasta que presiones Ctrl+C para dejar de reenviar el puerto. Luego puede abrir localhost:9090 en una ventana del navegador para ver su aplicación de libro de visitas de ejemplo.

Cualquier envío adicional a este repositorio de Github se reflejará automáticamente en ArgoCD, que resincronizará su implementación y brindará disponibilidad continua.


**Conclusión**

Ahora ha visto los fundamentos de la instalación e implementación de aplicaciones utilizando Argo CD. Debido a que Kubernetes requiere tantas capas de abstracción, es importante asegurarse de que sus implementaciones sean lo más fáciles de mantener posible y la filosofía GitOps es una buena solución.

