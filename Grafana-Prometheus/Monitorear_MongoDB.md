# Cómo monitorear MongoDB con Grafana y Prometheus en Ubuntu 20.04

## 1. Introducción

Es fundamental que los administradores de bases de datos eviten problemas de rendimiento o memoria. Herramientas como Prometheus y Grafana pueden ayudarlo a monitorear el rendimiento del clúster de su base de datos. Prometheus es una plataforma de alertas y monitoreo de código abierto que recopila y almacena métricas en datos de series temporales . Grafana es una aplicación web de código abierto para visualización y análisis interactivos. Le permite ingerir datos de una gran cantidad de fuentes de datos, consultarlos y mostrarlos en gráficos personalizables para facilitar el análisis. También es posible configurar alertas para que pueda recibir notificaciones rápida y fácilmente sobre comportamientos inesperados. Usarlos juntos le permite recopilar, monitorear, analizar y visualizar los datos de su instancia de MongoDB.

En este tutorial, configurará una base de datos MongoDB y la monitoreará con Grafana usando Prometheus como fuente de datos. Para lograr esto, configurará el exportador de MongoDB como un destino de Prometheus para que Prometheus pueda extraer las métricas de su base de datos y ponerlas a disposición de Grafana.


## 2. Requisitos previos

Para seguir este tutorial, necesitarás:

* Un servidor Ubuntu 20.04 con un usuario no root con sudoprivilegios y un firewall configurado con ufw.
* MongoDB instalado en el servidor Ubuntu 20.04.
* Grafana instalado en el servidor Ubuntu 20.04.

Para instalar Grafana, necesitará lo siguiente:

* Un nombre de dominio completamente registrado. Este tutorial se utiliza your_domain en todas partes. Puede comprar un nombre de dominio en Namecheap, obtener uno gratis en Freenom o utilizar el registrador de dominio de su elección.
* Los siguientes registros DNS configurados para su servidor. 
  - Un registro A your_domain que apunta a la dirección IP pública de su servidor.
  - Un registro A que apunta a la dirección IP pública de su servidor.www.your_domain
* Nginx configurado, incluido un bloque de servidor para su dominio.
* Un bloque de servidor Nginx con Let's Encrypt configurado.


## 3. Instalar y configurar Prometheus

Prometheus es un conjunto de herramientas de alertas y monitoreo de sistemas de código abierto que recopila y almacena métricas como datos de series de tiempo. Es decir, la información de las métricas se almacena con la marca de tiempo en la que se registró. En este paso, instalará Prometheus y lo configurará para que se ejecute como un servicio.


### 3.1 Instalación de Prometeo

Primero, necesitarás instalar Prometheus. Comience iniciando sesión en su servidor y actualizando las listas de paquetes de la siguiente manera:

``` 
sudo apt update
``` 

A continuación, creará los directorios de configuración y datos para Prometheus. Para crear un directorio de configuración llamado prometheus, ejecute el siguiente comando:

``` 
sudo mkdir -p /etc/prometheus
``` 

A continuación, cree los directorios de datos:

``` 
sudo mkdir -p /var/lib/prometheus
``` 

Después de crear los directorios, descargará el archivo de instalación comprimido. Los archivos de instalación de Prometheus vienen en binarios precompilados en archivos comprimidos. Para descargar Prometheus, visite la página de descarga.

Para descargar la versión 2.31.0, ejecute el siguiente comando y reemplace el número de versión según sea necesario:

``` 
wget https://github.com/prometheus/prometheus/releases/download/v2.31.0/prometheus-2.31.0.linux-amd64.tar.gz
```

Una vez descargado, extraiga el archivo tarball:

``` 
tar -xvf prometheus-2.31.0.linux-amd64.tar.gz
```

Después de extraer el archivo, navegue hasta la carpeta Prometheus:

``` 
cd prometheus-2.31.0.linux-amd64
``` 

Luego, mueva los archivos binarios prometheus y promtool al /usr/local/bin/directorio:

``` 
sudo mv prometheus promtool /usr/local/bin/
``` 

