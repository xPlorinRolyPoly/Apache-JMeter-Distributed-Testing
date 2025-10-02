# :whale: JMeter - Docker Image

Login to docker hub repository, by running below command:
```shell
docker login --username <username> --password <password>
```

Run below command to build JMeter docker image and push to the docker hub repository:
```shell
export REPO_DIR="<path\to\apache-jmeter-distributed-testing>"
cd "$REPO_DIR\docker-image"
docker build -t apache-jmeter .
docker tag apache-jmeter:latest alpanachaphalkar/jmeter:latest
docker push alpanachaphalkar/jmeter:latest
```