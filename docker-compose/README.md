# :ship: JMeter Distributed Environment using Docker-Compose 
JMeter distributed environment can be created on local system using docker-compose.
This page describes the steps and dependencies to set up the JMeter distributed environment using docker-compose.
It is quite easy to set up the JMeter distributed environment using docker-compose rather than running 
individual docker containers as described on [JMeter Distributed Environment using Docker](../README.md)

---
**Video Tutorial:**

[Apache JMeter Distributed Environment using Docker-Compose](https://youtube.com/playlist?list=PLm9ZfKz5SwIhVL7A5T5M3TUvJKKZVDr-d)

---


## :oil_drum: Redis Certificate and Key :closed_lock_with_key:
Create certificate and key file by running below command:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
cd "$REPO_DIR\redis"
openssl req -newkey rsa:2048 -nodes -keyout redis-domain.key -x509 -days 365 -out redis-domain.crt
```

## :closed_lock_with_key: Import Redis certificate in Java Keystore
Run below command to import redis db server certificate as trusted certificate in Java keystore:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>";
keytool -import -file "$REPO_DIR\redis\redis-domain.crt" \
  -alias redis-server-ca \
  -keystore "$REPO_DIR\redis\redis-db-ca.jks" \
  -storepass "redis-cert@alpana" \
  -storetype JKS -noprompt
```

## :ship: JMeter Distributed Environment + :watch: InfluxDB + :chart_with_upwards_trend: Grafana
Create directories to store influxdb data on local system, by running below commands:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
mkdir "$REPO_DIR\docker-compose\influxdb\docker-volume"
```
These created directories will be mapped to influxdb and grafana docker containers created as part of docker-compose deployment.

Run below command to create JMeter distributed environment with InfluxDB and Grafana using docker-compose:
```shell
cd "$REPO_DIR\docker-compose"
docker-compose up -d
```

Once all docker containers are up and running, below service can be accessed on `localhost`:

| Service Name | URL                    |
|--------------|------------------------|
| redis        | localhost:6379         |
| influxdb     | http://localhost:9086/ |
| grafana      | http://localhost:3000/ |


## :bar_chart: Deploy InfluxDB dashboard (Optional)
File [apache_jmeter.yaml](./influxdb/dashboards/apache_jmeter.yaml) has been mapped to docker container volume. In order to deploy
the dashboard based on this template file, run below commands:
```shell
docker exec -it influxdb sh
influx apply --file /home/dashboards/apache_jmeter.yaml
```

## <img src="../icons/jmeter.svg" alt= “jmeter” width="20" height="20"> Run Test Plan from JMeter Master Container
* Run below command to ssh into `jmeter-master` docker container:
  ```shell
  docker exec -it jmeter-master sh
  ```
  
* Run below command to execute tests inside `jmeter-master` docker container:
  ```shell
  export TEST_REPORT_TIMESTAMP=$(TZ='GMT-1' date +%d%m%Y-%H%M%S);
  export TEST_RUN_TIMESTAMP=$(TZ='GMT-1' date +%d.%m.%Y-%H:%M:%S);
  jmeter -n \
    -t /jmeter/repo/tests/sample.jmx \
    -Rjmeter-server-1,jmeter-server-2 \
    -j /jmeter/repo/logs/sample-docker-compose-jmeter-master.log \
    -l /jmeter/repo/results/sample-docker-compose-$TEST_REPORT_TIMESTAMP.csv \
    -Jserver.rmi.ssl.disable=true \
    -G /jmeter/repo/configs/local-docker-compose-dev.properties \
    -q /jmeter/repo/configs/local-docker-compose-dev.properties \
    -Jtest.runId=R-$TEST_RUN_TIMESTAMP \
    -Ljmeter.engine=DEBUG
  ```