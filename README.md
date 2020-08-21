# Couchbase Kafka Connector Demo

In each shell that you are running kafka commands you will need to setup some environment variables which will be used by
all subsequent commands.  

```
. ./setup_kafka.sh
```
> This script is based upon Apache Kafka and should be pointed to your environment

You can test that the environment variables were set correctly by executing the following commands

```
echo $KAFKA_HOME
echo $PATH
```

In one shell start the zookeeper process

```
zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
```

Open a second shell and start the kafka server

```
kafka-server-start.sh $KAFKA_HOME/config/server.properties
```

The next step is to create a topic for us to use

```
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

We will create the following topics for use in this POC

* inbound
* experiment
* full

We can now verify that our topic was created by running the following command

```
kafka-topics.sh --list --bootstrap-server localhost:9092
```

We now need to generate some sample messages.

```
cd $KAFKA_CONNECT_COUCHBASE_HOME/examples/json-producer
```

Run the command:
```
mvn compile exec:java -Dexec.args="1 10 justins_sweet_stat test"
```

The arguments are as follows:

* Starting ID
* Range
* Stat Name
* Kafka TOPIC

In this case,  the command would generate messages for I-1 through I-10,  setting a statistic called "justins_sweet_stat" and publishing these messages to the topic "test"

We can view the messages generated in the topic by running the following command

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

We will now setup the Couchbase Sink Connector to push the messages into Couchbase.  
The first step we need to do is to edit the $KAFKA_CONNECT_COUCHBASE_HOME/config/quickstart-couchbase-sink.properties file.  
Verify that the topics, bucket, and credentials are correct for the use case we are testing.

We can run the following command to start the couchbase-sink connector for the different tests

INBOUND APPROACH

cd $KAFKA_CONNECT_COUCHBASE_HOME
env CLASSPATH=./* connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties config/quickstart-couchbase-sink-inbound.properties

EXPERIMENT APPROACH

cd $KAFKA_CONNECT_COUCHBASE_HOME
env CLASSPATH=./* connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties config/quickstart-couchbase-sink-experiment.properties

FULL APPROACH

cd $KAFKA_CONNECT_COUCHBASE_HOME
env CLASSPATH=./* connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties config/quickstart-couchbase-sink-full.properties


--- DELETE A TOPIC ----

kafka-topics.sh --zookeeper localhost:2181 --delete --topic test
