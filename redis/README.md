# :oil_drum: Redis Database
Redis (REmote DIctionary Server) is an open source, in-memory data structure store, used as a database, cache and message broker. As an in-memory database, it keeps all the data mostly in the RAM. Redis enables high performance when reading/writing data, and is also useful when you need to ensure unique data is used across all test servers.

## :key: Using Docker with password 
* Run redis database using docker container, by running below command:
  ```shell
  docker run --name alpana-redis -p 6379:6379 -d redis redis-server --requirepass "redis@alpana"
  ```
* Run redis-cli with password inside docker container `alpana-redis`, by running below command:
  ```shell
  docker exec -it alpana-redis redis-cli -a 'redis@alpana' 
  ```

## :closed_lock_with_key: Using Docker with password and SSL certificate 
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
    -v "$REPO_DIR\redis\redis-domain.crt":/data/domain.crt \
    -v "$REPO_DIR\redis\redis-domain.key":/data/domain.key \
    -d redis redis-server \
    --requirepass "redis@alpana" --port 0 --tls-port 6379 --tls-cert-file domain.crt \
    --tls-key-file domain.key --tls-ca-cert-file domain.crt --tls-auth-clients no
  ```
  
* Access redis database on `localhost:6379` with password `redis@alpana` and certificate `$REPO_DIR\redis\redis-domain.crt`