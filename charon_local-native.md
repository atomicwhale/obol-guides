# Running Charon with Native Execution/Consensus Clients

## Introduction

This guide is for people who want to run Charon on an existing machine with local execution/consensus clients (EC/CC) running as system service (systemd). e.g. You have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar) when setting up your node.

*This guide has been tested on Debian 12 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*

  
**TLDR**  
In this configuration, EC/CC are already running as a system service (systemd) and can be reached on localhost.  
We will setup Charon in docker using the Obol official docker package and point it to local BN.

## Configuration
### 0. Initial setup  
1. First, your BN has to be configured to expose the validator REST port (e.g. 5052) at 0.0.0.0. Refer to the client' guide for this.
For example, the BN will need the following flags
```
# Nimbus
--rest-address=0.0.0.0
# Lighthouse
--http-address=0.0.0.0
# Teku
--validator-api-interface=0.0.0.0
# Lodestar
--rest.address=0.0.0.0
# Prysm
--rpc-host=0.0.0.0
```
For EthPillar, it can be done by going to `Consensus Client` - `8 Expose consensus client RPC Port`  
![Alt text](screenshots/ethpillar.png?raw=true "ethpillar")
  
2. Make sure this port is protected behind a firewall, because you don't want random people on the internet to connect to it. Please refer to other firewall guides for this.  

3. Please follow the [official guide](https://docs.obol.org/run/start/quickstart_group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
Stop **BEFORE** you do Step 4 and modify you configurations following the guide below.  

### 1. Disable EC/CC included in the Charon docker package
(This step is taken from the official guide, you can find it in the official guide [Step 4: Existing Beacon Node](https://docs.obol.org/run/start/quickstart_group#step-4-start-your-distributed-validator-node)  
1. Create the `docker-compose.override.yml` file from the example  
```
cp -n docker-compose.override.yml.sample docker-compose.override.yml
```

3. Modify `docker-compose.override.yml` file using an editor (`nano` for example)  
```
nano docker-compose.override.yml
```
   * Uncomment `service`, and both `nethermind` and `lighthouse` under `services`.  
   * Uncomment the `profiles: [disable]` line for both `nethermind` and `lighthouse`.  
The override file should now look like this:  
```
services:
  nethermind:
    # Disable nethermind
    profiles: [disable]
    # Bind nethermind internal ports to host ports
    #ports:
      #- 8545:8545 # JSON-RPC
      #- 8551:8551 # AUTH-RPC
      #- 6060:6060 # Metrics

  lighthouse:
    # Disable lighthouse
    profiles: [disable]
    # Bind lighthouse internal ports to host ports
    #ports:
      #- 5052:5052 # HTTP
      #- 5054:5054 # Metrics
```
Use `Ctrl+O` and `Ctrl+X` to save and exit if you using `nano`.
4. (Optional) Disable mev-boost  
*Charon does not talk to mev-boost, only CC needs to talk to it when proposaing blocks. You should configure your mev-boost when you set up your CC, check relevant guides you followed when you setting up your EC and CC.*  
You can use the same method to disable mev-boost container (by uncommenting the relevant lines in the `mev-boost` section).  
The section should now look like this:  
```
  mev-boost:
    # Disable mev-boost
    profiles: [disable]
    # Bind mev-boost internal ports to host ports
    #ports:
      #- 18550:18550 # Metrics
```

### 2. Configure Charon to use local Beacon node (Consensus client)  
1. Check the local Beacon node is reachable at `localhost:<port-number>`
2. Fpr example
```
curl http://localhost:5052/eth/v1/node/syncing
```
You should see something like this:  
> {"data":{"head_slot":"XXXXXXXX","sync_distance":"0","is_syncing":false,"is_optimistic":false,"el_offline":false}}

which indicates that the beaconnode is reachable at localhost, and `sync_distance` at (0 or 1) and `is_syncing` is (false) which suggest the consensus client is fully synced.  

2. Modify `docker-compose.override.yml` so Charon can connect to host network  
(Docker compose creates a [new docker network by default](https://docs.docker.com/compose/how-tos/networking/), containers within the network cannot reach service running on host without further configuration)  
Modify the `docker-compose.override.yml` file using an editor  
```
nano docker-compose.override.yml
```
Uncomment the `charon` line, and add two lines for `extra_host`.  
It should now look like this:  
```
  charon:
    # Configure any additional env var flags in .env.charon.more
    #env_file: [.env.charon.more]
    # Uncomment the extra_hosts section if you are trying to communicate with a CL running in a different docker network on the same machine 
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
Save and exit.  

3. Configure CDVN in the `.env` file
First create the `.env` file using the template provided:  
For Holesky testnet  
```
cp .env.sample.holesky .env
```
For Ethereum mainnet  
```
cp .env.sample.mainnet .env
```
Modify the BN endpoint in the `.env` file  
```
nano .env
```
Uncomment and set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file to localhost.  
Point the endpoint to docker host network  
```
http://host.docker.internal:5052
```
The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://host.docker.internal:5052
```
Save and exit.  

### 3. Start Charon  
*Make sure you are running this command under the charon folder, it should be `charon-distributed-validator-node` by default)*  
```
cd ~/charon-distributed-validator-node
```
Start Charon by running  
```
docker compose up -d
```

### 4. Check if Charon is running successfuly  
Check the logs of the Charon container by using `docker logs <charon-container-name> -f`, for example:  
```
docker logs charon-distributed-validator-node-charon-1 --tail 50 -f
```
(Tips: Using auto complete - You can try pressing `Tab` after typeing the first few letters of the container name)  
You can monitor the logs here if needed, and use `Ctrl+C` to breakout from the logs.

- If Charon cannot connect to the beacon node, you will see an error:  
>ERRO cmd        Fatal error: new eth2 http client: fetch fork schedule: beacon api fork_schedule: client is not active {"label": "fork_schedule"}  

If Charon fails to connect to the beacon node, double check everything has been configure corect it, or hop on the discord and ask for help.  

## Tips and Tricks
### Removing unused volumes  
If you have already started Charon before you disable Nethermind and Lighthouse, you will have the docker containers and volumes which take up some space on your disk.  
They can removed by running `docker volume prune` and choose `yes`. This will remove all local volumes not used by at least one container!  

### Port forwarding
Port forwarding is required if the machine is behind NAT. Charon use port 3610/tcp by defaut. This needs to be forwarded to your node. Please search for guide on how to do it for your specific router/gateway.  
You can use tools such as [yougetsignal](https://www.yougetsignal.com/tools/open-ports/) to check wheter the port are open corrently.
