# kubecon-etcd-metrics-lab

## Prerequisites

Note: image is only for linux/amd64 and linux/arm64. Tested on mac and linux.

Start 3-node etcd cluster with grafana and prometheus:
```bash
docker-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up --force-recreate -V
```
To verify:
```bash
docker ps --all
```
Output:
```bash
➜  ~ docker ps --all
CONTAINER ID  IMAGE                                  COMMAND               CREATED         STATUS         PORTS                                              NAMES
8c2060085521  docker.io/bkanivets/etcd:v3.5.9  --name=etcd-1 --i...  26 seconds ago  Up 21 seconds  0.0.0.0:2379->2379/tcp, 0.0.0.0:11180->11180/tcp   kubecon-etcd-metrics-lab_etcd-1_1
e89e0d64213e  docker.io/bkanivets/etcd:v3.5.9  --name=etcd-2 --i...  25 seconds ago  Up 20 seconds  0.0.0.0:21180->11180/tcp, 0.0.0.0:22379->2379/tcp  kubecon-etcd-metrics-lab_etcd-2_1
163a3b97507f  docker.io/bkanivets/etcd:v3.5.9  --name=etcd-3 --i...  24 seconds ago  Up 19 seconds  0.0.0.0:31180->11180/tcp, 0.0.0.0:32379->2379/tcp  kubecon-etcd-metrics-lab_etcd-3_1
81db7802ceba  docker.io/prom/prometheus:latest       --config.file=/et...  23 seconds ago  Up 18 seconds  0.0.0.0:9090->9090/tcp                             kubecon-etcd-metrics-lab_prometheus_1
f04a2e245717  docker.io/grafana/grafana:latest                             21 seconds ago  Up 17 seconds  0.0.0.0:3000->3000/tcp                             kubecon-etcd-metrics-lab_grafana_1
```

Try running benchmark:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --key-space-size=100000 --total=10000
```

Check out default etcd [dashboard](http://localhost:3000/d/e3f3beda-14fe-47ad-a431-c8227c997a53/etcd-by-prometheus).

Grafana credentials are:
`admin`
`foobar`

## Scenario 0: Puts and Ranges
Explore sequence diagram for [Puts](https://mermaid.live/edit#pako:eNqdVVlv2kAQ_iurfUpUbJmYw-yDpbSpqkq9BG0iVZbQsjsEC7x21msCjfLfO2tOg3Gqwos91zffXH6hIpVAGc3hqQAl4C7mj5onkZqkK_JJZyJSkSL4y7g2sYgzrgxZ-qhw58sR6CXoSIGSkTo2ACNkXirdj_i4s7Mxh3xqziNqlLrfMJVz1VEsa7UxKhGrmM984T7cfrmYCM-yRQz63t_k8XUpxDlYglL3QccG7mN4vqCPlYRVDesJF3MUuu-5EbOfK5tgpHhhUlUkE8vfRqvWzgnDd7XFYuRHYa6ua32csCpgucEsQI5NarAIn5XYOdaGtpj7eiOOTrM0h2YXJ6zPMiud-SIfZ8g8Vo_H6HsQ54Tmro-MPNtSkyFwuSZixtVZDjtTG2HbYUZGfLnPdytEiL0aH8bTfK3EWBaamzhV4xxEqmTufp-UcWvI7oEqVA-ZHpiKNElic6j3CExzvIst3rC3g7kW9fSrXaubZkZurX9NBnsT61od7Mp0VVXW-GSQGfmlcj6FETw1uh2Woym-E57mYt_HWWG29byVspFNQ0HLUg4hLxZ4pGL-Zk0vjfVRtze4zb3-r0W5g7fX9GTPbVWRXZaqHP7tMiB_uai7DG85ktIT87y0Oicz4tRMzYdyUa6uCWDENWl7XpLXOzvhmfNWsN22xk3e_GmLJqATHkv8nL1YnIiaGSQQUYaPkut5RCP1inb2Jo_wPlBmdAEtWmSSm92nj7IptgmleNMpe6EryoLA9bpBZ9Dp9vpBv91u0TVljud2bwa-3_c7vW7XD7yb7muL_klTjNB2vU7P7_R7nWAwaAdeu4z2u9RZyNe_HrOfug).
Run `put` benchmark with 1000 clients:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=1000 --conns=1000 put --sequential-keys --key-space-size=100000 --total=100000000
```
Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.

