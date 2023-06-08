## 1 - Create a MachineSet with label and taint
```shell
oc apply -f machineset.yaml
```
### Showing the taint
```shell
oc describe node <node-name> | grep -i taint
```

## 2 - Create a namespace com o node selector.
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: amq-streams
  labels:
    kubernetes.io/metadata.name: amq-streams
  annotations:
    openshift.io/display-name: amqstreams
    openshift.io/node-selector: env=infra-amqstreams
    scheduler.alpha.kubernetes.io/defaultTolerations: >-
      [{"operator": "Exists", "effect": "NoSchedule", "key":"reserved"}]
spec:
  finalizers:
    - kubernetes
```
## 3 - Install the AMQ Streams Operator via Operator Hub

## 4 - Create the a two configmaps to export the metrics from the JMX(Zookeeper and Broker)
```shell
oc apply -f kuberesources/kafka-metrics-configmap.yaml
```
## 5 - Create the broker with a super-admin, tls and scram-sha auth. 
```shell
oc apply -f kuberesources/kafka.yaml
```
## 6 - Check if all pods were deployed.
```shell
oc get pods -w 
amq-streams-cluster-operator-v2.4.0-0-64f5f6b9c6-kqf98   1/1     Running   0          24m
kafka-cluster-entity-operator-744c776cff-h9stk           3/3     Running   0          5m2s
kafka-cluster-kafka-0                                    1/1     Running   0          5m42s
kafka-cluster-kafka-1                                    1/1     Running   0          5m42s
kafka-cluster-kafka-2                                    1/1     Running   0          5m42s
kafka-cluster-kafka-exporter-58d494b9b5-b6rlp            1/1     Running   0          4m31s
kafka-cluster-zookeeper-0                                1/1     Running   0          6m31s
kafka-cluster-zookeeper-1                                1/1     Running   0          6m31s
kafka-cluster-zookeeper-2                                1/1     Running   0          6m31s
```
