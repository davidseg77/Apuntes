# Developing Event-driven Applications with Apache Kafka and Red Hat AMQ Streams

### Creación de tópicos en Kafka

**Creación de tópicos de Kafka con el operador de tópicos de AMQ Streams**

La distribución AMQ Streams Kafka proporciona un operador de tópicos para la gestión de tópicos Kafka. El operador de tópicos le permite crear y eliminar tópicos de Kafka mediante el uso de archivos YAML de recursos personalizados (CR).

Puede crear un tópico usando el tipo KafkaTopic, por ejemplo:

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: "example-topic"
  labels:
    strimzi.io/cluster: "example-cluster"
spec:
  partitions: 3
  replicas: 1
  config:
    retention.ms: 604800000 # 7 days
    segment.bytes: 1073741824
```

**Definición de particiones Kafka**

Cada tópico se divide en una o más particiones. Kafka distribuye particiones a los brókers. Las particiones almacenan físicamente los registros en el sistema de archivos. Cada partición se puede replicar a múltiples brókers para obtener resiliencia contra las fallas del bróker.

Por ejemplo, considere el siguiente tópico orders (pedidos) dividido en 3 particiones, cada partición tiene una réplica:

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: "orders"
  labels:
    strimzi.io/cluster: "example-cluster"
spec:
  partitions: 3
  replicas: 1
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
```

**Replicación de particiones**

Cada tópico puede especificar un factor de replicación. Cuando configura un factor de replicación, el nodo de bróker se convierte en el leader (líder) para cada una de las particiones del tópico.

El líder de la partición replica la partición en un número especificado de nodos de brókers del mismo nivel. Los agentes del mismo nivel que contienen una réplica de la partición se llaman followers (seguidores).

Cuando los seguidores contienen los mismos registros de partición que el líder, se convierten en in-sync replicas (ISR).

Por ejemplo, considere el tópico anterior con el factor de replicación 3:

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: "orders"
  labels:
    strimzi.io/cluster: "example-cluster"
spec:
  partitions: 3
  replicas: 3
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
```

Esto significa que cada una de las tres particiones se replica en tres nodos.

### Instalación de Kafka Streams

Las aplicaciones de Kafka Streams requieren las siguientes librerías del grupo org.apache.kafka:

kafka-streams: API de Kafka Streams

kafka-clients: API de productor y consumidor de Kafka, necesarias para interactuar con Kafka

Para instalar Kafka Streams en un proyecto Maven, debe añadir la dependencia kafka-streams a su archivo pom.xml.

```
<dependencies>
    ...output omitted...
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-streams</artifactId>
        <version>2.8.0</version>
    </dependency>
</dependency>
```

**Extensión Quarkus**

En los proyectos Quarkus, la forma preferida de usar Kafka Streams es la extensión Quarkus de kafka-streams.

```
[user@host ~] mvn quarkus:add-extension -Dextensions="kafka-streams"
```

Al instalar esta extensión, se añade la siguiente dependencia de Maven al archivo pom.xml del proyecto:

```
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-kafka-streams</artifactId>
</dependency>
```

**Creación de una tubería de procesamiento de flujo básica**

Las aplicaciones de Kafka Streams se organizan de acuerdo con los siguientes pasos lógicos:

Establecer los parámetros de configuración de Kafka y Kafka Streams.

Especificar la topología de procesamiento.

Cree la topología.

Inicializar e iniciar el cliente de Kafka Streams.

El siguiente es un ejemplo de una aplicación muy básica de Kafka Streams, que ilustra los pasos anteriores:

```
...output omitted...

import java.util.Properties;

import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.Consumed;
import org.apache.kafka.streams.kstream.Produced;

...output omitted...

Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "your-application");
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-server-host:kafka-server-port");
...more props omitted...
StreamsConfig config = new StreamsConfig(settings); 1

StreamsBuilder builder = new StreamsBuilder(); 2

