## Ejemplo Práctico ##

#### Simular una infraestructura donde exista un productos de datos que capture logins de usuarios en tiempo real y los envíe a un tópico llamado `user-tracker`, que será donde el consumidor irá a capturar esos datos

----------

## Instalación de entorno

`python3 -m venv .venv`
> Este comando crea un entorno virtual de Python en el directorio actual (representado por .venv). Un entorno virtual es un entorno de desarrollo aislado que contiene su propia instalación de Python y paquetes, lo que permite gestionar las dependencias de manera independiente para cada proyecto. El comando python3 -m venv utiliza la herramienta venv de Python para crear el entorno virtual


`source .venv/bin/activate`
> Este comando activa el entorno virtual recién creado. Cuando activas un entorno virtual, el sistema operativo utiliza la instalación de Python y los paquetes asociados con ese entorno en lugar de los instalados globalmente en el sistema. Esto ayuda a evitar conflictos entre las versiones de paquetes y garantiza que las dependencias específicas del proyecto se utilicen correctamente. El comando source se utiliza para ejecutar el script de activación del entorno virtual en el shell actual.

> **Nota**: Este comando solo aplicar para Linux y MacOS. Para Windows ejecutar `.\.venv\Scripts\activate`

`pip install -r requirements.txt`
>Este comando instala todas las dependencias del proyecto listadas en un archivo llamado requirements.txt. Este archivo suele contener una lista de paquetes de Python y sus versiones exactas o rangos de versiones compatibles para garantizar que todas las dependencias se instalen correctamente y se mantengan en sincronía entre diferentes entornos de desarrollo. El comando pip es el gestor de paquetes de Python y se utiliza para instalar paquetes desde el índice de paquetes de Python (PyPI) o desde un archivo local como requirements.txt.

## Ejecución

`docker-compose up -d`
> Levantar los contenedores de Kafka y Zookeeper

`python3 kafka_producer.py`
> Ejecutar el producer. Es importante activar el entorno antes de ejecutar el script del producer. En caso arroja error por no encontrar los modulos, intentar con `python kafka_producer.py`

`python3 kafka_consumer.py`
> Ejecutar el consumer en otra terminal. Es importante activar el entorno antes de ejecutar el script del producer.

[Enlace Referencia]([https://](https://medium.com/@juan.martin.ochoa/streaming-introduccion-practica-a-kafka-con-python-y-docker-bd53bfe6d7a2))