# Running Charon with Docker-based Execution/Consensus Clients

## Introduction

This guide is for people who want to run Charon on an existing machine with local execution/consensus clients (EC/CC) running as docker containers (such as ETH-Docker, Rocketpool Smart node).

*This guide has been tested on Debian 12 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*

  
**TLDR**  
In this configuration, EC/CC are already running as as docker containers within a docker bridge network. We will need to place Charon in this docker network so that it can connect to EC/CC.  
We will setup Charon in docker using the Obol official docker package and point it to the local CC.

## Configuration
### 0. Initial setup  
Please follow the [official guide](https://docs.obol.org/run/start/quickstart_group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
Stop **BEFORE** you do Step 4 and modify you configurations following the guide below.  

### 1. Inspect Docker network
We firt need to find out what docker network are running on the local machine.  
```
docker network ls
```
This should show you a list of all the Docker networks, it should look like this:
```
NETWORK ID     NAME                 DRIVER    SCOPE
802cde714b18   bridge               bridge    local
0022bd01f59d   eth-docker_default   bridge    local
0f8943e65fbf   rocketpool_net       bridge    local
607eb18ca2a8   host                 host      local
839941bca9ab   none                 null      local
```
- If you are running ETH-docker, you will notice a bridge network called `eth-docker_default`. (This first part of the network name will be the same as the name of the folder where ETH-docker is located).
- If you are running Rocketpool Smart Node, you will notice a bridge network called `rockerpool_net`.
- If you are running Sedge, you will notice a bridge network called `sedge`.
  
This is the docker network where you EC/CC are running. In the next step, we will connect charon to this network so it can talk to the EC/BN.  

### 2. Disable EC/CC included in the Charon docker package
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
### 3. Configure Charon to use additional docker network
(This following step use ETH-docker `eth-docker_default` as example, please adjust if you are running Rocketpool Smartnode or other packages)  
1. Modify `charon` section in `docker-compose.override.yml` file  
```
nano docker-compose.override.yml
```
Uncomment line `charon` under the service section, and add additional network configureation here.  
The section in the override file should now look like this:  
```
  charon:
    networks:
      - eth-docker_default
```

2. Add the follow lines to the bottom of the file:
```
networks:
  eth-docker_default:
    name: eth-docker_default
    external: true
```
Save and exit.  

### 4. Configure Charon to use local Beacon node (Consensus client)  
1. Set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file.  
(`http://eth2:port` works for both RP SmartNode and Eth-docker)
(`http://consensus:port` works for both Eth-docker and Sedge)
The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://eth2:5052
```

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
- If Charon connects to the BN sucessfully, you should not see any error.  
The logs will look like this if it can connect to the BN:  
![Alt text](screenshots/charon-connection-success.png?raw=true)
  
- If Charon cannot connect to the beacon node, you will see an error:  
>ERRO cmd        Fatal error: new eth2 http client: fetch fork schedule: beacon api fork_schedule: client is not active {"label": "fork_schedule"}

The logs will look like this if it fail to connect to the BN:  
![Alt text](screenshots/charon-connection-fail.png?raw=true)
If Charon fails to connect to the beacon node, double check everything has been configure corect it, or hop on the discord and ask for help.  

## Tips and Tricks
### Removing unused volumes  
If you have already started Charon before you disable Nethermind and Lighthouse, you will have the docker containers and volumes which take up some space on your disk.  
They can removed by running `docker volume prune` and choose `yes`. This will remove all local volumes not used by at least one container!  

### Port forwarding
Port forwarding is required if the machine is behind NAT. Charon use port 3610/tcp by defaut. This needs to be forwarded to your node. Please search for guide on how to do it for your specific router/gateway.  
You can use tools such as [yougetsignal](https://www.yougetsignal.com/tools/open-ports/) to check wheter the port are open corrently.