A continuación, moverá todos los archivos relacionados con Prometheus a una ubicación: /etc/prometheus/. Para mover los archivos de la consola en el consoles directorio y los archivos de la biblioteca en el console_libraries directorio, ejecute el siguiente comando:

``` 
sudo mv consoles/ console_libraries/ /etc/prometheus/
``` 

Los archivos de la consola y la biblioteca de la consola se utilizan para iniciar la GUI de Prometheus. Estos archivos se conservarán con los archivos de configuración para que puedan usarse al iniciar el servicio.

Finalmente, mueva el prometheus.yml archivo de configuración de la plantilla al /etc/prometheus/directorio:

``` 
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
``` 

prometheus.yml es el archivo de configuración de la plantilla donde configurará el puerto para Prometheus y qué archivos usar al iniciar el servicio.

Para verificar la versión de Prometheus instalada, ejecute el comando:

``` 
prometheus --version
``` 

Recibirá un resultado similar a este:

``` 
Output
prometheus, version 2.31.0 (branch: HEAD, revision: b41e0750abf5cc18d8233161560731de05199330)
  build user:       root@0aa1b7fc430d
  build date:       20220714-15:13:18
  go version:       go1.18.4
  platform:         linux/amd64
```

En esta sección, instaló Prometheus y verificó su versión. A continuación, lo iniciará como un servicio.


### 3.2 Ejecutando Prometheus como servicio

Ahora que ha instalado Prometheus, lo configurará para que se ejecute como un servicio.

Antes de crear el archivo del sistema para lograr esto, deberá crear un grupo y un usuario de Prometheus. Necesitará un usuario dedicado con acceso de propietario a los directorios necesarios. Para crear un prometheus grupo, ejecute el siguiente comando:

``` 
sudo groupadd --system prometheus
```

A continuación, crea un prometheus usuario y asígnalo al prometheus grupo que acabas de crear:

``` 
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
``` 

Cambie la propiedad y los permisos del directorio de la siguiente manera para que el usuario dedicado tenga los permisos correctos:

``` 
sudo chown -R prometheus:prometheus /etc/prometheus/ /var/lib/prometheus/
sudo chmod -R 775 /etc/prometheus/ /var/lib/prometheus/
``` 

A continuación, creará el archivo de servicio para ejecutar Prometheus como servicio. Usando nanoo su editor de texto favorito, cree un systemd archivo de servicio llamado prometheus.service:

``` 
sudo nano /etc/systemd/system/prometheus.service
``` 

Agregue las siguientes líneas de código:


``` 
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
``` 

Con este código, configura Prometheus para usar los archivos enumerados en el ExecStart bloque para ejecutar el servicio. El archivo de servicio indica systemd que ejecute Prometheus como prometheus usuario con el archivo de configuración /etc/prometheus/prometheus.yml y que almacene sus datos en el /var/lib/prometheus directorio. También configura Prometheus para que se ejecute en el puerto 9090.

Guarde y cierre su archivo. Si usa nano, presione CTRL+X y luego Y.

Ahora, inicie el servicio Prometheus:

``` 
sudo systemctl start prometheus
``` 

Habilite el servicio Prometheus para que se ejecute al inicio:

``` 
sudo systemctl enable prometheus
``` 

Puede verificar el estado del servicio usando el siguiente comando:

``` 
sudo systemctl status prometheus
```

El resultado confirmará que el servicio es active (running):

``` 
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-08-05 18:06:05 UTC; 13s ago
   Main PID: 7177 (prometheus)
      Tasks: 6 (limit: 527)
     Memory: 21.0M
     CGroup: /system.slice/prometheus.service
             └─7177 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/ --web.console.template>
```

Para acceder a Prometheus, inicie su navegador y visite la dirección IP de su servidor seguida del puerto 9090: .http://your_server_ip:9090.

**Nota:** Para acceder a la consola web de Prometheus, es posible que deba permitir el puerto 9090en su servidor. Para verificar su conjunto de reglas UFW actual, ejecute el siguiente comando:

``` 
sudo ufw status
``` 

Si el puerto 9090 aún no está permitido, puede agregarlo usando el siguiente comando:

``` 
sudo ufw allow 9090
``` 

Ahora puede acceder a la consola web de Prometheus.