builder 3
    .stream(
        "topic-name",
        Consumed.with(keySerde, valueSerde)
    ) 4
    .foreach((key, value) ->
        System.out.println("Received payment: " + value)
    ) 5
    .to(
        "output-topic",
        Produced.with(keySerde, valueSerde)
    ); 6

Topology topology = builder.build(); 7

KafkaStreams streams = new KafkaStreams(topology, config); 8

streams.start(); 9
```

1

Defina los parámetros de configuración de Kafka y Kafka Streams. El objeto StreamsConfig contiene la definición de todos los parámetros de configuración. Se requieren application.id y bootstrap.servers. Para obtener más detalles, consulte Referencia de los parámetros de configuración.

2

Cree una instancia de StreamsBuilder.

3

Use la instancia StreamsBuilder para comenzar a crear la topología de procesamiento.

4

Especifique el tópico Kafka para consumir el flujo y los deserializadores necesarios para consumir la clave y el valor de cada mensaje.

5

Procese el flujo. Este ejemplo en particular usa la transformación foreach para imprimir el valor de cada mensaje.

6

Envíe el resultado a otro tópico de Kafka.

7

Cree la topología.

8

Cree el cliente Kafka Streams con la topología y las propiedades de configuración.

9

Inicie el cliente de Kafka Streams.

**Extensión Kafka Streams Quarkus**

Si usa la extensión Quarkus Kafka Streams, no necesita ocuparse del cliente KafkaStreams.

Quarkus se encarga de lo siguiente:

Inicialización, creación y detención del cliente KafkaStreams.

Lectura de la configuración en el archivo application.properties e inserción de parámetros de configuración en el cliente KafkaStreams.

Para crear una tubería de procesamiento, debe crear un método productor de CDI que devuelva una topología, como muestra el siguiente ejemplo:

```
@Produces
public Topology buildTopology() {

    StreamsBuilder builder = new StreamsBuilder();

    builder
        .stream(
            "input-topic",
            Consumed.with(keySerde, valueSerde)
        )
        .filter((key, value) -> value > 0)
        .to(
            "output-topic",
            Produced.with(keySerde, valueSerde)
        );

    Topology topology = builder.build();
    return topology;
}
```

Finalmente, para establecer la configuración de Kafka Streams, establezca los siguientes valores de configuración en el archivo applications.properties de su aplicación Quarkus.

```
quarkus.kafka-streams.application-id=your-app-id
quarkus.kafka-streams.bootstrap-servers = kafka-server-host:kafka-server-port
quarkus.kafka-streams.topics = input-topic,output-topic

...output omitted...
```

Los parámetros quarkus.kafka-streams.application-id y quarkus.kafka-streams.bootstrap-servers no son necesarios. Si no está configurado, por defecto es el nombre de la aplicación Quarkus y localhost:9012, respectivamente.

El parámetro quarkus.kafka-streams.topics es obligatorio y evita que Quarkus arranque el motor de Kafka Streams si alguno de los tópicos de Kafka especificados no está listo.

### Configuración del clúster de Kafka Connect

En AMQ Streams en Red Hat OpenShift Container Platform (RHOCP), los clústeres de Kafka Connect se configuran usando el recurso personalizado KafkaConnect. La siguiente definición de YAML es un ejemplo básico de una configuración de clúster de Kafka Connect con dos réplicas.

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnect 1
metadata:
  annotations:
    strimzi.io/use-connector-resources: 'true' 2
  name: my-connect-cluster 3
spec:
  bootstrapServers: 'my-cluster-kafka-bootstrap:9092' 4
  image: 'quay.io/redhattraining/connect-cluster:latest' 5
  config: 6
    config.storage.topic: my-connect-cluster-configs 7
    offset.storage.topic: my-connect-cluster-offsets 8
    status.storage.topic: my-connect-cluster-status 9
    config.storage.replication.factor: 1
    offset.storage.replication.factor: 1
    status.storage.replication.factor: 1
  replicas: 2 10
  version: 2.8.0 11
```

1

Nombre del recurso de Kafka Connect que gestiona el operador de AMQ Streams.

2

