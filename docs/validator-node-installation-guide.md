# Validator node Installation Guide

Use this procedure for creating a new validator node for the Besu Network ("Red B").

> :information_source: *Validator nodes are used to collect transactions in blocks and muest not be used to send transactions.*



1. [Docker Installation](#docker)
2. [Besu Installation](#besu)
3. [Network Stats](#stats)
4. [Request access to the network & Validator Status](#access)
5. [Upgrade Besu](#upgrade-besu)

---



## <a name="docker"></a>1. Docker installation

```sh
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
$ sudo usermod -aG docker ubuntu
```

## Docker Images

Pull Besu Docker images and tag the image

```sh
$ docker image pull hyperledger/besu

$ docker image tag hyperledger/besu besu
```


## Create directories

```sh
$ mkdir -p $HOME/alastria-red-b/besu-node
$ cd $HOME/alastria-red-b/besu-node
$ mkdir data
```

---

## <a name="besu"></a>2. Besu Installation


### <a name="node_public_key"></a>Generate Node Public Key

```sh
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm besu --data-path=data public-key export --to=data/key.pub
```

> :information_source: *This value is your **Node Public Key***

#### <a name="node_address"></a>Check Node Address

```sh
$ cd $HOME/alastria-red-b/besu-node
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm besu --data-path=data public-key export-address --to=data/nodeAddress
```

> :warning: *Write down this value (it is your **Node Address**). You will need it for registering your Node as a Validator Node in the network*



### Generate genesis file

```sh
$ vi $HOME/alastria-red-b/besu-node/genesis.json
```

* Copy and paste from [genesis.json](../configs/genesis.json)
  * ([About our Genesis file](about-genesis-file.md))

### Generate config file

```sh
$ vi $HOME/alastria-red-b/besu-node/config.toml
```
* Copy and paste from [validator_node_config.toml](../configs/validator_node_config.toml), **changing p2p-host="127.0.0.1" to your node IP**


### Start Besu

Run Besu container

```sh
$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).
```

#### Check Besu logs (for detached mode)

```sh
$ docker container logs -f besu
```

#### <a name="enode"></a>Check enode
```sh
curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://127.0.0.1:8545
```

> :warning: *Write down this value (it is your **enode**). You will need it for registering your Node as a **Whitelisted Node** in the network*



## <a name="access"></a>3. Request access to the network & Validator Status

1. [Get your **enode**](#enode)
2. [Get your **Node Address**](#node_address)

3. Send to Alastria Besu Core Team
   - [ ] :warning: _HOW?? (TBD)_
   - your **enode** (for registering your Node as a **Whitelisted Node** in the network)
   - your **Node Address** (for voting up your node as a Validator)


## <a name="stats"></a>4. Network Stats

If you want your node to be displayed in [Red B Network Monitoring](http://52.48.45.179), you have to run the following command, changing __email__ and __node_name__:

:info: Only validators and selected nodes will be displayed in [Red B Network Monitoring](http://52.48.45.179)

```sh
docker run -d --restart=always --name ethstats-client --net host --entrypoint "./bin/ethstats-cli.js" alethio/ethstats-cli --register --account-email <email> --node-name <node_name> --server-url http://52.48.45.179:3000 --client-url ws://127.0.0.1:8546
```


## 5. Upgrade Besu

To upgrade besu to the latest version, simply run: 

```sh
$ docker image pull hyperledger/besu:latest
$ docker container stop besu
$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).
```