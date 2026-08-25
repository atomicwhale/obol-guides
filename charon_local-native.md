# Running Charon with Native Execution/Consensus Clients

## Introduction

This guide is for people who want to run Charon on an existing machine with local execution/consensus clients (EC/CC) running as system service (systemd). e.g. You have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar) when setting up your node.

*This guide has been tested on CDVN v1.10.3 and Debian 12 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*

  
**TLDR**  
In this configuration, EC/CC are already running as a system service (systemd) and can be reached on localhost.  
We will setup Charon in docker using the Obol official docker package and point it to local BN.

## Configuration
### 0. Initial setup  
1. Please follow the [official guide](https://docs.obol.org/run-a-dv/start/create-a-dv-with-a-group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
Stop **AFTER** you created the `.env` file in Step 4, modify you configurations following the guide below.  

*Make sure you are under the charon folder, it should be `charon-distributed-validator-node` by default)*  
```
cd ~/charon-distributed-validator-node
```

### 1. Check the local Beacon node is reachable  
1. First, your BN need to be configured to expose the validator REST port (e.g. 5052) to localhost at `0.0.0.0`. Refer to the client' guide for this. (Binding the BN to localhost or `127.0.0.1` does not always work on some system and docker compose, you can bind it to `0.0.0.0` as a workaround. Beware of the risks, and firewall this port!)    
For example, the BN may need the following flags  
```
# Nimbus
--rest-address=127.0.0.1
# Lighthouse
--http-address=127.0.0.1
# Teku
--validator-api-interface=127.0.0.1
# Lodestar
--rest.address=127.0.0.1
# Prysm
--rpc-host=127.0.0.1
```
For EthPillar, it can be done by going to `Consensus Client` - `8 Expose consensus client RPC Port`  
![Alt text](static/img/ethpillar.png?raw=true)

Binding the BN to `0.0.0.0` is a workaround, beware of the risk associated with it: [Read more](https://dev.to/mjnaderi/accessing-host-services-from-docker-containers-1a97)  
Make sure this port is protected behind a firewall, because you don't want random people on the internet to connect to it. Please refer to other firewall guides for this.  

2. Check the local Beacon node is reachable at `localhost:<port-number>`
For example
```
curl http://localhost:5052/eth/v1/node/syncing
```
You should see something like this:  
> {"data":{"head_slot":"XXXXXXXX","sync_distance":"0","is_syncing":false,"is_optimistic":false,"el_offline":false}}

which indicates that the beaconnode is reachable at localhost, and `sync_distance` at (0 or 1) and `is_syncing` is (false) which suggest the consensus client is fully synced.  

3. **Again** Make sure this port is protected behind a firewall (if binding BN to `0.0.0.0`), because you don't want random people on the internet to connect to it. Please refer to other firewall guides for this.  

### 2. Disable EC/CC and mev-boost included in the Charon docker package, and configure Charon to use local Beacon node
(This step is taken from the official guide, you can find it in the official guide [Step 4: Existing BN](https://docs.obol.org/run-a-dv/start/create-a-dv-with-a-group#step-4-start-your-distributed-validator-node)  
1. (Go into the CDVN folder) Modify `.env` file using an editor (`nano` for example)  
```
nano .env
```

2. Uncomment EL=el-none, CL=cl-none and MEV=mev-none variables in the .env file and comment out EL=el-nethermind,  CL=cl-lighthouse and MEV=mev-mevboost variables, so it looks like this:  
```
#EL=el-nethermind
...
EL=el-none
...
#CL=cl-lighthouse
...
CL=cl-none
...
#MEV=mev-mevboost
...
MEV=mev-none
```

3. Configure Charon to use local Beacon node (Consensus client)  
Uncomment and set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file  
The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://host.docker.internal:5052
```
Use `Ctrl+O` and `Ctrl+X` to save and exit if using `nano`.  

### 3. Enable Charon to connect to host network
1. Create a docker compose override file, for example `custom.yml`  
```
cp docker-compose.override.yml.sample custom.yml
```

2. Modify `charon` section in `custom.yml` file  
```
nano custom.yml
```
Uncomment line `services`  
Uncomment line `charon` under the service section, and uncomment additional network configureation here.  
The section in the override file should now look like this:  
```
  charon:
  ...
    # Uncomment the extra_hosts section if you are trying to communicate with a CL running in a different docker network on the same machine 
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
Use `Ctrl+O` and `Ctrl+X` to save and exit if using `nano`.  

3. Configure the override file in the `.env` file
We are back at editing the `.env` file again
```
nano .env
```
Find the line whcih starts with COMPOSE_FILE and add `:custom.yml` to the end.  
It shoud now look like this:  
```
# The actual adjustable values are specified above
...
COMPOSE_FILE=compose-el.yml:compose-cl.yml:compose-vc.yml:compose-mev.yml:docker-compose.yml:custom.yml
```
Use `Ctrl+O` and `Ctrl+X` to save and exit if using `nano`.  

### 4. Start Charon  
1. Start Charon by running  
```
docker compose up -d
```

2. Check the logs of the Charon container by using `docker logs <charon-container-name> -f`, for example:  
```
docker logs charon-distributed-validator-node-charon-1 --tail 50 -f
```
(Tips: Using auto complete - You can try pressing `Tab` after typeing the first few letters of the container name)  
You can monitor the logs here if needed, and use `Ctrl+C` to breakout from the logs.  
- If Charon connects to the BN sucessfully, you should not see any error.  
The logs will look like this if it can connect to the BN:  
![Alt text](static/img/charon-connection-success.png?raw=true)
  
- If Charon cannot connect to the beacon node, you will see an error:  
>ERRO cmd        Fatal error: new eth2 http client: fetch fork schedule: beacon api fork_schedule: client is not active {"label": "fork_schedule"}

The logs will look like this if it fail to connect to the BN:  
![Alt text](static/img/charon-connection-fail.png?raw=true)
If Charon fails to connect to the beacon node, double check everything has been configure corect it, or hop on the discord and ask for help.  

## Tips and Tricks
### Removing unused volumes  
If you have already started Charon before you disable Nethermind and Lighthouse, you will have the docker containers and volumes which take up some space on your disk.  
They can removed by running `docker volume prune` and choose `yes`. This will remove all local volumes not used by at least one container!  

### Port forwarding
Port forwarding is required if the machine is behind NAT. Charon use port 3610/tcp by defaut. This needs to be forwarded to your node. Please search for guide on how to do it for your specific router/gateway.  
You can use tools such as [yougetsignal](https://www.yougetsignal.com/tools/open-ports/) to check wheter the port are open corrently.
  
##   
*If you run into issue or have any suggestion feel free to open a PR or contact me (atomicwhale|at0micwhale) on Obol discord.*
