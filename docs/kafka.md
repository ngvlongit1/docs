# Install kafka

## Environment

```export SCALA_VERSION="2.11"```

```export KAFKA_VERSION="1.0.0"```

## 1. Download & Extract Kafka
```
wget http://mirror.downloadvn.com/apache/kafka/$KAFKA_VERSION/kafka_$SCALA_VERSION-$KAFKA_VERSION.tgz
wget https://apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz

tar -xzf kafka_$SCALA_VERSION-$KAFKA_VERSION.tgz -C /opt
tar -zxf zookeeper-3.4.10.tar.gz -C /opt

mkdir -p /kafkadata/kafka_$KAFKA_VERSION/kafka-logs
mkdir -p /kafkadata/kafka_$KAFKA_VERSION/zookeeper
```
## 2. Edit config

### Edit /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/server.properties
```
# Broker id
broker.id=1

delete.topic.enable=true
auto.leader.rebalance.enable=true
log.dirs=/kafkadata/kafka_$KAFKA_VERSION/kafka-logs

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

### Edit /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/zookeeper.properties
```
dataDir=/kafkadata/kafka_$KAFKA_VERSION/zookeeper
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
echo <broker.id> > /kafkadata/kafka_$KAFKA_VERSION/zookeeper/myid
```

### 3. Start Server

- Disable swap

```swapoff -a```

- Start Zookeeper Server
```
/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/zookeeper-server-start.sh -daemon /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/zookeeper.properties
```
- Start kafka server
```
/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-server-start.sh -daemon /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/server.properties
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
rm -rf /kafkadata/kafka_$KAFKA_VERSION/kafka-logs
rm -rf /kafkadata/kafka_$KAFKA_VERSION/zookeeper/version-2
``` 
## 3. Start Server

- Start Zookeeper Server
```
/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/zookeeper-server-start.sh -daemon /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/zookeeper.properties
```
- Start kafka server
```
/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-server-start.sh -daemon /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/config/server.properties
```

# Use Guide
- Create topic

```/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-topics.sh --zookeeper localhost:2181  --create --topic test --partitions 32 --replication-factor 2```

- List all topic

```/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-topics.sh --zookeeper localhost:2181 --list```

- Describe all topic

```/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-topics.sh --zookeeper localhost:2181 --describe```

- Describe topic test

```/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test```

- Set retention time of topic test to 1 day.

``` /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name test --add-config retention.ms=864000000```

- Delete topic

```/opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic test```

- Tail log Kafka

```tail -f -n 100 /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/logs/server.log```

- Tail log Zookeeper

```tail -f -n 100 /opt/kafka_$SCALA_VERSION-$KAFKA_VERSION/logs/zookeeper.out```