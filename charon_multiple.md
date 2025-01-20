# Multiple Charon nodes on a single machine

TLDR: Docker compose makes this setup very easy. Clone the Charon Distributed Validator Node (CDVN) repo to a new directory and they will run as two isolated instances.

### 0. Important notes

This guide is based on this assumptions:  
**You are runing the BN/EC separately to Charon.** This is highly recommended because it makes managing each Charon node much easier, and you don't need to bring the BN/EC down when making changes to the ocnfiguration.  
**If you arae running one CDVN with the EC/BN clients and wants to spin up more CDVN instance on the same machine. Please read the short section "Pluggin more Charon to one CDVN with EC/BN" at the end of the guide.  
  
1. [Disclaimer] This setup is for testing, or if you are running multiple nodes for different clusters. Please do not run more than one Charon nodes from the same cluster on a single machine as it creats a single point of failure. Please run each node on a separated machine in a production environment.  
2. [Hardware] Make sure the machine is capable of running multiple nodes. Around 1-1.5 GB RAM is needed per node (without EC/BN).
3. [Ports] Different P2P ports are needed for different nodes, DO NOT use the same ports. Instruction on how to change the port can be found below.  
4. [Note] Each P2P port need to be forwarded if you are running behind a NAT (e.g. home router/gateway).

### 1. Clone each CDVN into separate directories  

1. Specify a directory when first cloning the repo, for example `charon-distributed-validator-node-1`:  
```
# Clone the repo
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git charon-distributed-validator-node-1`
# Change directory
cd charon-distributed-validator-node-1/
```

2. If you want to creat another charon node, change the name of the folder to, for example `charon-distributed-validator-node-2`  
notice the folder name is different, it ends with "-2" instead of "-1" in the above axample.  

```
# Go back to your home directory
cd ~
# Copy the CDVN directory
cp -r charon-distributed-validator-node-1/ charon-distributed-validator-node-2/`
# Go into the directory
cd charon-distributed-validator-node-2/
```
If you are cloning from a current charon directory which already contains an ENR and relervant cluster files, make sure to remove them before you try to start it.
```
# Remove all previous cluster configuration
rm -r .charon/
```

3. Repeat the above step 2 if you want to run more than 2 Charon nodes.  
The folder and file structure will look like this:  
```
\charon-distributed-validator-node-1
|-.env
|-.charon
|-docker-compose.yml
|...
\charon-distributed-validator-node-2
|-.env
|-.charon
|-docker-compose.yml
|...
\charon-distributed-validator-node-3
...
```

### 2. Adjust CDVN configuration  

The configuration of each CDVN needs to be done separatly under each folder. Most of the settings can be done in `.env` and `docker-compose.override.yml` files.  
Look at the official guide for other steps requires for setting up a Charon node: [official guide] (https://docs.obol.org/run/start/quickstart_group) or other [guide here](https://github.com/atomicwhale/obol-guides)  

Ports that are mapped to the host machine also needs to be adjusted to avoid conflicts.

1. Each additional Charon needs a different P2P ports (3610 by default)
(If you are configuring Charon for the first time, start with the template by running `cp .env.sample.mainnet .env`(mainnet) or `cp .env.sample.holesky .env` (holesky))  
`nano .env`
Uncomment the P2P port line and add your port here (3611 for example), it shoud look like this after:  
```
# Charon host exposed ports
CHARON_PORT_P2P_TCP=3611
```
* (You will need to do portforwarding for each Charon port if you are running behind a NAT. Google is your friend, search for port forwarding guide for the specific router/gateway model you have)  

2. Change Grafana port
Each additional Grafana needs to be mapped to another port.  
For example, we can change the Grafana on the 2nd Charon node to 3001.  
The settings can be changed in the `.env` file  
`nano .env`
Uncomment the Grafana port line and change the port number (e.g. 3001)  
It should now looks like this:  
```
# Grafana host exposed ip and port.
#MONITORING_IP_GRAFANA=
MONITORING_PORT_GRAFANA=3001
```

### 3. Start Charon  

Go in the directory of the CDVN which you want to start by:  
```
# Go back to your home directory
cd ~
# Go in the the CDVN directory to start
cd charon-distributed-validator-node-1/
```

Start Charon by running  
```
docker compose up -d
```  
(Make sure you are running this command under the correct charon folder. You will need to run this again for each charon node under their correspding directories)  

### 4. Check if Charon is running successfuly

Check the logs of different Charon containers by using:
`docker logs charon-distributed-validator-node-charon-1 --tail 50 -f`
`docker logs charon-distributed-validator-node-charon-2 --tail 50 -f`
(Tips: Using auto complete - You can try pressing `Tab` after typeing the first few letters of the container name)  
You can monitor the logs here if needed, and use `Ctrl+C` to breakout from the logs.  
- If Charon connects to the BN sucessfully, you should not see any error.  
The logs will look like this if it can connect to the BN:  
![Alt text](screenshots/charon-connection-success.png?raw=true)
  
- If Charon cannot connect to the beacon node, you will see an error:  
>ERRO cmd        Fatal error: new eth2 http client: fetch fork schedule: beacon api fork_schedule: client is not active {"label": "fork_schedule"}

The logs will look like this if it fail to connect to the BN:  
![Alt text](screenshots/charon-connection-fail.png?raw=true)
If Charon fails to connect to the beacon node, double check everything has been configure corect it, or hop on discord and ask for help.  

## Tips and Tricks  
### Pluggin more Charon to one CDVN with EC/BN  
(Official support for this is coming soon too, stay tune.)  
If you already have a CDVN with EC/BN (Nethermind/Lighthouse) running, and want to spin up more CDVNs on the same machine. You can use docker network to achieve it. Follow the previous steps in this guide to put additional CDVN in their own folder
1. Disable EC/BN in the additional Charon nodes.  
2. Point additional Charon nodes to the BN running in Charon1, edit the `.env` file:  
```
CHARON_BEACON_NODE_ENDPOINTS=http://lighthouse:5052
```

### Removing unused volumes  
If you have already started Charon before you disable Nethermind and Lighthouse, you will have the docker containers and volumes which take up some space on your disk.  
They can removed by running `docker volume prune` and choose `yes`. This will remove all local volumes not used by at least one container!  

### Port forwarding
Port forwarding is required if the machine is behind NAT. Charon use port 3610/tcp by defaut. This needs to be forwarded to your node. Please search for guide on how to do it for your specific router/gateway.  
You can use tools such as [yougetsignal](https://www.yougetsignal.com/tools/open-ports/) to check wheter the port are open corrently.

-----------
**Some other guides here to help with setting up CDVN**  
https://github.com/atomicwhale/obol-guides/blob/main/README.md
  
##   
*If you run into issue or have any suggestion feel free to open a PR or contact me (atomicwhale|at0micwhale) on Obol discord.*
