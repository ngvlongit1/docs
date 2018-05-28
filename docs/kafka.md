# Install kafka

## 1. Download & Extract Kafka
```
wget http://mirror.downloadvn.com/apache/kafka/1.0.0/kafka_2.11-1.0.0.tgz
wget https://apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

tar -xzf kafka_2.11-1.0.0.tgz -C /opt
tar -zxf zookeeper-3.4.10.tar.gz -C /opt

mkdir -p /kafkadata/kafka_1.0.0/kafka-logs
mkdir -p /kafkadata/kafka_1.0.0/zookeeper
```
## 2. Edit config

### Edit /opt/kafka_2.11-1.0.0/config/server.properties
```
# Broker id
broker.id=1

delete.topic.enable=true
auto.leader.rebalance.enable=true
log.dirs=/kafkadata/kafka_1.0.0/kafka-logs

# Number of replica
default.replication.factor=2

log.retention.hours=2  
# 100GB
log.retention.bytes=107374182400

group.initial.rebalance.delay.ms=3

offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3

# List of Zookeeper server
zookeeper.connect=kafka01:2181,kafka02:2181,kafka03:2181
```

### Edit /opt/kafka_2.11-1.0.0/config/zookeeper.properties
```
dataDir=/kafkadata/kafka_1.0.0/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
tickTime=2000
maxClientCnxns=0
initLimit=5
syncLimit=2
#Zookeeper Cluster
server.1=kafka01:2888:3888
server.2=kafka02:2888:3888
server.3=kafka03:2888:3888
```
> For Kafka Cluster
```
echo <broker.id> > /kafkadata/kafka_1.0.0/zookeeper/myid
```

### 3. Start Server
- Start Zookeeper Server
```
/opt/kafka_2.11-1.0.0/bin/zookeeper-server-start.sh -daemon /opt/kafka_2.11-1.0.0/config/zookeeper.properties
```
- Start kafka server
```
/opt/kafka_2.11-1.0.0/bin/kafka-server-start.sh -daemon /opt/kafka_2.11-1.0.0/config/server.properties
```


# Re Install kafka
## 1. Stop Server
> Stop Kafka
```
ps aux | grep -v grep | grep kafkaServer-gc.log | awk '{print $2}' | xargs kill -s TERM
```

> Stop Zookeeper
```
ps aux | grep -v grep | grep zookeeper-gc.log | awk '{print $2}' | xargs kill -s TERM
```
## 2. Delete old data
```
rm -rf /kafkadata/kafka_1.0.0/kafka-logs
rm -rf /kafkadata/kafka_1.0.0/zookeeper/version-2
``` 
## 3. Start Server

- Start Zookeeper Server
```
/opt/kafka_2.11-1.0.0/bin/zookeeper-server-start.sh -daemon /opt/kafka_2.11-1.0.0/config/zookeeper.properties
```
- Start kafka server
```
/opt/kafka_2.11-1.0.0/bin/kafka-server-start.sh -daemon /opt/kafka_2.11-1.0.0/config/server.properties
```

# Use Guide
- Create topic

```/opt/kafka_2.11-1.0.0/bin/kafka-topics.sh --zookeeper localhost:2181  --create --topic test --partitions 32 --replication-factor 2```

- List all topic

```/opt/kafka_2.11-1.0.0/bin/kafka-topics.sh --zookeeper localhost:2181 --list```

- Describe all topic

```/opt/kafka_2.11-1.0.0/bin/kafka-topics.sh --zookeeper localhost:2181 --describe```

- Describe topic test

```/opt/kafka_2.11-1.0.0/bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test```

- Set retention time of topic test to 1 day.

``` /opt/kafka_2.11-1.0.0/bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name test --add-config retention.ms=864000000```

- Delete topic

```/opt/kafka_2.11-1.0.0/bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic test```

- Tail log
```tail -f -n 100 /opt/kafka_2.11-1.0.0/logs/server.log```
```tail -f -n 100 /opt/kafka_2.11-1.0.0/logs/zookeeper.out```