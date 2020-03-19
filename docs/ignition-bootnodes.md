# Alastria Besu Network (Red B) - Ignition Process

We have called "Ignition" to the process of generation of the first 4 bootnodes / validators for the Besu Network ("Red B"). This process include the creation of the network (using Docker) with on-chain permissioning, and the setting of 4 Orion services (as Orion bootnodes)



1. [Docker Installation](#docker)
2. [Orion Installation](#orion)
3. [Besu Installation](#besu)
4. [Permissioning Configuration](#perm)
5. [Network Stats](#stats)
6. [Upgrade Besu](#upgrade-besu)
I. [Annex - January 2020 reboot](#annex-january-2020-reboot)

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
$ docker image pull pegasyseng/orion:1.5.1-SNAPSHOT
$ docker image pull hyperledger/besu

$ docker image tag pegasyseng/orion:1.5.1-SNAPSHOT orion
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

> :information_source: *Besu bootnodes have adjacent Orion instances to be able to function as Orion bootnodes*

### Prerequisites

Variable | Description
-------- | ------------
`YOUR_PASSWORD` | Choose a password to encrypt the Orion node key pair
`YOUR_ORION_NODE_IP` | Your Orion node IP




### Create password and config files for Orion
```sh
$ cd $HOME/alastria-red-b/besu-node/orion
$ echo "YOUR_PASSWORD" > passwordFile
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


In all the bootnodes, we have created a Signing Key file for registering the marker of the Orion private transactions

```sh
$ vi $HOME/alastria-red-b/besu-node/signer.key
```
* Copy and paste in this file **the SignerKey of this node**


---

## <a name="besu"></a>3. Besu


### <a name="node_public_key"></a>Generate Node Public Key & Node Address

```sh
$ cd $HOME/alastria-red-b/besu-node
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm besu --data-path=data public-key export-address --to=data/nodeAddress
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm besu --data-path=data public-key export --to=data/key.pub
```



### Generate genesis file

```sh
$ vi $HOME/alastria-red-b/besu-node/genesis.json
```

* Copy and paste from [genesis.json](../configs/genesis.json)
  * ([About our Genesis file](about-genesis-file.md))

### Generate config files

* provisional_config.toml
* config.toml
* data/permissions_config.toml


#### Provisional config file

```sh
$ vi $HOME/alastria-red-b/besu-node/provisional_config.toml
```
* Copy and paste from [provisional_config.toml](../configs/provisional_config.toml), **changing p2p-host="127.0.0.1" to your node IP**


#### Final config file

```sh
$ vi $HOME/alastria-red-b/besu-node/config.toml
```
* Copy and paste from [validator_node_config.toml](../configs/validator_node_config.toml), **changing p2p-host="127.0.0.1" to your node IP**


#### Permissions (provisional) config file


```sh
$ vi $HOME/alastria-red-b/besu-node/data/permissions_config.toml
```

* Copy and paste from  [permissions_config.toml](../configs/permissions_config.toml)

### Start Besu

Run Besu container with **provisional config** ([provisional_config.toml](../configs/provisional_config.toml))


```sh
$ cd $HOME/alastria-red-b/besu-node
$ docker container run -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="provisional_config.toml"
# Add -d to run in detached mode (background).
```

---

## <a name="perm"></a>4. Permissioning & Privacy Configuration

### Deploy permissioning Smart Contract

After starting the nodes, we have deployed the Permissioning Smart Contracts usign our first Admin Account.

#### permissioning-smart-contracts repo

```sh
$ cd $HOME/alastria-red-b/besu-node
$ git clone https://github.com/PegaSysEng/permissioning-smart-contracts.git
$ cd $HOME/alastria-red-b/besu-node/permissioning-smart-contracts
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm node:12 yarn
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm -e "BESU_NODE_PERM_ACCOUNT=0xC1dbb16EfC26917d8179F6Ea14c2EF037622E51a" -e "BESU_NODE_PERM_KEY=<Account_private_key>" -e "ACCOUNT_INGRESS_CONTRACT_ADDRESS=0x0000000000000000000000000000000000008888" -e "NODE_INGRESS_CONTRACT_ADDRESS=0x0000000000000000000000000000000000009999" -e "BESU_NODE_PERM_ENDPOINT=http://52.16.154.220:8545" -e "NETWORK_ID=2020" -e "INITIAL_ADMIN_ACCOUNTS=0xC1dbb16EfC26917d8179F6Ea14c2EF037622E51a,0x969F24a839B478c0C44B96dEC38A460FC1ef2D2e,0xC5E0Aae3C913c76F6338835e4D803f8774Fa0bA5" -e "INITIAL_WHITELISTED_ACCOUNTS=0x3C1eB70aE4e6EECF5353Ce0104b56d27f819409B,0x3af5eEC8856c4E6bc8479A879FDc7b3D8D89f8a9,0xC44d78f4d822adC21D2f5261Bb9a2a176838807C,0xF3E77F2d04B93a0e1b0655F7504a9739f3a1c1eF,0x377cE19acD4B3b39D2f05EebA8c7894E8166cd50,0xC5E0Aae3C913c76F6338835e4D803f8774Fa0bA5,0xfF107792c0a01765F4c8FaE7d5990c8E0Fd1e2b5" -e "INITIAL_WHITELISTED_NODES=enode://5e15792a10fefc24bf495c44896734a177ed01857b8b162879b317c5fdcd7f16a0cb8af877a6bb510dc0eb15990cfa92deaa85a87c9edd03e833d928eb6f9f78@18.202.38.195:30303","enode://f5a31862af9adbe702562481663c0fecfc3fa4a6e5a21b16907e23e537f641d40ee361880d68e4d5d97a105a06fe6775c4fca07e507d0ea322018dc0f754546b@52.16.154.220:30303","enode://abe1d7baf32e88849526f01369bd432119188ff3f8e369d1abab89e75e14557c33322ea8eced2ab2b4722cf3e3301ef5af9385411c509060e97aa35f1ea1b60c@158.176.139.92:30303","enode://76af726fa65c4a1fb6e150960eda7e5ac12a58f34dc544911190c44ee32dff3357a5c323639626e9b35e5c3fef4e2a07b21dc82d099560808ae8e4ae870425d5@5.153.57.78:30303" node:12 yarn truffle migrate --reset
```

* *(Note: the permissioning-smart-contracts can be cloned outside the `besu-node` directory)*
* *In this deployment, we changed <Account_private_key> for the real value*

__Optional__. To launch the dApp, run
```sh
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm -e "ACCOUNT_INGRESS_CONTRACT_ADDRESS=0x0000000000000000000000000000000000008888" -e "NODE_INGRESS_CONTRACT_ADDRESS=0x0000000000000000000000000000000000009999" -e "BESU_NODE_PERM_ENDPOINT=http://52.16.154.220:8545" -e "NETWORK_ID=2020" -p 3000:3000 node:12 yarn start
```


### Restart nodes with On-chain Permissioning

In all the bootnodes, we have stopped and removed the container

```sh
$ docker container stop besu
$ docker container rm besu
```

Then we have run the Besu containers with the **final config** files ([validator_node_config.toml](../configs/validator_node_config.toml))

```sh
$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).
```


## <a name="stats"></a>5. Network Stats

If you want your node to be displayed in [Red B Network Monitoring](http://52.48.45.179), you have to run the following command, changing __email__ and __node_name__:

:info: Only validators and selected nodes will be displayed in [Red B Network Monitoring](http://52.48.45.179)

```sh
docker run -d --restart=always --name ethstats-client --net host --entrypoint "./bin/ethstats-cli.js" alethio/ethstats-cli --register --account-email <email> --node-name <node_name> --server-url http://52.48.45.179:3000 --client-url ws://127.0.0.1:8546
```


## 6. Upgrade Besu

To upgrade besu to the latest version, simply run: 

```sh
$ docker image pull hyperledger/besu:latest
$ docker container stop besu
$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).
```


## I. Annex - January 2020 reboot

### Network Reboot Process

1. Download latest versions (Besu & Orion)
2. Stop validation nodes (Besu & Orion)
3. Change genesis file to [current version](../configs/genesis.json)
4. Delete Besu database & Orion database
5. Start Orion nodes
6. Start besu nodes with Local Permissioning
7. Deploy permissioning Smart Contracts
8. Restart nodes with OnChain Permissioning
9. Restart Ethstats

```sh
$ docker image pull hyperledger/besu:1.4
$ docker image tag hyperledger/besu:1.4 besu
$ docker image pull pegasyseng/orion:1.5.1-SNAPSHOT
$ docker image tag pegasyseng/orion:1.5.1-SNAPSHOT orion

$ docker container stop besu
$ docker container stop orion

$ cd $HOME/alastria-red-b/besu-node
$ curl -LJO https://github.com/Sngular-Blockchain/Alastria-Besu-Core-Team/blob/master/configs/genesis.json

$ sudo rm -r data/DATABASE_METADATA.json data/database/ data/uploads/
$ sudo rm -r orion/file-uploads/ orion/routerdb/

$ cd $HOME/alastria-red-b/besu-node/orion
$ docker container run -d -v `pwd`:`pwd` -w `pwd` -p 8080:8080 -p 8888:8888 --name orion orion orion.conf
# Add -d to run in detached mode (background).

$ cd $HOME/alastria-red-b/besu-node/
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="provisional_config.toml"
# Add -d to run in detached mode (background).

# (Wait for network sync:)
$ curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":53}' http://127.0.0.1:8545
# (Deploy permissioning-smart-contracts repo)

$ cd $HOME/alastria-red-b/besu-node/
$ docker container stop besu
$ docker container rm besu
$ docker container run -d -v `pwd`:`pwd` -w `pwd` --network host --name besu besu --config-file="config.toml"
# Add -d to run in detached mode (background).

# Changed variables in each node!
$ docker container stop ethstats-client
$ docker container rm ethstats-client
$ docker run -d --restart=always --name ethstats-client --net host alethio/ethstats-cli --register --account-email <email> --node-name <node_name> --server-url http://52.48.45.179:3000 --client-url ws://127.0.0.1:8546
```