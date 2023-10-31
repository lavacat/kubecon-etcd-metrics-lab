# kubecon-etcd-metrics-lab

## Prerequisites

Start 3-node etcd cluster with grafana and prometheus
```bash
docker-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up
```
To verify:
```bash
docker ps --all
```
Output:
```bash
➜  ~ podman ps --all
CONTAINER ID  IMAGE                                  COMMAND               CREATED         STATUS         PORTS                                              NAMES
8c2060085521  docker.io/bkanivets/etcd:v3.5.9-arm64  --name=etcd-1 --i...  26 seconds ago  Up 21 seconds  0.0.0.0:2379->2379/tcp, 0.0.0.0:11180->11180/tcp   kubecon-etcd-metrics-lab_etcd-1_1
e89e0d64213e  docker.io/bkanivets/etcd:v3.5.9-arm64  --name=etcd-2 --i...  25 seconds ago  Up 20 seconds  0.0.0.0:21180->11180/tcp, 0.0.0.0:22379->2379/tcp  kubecon-etcd-metrics-lab_etcd-2_1
163a3b97507f  docker.io/bkanivets/etcd:v3.5.9-arm64  --name=etcd-3 --i...  24 seconds ago  Up 19 seconds  0.0.0.0:31180->11180/tcp, 0.0.0.0:32379->2379/tcp  kubecon-etcd-metrics-lab_etcd-3_1
81db7802ceba  docker.io/prom/prometheus:latest       --config.file=/et...  23 seconds ago  Up 18 seconds  0.0.0.0:9090->9090/tcp                             kubecon-etcd-metrics-lab_prometheus_1
f04a2e245717  docker.io/grafana/grafana:latest                             21 seconds ago  Up 17 seconds  0.0.0.0:3000->3000/tcp                             kubecon-etcd-metrics-lab_grafana_1
```

Try running benchmark:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=27 --conns=20 put --sequential-keys --key-space-size=100000 --total=100000
```

Check out default etcd [dashboard](http://localhost:3000/d/e3f3beda-14fe-47ad-a431-c8227c997a53/etcd-by-prometheus)

Explore sequence diagrams for [Puts](https://mermaid.live/edit#pako:eNqNVdmO2jAU_RXLTzMqiQhhCX5AoouqSt0E7YxURULGvgzRECfjOAwU8e-9DmsgZAov4S7nnLuFDRWJBMpoBi85KAEfI_6keRyqabIin3UqQhUqgp-UaxOJKOXKkKWPDvd5OQa9BB0qUDJU5wFghMwKp_sJHw9xFnPEZ-YaUaPV_Y5Srl1nWDZqF1Qwljlf-cJ9HH69KYSn6SIC_eDvdHxbCnFNFqPVfdSRgYcIXm_4IyVhVVH1lItnNLrvuRHzXysrMFQ8N4nK46mt36KVe-cMBu8qm8XIz9zc3VfmOIOygWUGVYCcmMRgE74ocUishLacx34jj07SJIP6FGdQrTItkvkim6RYeaSeztmPJM5FmYc5MvJqW01GwOWaiDlXVxoOoRZhP2FGxnx51Ls3IsXRjQ-TWbZWYiJzzU2UqEkGIlEyc39MC9yKYo9EpVJPSk-ViiSOI3Pq9xhMPd7NEe-qt4u5FtXll6dWtc2MDG1-hYJjiE0tL3Zpu8ouG3yxyIz8VhmfwRheatNOx1GH7wwutdjfkzQ3-34OpaytpqahRStHkOULfElF_M2e3lrrs2nveOtnXZ7SxXHaVqCkNFEZ_N85o2i5qDrntxJJkYlXeGvfLwbrVIz6Q7Hdd_cEEHFNvGYzzqqTncFV8t6wP5Ha89t9aYPGoGMeSfwP2liekJo5xBBSho-S6-eQhmqLcfZFOsajpszoHBo0TyU3h_8rymY4LLTii5iyDV1R1u65Hb_lt4Ou3wq8VqdB15Q5bbfZ6bS7_X7Q7_W6Xc_bNujfJEEADz1eP-gEvU4r6AW-5xdofwqnpdz-AxFKhD0) and [Ranges](https://mermaid.live/edit#pako:eNqFVMGOmzAQ_RXLp1YNKOAtJD4gVdo97KWVsu0eKqTIsScJChhqG5I0yr_XZstqYVmCL8jz3ps3M7YvmJcCMMUa_tQgOdxnbKdYkUq3KqZMxrOKSYMaoiruH5onUA2oYRQMF7qN-A_2twNtyhNasa1JJbLfW4Kyu_53m_t96I2WQ72AQIoPM7KqyjNQz6SPKBrO_RUw8ZzBsR_aMH6wim305-mlWFabUtbFxvl2lvoFe0nyZbRIaguUO_j0eZTlJf0Nqo21AWJtSsNy_1Hyjjgq7rK-dspmsnYfpYDTNMlLxp3qvDyuldVYZ04E9HsXr8m8QcHdJCg6qsxAa-WM-J7JG_bH5jTo2RjEUXsDHHB6MQfuj5SibakQML5HChqKfknNtjCh4CWDbMph_zfomxAf5p2obwW6zs10jVNnyvKrUmq40eHBCRvj3jqVdowiHzuVt4ioZWZyt9bASym0_2PTmnQC3cIzXIAqWCbsQ3Nxwik2eyggxdT-CqYOKU7l1eLcJXw6S46pUTXMcF0JZrpHCdMty7XdtXcY0ws-YRou_DmZLxfBPCKLuzCOyQyfMY1jP4yigERBtLwjJLrO8N-ytAJzf0m-hlEYBDFZxhYTtWq_26BLef0HJo_FLg)

## Scenario 1: delay fsync

### Add small delay
```bash
curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(100)'
```

Run `put` benchmark with 1000 clients
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=1000 --conns=1000 put --sequential-keys --key-space-size=100000 --total=100000
```
Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.

