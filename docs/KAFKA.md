# Install Kafka Cluster

## References

http://kafka.apache.org/documentation/#quickstart

## Machines

machine | ip address | os | hostname 
-|-|-|-
master | 192.168.56.150 | centos 7 | aiden-master
slave1 | 192.168.56.151 | centos 7 | aiden-slave1
slave2 | 192.168.56.152 | centos 7 | aiden-slave2

## Prerequisites

- Zookeeper installed


## Installation

```
tar xzvf kafka_2.12-2.4.0.tgz -C /usr/local
```

vi ~/.bashrc
```
export KAFKA_HOME=/usr/local/kafka_2.12-2.4.0
export PATH=$KAFKA_HOME/bin:$PATH
```


## Configuration

```
mkdir -p /root/kafka/logs

cd /usr/local/kafka_2.12-2.4.0/
vi config/server.properties
```

```
broker.id=1
listeners=PLAINTEXT://aiden-master:9092

log.dirs=/root/kafka/logs

zookeeper.connect=aiden-master:2181,aiden-slave1:2181,aiden-slave2:2181
```



## Start

```
bin/kafka-server-start.sh config/server.properties
```

## Check

- jps
```
[root@aiden-master kafka_2.12-2.4.0]# jps
...
5072 Kafka
```


## Client

```
bin/kafka-topics.sh --create --bootstrap-server aiden-master:9092 --replication-factor 2 --partitions 1 --topic my-replicated-topic

bin/kafka-topics.sh --describe --bootstrap-server aiden-master:9092 --topic my-replicated-topic

bin/kafka-topics.sh --create --zookeeper aiden-master:2181,aiden-slave:2181 --partitions 2 --replication-factor 2 --topic test

bin/kafka-topics.sh --describe --bootstrap-server aiden-master:9092 --topic test

bin/kafka-topics.sh --describe --bootstrap-server aiden-master:9092,aiden-slave:9092 --topic test
```

