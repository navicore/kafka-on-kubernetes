# Apache Kafka on Kubernetes

Created from combining parts of [ramhiser/kafka-kubernetes](https://github.com/ramhiser/kafka-kubernetes) and [UKHomeOffice/docker-kafka](https://github.com/UKHomeOffice/docker-kafka) with references to docker images I build myself.

###Caveats
* presumes you have a healthy kubernetes environment.  If not see [this](https://gist.github.com/navicore/5b67aba5c6c0c30f91e3c65f1e31507f) or [this](http://kubernetes.io/docs/getting-started-guides).
* Uses RC for kafka and Deployment for zk.  todo: Deployment for kafka
* Does not configure disk, replacing more than n-1 of your replication factor will loose data. todo: StatefulSets in 1.5

###DO THIS
```
kubectl create -f zookeeper-services.yaml
kubectl create -f zookeeper-cluster.yaml
```

After the Zookeeper cluster is launched, check that all three deployments are
`Running`.

```
$ kubectl get pods
zookeeper-deployment-1-dbauf   1/1       Running   0          2h
zookeeper-deployment-2-mp6nb   1/1       Running   0          2h
zookeeper-deployment-3-26ere   1/1       Running   0          2h
```

One of the deployments should be `LEADING`, while the other two should be
`FOLLOWERS`. To check that, look at the pod logs.

```
kubectl logs zookeeper-deployment-1-dbauf
...
2016-10-06 14:04:05,904 [myid:2] - INFO [QuorumPeer[myid=2]/0:0:0:0:0:0:0:0:2181:Leader@358] - LEADING - LEADER ELECTION TOOK - 2613
```

```
kubectl create -f kafka-services.yaml
```

```
kubectl create -f kafka-cluster.yaml
```

# instructions for a kafka cli

find a kafka pod and exec bash inside - you then have all the kafka scripts in
an env that can find the brokers and zookeepers via service disco
```
exec <KAFKA POD NAME> -i -t -- bash -il
cd /kafka/bin
ls
```

list topics

```
./kafka-topics.sh --list --zookeeper zoo1,zoo2,zoo3
```

create a topic

```
./kafka-topics.sh --create --zookeeper zoo1 --topic test1 --partitions 4 --replication-factor 3
```

post to a topic

```
./kafka-console-producer.sh --broker-list kafka-1:9092 --topic test1
```

read a topic

```
./kafka-console-consumer.sh --bootstrap-server kafka-1:9092 --topic test1 --from-beginning
```

