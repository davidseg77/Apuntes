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

