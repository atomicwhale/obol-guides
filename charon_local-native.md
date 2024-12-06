# Running Charon with Native Execution/Consensus Clients
## Introduction
This guide is for people who want to run Charon on an existing machine with local execution/consensus clients (EC/CC) running as system service (systemd). e.g. You have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar) when setting up your node.  
  
*This guide has been tested on Debian 12.7 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*  
  
**TLDR**  
In this configuration, EC/CC are already running as a system service (systemd) and can be reached on localhost.  
We will setup Charon in docker using the Obol official docker package and point it to local BN.  

## Configuration
### 0. Initial setup  
Please follow the [official guide](https://docs.obol.org/start/quickstart_group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
Please modify you configurations following the guide below before starting Charon.  

### 1. Disable EC/CC included in the Charon docker package
(This step is taken from the official guide, you can find it in the official guide [Step 4: Existing Beacon Node](https://docs.obol.org/start/quickstart_group#step-4-start-your-distributed-validator-node))  
1. Creat the `docker-compose.override.yml` file from the example  
`cp -n docker-compose.override.yml.sample docker-compose.override.yml`  
2. Uncomment the `profiles: [disable]` line for both `nethermind` and `lighthouse`. The override file should now look like this:  
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
### 2. Configure Charon to use local Beacon node (Consensus client)  
1. Check the local Beacon node is reachable  
`curl http://localhost:5052/eth/v1/node/syncing`  
You should see something like this:  
```
{"data":{"head_slot":"XXXXXXXX","sync_distance":"0","is_syncing":false,"is_optimistic":false,"el_offline":false}}
```  
which indicates that the beaconnode is reachable at localhost, and `sync_distance` at (0 or 1) and `is_syncing` is (false) which suggest the consensus client is synced.  
  
3. Set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file to localhost. The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://localhost:5052
```
### 3. Start Charon
`docker compose up -d`  

## Tips and Tricks
### Removing unused volumes  
If you have already started Charon before you disable Nethermind and Lighthouse, you will have the docker containers and volumes which take up some space on your disk.  
They can removed by running `docker volume prune -a` and choose `yes`. This will remove all local volumes not used by at least one container!  

### Port forwarding
Port forwarding is required if the machine is behind NAT. Charon use port 3610/tcp by defaut. This needs to be forwarded to your node. Please search for guide on how to do it for your specific router/gateway.  
You can use tools such as [yougetsignal](https://www.yougetsignal.com/tools/open-ports/) to check wheter the port are open corrently.
