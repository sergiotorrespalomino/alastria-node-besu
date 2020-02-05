# Ethstats Server and Dashboard installation guide

Use this procedure to create an Ethstats Server that recieves info from nodes and a dashboard to display it.

1. [Docker Installation](#docker-installation)
2. [Docker Compose Installation](#docker-compose-installation)
3. [Ethstats Server Installation](#ethstats-server-installation)
4. [Restart Ethstats Server](#restart-ethstats-server)
5. [Required Ports](#required-ports)

## Docker Installation

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
```

## Docker Compose Installation

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

## Ethstats Server Installation

You will have to edit the `docker-compose.yml` file to allow remote connections and also change the id and name of the network

```sh
git clone https://github.com/Alethio/ethstats-network-server.git
cd ethstats-network-server/docker/lite-mode/memory-persistence
vi docker-compose.yml
docker-compose up -d
```

[Edited docker-compose.yml to allow remote connections](../configs/docker-compose.yml)

```yml
version: '3.7'
services:
  server:
    container_name: ethstats-network-server
    image: alethio/ethstats-network-server:latest
    restart: always
    depends_on:
      - deepstream
    ports:
      - 0.0.0.0:3000:3000
      - 0.0.0.0:3030:3030
      - 127.0.0.1:8888:8888
    environment:
      - NETWORK_ID=2020
      - NETWORK_NAME=alastria-besu
      - LITE=1
      - LITE_DB_PERSIST=0
      - LITE_API_PORT=3030
      - APP_PORT=3000
      - METRICS_PORT=8888
      - DEEPSTREAM_HOST=deepstream
    command: ["./bin/app.js", "-v"]
  deepstream:
    container_name: ethstats-network-deepstream
    image: deepstreamio/deepstream.io:3.2.2
    restart: always
    ports:
      - 0.0.0.0:6020:6020
  dashboard:
    container_name: ethstats-network-dashboard
    image: alethio/ethstats-network-dashboard:latest
    restart: always
    depends_on:
      - server
    volumes:
      - ../../config/nginx/conf.d:/etc/nginx/conf.d
    ports:
      - 0.0.0.0:80:80
    environment:
      - NETSTATS_API_URL=http://52.48.45.179:3030
      - DS_URL=52.48.45.179:6020
      - DS_USER=frontend
      - DS_PASS=
```

## Restart Ethstats Server

```sh
cd ethstats-network-server/docker/lite-mode/memory-persistence
docker-compose down -v
docker-compose up -d
```

## Required Ports

Your instance must have the following ports open:

* 22 (ssh)
* 80
* 6020
* 3030
* 3000