Active el escaneo de recursos de KafkaConnector y cree conectores sin usar la API de REST.

3

Nombre del clúster de Kafka Connect.

4

Dirección bootstrap del clúster de Kafka a la que se conecta el clúster de Kafka Connect.

5

Dirección de registro para la imagen personalizada de Kafka Connect. AMQ Streams crea esta imagen y la envía al registro.

6

Configuraciones para el clúster de Kafka Connect.

7

El tópico en el que se almacenará el estado del conector y de la configuración de la tarea.

8

El tópico en el que se almacenará el estado de la compensación del conector.

9

El tópico en el que se almacenarán las actualizaciones de estado para los conectores y las tareas

10

conteo de réplicas para el clúster de Kafka Connect.

11

Versión de Kafka que se usa en el clúster de Kafka Connect.

La creación manual de una imagen no es la única forma de crear un clúster de Kafka Connect. Tiene dos opciones para preparar una imagen de clúster de Kafka Connect:

* Crear manualmente una imagen.

* Usar el operador AMQ Streams para crear la imagen.

### Creación de una imagen de clúster con el operador AMQ Streams

Para hacer que AMQ Streams cree una imagen, debe definir el valor KafkaConnect.spec.build en lugar de KafkaConnect.spec.image. La siguiente configuración de YAML es un ejemplo de un recurso de clúster KafkaConnect que está configurado para crear las imágenes en RHOCP.

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnect
metadata:
  annotations:
    strimzi.io/use-connector-resources: 'true'
  name: my-connect-cluster
spec:
  bootstrapServers: my-cluster-kafka-bootstrap:9092
  build: 1
    output: 2
      image: quay.io/redhattraining/connect-cluster:latest 3
      pushSecret: my-connect-cluster-push-secret 4
      type: docker 5
    plugins: 6
    - name: debezium-postgres-connector
      artifacts:
      - type: tgz 7
        url: https://a.plugin.url/debezium-connector-postgres-1.3.1.Final-plugin.tar.gz 8
        sha512sum: 962a12151bdf9a5a30627...f035a0447cb820077af00c03 9
    - name: camel-elasticsearch-rest-kafka-connector
      artifacts:
      - type: tgz
        url: https://a.plugin.url/camel-elasticsearch-rest-kafka-connector-0.10.0-package.tar.gz
  config:
    config.storage.topic: my-connect-cluster-configs
    offset.storage.topic: my-connect-cluster-offsets
    status.storage.topic: my-connect-cluster-status
    config.storage.replication.factor: 1
    offset.storage.replication.factor: 1
    status.storage.replication.factor: 1
  replicas: 2
  version: 2.8.0
```

1

En lugar de un valor image, debe usar un valor build en este caso.

2

En output se definen la salida de la imagen y sus detalles.

3

La dirección de registro para la imagen personalizada de Kafka Connect que el operador AMQ Streams crea y envía.

4

Envíe el nombre del secreto para enviar la imagen al registro definido. Debería crear este secreto de envío por adelantado.

5

El tipo de salida puede ser docker o imagestream. Mientras que el tipo de salida docker define una imagen externa que se va a crear, imagestream define una imagen interna de RHOCP a la que se accede a través de ImageStream.

6

Lista de complementos (plug-ins) de conectores y sus artefactos para agregar a la nueva imagen de contenedor. Cada complemento (plug-in) debe estar configurado con al menos un artefacto.

7

Tipo de artefacto para el complemento (plug-in). Este campo puede tener valores tgz, zip y jar.

8

URL desde la que se descarga el artefacto del complemento (plug-in).

9

(Opcional) Suma de comprobación de SHA-512 para verificar el artefacto de complemento (plug-in).

### Uso del recurso KafkaConnector

Con el recurso personalizado KafkaConnector, puede definir el estado deseado de un conector en un archivo YAML e implementar el recurso en OpenShift para crear las instancias del conector. Debe implementar los recursos personalizados de KafkaConnector en el mismo espacio de nombres que su clúster de Kafka Connect.

En el siguiente fragmento se muestra cómo definir una instancia de conector.

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnector
metadata:
  name: my-source-connector 1
  labels:
    strimzi.io/cluster: my-connect-cluster 2
spec:
  class: org.apache.kafka.connect.file.FileStreamSourceConnector 3
  tasksMax: 2 4
  config: 5
    file: "/opt/kafka/LICENSE"
    topic: my-topic
```

