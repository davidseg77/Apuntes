# Info extraida del curso Monitorización con Grafana y Prometheus

## 1. Introducción

### 1.1 Instalación de Prometheus

1. Crear el usuario prometheus
 
``` 
$ sudo useradd --no-create-home --shell /bin/false prometheus
``` 
 
2. Crear los directorios para prometheus:
 
``` 
$ sudo mkdir /etc/prometheus

$ sudo mkdir /var/lib/prometheus

$ sudo chown prometheus:prometheus /etc/prometheus

$ sudo chown prometheus:prometheus /var/lib/prometheus
``` 

3. Descargar Prometheus (descargar última versión disponible):
 
``` 
$ curl -LO https://github.com/prometheus/prometheus/releases/download/v2.28.0/prometheus-2.28.0.linux-amd64.tar.gz
``` 
 
4. Descomprimir prometheus:

```  
$ tar xvf prometheus-2.28.0.linux-amd64.tar.gz
``` 
 
5. Copiar prometheus a los directorios de binarios:
 
``` 
$ sudo cp prometheus-2.28.0.linux-amd64/prometheus /usr/local/bin/

$ sudo cp prometheus-2.28.0.linux-amd64/promtool /usr/local/bin/

$ sudo chown prometheus:prometheus /usr/local/bin/prometheus

$ sudo chown prometheus:prometheus /usr/local/bin/promtool

$ sudo cp -r prometheus-2.28.0.linux-amd64/consoles /etc/prometheus

$ sudo cp -r prometheus-2.28.0.linux-amd64/console_libraries /etc/prometheus

$ sudo chown -R prometheus:prometheus /etc/prometheus/consoles

$ sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries

$ rm -rf prometheus-2.28.0.linux-amd64.tar.gz prometheus-2.28.0.linux-amd64
``` 
 
6. Editar fichero de prometheus con el comando:
 
``` 
$ sudo vim /etc/prometheus/prometheus.yml
``` 
 
7. Y añadir la siguiente configuración:
 
```  
global:

scrape_interval: 15s

scrape_configs:

  - job_name: 'prometheus'

    scrape_interval: 5s

    static_configs:

      - targets: ['localhost:9090']
``` 
 
8. Cambiar el propietario del archivo de configuración prometheus.yml a prometheus:
 
```  
$ sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
``` 
 
9. Crear el servicio de Prometheus:
 
``` 
$ sudo vim /etc/systemd/system/prometheus.service

 
[Unit]

Description=Prometheus

Wants=network-online.target

After=network-online.target

 

[Service]

User=prometheus

Group=prometheus

Type=simple

ExecStart=/usr/local/bin/prometheus \

    --config.file /etc/prometheus/prometheus.yml \

    --storage.tsdb.path /var/lib/prometheus/ \

    --web.console.templates=/etc/prometheus/consoles \

    --web.console.libraries=/etc/prometheus/console_libraries

 

[Install]

WantedBy=multi-user.target
``` 

10. Arrancar y habilitar el servicio de Prometheus
 
``` 
$ sudo systemctl daemon-reload

$ sudo systemctl start prometheus

$ sudo systemctl status prometheus

$ sudo systemctl enable prometheus
``` 

### 1.2 Instalación de Grafana

1. Instalar las dependencias de Grafana:

``` 
$ sudo apt install -y apt-transport-https

$ sudo apt install -y software-properties-common wget
``` 
 
2. Instalar repositorio de Grafana:

```
$ wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

$ sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

$ sudo apt update
``` 
 
3. Instalar Grafana:

``` 
$ sudo apt install grafana
``` 
 
4. Iniciar/Habilitar servicio Grafana:

```
$ sudo systemctl daemon-reload

$ sudo systemctl enable grafana-server

$ sudo systemctl start grafana-server

$ sudo systemctl status grafana-server
``` 


## 2. Recogiendo métricas con Prometheus

### 2.1 Exporters de métricas

Los exporters son librerías que se utilizan para recoger métricas del sistema, servicios, bases de datos, etc. y publicarlos en un endpoint HTTP, del que posteriormente Prometheus lee.

Parámetros de configuración de Jobs (exporters) en Prometheus:

* job_name: Nombre del job (sirve para diferenciarlos)
* scrape_interval: Intervalo que Prometheus leerá métricas en este job
* scrape_timeout: Si el exporter tarda más de 1 minuto en responder con las métricas salta el timeout
* metric_path: El endpoint en el que encontramos las métricas  (default /metrics)
* targets: Los servidores de los que vamos a leer las métricas


### 2.2 Instalar exporters

**Instalar NODE EXPORTER**

1. Crear usuario node exporter

``` 
$ sudo useradd --no-create-home --shell /bin/false node_exporter
``` 
 
2. Descargar node exporter

``` 
$ curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
``` 
 
3. Descomprimir y copiar binario

``` 
$ tar xvf node_exporter-1.1.2.linux-amd64.tar.gz

$ sudo cp node_exporter-1.1.2.linux-amd64/node_exporter /usr/local/bin

$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

$ rm -rf node_exporter-1.1.2.linux-amd64.tar.gz node_exporter-1.1.2.linux-amd64
``` 
 