### Add large delay
```bash
curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(1000)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(1000)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(1000)'
```

Run `range` benchmark with 100 clients (while prior benchmark is still running)
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 range /
```

Observe behavior at [Ranges](http://localhost:3000/d/ad0da30b-2128-4455-8cef-31424b06b7b9/ranges) dashboard.

Try `range` benchmark with `--consistency=s`
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 --consistency=s range /
```

## Scenario 2: delay network

Stop cluster after scenario 1.

Adjust `-rx-delay` in docker-compose-etcd-bridge.yml

Start cluster with `bridge` interface.
```bash
docker-compose -f docker-compose-etcd-bridge.yml -f docker-compose-metrics.yml up
```

Check [Peers](http://localhost:3000/d/f3469871-37dd-4af7-acb1-ef0c1f7d5747/peers) dashboard.

Run `put` benchmark with 1000 clients
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=1000 --conns=1000 put --sequential-keys --key-space-size=100000 --total=100000
```
Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.


## Scenario 3: reaching DB size limit
Reset delay if any:
```bash
curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(1)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(1)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(1)'
```

Run `put` benchmark with increased val size
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=10000 --key-space-size=100000 --total=10000000
```

Default limit it 2Gb. Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.

## Scenario 4: compaction and defragmentation

Run `put` benchmark with `compaction`
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000 compact-index-delta=100 --compact-interval=10s
```

Observe behavior at [Compaction](http://localhost:3000/d/eb5c5196-8de5-4435-9a7d-d2bb4da869f4/compaction) dashboard.

### Run defragmentation

```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9-arm64 defrag --endpoints=127.0.0.1:2379
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9-arm64 defrag --endpoints=127.0.0.1:22379
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9-arm64 defrag --endpoints=127.0.0.1:32379
```

For explanation of defragmentation process see [docs](https://etcd.io/docs/v3.5/op-guide/maintenance/#defragmentation).

### Run defragmentation with delay

Make sure that `put` benchmark is still running:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000 compact-index-delta=100 --compact-interval=10s
```

Add delay:
```bash
curl http://127.0.0.1:11180/defragBeforeCopy -XPUT -d'sleep(10000)'
curl http://127.0.0.1:21180/defragBeforeCopy -XPUT -d'sleep(10000)'
curl http://127.0.0.1:31180/defragBeforeCopy -XPUT -d'sleep(10000)'
```

Run defrag for non-leader:
```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9-arm64 defrag --endpoints=127.0.0.1:32379
```

Observe behavior at [Defrag](http://localhost:3000/d/bb7bc45f-5370-401d-8212-3408c6936f5c/defrag) dashboard. Check out grpc error rate.

Run defrag for leader:
```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9-arm64 defrag --endpoints=127.0.0.1:2379
```

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
