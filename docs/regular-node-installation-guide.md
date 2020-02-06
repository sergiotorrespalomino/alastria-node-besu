# Regular node Installation Guide

Use this procedure for creating a new regular node for the Besu Network ("Red B"), including a adjacent Orion node for registering private transactions.

> :information_source: *Regular nodes are used to receive transactions for the Blockchaion network, and dispacth them to the validators in order to collect them in blocks.*



1. [Docker Installation](#docker)
2. [Orion Installation](#orion)
3. [Besu Installation](#besu)
4. [Request access to the network](#access)
5. [Upgrade Besu](#5-upgrade-besu)

---



## <a name="docker"></a>1. Docker installation

```sh
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
$ sudo usermod -aG docker ubuntu
```

## Docker Images

Pull Orion and Besu Docker images and tag the images

```sh
$ docker image pull pegasyseng/orion:1.5.0-SNAPSHOT
$ docker image pull hyperledger/besu

$ docker image tag pegasyseng/orion:1.5.0-SNAPSHOT orion
$ docker image tag hyperledger/besu besu
```


## Create directories

```sh
$ mkdir -p $HOME/alastria-red-b/besu-node
$ cd $HOME/alastria-red-b/besu-node
$ mkdir data orion
```

---

## <a name="orion"></a>2. Orion Installation


### Prerequisites

Variable | Description
-------- | ------------
`YOUR_PASSWORD` | Choose a password to encrypt the Orion node key pair
`YOUR_ORION_NODE_IP` | Your Orion node IP




### Create password and config files for Orion
```sh
$ cd $HOME/alastria-red-b/besu-node/orion
$ echo "YOUR_PASSWORD" > passwordFile
$ chmod 660 passwordFile
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --name orion --rm orion -g nodekey
$ vi $HOME/alastria-red-b/besu-node/orion/orion.conf
```

* Copy and paste from [orion.conf](../configs/orion.conf), **changing nodeurl = "http://YOUR_ORION_NODE_IP:8080/" to your node IP**


<!--```toml
nodeurl = "http://YOUR_ORION_NODE_IP:8080/" # URL advertised to Orion nodes. Required
nodeport = 8080 # Port on which to listen for Orion nodes. Required
nodenetworkinterface = "0.0.0.0" # Host on which to listen for Orion nodes
clienturl = "http://127.0.0.1:8888" # URL advertised to Ethereum clients
clientport = 8888 # Port on which to listen for Ethereum clients
clientnetworkinterface = "0.0.0.0" # Host on which to listen for Ethereum clients 	
publickeys = ["nodekey.pub"] # List of files containing public keys hosted by node
privatekeys = ["nodekey.key"] # List of files containing private keys hosted by node (corresponding order to public keys)
passwords = "passwordFile" # File containing passwords to unlock privatekeys. Include an empty line for keys that are not locked.
othernodes = ["http://18.202.38.195:8080/", "http://52.16.154.220:8080/", "http://158.176.139.92:8080/", "http://5.153.57.78:8080/"] # Bootnodes for Orion network
tls = "off" # TLS status options
```-->



### Start Orion

Run Orion container

```sh
$ cd $HOME/alastria-red-b/besu-node/orion
$ docker container run -d -v `pwd`:`pwd` -w `pwd` -p 8080:8080 -p 8888:8888 --name orion orion orion.conf
# Add -d to run in detached mode (background).
```

Check Orion logs (for detached mode)

```sh
$ docker container logs -f orion
```

<!--Expose the ports you specified in file __orion.conf__ (make sure you have those ports open in your instance)
To stop __orion__, you need to kill the container and, if you need, remove it (Although, if you append the flag --rm to the docker container run command, it deletes the container once it is killed). In this case:-->

### Confirm Orion is Running

```sh
curl http://localhost:8888/upcheck
# Must return 'I'm up!'
```

### Kill Orion

* _To stop __orion__, you need to stop the container and remove it_

```sh
$ docker container stop orion
$ docker container rm orion
```




### <a name="signer_key"></a>Create Privacy Marker Transaction Signing Key file (signer.key)

 
You will need a Signing Key file for registering the marker of the Orion private transactions 

```sh
$ vi $HOME/alastria-red-b/besu-node/signer.key
$ chmod 660 $HOME/alastria-red-b/besu-node/signer.key
```
* Create a new private-public key pair and copy and paste **private key** of this pair in this file (**SignerKey Private Key**). For example, with MetaMask.

> :warning: *Write down the value of the **address** associated to this key pair (it is your **SignerKey Address**). You will need it for registering as a Whitelisted Account in the network*

---

## <a name="besu"></a>3. Besu


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
* Copy and paste from [regular_node_config.toml](../configs/regular_node_config.toml), **changing p2p-host="127.0.0.1" to your node IP**
* :warning: The configuration of [regular_node_config.toml](../configs/regular_node_config.toml) restrict access to localhost (rpc-http-host="127.0.0.1", rpc-ws-host="127.0.0.1"). Change this configuration to open access to everyone (rpc-http-host="0.0.0.0", rpc-ws-host="0.0.0.0").


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


## <a name="access"></a>4. Request access to the network

1. [Get your **enode**](#enode)
2. [Get your **SignerKey Address**](#signer_key)
2. Send to Alastria Besu Core Team (for registering your node)
   - [ ] :warning: _HOW?? (TBD)_
   - your **enode** (for registering your Node as a **Whitelisted Node** in the network)
   - your **SignerKey Address** (for adding to the Accounts Whitelist)
   - **any Address** you want to send transactions from (for adding to the Accounts Whitelist)


## 5. Upgrade Besu

To upgrade besu to the latest version, simply run: 

```sh
$ docker image pull hyperledger/besu:latest
$ docker container stop besu
$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).
```