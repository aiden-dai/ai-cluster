# Install Redis Cluster

## References

https://redis.io/topics/cluster-tutorial


## Machines

machine | ip address | os | hostname | Role
-|-|-|-|-
master | 192.168.56.150 | centos 7 | aiden-master | Master
slave1 | 192.168.56.151 | centos 7 | aiden-slave1 | Worker
slave2 | 192.168.56.152 | centos 7 | aiden-slave2 | Worker

For your first tests it is strongly suggested to start a six nodes cluster with three masters and three slaves.

## Pre-requisites

- ...

```
yum install tcl
```

## Installation

```
tar xzvf redis-5.0.7.tar.gz
cd redis-5.0.7
make
make test
make install
```

```
[root@localhost redis-5.0.7]# make test
  ...
  99 seconds - integration/replication-psync
  132 seconds - integration/replication

\o/ All tests passed without errors!

Cleanup: may take some time... OK
make[1]: Leaving directory `/root/redis-5.0.7/src'
```




## Configuration

- redis.conf



## Start

Running Redis
-------------

To run Redis with the default configuration just type:

```
redis-server
redis-server /path/to/redis.conf
```

Start
```
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 18565
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                  
```


## Create cluster


```
mkdir redis-cluster
cd redis-cluster
mkdir 7000 7001 7002 7003 7004 7005
```

create a redis.conf in each folder
```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

start each servers
```
cd 7000
redis-server redis.conf
```

create cluster
```
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1

[root@localhost redis-cluster]# redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
> 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
> --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
...
[OK] All 16384 slots covered.
```


```
redis-cli -p 7000 cluster nodes
```

## Create cluster - Quickstart

```bash
cd utils/create-cluster

# init
./create-cluster start

# Create cluster
./create-cluster create

# Stop cluster
./create-cluster stop

# clean
./create-cluster clean
```


```
[root@localhost create-cluster]# ./create-cluster create
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:30005 to 127.0.0.1:30001
Adding replica 127.0.0.1:30006 to 127.0.0.1:30002
Adding replica 127.0.0.1:30004 to 127.0.0.1:30003
...
[OK] All 16384 slots covered.
```

## Check

- check nodes
```
redis-cli -p 7000 cluster nodes
```


## Client


```
[root@localhost src]# ./redis-cli 
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"

```
