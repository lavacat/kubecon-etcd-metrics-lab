# kubecon-etcd-metrics-lab


# Scenario 1: delay fsync

podman-compose -f docker-compose-etcd.yml -f docker-compose-metrics.yml up

curl http://127.0.0.1:11180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:21180/walBeforeFdatasync -XPUT -d'sleep(100)'
curl http://127.0.0.1:31180/walBeforeFdatasync -XPUT -d'sleep(100)'


docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=27 --conns=20 put --sequential-keys --key-space-size=100000 --total=100000

# Scenario 2: delay network

Adjust `-rx-delay` in docker-compose-etcd-bridge.yml

podman-compose -f docker-compose-etcd-bridge.yml -f docker-compose-metrics.yml up

docker run --network="host" -it --entrypoint benchmark --rm docker.io/bkanivets/etcd:v3.5.9-arm64 --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --clients=27 --conns=20 put --sequential-keys --key-space-size=100000 --total=100000

