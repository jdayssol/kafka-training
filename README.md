# kafka-training

Kafka commands:
Prerequisites :

Download kafka https://kafka.apache.org/downloads
Open a terminal inside the unzipped Kafka folder:

Start Zookeeper	bin/zookeeper-server-start.sh config/zookeeper.properties	

Zookeeper is responsible to check the kafka system and distribute the work.
It will redirect the request from the user of kafka to the right broker, will check the broker health status.

Start a broker	bin/kafka-server-start.sh config/server.properties	

A broker is a unit worker in kafka. it receives message from the producer and send them to the consumers.

To start multiple broker in the same machine, duplicate the server.properties files and use this commands
 bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties

To run multiple broker on the same machine, they need to have different name and port.
3 properties to change on the server properties:
broker.id=1 -> unique
listeners=PLAINTEXT://:9093 -> increment it
log.dirs=/tmp/kafka-logs-1
Create a topic	bin/kafka-topics.sh --create --topic my_topic --zookeeper localhost:2181 --replication-factor 1 --partitions 1	
This is a simple topic with one partition and no replication. it will contact the zookeeper to be managed by him.

A partition is a stack that contains messages.

List the topics	bin/kafka-topics.sh --list --zookeeper localhost:2181	
Describe a topic	bin/kafka-topics.sh --describe --topic my_topic --zookeeper localhost:2181	
Create a producer	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my_topic	Type any message that you want and enter to send message
Create a consumer / Listen to a topic	bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my_topic --from-beginning	
Produce messages in a topic	bin/kafka-producer-perf-test.sh --topic my_topic --num-records 50 --record-size 1 --throughput 10 --producer-props bootstrap.servers=localhost:9093	This will create parametrised message inside a topic. 
Change the number of partition of a topic using the zookeeper	bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic my_first_topic --partitions 4	
Adding more partitions.

A partition is a stack that contains messages. A topic can have multiple partitions, that will be managed by the brokersin charge of the topic.

If replication factor > 1, then partition will be also use to replicate data.

A group on consumer will listen this partition, normally the consumer will always listen the same partition during time. This way all the message inside the partitions will be catched.

If partition < consumer, some consumer will listen more than one partition
If partition = consumer, each consumer will always listen the same partition
If partition > consumer, the excedent consumer will be inactive.
Consult the consumer offset topics	bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic __consumer_offsets	
Offset topic is a special topic managed by Kafka.

It will let us now on which message the consumer is reading of the messages flow inside the topic.

This way when a consumer listen the next message, he knows where it stop reading.

Generate java class from AVRO schema.	
Create avro schema manually in json format. Example user_schema.avsc

{
"type": "record",
"namespace": "com.pluralsight.kafka.model",
"name": "User",
"fields": [
{
"name": "userId",
"type": "string"
}, {
"name": "username",
"type": "string"
}, {
"name": "dateOfBirth",
"type": "int",
"logicalType": "date" // Date stored as a integer, counting day since 1/1/1970
}
]
}
Download avro tools:
wget https://repo1.maven.org/maven2/org/apache/avro/avro-tools/1.9.1/avro-tools-1.9.1.jar

In the rep of avro tools:
java -jar avro-tools-1.9.1.jar compile schema PATH/schemas/user_schema.avsc ./generated

Avro is useful in order to generate class that contains serialize / deserialise methods.

This classes can be send throw a producer and received from a consumer using the AvroSerializer / Deserialized .

The generated classes contains method to access the fields and for serialization.
Generated the classes needed and copy paste then inside your project



Adapt a producer to use Avro serialisation	
Copy paste the generated classes from avro tools inside your project

Add to the pom.xml of the producer

<dependency>
<groupId>io.confluent</groupId>
<artifactId>kafka-avro-serializer</artifactId>
<version>5.2.0</version>
</dependency>
<dependency>
<groupId>org.apache.avro</groupId>
<artifactId>avro</artifactId>
<version>1.8.2</version>
</dependency>


By using Avro, we will use his serializer instead of the StringSerializer:
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");


We need to add this property to retrieve the schema from the schema registry
props.put("schema.registry.url", "http://localhost:8081");


Adapt a consumer to use Avro deserialisation	
props.put("bootstrap.servers", "localhost:9093,localhost:9094");
props.put("group.id", "user-tracking-consumer");
props.put("key.deserializer", "io.confluent.kafka.serializers.KafkaAvroDeserializer");
props.put("value.deserializer", "io.confluent.kafka.serializers.KafkaAvroDeserializer");
props.put("specific.avro.reader", "true");
props.put("schema.registry.url", "http://localhost:8081");

Using Schema registry to share Avro encryption between producer and consumer	
git clone https://github.com/confluentinc/schema-registry.git


Use mvn package to compile the schema registry project

Launch the service:
bin/schema-registry-start config/schema-registry.properties

The Schema registry make possible the storage of the schemas and the share of this schemas between producers and consumers.

This way we can send complex classes in our message and we avoid serialisation problems.

Schema registry allow the producer and consumer to access schema in order to serialize or deserialize transfered objects
Confluent offer solution for that.

Notice: from an unknown reason the schema registry was not able to connect to the zookeper. Edit the settings in order to activate the bootstrap server configuration instead.





Example of a project of a producer and a consumer exchanging messages throw Kafka:
Example of a project using 1 zookeeper, 2 broker, 1 schemas registry. 	
Create a zookeper instance
bin/zookeeper-server-start.sh config/zookeeper.properties

Create two brokers
bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties

Create a topic named user-tracking-avro
bin/kafka-topics.sh --create --bootstrap-server localhost:9093 --partitions 2 --replication-factor 2 --topic user-tracking-avro

Start the schema registry 
bin/schema-registry-start config/schema-registry.properties

Mvn package inside the projects
Launch the consumer main method
Launch the producer main method
Code and configuration file available here:

https://github.com/jdayssol/kafka-training

Producer console will print the object, and the consumer will receive them and print them.

