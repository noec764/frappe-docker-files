
### Need to do before

[Install Docker](https://docs.docker.com/engine/install/ubuntu/)
[Post Install for development environement](https://docs.docker.com/engine/install/linux-postinstall/)
[Install docker-compose](https://docs.docker.com/compose/install/)
[Docker Compose Completion](https://docs.docker.com/compose/completion/)


### Create the Docker volume

Create directory if not exists
```sh
 mkdir -p  /opt/data/frappedev/mysql
```
Assuming now /opt/data/frappedev/mysql directory exists 
```sh
 docker volume create --driver local --opt type=none --opt device=/opt/data/frappedev/mysql --opt o=bind frappedev-mariadb-vol
```

Create directory if not exists
```sh
 mkdir -p  /opt/data/frappedev/redis_cache;
 mkdir -p  /opt/data/frappedev/redis_queue;
 mkdir -p  /opt/data/frappedev/redis_socketio; 
```

Assuming now /opt/data/frappedev/redis_cache/, /opt/data/frappedev/redis_queue/,  /opt/data/frappedev/redis_socketio directories exists
```sh
docker volume create --driver local --opt type=none --opt device=/opt/data/frappedev/redis_cache --opt o=bind frappedev-redis-cache-data;
docker volume create --driver local --opt type=none --opt device=/opt/data/frappedev/redis_queue --opt o=bind frappedev-redis-queue-data;
docker volume create --driver local --opt type=none --opt device=/opt/data/frappedev/redis_socketio --opt o=bind frappedev-redis-socketio-data;
```

### Network

To name it to use it for others application like metabase

```sh
docker network create frappe-network
```

### Directory sctruture

/home/..../frappe

/home/..../frappe_docker_dev => clone of https://github.com/frappe/frappe_docker

/home/..../frappe_docker_file => Clone of this repo

Then follow the step given by official documentation
[ERPNext/Frappe dev env install](https://github.com/frappe/frappe_docker/tree/develop/development)

### Run compose

```sh
    cd /home/..../frappe_docker_file/frappe
    docker-compose up
```

Enter into bash of the container
```sh
docker exec -e "TERM=xterm-256color" -w /workspace/development -it frappedev_frappe_1 bash
```

inside container bash

```sh
bench init --skip-redis-config-generation --frappe-branch develop frappe-bench
cd frappe-bench

bench set-mariadb-host mariadb
bench set-redis-cache-host redis-cache:6379
bench set-redis-queue-host redis-queue:6379
bench set-redis-socketio-host redis-socketio:6379

bench new-site [url without https:// probably local site name erpnext.local manage into /etc/hosts for local dev env] --mariadb-root-password 123 --admin-password admin --no-mariadb-socket --db-name [dbname]

bench get-app --branch develop erpnext https://github.com/frappe/erpnext.git
bench --site [url without https:// probably local site name erpnext.local manage into /etc/hosts for local dev env] install-app erpnext
bench --site [url without https:// probably local site name erpnext.local manage into /etc/hosts for local dev env] set-config developer_mode 1
bench --site [url without https:// probably local site name erpnext.local manage into /etc/hosts for local dev env] clear-cache
```

If you'r going to intall more than one site

```sh
bench config dns_multitenant on
```


Once first bench install is done you can revert docker-compose

```yaml
frappe:
    image: frappe/bench:latest
    # command: bash -c "cd frappe-bench;bench start"
    command: sleep infinity
```
by
```yaml
frappe:
    image: frappe/bench:latest
    command: bash -c "cd frappe-bench;bench start"
    # command: sleep infinity
```

