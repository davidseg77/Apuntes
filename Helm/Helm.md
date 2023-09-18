# Apuntes sobre la herramienta Helm

Un Helm chart consta principalmente de dos archivos YAML y una lista de plantillas.

Los archivos YAML son los siguientes:

    Chart.yaml: contiene la información de definición del gráfico.

    values.yaml: contiene los valores que Helm usa en las plantillas predeterminadas y creadas por el usuario.

La estructura básica del archivo Chart.yaml es la siguiente:

```
apiVersion: v2 1
name: mychart 2
description: A Helm chart for Kubernetes 3
type: application 4
version: 0.1.0 5
appVersion: "1.0.0" 6
```

1

Versión de la API de Helm para utilizar

2

Nombre de Helm Chart

3

Descripción de Helm Chart

4

Tipo de Helm Chart: aplicación o biblioteca

5

Versión de Helm Chart

6

Versión de la aplicación de paquetes de este gráfico

---------------------------------------------------------------------------------------------------------------

El archivo Chart.yaml puede contener otros valores opcionales; uno de los valores más importantes es la sección dependencies. La sección dependencies contiene una lista de otros Helm Charts que permiten que funcione este gráfico de Helm.

La estructura de la sección dependencies es la siguiente:

```
dependencies: 1
  - name: dependency1 2
    version: 1.2.3 3
    repository: https://examplerepo.com/charts 4
  - name: dependency2
    version: 3.2.1
    repository: https://helmrepo.example.com/charts
```

1

Enumeración de dependencias

2

Nombre del primer Helm Chart requerido

3

Versión del gráfico de la que depende

4

Repositorio que depende de Helm Chart

## 1. Actualice las dependencias para el gráfico

Esto descarga los gráficos agregados como dependencias y bloquea sus versiones.

```
[student@workstation famouschart]$ helm dependency update
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://charts.bitnami.com/bitnami" chart repository
Saving 1 charts
Downloading mariadb from repo https://charts.bitnami.com/bitnami
Deleting outdated charts
```

## 2. Use el comando helm install para implementar la aplicación en el clúster de RHOCP:

```
[student@workstation famouschart]$ helm install famousapp .
NAME: famousapp
LAST DEPLOYED: Thu May 20 19:12:09 2021
NAMESPACE: yourdevuser-multicontainer-helm
STATUS: deployed
REVISION: 1
NOTES:
...output omitted...
```

## 3. Para crear un helm chart

```
[student@workstation multicontainer-review]$ helm create exochart
Creating exochart
```

--------------------------------------------------------------------------------------------

Las dependencias se agregan al final del archivo chart.yaml, y después se actualizan.

Las variables de entorno se agregan al final del archivo values.yaml

Después se accede a openshift y se inserta el helm con helm install + nombre del helm

## 4. Use el comando helm template para extraer las definiciones de objetos de un Helm Chart en la definición base del kustomize:

```
[student@workstation exokustom]$ helm template exoplanets \
../exochart > base/deployment.yaml
```