En este paso, instaló Prometheus y lo configuró para que se ejecute como un servicio. A continuación, vinculará su base de datos MongoDB a Prometheus utilizando el exportador MongoDB.


## 4. Configurar el exportador MongoDB

Prometheus funciona raspando objetivos para recopilar métricas. En este paso, instalará el exportador de MongoDB y lo configurará como destino de Prometheus para que Prometheus pueda recopilar los datos de su instancia de MongoDB.

### 4.1 Instalación del exportador MongoDB

En esta sección, instalará el exportador MongoDB. Primero, cree un directorio para el exportador y navegue hasta él:

``` 
mkdir mongodb-exporter
cd mongodb-exporter
``` 

El exportador MongoDB se puede descargar desde Github. El exportador viene como un archivo binario en un archivo, pero lo configurará como un servicio. Descargue el archivo binario con el siguiente comando:

```
wget https://github.com/percona/mongodb_exporter/releases/download/v0.7.1/mongodb_exporter-0.7.1.linux-amd64.tar.gz
```

A continuación, extraiga el archivo descargado en su carpeta actual:

``` 
tar xvzf mongodb_exporter-0.7.1.linux-amd64.tar.gz
``` 

Finalmente, mueva el mongodb_exporter binario a usr/local/bin/:

``` 
sudo mv mongodb_exporter /usr/local/bin/
``` 

En esta sección, instaló el exportador MongoDB. A continuación, habilitará la autenticación MongoDB y creará un usuario para monitorear.

### 4.2 Habilitar la autenticación de MongoDB

En esta sección, configurará la autenticación de MongoDB para el exportador de MongoDB y creará un usuario para monitorear las métricas del clúster.

Comience conectándose a su instancia de MongoDB con mongo:

``` 
mongo
``` 

Creará una cuenta de administrador para su exportador con la función de monitor de clúster. Cambiar a la adminbase de datos:

``` 
use admin
``` 

Después de cambiar a la admin base de datos, cree un usuario con el clusterMonitor rol:

``` 
db.createUser({user: "test",pwd: "testing",roles: [{ role: "clusterMonitor", db: "admin" },{ role: "read", db: "local" }]})
```

Recibirá el siguiente resultado:

``` 
Successfully added user: {
        "user" : "test",
        "roles" : [
                {
                        "role" : "clusterMonitor",
                        "db" : "admin"
                },
                {
                        "role" : "read",
                        "db" : "local"
                }
        ]
}
```

Después de crear el usuario, salga del shell de MongoDB:

``` 
exit
``` 

A continuación, configure su variable de entorno URI de MongoDB con las credenciales de autenticación adecuadas:

``` 
export MONGODB_URI=mongodb://test:testing@localhost:27017
``` 

Configure MONGODB_URI para especificar la mongodb instancia que utiliza las credenciales de autenticación que configuró anteriormente (el test usuario y testing la contraseña). 27017 es el puerto predeterminado para una mongodb instancia. Cuando configura la variable de entorno, tiene prioridad sobre el perfil almacenado en el archivo de configuración.

Para verificar que la variable de entorno URI de MongoDO esté configurada correctamente, ejecute el siguiente comando:

``` 
env | grep mongodb
``` 

Recibirá el siguiente resultado:

``` 
MONGODB_URI=mongodb://mongodb_exporter:password@localhost:27017
``` 

En esta sección, creó un usuario de MongoDB con el clusterMonitor rol, que ayuda a monitorear las métricas del clúster. A continuación, configurará el exportador de MongoDB para que se ejecute como un servicio.


### 4.3 Creando un servicio para el exportador MongoDB

En esta sección, creará un archivo de sistema para el exportador MongoDB y lo ejecutará como un servicio.

Navegue /lib/systemd/system y cree un nuevo archivo de servicio para el exportador utilizando nano su editor de texto favorito:

``` 
cd /lib/systemd/system/
sudo nano mongodb_exporter.service
``` 

Pegue la siguiente configuración en su archivo de servicio:

``` 
/lib/systemd/system/mongodb_exporter.service
[Unit]
Description=MongoDB Exporter
User=prometheus

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/mongodb_exporter

[Install]
WantedBy=multi-user.target
``` 