1

Nombre único asignado al conector.

2

Nombre del clúster de Kafka Connect con el que se va a enlazar.

3

Nombre o alias completamente calificado de la clase que implementa la lógica del conector.

4

Cantidad máxima de tareas que el conector puede crear.

5

Configuración del conector como pares de clave-valor. Cada implementación de conector puede tener diferentes opciones de configuración.

**Ejemplo**

Cree un conector de origen GitHub usando el recurso personalizado KafkaConnector.

Abra el archivo resources/github-source-connector.yaml y configure el usuario de GitHub y el token de acceso.

El contenido del archivo debería verse de la siguiente manera:

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnector
metadata:
  labels:
    strimzi.io/cluster: my-connect-cluster
  name: github-source-connector
spec:
  class: org.apache.camel.kafkaconnector.github.CamelGithubSourceConnector 1
  config:
    camel.source.endpoint.oauthToken: YOUR_GITHUB_TOKEN 2
    camel.source.endpoint.repoName: ad482-connectors 3
    camel.source.endpoint.repoOwner: YOUR_GITHUB_USERNAME 4
    camel.source.path.branchName: main 5
    camel.source.path.type: event 6
    key.converter: org.apache.kafka.connect.json.JsonConverter 7
    value.converter: org.apache.kafka.connect.json.JsonConverter 8
    topics: github-events 9
  tasksMax: 1
```

1

Clase de conector que implementa la integración con GitHub.

2

Configuración específica para el conector origen de Camel. Define el token de acceso que se va a usar.

3

Configuración específica para el conector origen de Camel. Define el repositorio de GitHub.

4

Configuración específica para el conector origen de Camel. Define el usuario del repositorio.

5

Configuración específica para el conector origen de Camel. Define el nombre de la rama de GitHub.

6

Configuración específica para el conector origen de Camel. Define el tipo de ruta.

7

Convertidor de claves.

8

Convertidor de valores.

9

Tema al que se envían los mensajes.

### Captura de los datos de eventos de cambio con Debezium

Cada implementación de conector puede tener diferentes opciones de configuración. El siguiente es un ejemplo de un conector Debezium MySQL, que se define como un recurso personalizado de KafkaConnector.

```
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnector
metadata:
  labels:
    strimzi.io/cluster: my-connect-cluster
  name: mysql-debezium-connector
spec:
  class: io.debezium.connector.mysql.MySqlConnector 1
  config:
    database.hostname: mysql 2
    database.port: 3306 3
    database.user: dbuser 4
    database.password: dbpassword 5
    database.dbname: neverendingblog 6
    database.server.name: db 7
    table.include.list: neverendingblog.posts 8
    database.history.kafka.topic: dbhistory.posts 9
    database.history.kafka.bootstrap.servers: 'my-cluster-kafka-bootstrap:9092' 10
  tasksMax: 1
```

1

Clase de conector Debezium MySQL que habilita la conexión MySQL para CDC.

2

Dirección de host del servidor MySQL.

3

Número de puerto del servidor MySQL.

4

Usuario de MySQL con los privilegios adecuados.

5

Contraseña del usuario de MySQL.

6

Base de datos MySQL que incluye las tablas relevantes a las que se conectará.

7

Nombre lógico del servidor MySQL o clúster.

8

Nombre de la tabla en la base de datos cuyos datos deben ser capturados por Debezium.

9

Dirección del clúster de Kafka que el conector usa para escribir y recuperar instrucciones de Lenguaje de definición de datos (DDL) en el tópico del historial de la base de datos.

10

Lista de brókers de Kafka que usa el conector para escribir y recuperar instrucciones DDL en el tópico del historial de la base de datos.



