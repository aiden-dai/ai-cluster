# Install ETCD Cluster

## References

https://etcd.io/docs/v3.4.0/op-guide/clustering/


## Machines

machine | ip address | os | hostname
-|-|-|-
node1 | 192.168.56.150 | centos 7 | aiden-master
node2 | 192.168.56.151 | centos 7 | aiden-slave1
node3 | 192.168.56.152 | centos 7 | aiden-slave2

## Pre-requisites

- N/A

## Installation

```
ETCD_VER=v3.4.4
curl -L https://storage.googleapis.com/etcd/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o etcd-${ETCD_VER}-linux-amd64.tar.gz
mkdir -p /usr/local/etcd
tar xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz -C /usr/local/etcd --strip-components=1
```


```
export PATH=$PATH:/usr/local/etcd
```


## Configuration

To be updated.


## Start

On node1:
```
etcd --name infra0 --initial-advertise-peer-urls http://192.168.56.150:2380 \
  --listen-peer-urls http://192.168.56.150:2380 \
  --listen-client-urls http://192.168.56.150:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.56.150:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://192.168.56.150:2380,infra1=http://192.168.56.151:2380,infra2=http://192.168.56.152:2380 \
  --initial-cluster-state new
```

On node2:
```
$ etcd --name infra1 --initial-advertise-peer-urls http://192.168.56.151:2380 \
  --listen-peer-urls http://192.168.56.151:2380 \
  --listen-client-urls http://192.168.56.151:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.56.151:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://192.168.56.150:2380,infra1=http://192.168.56.151:2380,infra2=http://192.168.56.152:2380 \
  --initial-cluster-state new
```

On node3:
```
etcd --name infra2 --initial-advertise-peer-urls http://192.168.56.152:2380 \
  --listen-peer-urls http://192.168.56.152:2380 \
  --listen-client-urls http://192.168.56.152:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://192.168.56.152:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://192.168.56.150:2380,infra1=http://192.168.56.151:2380,infra2=http://192.168.56.152:2380 \
  --initial-cluster-state new
```

## Check

- etcd --version
```
[root@aiden-master ~]# etcd --version
etcd Version: 3.4.4
Git SHA: c65a9e2dd
Go Version: go1.12.12
Go OS/Arch: linux/amd64
```

- etcdctl version
```
[root@aiden-master ~]# etcdctl version
etcdctl version: 3.4.4
API version: 3.4
```

- Check cluster members
```
ENDPOINTS=192.168.56.150:2379,192.168.56.151:2379,192.168.56.152:2379
etcdctl --endpoints=$ENDPOINTS member list

[root@aiden-master ~]# etcdctl --endpoints=$ENDPOINTS member list
837735775bff0d0, started, infra2, http://192.168.56.152:2380, http://192.168.56.152:2379, false
673ae722c15233dc, started, infra1, http://192.168.56.151:2380, http://192.168.56.151:2379, false
696a54bf5f98b629, started, infra0, http://192.168.56.150:2380, http://192.168.56.150:2379, false
```

- Check cluster status and health

```
[root@aiden-master ~]# etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|      ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 192.168.56.150:2379 | 696a54bf5f98b629 |   3.4.4 |   20 kB |      true |      false |        12 |         34 |                 34 |        |
| 192.168.56.151:2379 | 673ae722c15233dc |   3.4.4 |   20 kB |     false |      false |        12 |         34 |                 34 |        |
| 192.168.56.152:2379 |  837735775bff0d0 |   3.4.4 |   20 kB |     false |      false |        12 |         34 |                 34 |        |
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
[root@aiden-master ~]# etcdctl --endpoints=$ENDPOINTS endpoint health
192.168.56.150:2379 is healthy: successfully committed proposal: took = 1.816663ms
192.168.56.151:2379 is healthy: successfully committed proposal: took = 2.313504ms
192.168.56.152:2379 is healthy: successfully committed proposal: took = 2.460979ms
```


## Client

- etcdctl
```
[root@aiden-master ~]# etcdctl --endpoints=$ENDPOINTS put foo 'hello'
OK
[root@aiden-master ~]# etcdctl --endpoints=$ENDPOINTS get foo
foo
hello
[root@aiden-master ~]# etcdctl --endpoints=$ENDPOINTS del foo
1
```