Este archivo de servicio indica systemd que se ejecute el exportador MongoDB como un servicio bajo el prometheus usuario. ExecStart ejecutará el mongodb_exporter binario desde usr/local/bin/. 

Guarde y cierre su archivo.

A continuación, reinicie el demonio de su sistema para recargar los archivos de la unidad:

``` 
sudo systemctl daemon-reload
``` 

Ahora, inicia tu servicio:

``` 
sudo systemctl start mongodb_exporter.service
``` 

Para verificar el estado del servicio exportador de MongoDB, ejecute el siguiente comando:

``` 
sudo systemctl status mongodb_exporter.service
``` 

El resultado confirmará que el servicio es active (running):

``` 
● mongodb_exporter.service - MongoDB Exporter
     Loaded: loaded (/lib/systemd/system/mongodb_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-08-05 18:18:38 UTC; 1 weeks 3 days ago
   Main PID: 7352 (mongodb_exporte)
      Tasks: 5 (limit: 527)
     Memory: 14.2M
     CGroup: /system.slice/mongodb_exporter.service
             └─7352 /usr/local/bin/mongodb_exporter
```

Para asegurarse de que todo funcione como se esperaba, navegue hasta la raíz del proyecto y ejecute un comando curl en el puerto 9216, que es donde se ejecuta el exportador:

``` 
cd ~
sudo curl http://localhost:9216/metrics
``` 

El resultado será largo y contendrá líneas similares a esta:

``` 
Output
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 11
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 1.253696e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 1.253696e+06
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 3054
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 2866
# HELP go_memstats_gc_sys_byte
.
.
.
# HELP mongodb_asserts_total The asserts document reports the number of asserts on the database. While assert errors are typically uncommon, if there are non-zero values for the asserts, you should check the log file for the mongod process for more information. In many cases these errors are trivial, but are worth investigating.
# TYPE mongodb_asserts_total counter
mongodb_asserts_total{type="msg"} 0
mongodb_asserts_total{type="regular"} 0
mongodb_asserts_total{type="rollovers"} 0
mongodb_asserts_total{type="user"} 19
mongodb_asserts_total{type="warning"} 0
# HELP mongodb_connections The connections sub document data regarding the current status of incoming connections and availability of the database server. Use these values to assess the current load and capacity requirements of the server
# TYPE mongodb_connections gauge
mongodb_connections{state="available"} 51198
mongodb_connections{state="current"} 2
# HELP mongodb_connections_metrics_created_total totalCreated provides a count of all incoming connections created to the server. This number includes connections that have since closed
# TYPE mongodb_connections_metrics_created_total counter
mongodb_connections_metrics_created_total 6
# HELP mongodb_exporter_build_info A metric with a constant '1' value labeled by version, revision, branch, and goversion from which mongodb_exporter was built.
# TYPE mongodb_exporter_build_info gauge
mongodb_exporter_build_info{branch="v0.7.1",goversion="go1.11.10",revision="3002738d50f689c8204f70f6cceb8150b98fa869",version="0.7.1"} 1
# HELP mongodb_exporter_last_scrape_duration_seconds Duration of the last scrape of metrics from MongoDB.
# TYPE mongodb_exporter_last_scrape_duration_seconds gauge
mongodb_exporter_last_scrape_duration_seconds 0.003641888
# HELP mongodb_exporter_last_scrape_error Whether the last scrape of metrics from MongoDB resulted in an error (1 for error, 0 for success).
# TYPE mongodb_exporter_last_scrape_error gauge
mongodb_exporter_last_scrape_error 0
.
.
.
...
``` 

El resultado confirma que el exportador de MongoDB está recopilando métricas, como la mongodb versión metrics-document y los detalles de las conexiones.

En esta sección, configurará el exportador de MongoDB como un servicio y recopilará métricas de MongoDB. A continuación, configurará el exportador como destino para Prometheus.


## 4.4 Configuración del exportador MongoDB como destino de Prometheus

En esta sección, configurará el exportador MongoDB como destino de Prometheus. Navegue hasta el directorio que contiene su archivo de configuración de Prometheus:

``` 
cd /etc/prometheus/
``` 

Usando nano o su editor de texto favorito, abra el archivo para editarlo:

```
sudo nano prometheus.yml
```

Agregue el exportador MongoDB como destino copiando las líneas resaltadas en su archivo:

``` 
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    static_configs:
            - targets: ["localhost:9090", "localhost:9216"]
``` 

9216es el puerto predeterminado para el exportador MongoDB.

Guarde y cierre su archivo.

Después de agregar el objetivo, reinicie Prometheus:

```
sudo systemctl restart prometheus
``` 

Navegue para http://localhost:9090/targets verificar que Prometheus esté eliminando su exportador recién agregado.

Accederás a una lista de objetivos de Prometheus.

El 9090 punto final es que Prometheus se rasque a sí mismo. El 9216 punto final es el exportador de MongoDB, que confirma que su configuración funciona como se esperaba.

En este paso, instaló el exportador MongoDB y lo configuró como un destino de Prometheus para recopilar métricas. A continuación, creará un panel de MongoDB en la consola web de Grafana para ver y analizar estas métricas.

## 5. Creación de un panel de MongoDB en Grafana

En este paso, creará un panel para visualizar sus datos de MongoDB en Grafana. Para lograr esto, agregará Prometheus como fuente de datos en Grafana e importará un panel de MongoDB desde Percona. Percona proporciona varios paneles para MongoDB, que puede encontrar en los documentos del producto Percona. Para este tutorial, importará el panel de descripción general de MongoDB a su instancia de Grafana. Para comenzar, configurará Prometheus como fuente de datos de Grafana.

Como parte de los requisitos previos, instaló y aseguró Grafana. Navegue a su instancia de Grafana en your_domain:3000 e inicie sesión con las credenciales que creó durante los requisitos previos.

En el panel izquierdo, haga clic en el ícono de ajustes para Configuración y luego seleccione Fuentes de datos.

Haga clic en Agregar fuente de datos. Luego seleccione Prometeo. En la siguiente pantalla, configurará los ajustes para su fuente de datos Prometheus. 

En el campo URL, proporcione la URL de su instancia de Prometheus:

``` 
http://your_server_ip:9090/
``` 

Haga clic en Guardar y probar en la parte inferior de la pantalla. Ahora se agrega Prometheus como fuente de datos para Grafana.

A continuación, importará el panel de descripción general de MongoDB para Grafana. Puede importar el panel cargando un archivo JSON o importando una ID del panel, que puede encontrar en los documentos del producto Grafana para paneles. Aquí, utilizará el ID del panel para importar el panel.

En el menú de la izquierda, haga clic en el ícono más para Crear y seleccione Importar. Desde allí, debería ser llevado a la página Importar.

Aquí, puede cargar el archivo JSON del panel o pegar el ID del panel de Grafana.

Agregue el ID del panel de Grafana, que puede encontrar en la página de Grafana para el panel de descripción general de MongoDB:

https://grafana.com/grafana/dashboards/7353

Hay muchos paneles disponibles. Puede encontrar más información visitando la página de Grafana en paneles.

Después de agregar la ID del panel, haga clic en Cargar.

Ahora se abrirá una página de Opciones, donde puede proporcionar un nombre para el panel, seleccionar la carpeta para el panel y seleccionar una fuente de datos. Puede dejar el panel y los nombres de las carpetas como predeterminados. Para la fuente de datos, elija Prometheus. Una vez que hayas completado las opciones, haz clic en Importar.

Se creará el panel de control.

Su panel mostrará actualizaciones en tiempo real de su base de datos MongoDB, incluidas operaciones de comandos, conexiones, cursores, operaciones de documentos y operaciones en cola.


## 6. Conclusión

En este artículo, configurará un panel de Grafana para monitorear las métricas de Prometheus para su base de datos MongoDB, lo que le permite monitorear su base de datos a través de un panel GUI. Primero, instaló Prometheus y configuró el exportador MongoDB. Luego, agregó Prometheus como fuente de datos en Grafana, donde podía monitorear y visualizar datos desde su instancia de MongoDB.





