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

## Kafka Producer
There are two option that I can show you: 
1. First is the from the AMQ Streams/Kafka package:
Create file ssl-perf-test.properties with the content below
```console
security.protocol=SSL
ssl.truststore.location=/path/to/your/kafka.client.truststore.jks
ssl.keystore.location=/path/to/your/kafka.client.keystore.jks
ssl.truststore.password=your_SSL_certificate_passphrase
ssl.keystore.password=passphrase
```
2. Run the kafka-producer-perf-test.sh
```shell
  bin/kafka-producer-perf-test.sh \
  --topic <topic_name> \
  --throughput -1 \
  --num-records 3000000 \
  --record-size 1024 \
  --producer-props acks=all bootstrap.servers=broker0:9093,broker1:9093,broker2:9093 \
  --producer.config /path/to/ssl-perf-test.properties
```

## Kafka Consumer


## List Topics
```shell
./bin/kafka-topics.sh --bootstrap-server=localhost:9092 --list
```
Show topic details 
```shell
 ./bin/kafka-topics.sh --bootstrap-server=localhost:9092 --describe --topic users.registrations
```
In Openshift
```shell
oc run kafka-consumer -ti --image=registry.redhat.io/amq7/amq-streams-kafka-27-rhel7:1.7.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server cluster-name-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

## SASL SCRAM-SHA-512
### To configure Kafka Connect to use SASL-based SCRAM-SHA-512 authentication, set the type property to scram-sha-512. This authentication mechanism requires a username and password.
1. Create a KafkaUser with SCRAM_SHA auth
```shell
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaUser
metadata:
  name: user-sample
  labels:
    strimzi.io/cluster: my-kafka-cluster
spec:
  authentication:
    type: scram-sha-512
```
2. It will create a secret that contains the password in base64, e.g:
```yaml
apiVersion: v1
kind: Secret
name: kafka-scram-client-credentials
data:
  password: SnpteEQwek1DNkdi
```
3. Downloadinf the password, ca and user.p12:
```shell
oc get secret user-sample -o jsonpath='{.data.password}' | base64 --decode > user.password
oc get secret <kafka-cluster>-cluster-ca-cert -n kafka -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crt
oc get secret <kafka-cluster>-cluster-ca-cert -n kafka -o jsonpath='{.data.ca\.password}' | base64 --decode > ca.password
```
4. Creating th trustore
```shell
keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca.cert 
```
It will generate a file *kafka.client.truststore.jks*
5. Now you can create a file call client.properties with the content bellow, I should replace the values with <REPLACE_ME>
```yaml
bootstrap.servers=[replace with public-ip:9094]
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
ssl.truststore.location=<REPLACE_ME_WITH_TRUSTORE.jks>
ssl.truststore.password=<REPLACE_ME_WITH_TRUSTORE_PASSWORD>
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="<REPLACE_ME_USER>" password="<REPLACE_ME_USER_PASSWORD"
```
5. Testing with the kafka cli
```shell
./kafka-console-consumer.sh --bootstrap-server <BOOTSTRAP_SERVER_URL>:443 --topic <TOPIC_NAME> --property print.timestamp=true --property print.key=true --consumer.config client.properties
```

Other example is using the cli directly with the properties
```shell
bin/kafka-console-producer.sh \
--bootstrap-server <BOOTSTRAP_SERVER>:443 \
--producer-property ssl.truststore.password=password \
--producer-property ssl.truststore.location=truststore.jks \
--producer-property security.protocol=SASL_SSL \
--producer-property sasl.mechanism=SCRAM-SHA-512 \
--producer-property sasl.jaas.config="org.apache.kafka.common.security.scram.ScramLoginModule required username=<USERNAME> password=<USERPASSWORD>;" \
--producer-property ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1 \
--topic <TOPIC_NAME>
```
