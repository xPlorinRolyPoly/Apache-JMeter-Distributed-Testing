# :ship: JMeter Distributed Environment using Docker
JMeter distributed environment can be created on local system using docker containers. 
This page describes the steps and dependencies to set up the JMeter distributed environment using docker containers. 

---
**Video Tutorial:**

[Apache JMeter Distributed Testing with Docker + InfluxDB + Grafana](https://youtube.com/playlist?list=PLm9ZfKz5SwIi5WgXkSFjgbiMYAA_EmHiV)

---


## :spider_web: Docker Network Setup
Create a custom docker network by running below command:
```shell
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  jmeter
```
In this network, JMeter master container and server container/s will be created.

## :oil_drum: Redis DB Setup
It is necessary that redis database is up and running while executing tests because test data will be sent to redis database during test execution.
Redis database should be SSL secured with the password.

To run redis database, follow below steps:
* Download `Git Bash` shell which will have OpenSSL utility

* Create certificate and key file by running below command:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  cd "$REPO_DIR\redis"
  openssl req -newkey rsa:2048 -nodes -keyout redis-domain.key -x509 -days 365 -out redis-domain.crt
  ```

* Run redis DB as docker container with SSL, using below command:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  docker run -it --name alpana-redis -p 6379:6379 \
    --net jmeter --ip 172.28.5.30 \
    -v "$REPO_DIR\redis\redis-domain.crt":/data/domain.crt \
    -v "$REPO_DIR\redis\redis-domain.key":/data/domain.key \
    -d redis redis-server \
    --requirepass "redis@alpana" --port 0 --tls-port 6379 --tls-cert-file domain.crt \
    --tls-key-file domain.key --tls-ca-cert-file domain.crt --tls-auth-clients no
  ```

* Access redis database on `localhost:6379` with password `redis@alpana` and certificate `$REPO_DIR\redis\redis-domain.crt`

## :watch: InfluxDB Setup
It is necessary that InfluxDB is up and running while executing tests because test result metrics will be sent to InfluxDB during test execution.

To run influxdb, follow below steps:
* Create directory for storing influxdb data in local system, by running below command:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  mkdir "$REPO_DIR\influxdb\docker-volume"
  ```
  This created directory will be mapped to the influxdb data directory in influxdb docker container.

* Run below command to start Influx DB on docker:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  docker run -itd --name alpana-influxdb -p 9086:8086 \
    --net jmeter --ip 172.28.5.31 \
    -v "$REPO_DIR\influxdb\docker-volume":/var/lib/influxdb2 \
    -v "$REPO_DIR\influxdb\dashboards":/home/dashboards \
    -e DOCKER_INFLUXDB_INIT_MODE=setup \
    -e DOCKER_INFLUXDB_INIT_USERNAME=alpana \
    -e DOCKER_INFLUXDB_INIT_PASSWORD=influxdb@alpana \
    -e DOCKER_INFLUXDB_INIT_ORG=alps \
    -e DOCKER_INFLUXDB_INIT_BUCKET=jmeter \
    -e DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=ScHCbdo6A2goQ9afT3iGnh4VoZUvViEwmH8dpcGWm45B2mfAN2n1EM33oG5otv2cCTvkiO92-CFpC9wHpQXNVQ== \
    influxdb:2.0.6-alpine
  ```

* File [apache_jmeter.yaml](./influxdb/dashboards/apache_jmeter.yaml) has been mapped to docker container volume. In order to deploy
  the dashboard based on this template file, run below commands:
  ```shell
  docker exec -it alpana-influxdb sh
  influx apply --file /home/dashboards/apache_jmeter.yaml
  ```

* Access Influx database on http://localhost:9086/ with username `alpana` and password `influxdb@alpana`

## :bar_chart: Grafana Setup (Optional)
It is NOT necessary to run grafana during or after test execution, because JMeter tests do not interact with grafana.
But grafana offers user-friendly dashboard based on influxdb as datasource. 
So, to see test results/metrics stored in influxdb with user-friendly dashboard, grafana can be used.

To run grafana, follow below steps:
* Create directory for storing grafana data in local system, by running below command:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  mkdir "$REPO_DIR\grafana\docker-volume"
  ```
  This created directory will be mapped to the grafana data directory in grafana docker container.

* Run below command to start Grafana dashboard:
  ```shell
  export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
  docker run -d --name=alpana-grafana -p 3000:3000 \
    --net jmeter --ip 172.28.5.32 \
    -v "$REPO_DIR\grafana\provisioning":/etc/grafana/provisioning \
    -v "$REPO_DIR\grafana\dashboards":/var/lib/grafana/dashboards \
    -e GF_SECURITY_ADMIN_USER=alpana \
    -e GF_SECURITY_ADMIN_PASSWORD=grafana@alpana \
    grafana/grafana:8.0.1
  ```

* Access Grafana on http://localhost:3000/ with username `alpana` and password `grafana@alpana`

## :closed_lock_with_key: Create Java keystore
Run below command to import redis db server certificate as trusted certificate in Java keystore:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>";
keytool -import -file "$REPO_DIR\redis\redis-domain.crt" \
  -alias redis-server-ca \
  -keystore "$REPO_DIR\redis\redis-db-ca.jks" \
  -storepass "redis-cert@alpana" \
  -storetype JKS -noprompt
```

## :package: Start JMeter Slave(Server) Container/s
Run below command to start JMeter server (slave) container with IP address:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
touch "$REPO_DIR\logs\sample-docker-server.log"
docker run -dit --name jmeter-server \
  --net jmeter --ip 172.28.5.20 \
  -v $REPO_DIR:/jmeter/repo \
  -v "$REPO_DIR\redis\redis-db-ca.jks":/tmp/server-ca.jks \
  -v "$REPO_DIR\logs\sample-docker-server.log":/jmeter/apache-jmeter-5.4.3/jmeter-server.log \
  alpanachaphalkar/jmeter:latest server
```

## :package: Start JMeter Master(Client) Container
Run below command to start JMeter master container with IP address:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
docker run -it --name jmeter-master \
  --net jmeter --ip 172.28.5.10 \
  -v $REPO_DIR:/jmeter/repo \
  -v "$REPO_DIR\redis\redis-db-ca.jks":/tmp/server-ca.jks \
  alpanachaphalkar/jmeter:latest sh
```

## <img src="./icons/jmeter.svg" alt= “jmeter” width="20" height="20"> Run Test Plan from JMeter Master Container
* Run below command to execute tests inside container `jmeter-master`:
  ```shell
  export TEST_REPORT_TIMESTAMP=$(TZ='GMT-1' date +%d%m%Y-%H%M%S);
  export TEST_RUN_TIMESTAMP=$(TZ='GMT-1' date +%d.%m.%Y-%H:%M:%S);
  jmeter -n \
    -t /jmeter/repo/tests/sample.jmx \
    -R172.28.5.20 \
    -j /jmeter/repo/logs/sample-docker-master.log \
    -l /jmeter/repo/results/sample-docker-distributed-$TEST_REPORT_TIMESTAMP.csv \
    -Jserver.rmi.ssl.disable=true \
    -G /jmeter/repo/configs/local-docker-dev.properties \
    -q /jmeter/repo/configs/local-docker-dev.properties \
    -Jtest.runId=R-$TEST_RUN_TIMESTAMP \
    -Ljmeter.engine=DEBUG
  ```
  