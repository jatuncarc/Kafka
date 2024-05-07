# Apache Kafka y NodeJs #

- Iniciar un contenedor en base a la imagen de zookeeper dede docker hub en caso no se encuentre en local.

`docker run -p 2181:2181 zookeeper`

- Iniciar un contenedor de Kafka incluyendo variables de entorno
  
`docker run -p 9092:9092 -e KAFKA_ZOOKEEPER_CONNECT=192.168.29.176:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.29.176:9092 -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 confluentinc/cp-kafka`

> La variable de entorno KAFKA_ZOOKEEPER_CONNECT contiene la ip y puerto de zookeeper. En caso se encuentre en un contenedor en local, se puede obtener la ip  y puerto de zookeeper con los siguientes scripts:

|`docker inspect --format "{{ .NetworkSettings.IPAddress }}" efbd8535aabb`

|  `docker port efbd8535aabb`

El valor efbd8535aabb representa el containerid.

<blockquote>
    &nbsp;Este es un texto con sangría utilizando HTML.
</blockquote>