4. Crear servicio node exporter

``` 
$ vim /etc/systemd/system/node_exporter.service

 

[Unit]

Description=Node Exporter

Wants=network-online.target

After=network-online.target

 

[Service]

User=node_exporter

Group=node_exporter

Type=simple

ExecStart=/usr/local/bin/node_exporter

 

[Install]

WantedBy=multi-user.target
``` 
 
5. Arrancar/Habilitar servicio node exporter

``` 
$ sudo systemctl daemon-reload

$ sudo systemctl start node_exporter

$ sudo systemctl status node_exporter

$ sudo systemctl enable node_exporter
``` 


### 2.3 Configurando Prometheus con los exporters

**Configurar NODE_EXPORTER en PROMETHEUS**

1. Editar configuracion de Prometheus

``` 
$ sudo vim /etc/prometheus/prometheus.yml

 - job_name: 'node_exporter'

    scrape_interval: 5s

    static_configs:

      - targets: ['localhost:9100']
``` 
 
2. Reiniciar servicio prometheus

```
$ sudo systemctl restart prometheus

$ sudo systemctl status prometheus
``` 


## 3. Monitorizando con Grafana y Prometheus

### 3.1 Crear un dashboard personalizado 

**Desarrollando un Dashboard:**

* CPU USAGE

- Query A:

``` 
(((count(count(node_cpu_seconds_total{instance="$node"}) by (cpu))) - avg(sum by (mode)(rate(node_cpu_seconds_total{mode='idle',instance="$node"}[$__rate_interval])))) * 100) / count(count(node_cpu_seconds_total{instance="$node"}) by (cpu))
``` 

* RAM USAGE

- Query A:

``` 
100 - ((node_memory_MemAvailable_bytes{instance="$node"} * 100) / node_memory_MemTotal_bytes{instance="$node"})
``` 
 
* CPU GRÁFICO

- Query A: Busy system

``` 
sum by (instance)(rate(node_cpu_seconds_total{mode="system",instance="$node"}[$__rate_interval])) * 100
``` 

- Query B: Busy User

``` 
sum by (instance)(rate(node_cpu_seconds_total{mode='user',instance="$node"}[$__rate_interval])) * 100
```

- Query C: Busy Iowait

``` 
sum by (instance)(rate(node_cpu_seconds_total{mode='iowait',instance="$node"}[$__rate_interval])) * 100
```

- Query D: Busy IRQs

``` 
sum by (instance)(rate(node_cpu_seconds_total{mode=~".*irq",instance="$node"}[$__rate_interval])) * 100
``` 
 
- Query E: Busy Other

``` 
sum by (rate(node_cpu_seconds_total{mode!='idle',mode!='user',mode!='system',mode!='iowait',mode!='irq',mode!='softirq',instance="$node"}[$__rate_interval])) * 100
``` 
 
- Query F: Idle

``` 
sum by (mode)(rate(node_cpu_seconds_total{mode='idle',instance="$node"}[$__rate_interval])) * 100
``` 

* GRÁFICO MEMORIA

- Query A: Ram total

``` 
node_memory_MemTotal_bytes{instance="$node"}
```

- Query B: Ram Used

```
node_memory_MemTotal_bytes{instance="$node"} - node_memory_MemFree_bytes{instance="$node"} - (node_memory_Cached_bytes{instance="$node"} + node_memory_Buffers_bytes{instance="$node"})
```
 
- Query C: RAM Cache + Buffer

```
node_memory_Cached_bytes{instance="$node"} + node_memory_Buffers_bytes{instance="$node"}
``` 
 
- Query D: Ram free

```
node_memory_MemFree_bytes{instance="$node"}
```
 
- Query E: SWAP Used

``` 
(node_memory_SwapTotal_bytes{instance="$node"} - node_memory_SwapFree_bytes{instance="$node"})
``` 


## 4. Configurar sistema de alertas vía e-mail

### 4.1 Configurar alertas en Grafana

Como crear alertas en Grafana:

* Primero creamos un AlertChannel que sería el canal que Grafana va a utilizar para enviar las alertas en caso de que hubiera alguna.

 * Creamos la regla, que queremos validar y en caso de que la regla se cumpla Grafana enviará alertas usando el canal definido.

Para ello, vamos al archivo grafana.ini dentro de /etc/grafana y dentro del módulo smtp habilitamos el host del mail e indicamos su usuario. Indicamos la contraseña, el from_address y el from_name. Reiniciamos el servicio de grafana-server y ya tendremos configurado nuestro email para recibir alertas.

Dentro de la configuración de Grafana, vamos a Alerting, a los canales de notificación. Hecho esto, dentro del dashboard en cuestión, damos en Editar. Hay que decir que no pueden crearse alertas cuando la regla de métrica lleva incorporada variables. Vamos dentro de Editar a la pestaña de Alertas y creamos una. En Notifications indicamos donde enviar las alertas y con qué mensaje. 


