# Running Charon with Docker-based Execution/Consensus Clients

## Introduction

This guide is for people who want to run Charon on an existing machine with local execution/consensus clients (EC/CC) running as docker containers (such as ETH-Docker, Rocketpool Smart node).

*This guide has been tested on CDVN v1.10.3 and Debian 12 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*

  
**TLDR**  
In this configuration, EC/CC are already running as as docker containers in their own docker network. We will need to place Charon in this docker network so that it can connect to the EC/CC.  
We will setup Charon in docker using the Obol official CDVN docker stack and point it to the local CC.

## Configuration
### 0. Initial setup  
Please follow the [official guide](https://docs.obol.org/run-a-dv/start/create-a-dv-with-a-group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
Stop **AFTER** you created the `.env` file in Step 4, modify you configurations following the guide below.  

*Make sure you are under the charon folder, it should be `charon-distributed-validator-node` by default)*  
```
cd ~/charon-distributed-validator-node
```

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
- If you are running ETH-docker, you will notice a bridge network called `eth-docker_default`. (This first part of the network name will be the same as the name of the folder where ETH-docker is located by default).
- If you are running Rocketpool Smart Node, you will notice a bridge network called `rockerpool_net`.
- If you are running Sedge, you will notice a bridge network called `sedge`.
  
This is the docker network where you EC/CC are running. In the next step, we will connect charon to this network so it can talk to the EC/BN.  

### 2. Disable EC/CC and mev-boost included in the Charon docker package
(This step is taken from the official guide, you can find it in the official guide [Step 4: Existing BN](https://docs.obol.org/run-a-dv/start/create-a-dv-with-a-group#step-4-start-your-distributed-validator-node)  
1. (Go into the CDVN folder) Modify `.env` file using an editor (`nano` for example)  
```
nano .env
```

2. Uncomment EL=el-none, CL=cl-none and MEV=mev-none variables in the .env file and comment out EL=el-nethermind,  CL=cl-lighthouse and MEV=mev-mevboost variables, so it looks liek this:  
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

### 3. Configure Charon to use local Beacon node (Consensus client)  
1. Set the `CHARON_BEACON_NODE_ENDPOINTS` variable.  
Uncomment and set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file  
(`http://eth2:port` works for both RP SmartNode and Eth-docker)
(`http://consensus:port` works for both Eth-docker and Sedge)
The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://eth2:5052
```
Use `Ctrl+O` and `Ctrl+X` to save and exit if using `nano`.  

### 4. Configure Charon to connect to CC's docker network
1. Create a docker compose override file, for example `custom.yml`  
```
cp docker-compose.override.yml.sample custom.yml
```

2. Modify `charon` section in `custom.yml` file  
```
nano custom.yml
```
Uncomment line `services`  
Uncomment line `charon` under the service section, and add additional network configureation here.  
The section in the override file should now look like this:  
```
  charon:
    ...
    networks:
      - ethdocker
```

3. Add the follow lines to the bottom of the file:  
(This following step use ETH-docker `eth-docker_default` as example, please adjust if you are running Rocketpool Smartnode or other packages)  
```
networks:
  ethdocker:
    name: eth-docker_default
    external: true
```

4. Add 
Use `Ctrl+O` and `Ctrl+X` to save and exit if using `nano`.  

### 5. Start Charon  
1. Start Charon by running  
```
docker compose up -d
```

2. Check if Charon is running happily  
Check the logs of the Charon container, for example:  
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
