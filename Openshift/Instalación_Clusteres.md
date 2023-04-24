# Apuntes sobre el curso DO322 de Red Hat

### Personalización de las instalaciones sin conexión

Después de copiar el registro local de contenedores, inicie el proceso de instalación de OpenShift. Debe modificar el archivo install-config.yaml para personalizar la instalación sin conexión de OpenShift.

 **El siguiente es un archivo install-config.yaml de muestra que se usa en una instalación con conexión:**

```
[user@demo ~]$ cat ${HOME}/ocp4-cluster/install-config.yaml
apiVersion: v1
baseDomain: example.com
#proxy: 1
#  httpProxy: http://<username>:<pswd>@<ip>:<port>
#  httpsProxy: http://<username>:<pswd>@<ip>:<port>
#  noProxy: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: |
  {"auths":...output omitted...} 2
sshKey: |
  ssh-rsa AA...output omitted...
```

1

Si es necesario, puede usar un proxy de red en una instalación con conexión.

2

pull-secret contiene las credenciales para autenticarse con quay.io y registry.redhat.io.

**El siguiente es un archivo install-config.yaml de muestra que se usa en una instalación sin conexión:**

```
[user@demo ~]$ cat ${HOME}/ocp4-cluster/install-config.yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: |
  {"auths":...output omitted...} 1
sshKey: |
  ssh-rsa AA...output omitted...
additionalTrustBundle: | 2
  -----BEGIN CERTIFICATE-----
  ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
  -----END CERTIFICATE-----
imageContentSources: 3
- mirrors:
  - nexus-registry-int.apps.tools.dev.nextcle.com/openshift/ocp4
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - nexus-registry-int.apps.tools.dev.nextcle.com/openshift/ocp4
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
```

1

pull-secret contiene las credenciales para autenticarse con el registro local de contenedores nexus-registry-int.apps.tools.dev.nextcle.com.

2

Si no se confía en el certificado usado por el registro local de contenedores, proporcione el conjunto de certificados que usó para el registro local de contenedores.

3

Agregue la sección imageContentSources de la salida del comando que se usó para copiar el repositorio. Esta personalización configurará el archivo /etc/containers/registries.conf en cada nodo del clúster para habilitar la duplicación de imágenes.

**En la siguiente tabla, se muestran algunas opciones útiles disponibles para el comando openshift-install:**

 | Opción |	Descripción |
|:--------:|:------------:|
| help | Imprime información de ayuda |
| explain -h	| Explica los campos relacionados con cada API InstallConfig soportada |
| explain installconfig	| Explica el recurso installconfig y sus campos |
| --log-level debug	| Habilita el modo de depuración |
| --dir	| Configura la ruta del directorio para almacenar los activos generados |
| create install-config	| Crea el archivo de configuración de instalación install-config.yaml |
| create manifests	| Crea manifiestos de Kubernetes |
| create ignition-configs | Crea archivos de configuración de encendido (ignition) |
| create cluster	| Implementa el clúster |
| wait-for bootstrap-complete	| Espera hasta que finalice la fase de instalación de bootstrap |
| wait-for install-complete	| Espera hasta que finaliza la instalación del clúster |
| destroy cluster	| Elimina el clúster |
