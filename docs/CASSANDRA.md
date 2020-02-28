# Install Cassandra Cluster

## References

http://cassandra.apache.org/doc/latest/getting_started/installing.html#prerequisites


## Machines

machine | ip address | os | hostname
-|-|-|-
node1 | 192.168.56.150 | centos 7 | aiden-master
node2 | 192.168.56.151 | centos 7 | aiden-slave1
node3 | 192.168.56.152 | centos 7 | aiden-slave2

## Pre-requisites

- JDK installed

## Installation

Download the tar file.
```
tar zxvf apache-cassandra-3.11.6-bin.tar.gz -C /usr/local
mv /usr/local/apache-cassandra-3.11.6 /usr/local/cassandra-3.11.6
```

Add below to profile
```
export CASSANDRA_HOME=/usr/local/cassandra-3.11.6
export PATH=$PATH:$CASSANDRA_HOME/bin
```


## Configuration
On each of the nodes
- Create directions
```bash
mkdir -p /root/cassandra/data
mkdir -p /root/cassandra/commitlog
mkdir -p /root/cassandra/saved_caches
mkdir -p /root/cassandra/hints
```

- conf/cassandra.yaml

```yaml
cluster_name: 'Test Cluster'
...
seed_provider:
    
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          
          - seeds: "192.168.56.150,192.168.56.151,192.168.56.152"
...
# For other node, change this accordingly
listen_address: 192.168.56.150


data_file_directories:
    - /root/cassandra/data

commitlog_directory: /root/cassandra/commitlog
saved_caches_directory: /root/cassandra/saved_caches
hints_directory: /root/cassandra/hints
```
## Start

Run as Root
```
# foreground
cassandra -f -R

# background
cassandra -R
```

## Check

- nodetool status
```
[root@aiden-master conf]# nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address         Load       Tokens       Owns (effective)  Host ID                               Rack
UN  192.168.56.152  114.87 KiB  256          66.5%             e901e4e5-7b05-4ef6-9777-7bb5cb22701f  rack1
UN  192.168.56.150  80.64 KiB  256          64.8%             2d390507-c8d7-4d24-8729-56c6527038aa  rack1
UN  192.168.56.151  137.61 KiB  256          68.7%             e6cd3418-4024-4234-93d6-2eb3eec0ab65  rack1
```


## Client

- cqlsh
```
[root@aiden-master bin]# cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.6 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> 
```
