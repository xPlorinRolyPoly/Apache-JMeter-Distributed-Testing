## :watch: Influx DB
Ingest metrics, events and logs in a high-performing time series database capable of ingesting millions of 
data points per second.

### :whale: Docker Setup
Create directory for storing influxdb data in local system, by running below command:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
mkdir "$REPO_DIR\influxdb\docker-volume"
```
This created directory will be mapped to the influxdb data directory in influxdb docker container.

Run below command to start Influx DB on docker:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
docker run -d --name alpana-influxdb \
  -p 9086:8086 \
  -v "$REPO_DIR\influxdb\docker-volume":/var/lib/influxdb2 \
  -v "$REPO_DIR\influxdb\dashboards":/home/dashboards \
  -e DOCKER_INFLUXDB_INIT_MODE=setup \
  -e DOCKER_INFLUXDB_INIT_USERNAME=alpana \
  -e DOCKER_INFLUXDB_INIT_PASSWORD=influxdb@alpana \
  -e DOCKER_INFLUXDB_INIT_ORG=alps \
  -e DOCKER_INFLUXDB_INIT_BUCKET=jmeter \
  influxdb:2.0.6-alpine
```

### :bar_chart: Deploy InfluxDB dashboard
File [apache_jmeter.yaml](./dashboards/apache_jmeter.yaml) has been mapped to docker container volume. In order to deploy
the dashboard based on this template file, run below commands:
```shell
docker exec -it alpana-influxdb sh
influx apply --file /home/dashboards/apache_jmeter.yaml
```

### :fireworks: UI Setup
Access URL:
```shell
http://localhost:9086/
```

Login details:
* Username - alpana
* Password - influxdb@alpana

API Token:
Below API token is automatically created.
* name - alpana's Token