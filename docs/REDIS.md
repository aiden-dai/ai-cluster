# Install Redis Cluster

## References

https://redis.io/topics/cluster-tutorial


## Machines

machine | ip address | os | hostname | Role | Hash Slots
-|-|-|-|-|-
node1 | 192.168.56.150 | centos 7 | aiden-master | master-slave | 0 - 5460
node2 | 192.168.56.151 | centos 7 | aiden-slave1 | master-slave | 5461 - 10922
node3 | 192.168.56.152 | centos 7 | aiden-slave2 | master-slave | 10923 - 16383


## Pre-requisites

- Install tcl

```bash
# tcl.x86_64 1:8.5.13-8.el7 to support make test
yum install tcl
```

## Installation

On each node:
```
tar xzvf redis-5.0.7.tar.gz
cd redis-5.0.7
make
make test
make install
```

```
[root@aiden-master redis-5.0.7]# make test
  ...
  99 seconds - integration/replication-psync
  132 seconds - integration/replication

\o/ All tests passed without errors!

Cleanup: may take some time... OK
make[1]: Leaving directory `/root/redis-5.0.7/src'
```




## Configuration

on each node:
- master 

```
mkdir -p /root/redis/6379
vi /root/redis/6379/redis.conf
```

content as below:
```
requirepass myredis
masterauth myredis
port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

- slave (replica)
```
mkdir -p /root/redis/6380
vi /root/redis/6380/redis.conf
```

content as below:
```
requirepass myredis
masterauth myredis
port 6380
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

## Start

Start redis on each node

```
cd /root/redis/6379
redis-server redis.conf

cd /root/redis/6380
redis-server redis.conf
```

Start log
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

On master
```
redis-cli --cluster create 192.168.56.150:6379 192.168.56.151:6379 192.168.56.152:6379 \
192.168.56.150:6380 192.168.56.151:6380 192.168.56.152:6380 \
--cluster-replicas 1 -a myredis
```

```
[root@aiden-master ~]# redis-cli --cluster create 192.168.56.150:6379 192.168.56.151:6379 192.168.56.152:6379 \
> 192.168.56.150:6380 192.168.56.151:6380 192.168.56.152:6380 \
> --cluster-replicas 1 -a myredis
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.56.151:6380 to 192.168.56.150:6379
Adding replica 192.168.56.152:6380 to 192.168.56.151:6379
Adding replica 192.168.56.150:6380 to 192.168.56.152:6379

...
[OK] All 16384 slots covered.
```


## Check

- check nodes
```
[root@aiden-master ~]# redis-cli -p 6379 -a myredis cluster nodes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
2085870446259a94f2746c206bdad200a8c8f65b 192.168.56.150:6379@16379 myself,master - 0 1582870151000 1 connected 0-5460
2e38f851bd12b532287d59663a2e90b00c31c335 192.168.56.152:6379@16379 master - 0 1582870152817 3 connected 10923-16383
417e1c680db810557d1c3d8fa05f1ff45bed96d5 192.168.56.151:6380@16380 slave 2085870446259a94f2746c206bdad200a8c8f65b 0 1582870151015 5 connected
f4dd0e581f16ae90a3f00c6ce90d9241f39a1b94 192.168.56.152:6380@16380 slave 55d759bd6e8d5308b9e2c35ed6392a730e52d242 0 1582870152349 6 connected
30c622d8f9962018e275d3e439b84cc352f48780 192.168.56.150:6380@16380 slave 2e38f851bd12b532287d59663a2e90b00c31c335 0 1582870152045 4 connected
55d759bd6e8d5308b9e2c35ed6392a730e52d242 192.168.56.151:6379@16379 master - 0 1582870153141 2 connected 5461-10922
```

- Check failover

Manually stop 192.168.56.151:6379, then run cluster nodes again, now 152:6380 become master
```
[root@aiden-master ~]# redis-cli -p 6379 -a myredis cluster nodes
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
2085870446259a94f2746c206bdad200a8c8f65b 192.168.56.150:6379@16379 myself,master - 0 1582870348000 1 connected 0-5460
2e38f851bd12b532287d59663a2e90b00c31c335 192.168.56.152:6379@16379 master - 0 1582870349211 3 connected 10923-16383
417e1c680db810557d1c3d8fa05f1ff45bed96d5 192.168.56.151:6380@16380 slave 2085870446259a94f2746c206bdad200a8c8f65b 0 1582870349101 5 connected
f4dd0e581f16ae90a3f00c6ce90d9241f39a1b94 192.168.56.152:6380@16380 master - 0 1582870348177 8 connected 5461-10922
30c622d8f9962018e275d3e439b84cc352f48780 192.168.56.150:6380@16380 slave 2e38f851bd12b532287d59663a2e90b00c31c335 0 1582870348590 4 connected
55d759bd6e8d5308b9e2c35ed6392a730e52d242 192.168.56.151:6379@16379 master,fail - 1582870326070 1582870325965 2 disconnected
```


## Client


```
[root@aiden-master ~]# redis-cli
127.0.0.1:6379> auth myredis
OK
127.0.0.1:6379> ping
PONG
```

connect to cluster
```
[root@aiden-master ~]# redis-cli -c -a myredis
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set foo bar
-> Redirected to slot [12182] located at 192.168.56.152:6379
OK
192.168.56.152:6379> get foo
"bar"
```