Explore sequence diagram for [Ranges](https://mermaid.live/edit#pako:eNqFVMGO2jAQ_ZXIp65KIpKwCfiAVLVVtYe2ErQ9VJGQsQeIIA61nQBF_PuOA2EJZCG5WDNv3rx5mXhPeC6AUKLhXwGSw5eUzRXLEmnfab51vqk1t2cHnzVTJuXpmknjlCEmvGU5BlWCSiRIYWGXEDBc6CrtfcVjjbSsIzYzt5wKo94PFHSbuuCyqCOo6nmJMtuT6u8l57ckGUa9ETDxJ4XNO-lUCti2jjNlfInhiuDX9ugQK0wui2xq57J0TVfc4fBjqwkUDZBz-PDUWuUOmwGqDcoAMTG5YSvvRfK6sJXcdj07iZ1Q7osd6n6RO2xXqlf5ZqKQY1I5A_pWxbmZezVw_aWos1GpgUrKzuELJm-U1FD3jmdHEqtlbJiBz-1MTSNwI67MxohNNHbhCtLIuWd0ZYA1tEx1mkvdDkd0c1OoM8uVA4wvUHtJnd9SsxncbXglTlnsyfdPQrzb923YEehiZRoD31tEhK9xIHhg5tVattU-WmX8YmLVtsqPCp2qMpXziQaeS6G9n9NKpCWoX9IhGaiMpQKvtL0lTohZQAYJoXgUTC0TksgD4uyfO95JTqhRBXRIsRa4U6frj9AZW2mM4o9P6J5sCQ36XjfsDvp-Nwr7vSCOww7ZERrHXhBFfhj50aAXhtGhQ_7nORJ0vUH4HESBHz_3_NiPeoOK7W-VtC0Prw815sI).

Run `range` benchmark with 100 clients (while prior benchmark is still running):
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 range / --total=100000
```
Observe behavior at [Ranges](http://localhost:3000/d/ad0da30b-2128-4455-8cef-31424b06b7b9/ranges) dashboard.

## Scenario 1: delay fsync

Run `put` benchmark with 1000 clients (if prior benchmark stopped):
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=1000 --conns=1000 put --sequential-keys --key-space-size=100000 --total=1000000
```

### Add small delay
```bash
curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(100)'
```

Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.

### Add large delay
```bash
curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(1000)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(1000)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(1000)'
```

Run `range` benchmark with 100 clients (while 'put' benchmark is still running):
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 range / --total=100000
```

Observe behavior at [Ranges](http://localhost:3000/d/ad0da30b-2128-4455-8cef-31424b06b7b9/ranges) dashboard.

Try `range` benchmark with `--consistency=s`:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 --consistency=s range / --total=100000
```

## Scenario 2: delay network

Stop cluster after scenario 1.

Check out `-rx-delay` in docker-compose-etcd-bridge.yml. It should be set to 1000ms.

Start cluster with `bridge` interface:
```bash
docker-compose -f docker-compose-etcd-bridge.yml -f docker-compose-metrics.yml up --force-recreate -V
```

Check [Peers](http://localhost:3000/d/f3c3b742-47e8-4bda-8631-a1d540d0f130/peers) dashboard.

Run `put` benchmark with 1000 clients:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=1000 --conns=1000 put --sequential-keys --key-space-size=100000 --total=100000
```
Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.


## Scenario 3: reaching DB size limit
Stop previous cluster and start new:
```bash
docker-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up --force-recreate -V
```

Run `put` benchmark with increased val size:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=10000 --key-space-size=100000 --total=10000000
```

Default limit it 2Gb. Observe behavior at [Puts](http://localhost:3000/d/ac2a8573-2a57-4b18-a9fd-d007b565f5e6/puts) dashboard.

## Scenario 4: compaction
Compaction [docs](https://etcd.io/docs/v3.6/op-guide/maintenance/#history-compaction-v3-api-key-value-database).

Explore sequence diagram for [Compaction](https://mermaid.live/edit#pako:eNqdVW1v2jAQ_iuWP20aRAEKpP6A1I1pmta9qGydNEVCxj5KBLEz26EwxH_fOUCAEmjV5Et0fu655158WVGhJVBGLfzNQQnoJ_zB8DRWI70gn0wmYhUrgk_GjUtEknHlyLyFB8F0PgAzBxMrUDJWhwBwQtriMPiInzuc57zjY3fKaNAafEMpp0cHXB61ARURj2M-8lnw--b2rBCeZbMEzH1ro-PrXIjTYClagy_3Zw4SJWFRBvcvz51WeTry2XmX48rUe713laVg5INOMy7cm7eVfvXesYFZh1pADp12mOZnJXaOlfQ-bllRRn4YnWkLl13qvWqlWeHMZ3aYYd6JejiMXgapP0l11ylGHk3igNwBl0siJlydaNhBPcO2h4wM-LzUuzViiPIYP4Zju1RiKHPDXaLV0ILQStrg-6jgrUi2DHSU6l7pPlOh0zRx-3oPwF3mO9vmTfZ-9JaiOv3jrlXNKyM33r9CQQkpJGxn92S2tvZjjBUTkPkMtlis4Fl4MfUvYy2gvnxbzmHGcwtlk54255BjxMUUByx4z52Y_MR4Y20IcDEhBuZITPZKC8htgi1i5JeyfAx9mIGDi9Lk6HW6zhAUg1FJEKvXXLOD6dt09fLsveri9uH5tfFk7-yqfgc208rCy7YVzrmcVW2r5xxJ4Ylaq67z5qU1moJJeSLxr7XytDF1E0ghpgw_JTfTmMZqjTi_nAe4JChzJocazTPJ3e4PR9kYa4NWXO-UreiCskYzDMKo3eo2r5tXUXjVua7RJZrDRhC2G81O4zpsRO32VXddo_-0RoowiKJuFIZRs9XptsJup1Xw_SkOfdD1fzHMkrs).

Stop previous cluster and start new:
```bash
docker-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up --force-recreate -V
```

Run `put` benchmark with `compaction`:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000 --compact-index-delta=10000 --compact-interval=10s
```
Observe behavior at [Compaction](http://localhost:3000/d/eb5c5196-8de5-4435-9a7d-d2bb4da869f4/compaction) dashboard.

Compare database size total/in_use when running `put` benchmark without compaction:
```bach
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000
```

### Generating ErrTooManyRequests (optional)
Run `put` benchmark with increased value size and `compaction`
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=10000 --key-space-size=100000 --total=10000000 --compact-index-delta=1000 --compact-interval=30s
```

## Scenario 5: defragmentation
For explanation of defragmentation process see [docs](https://etcd.io/docs/v3.5/op-guide/maintenance/#defragmentation).

Stop previous cluster and start new:
```bash
docker-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up --force-recreate -V
```

Increase db size by running `put` benchmark:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=10000 --key-space-size=100000 --total=101000
```
Run `put` benchmark with `compaction`:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000 --compact-index-delta=10000 --compact-interval=10s
```

Run defrag on non-leader:
```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9 defrag --endpoints=127.0.0.1:32379
```
Observe behavior at [Defrag](http://localhost:3000/d/bb7bc45f-5370-401d-8212-3408c6936f5c/defrag) dashboard.


### Run defragmentation with delay

Make sure that `put` benchmark is running:
```bash
docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=100 --conns=100 put --sequential-keys --val-size=100 --key-space-size=100000 --total=10000000
```

Add delay:
```bash
curl http://127.0.0.1:11180/defragBeforeCopy -XPUT -d'sleep(10000)'
curl http://127.0.0.1:21180/defragBeforeCopy -XPUT -d'sleep(10000)'
curl http://127.0.0.1:31180/defragBeforeCopy -XPUT -d'sleep(10000)'
```

Run defrag for non-leader:
```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9 defrag --endpoints=127.0.0.1:32379
```

Observe behavior at [Defrag](http://localhost:3000/d/bb7bc45f-5370-401d-8212-3408c6936f5c/defrag) dashboard. Check out grpc error rate.

Run defrag for leader:
```bash
docker run --network="host" -it --entrypoint etcdctl --rm docker.io/bkanivets/etcd:v3.5.9 defrag --endpoints=127.0.0.1:2379
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
