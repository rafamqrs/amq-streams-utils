# amq-streams-utils
Repository with commands utils for kafka. 


## List groups
```command
  ./bin/kafka-consumer-groups.sh  --list --bootstrap-server localhost:9092
```
## Describe group
```command 
/bin/kafka-consumer-groups.sh  --describe --group <group_name> --bootstrap-server localhost:9092
```
Understanding each describe group:
|Column Name   |Column Description|
|---           |---|
|GROUP         |Group Name - Kafka assigns the partitions of a topic to the consumer in a group, so that each partition is consumed by exactly one consumer in the group|
|TOPIC         |Topic Name - Kafka topics are the categories used to organize messages |
|PARTITION     | Partitioning takes the single topic log and breaks it into multiple logs, each of which can live on a separate node in the Kafka cluster   |
|CURRENT-OFFSET| Is a pointer to the last record that Kafka has already sent to a consumer in the most recent poll |
|LOG-END-OFFSET|Still remaing to be read by consumer |
|LAG           |Lag is simply the delta between the last produced message and the last consumer's committed offset|
|CONSUMER-ID   |An optional identifier of a Kafka consumer (in a consumer group) that is passed to a Kafka broker with every request|
|HOST          | Is the host           |
|CLIENT-ID     | An optional identifier of a Kafka consumer (in a consumer group) that is passed to a Kafka broker with every request      |
