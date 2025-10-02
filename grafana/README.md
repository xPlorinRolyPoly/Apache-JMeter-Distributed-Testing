## :chart_with_upwards_trend: Grafana
Grafana is used to compose observability dashboards with everything from Prometheus & Graphite metrics, 
to logs and application data to power plants and beehives.

### :whale: Docker Setup
Create directory for storing grafana data in local system, by running below command:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
mkdir "$REPO_DIR\grafana\docker-volume"
```
This created directory will be mapped to the grafana data directory in grafana docker container.

Run below command to start Grafana dashboard:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
docker run -d --name=alpana-grafana \
  -p 3000:3000 \
  -v "$REPO_DIR\grafana\docker-volume":/var/lib/grafana \
  -e GF_SECURITY_ADMIN_USER=alpana \
  -e GF_SECURITY_ADMIN_PASSWORD=grafana@alpana \
  grafana/grafana:8.0.1
```

### :fireworks: UI Setup
Access URL:
```shell
http://localhost:3000/
```

Login Details:
* Username - alpana
* Password - grafana@alpana