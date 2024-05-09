# Apache Kafka y NodeJs #

En este ejemplo se implementa el caso de 

**Requisitos:** 

- Construir entorno local de Kafka usando Docker. Para ello crear un archivo *kafka-compose.yml* con el siguiente contenido:
    ```yaml 
    version: '2'
    services:
    zookeeper:
        image: confluentinc/cp-zookeeper:latest
        environment:
        ZOOKEEPER_CLIENT_PORT: 2181
        ZOOKEEPER_TICK_TIME: 2000
    kafka-ui:
        container_name: kafka-ui
        image: provectuslabs/kafka-ui:latest
        ports:
        - "9000:8080"
        environment:
        KAFKA_CLUSTERS_0_NAME: dev-local
        KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
        KAFKA_CLUSTERS_0_METRICS_PORT: 9997
        DYNAMIC_CONFIG_ENABLED: true
        depends_on:
        - "kafka"
    kafka:
        image: confluentinc/cp-kafka:latest
        depends_on:
        - zookeeper
        ports:
        - "9092:9092"
        - "9997:9997"
        environment:
        KAFKA_BROKER_ID: 1
        KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
        KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
        KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
        KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        KAFKA_JMX_PORT: 9997
    ```

- Levantar los contenedores

    ```yaml
        docker-compose -f .\kafka-compose.yml up -d
    ```
  
- Iniciar proyecto node:
   ```javascript
        npm init -y
    ```
- Instalar dependencia KafkaJs
    ```javascript
        npm install kafkajs
    ```

- Crear un archivo input.json
    ```json
    [
    {
      "eventId": "001",
      "firstName": "Zac",
      "lastName": "Ryan",
      "age": 34,
      "Gender": "M",
      "bodyTemperature": 36.9,
      "overseasTravelHistory": true,
      "eventTimestamp": "2020-03-02 02:09:00"
    },
    {
      "eventId": "002",
      "firstName": "Jade",
      "lastName": "Green",
      "age": 23,
      "Gender": "M",
      "bodyTemperature": 36.8,
      "overseasTravelHistory": false,
      "eventTimestamp": "2020-03-02 03:09:11"
    },
    {
      "eventId": "003",
      "firstName": "Claudia",
      "lastName": "Wright",
      "age": 18,
      "Gender": "F",
      "bodyTemperature": 36.8,
      "overseasTravelHistory": true,
      "eventTimestamp": "2020-03-04 02:09:00"
    },
    {
      "eventId": "004",
      "firstName": "Mia",
      "lastName": "Schmidt",
      "age": 70,
      "Gender": "F",
      "bodyTemperature": 37.2,
      "overseasTravelHistory": false,
      "eventTimestamp": "2020-03-06 01:09:00"
    }
  ]
    ```
- Crear archivo de configuracion **config.js**:
    ```javascript
    module.exports = {
        kafka: {
        TOPIC: 'test',
        BROKERS: ['localhost:9092'],
        GROUPID: 'my-group-demo',
        CLIENTID: 'kafka-client'
        }
    }
    ```

- Crear el productor **producer.js**
    ```javascript
    const { Kafka } = require('kafkajs')
    const config = require('./config')
    const messages = require('../input.json')

    const client = new Kafka({
    brokers: config.kafka.BROKERS,
    clientId: config.kafka.CLIENTID
    })

    const topic = config.kafka.TOPIC

    const producer = client.producer()

    let i = 0

    const sendMessage = async (producer, topic) => {
    await producer.connect()

    setInterval(function() {
        i = i >= messages.length - 1 ? 0 : i + 1
        payloads = {
        topic: topic,
        messages: [
            { key: 'coronavirus-alert', value: JSON.stringify(messages[i]) }
        ]
        }
        console.log('payloads=', payloads)
        producer.send(payloads)
    }, 5000)
    }

    sendMessage(producer, topic)   
    ```

- Crear el consumidor **consumer.js**:
  ```javascript
    const { Kafka } = require('kafkajs')
    const config = require('./config')

    const kafka = new Kafka({
    clientId: config.kafka.CLIENTID,
    brokers: config.kafka.BROKERS
    })

    const topic = config.kafka.TOPIC
    const consumer = kafka.consumer({
    groupId: config.kafka.GROUPID
    })

    const run = async () => {
    await consumer.connect()
    await consumer.subscribe({ topic, fromBeginning: true })
    await consumer.run({
        eachMessage: async ({ message }) => {
        try {
            const jsonObj = JSON.parse(message.value.toString())
            let passengerInfo = filterPassengerInfo(jsonObj)
            if (passengerInfo) {
            console.log(
                '******* Alert!!!!! passengerInfo *********',
                passengerInfo
            )
            }
        } catch (error) {
            console.log('err=', error)
        }
        }
    })
    }

    function filterPassengerInfo(jsonObj) {
    let returnVal = null

    console.log(`eventId ${jsonObj.eventId} received!`)

    if (jsonObj.bodyTemperature >= 36.9 && jsonObj.overseasTravelHistory) {
        returnVal = jsonObj
    }

    return returnVal
    }

    run().catch(e => console.error(`[example/consumer] ${e.message}`, e))

    const errorTypes = ['unhandledRejection', 'uncaughtException']
    const signalTraps = ['SIGTERM', 'SIGINT', 'SIGUSR2']

    errorTypes.map(type => {
    process.on(type, async e => {
        try {
        console.log(`process.on ${type}`)
        console.error(e)
        await consumer.disconnect()
        process.exit(0)
        } catch (_) {
        process.exit(1)
        }
    })
    })

    signalTraps.map(type => {
    process.once(type, async () => {
        try {
        await consumer.disconnect()
        } finally {
        process.kill(process.pid, type)
        }
    })
    })

    module.exports = {
    filterPassengerInfo
    }
  ```

- Agregar los siguientes script en el package.json:
    ```json
    {
    "name": "lab02",
    "version": "1.0.0",
    "description": "kafka y node`",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "consumer": "node ./src/consumer.js",
        "producer": "node ./src/producer.js"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "dependencies": {
        "kafkajs": "^2.2.4"
        }
    }
    ```
- Ejecutar los siguientes comandos en una terminal por separado cada comando:
    

    `npm run producer`
    
    `npm run consumer`


    Referencia:
  [https://github.com/billonline33/kafka-nodejs-tutorial](https://github.com/billonline33/kafka-nodejs-tutorial)