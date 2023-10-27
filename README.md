# kubecon-etcd-metrics-lab

## Scenario 1: delay fsync

`podman-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up`

`curl <http://127.0.0.1:11180/walBeforeFdatasync> -XPUT -d'sleep(100)'`
`curl <http://127.0.0.1:21180/walBeforeFdatasync> -XPUT -d'sleep(100)'`
`curl <http://127.0.0.1:31180/walBeforeFdatasync> -XPUT -d'sleep(100)'`

`docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=27 --conns=20 put --sequential-keys --key-space-size=100000 --total=100000`

## Scenario 2: delay network

Adjust `-rx-delay` in docker-compose-etcd-bridge.yml

`podman-compose -f docker-compose-etcd-bridge.yml -f docker-compose-metrics.yml up`

`docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=27 --conns=20 put --sequential-keys --key-space-size=100000 --total=100000`

## Scenario 3: range

## Glossary

- **Compaction** : etcd keeps an exact history of its keyspace, the process of compacting the keyspace history to drop all information about keys superseded prior to a given keyspace revision is compaction.
- **KV** : Key-Value pair.
- **WAL** : Write-Ahead Log.
- **MVCC** : Multi-Version Concurrency Control.
- **boltDB** : bolt database - backend used by etcd, more info [here](https://github.com/boltdb/bolt).
- **raft** : Raft is a consensus algorithm that is designed to be easy to understand. It's equivalent to Paxos in fault-tolerance and performance. More info [here](https://raft.github.io)
- **txn** : Transaction, a compound operation type in etcd, that can be any* combination of Read/Write/Delete.
- **gRPC** : gRPC is a modern open source high performance Remote Procedure Call (RPC) framework. More info [here](https://grpc.io/).
- **bidi** : bi-directional.
- **fsync** : `fsync()` transfers all modified in-core data of the file referred to by the file descriptor fd to the disk device (or other permanent storage device) so that all changed information can be retrieved even if the system crashes or is rebooted.

## Citations/References

- [gRPC](https://grpc.io)
- [raft](https://raft.github.io/)
- [draw.io](https://draw.io)
- [boltDB](https://github.com/boltdb/bolt.git)
- [etcd](https://etcd.io/)

- grafana and prometheus configs are from [docker-compose-stacks](https://github.com/ninadingole/docker-compose-stacks)
