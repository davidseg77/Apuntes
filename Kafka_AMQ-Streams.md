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

