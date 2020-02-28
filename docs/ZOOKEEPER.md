# Install Hadoop Cluster

## References

...

## Machines

machine | ip address | os | hostname
-|-|-|-
master | 192.168.56.150 | centos 7 | aiden-master
slave1 | 192.168.56.151 | centos 7 | aiden-slave1
slave2 | 192.168.56.152 | centos 7 | aiden-slave2

## Pre-requisites

- N/A

## Installation

Run
```
tar xzvf apache-zookeeper-3.5.7-bin.tar.gz  -C /usr/local
mv /usr/local/apache-zookeeper-3.5.7-bin /usr/local/zookeeper-3.5.7
```

vi ~/.bashrc
```
export ZK_HOME=/usr/local/zookeeper-3.5.7
export PATH=$PATH:$ZK_HOME/bin
```

## Configuration
Create Data folder
```
mkdir -p /root/zookeeper
```

Create myid file.
- aiden-master
```
echo '1' > /root/zookeeper/myid
```
- aiden-slave1
```
echo '2' > /root/zookeeper/myid
```
- aiden-slave2
```
echo '3' > /root/zookeeper/myid
```

For replicated mode, a minimum of three servers are required, and it is strongly recommended that you have an odd number of servers.

Create zoo.cfg
```
vi  /usr/local/zookeeper-3.5.7/conf/zoo.cfg

tickTime=2000
dataDir=/root/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=aiden-master:2888:3888
server.2=aiden-slave1:2888:3888
server.3=aiden-slave2:2888:3888
```

## Start Zookeeper

Run in all machines
```
bin/zkServer.sh start
```

## Check

- zkServer.sh status

```
[root@aiden-master zookeeper-3.5.7]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader

[root@aiden-slave1 zookeeper-3.5.7]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

- JPS

```
[root@aiden-master ~]# jps
...
4204 QuorumPeerMain

[root@aiden-slave1 ~]# jps
...
3428 QuorumPeerMain
```


## client

Run zkCli
```
bin/zkCli.sh -server 127.0.0.1:2181
```