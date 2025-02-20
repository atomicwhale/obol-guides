# Running Charon with Remote Execution/Consensus Clients

## Introduction

This guide is for people who want to run Charon connecting to execution/consensus clients (EC/CC) running on a external/remote machine. e.g. You running Charon on a small VPS and want to plug it into a current EC/CC pair.)  

*This guide has been tested on Debian 12 and should work on all Debian-basd (Debian/Ubuntu etc.) system.*  
*The term Beacon node (BN) and Consensus client (CC) are used interchangably in this guide.*
  
**Connecting to existing EC/BN is already covered in the Obol official guide: [Step 4: Existing Beacon Node](https://docs.obol.org/start/quickstart_group#step-4-start-your-distributed-validator-node).  
This guide aims to provide some extra tips and tricks on top of the official guide.**

## Configuration of the remote Beacon Node

A few things to consider when you are using a remote BN.
1. Expose and safeguard the BN RPC port (e.g. 5052)
   - If you are running `ufw` firewall, you can adding a rule such as `sudo ufw allow from <CHARON_IP> proto tcp to any port 5052` to allow only charon machien to connect to the BN port.  
   - If you are running ETH-docker on LAN or in a secure network, you can add `cl-shared.yml` to expose your BN. Consider using `cl-traefik.yml` if you are running on public network.  
   - If you are running Rocketpool Smart Node, please choose `Open to exteranl hosts` on the Beacon Node configureation page.  
2. If using `ufw` and `docker`, please follow this [guide](https://ethdocker.com/Support/Cloud) to ensure firewall is working properly for docker.  
   - Use a port checker such as `https://www.yougetsignal.com/tools/open-ports/` to test your port 5052 on the BN IP. It should be closed to the public network.  
   - Use `curl http://BN-IP:5052/eth/v1/node/syncing` to check if you can reach your BN from your Charon machine.  
3. Optimize the latency between the beacon node and charon--ideally they should be on the same LAN or same data center. Any latency here will just add to the time your cluster's need to reach consensus.  

## Configuration
### 0. Initial setup  
Please follow the [official guide](https://docs.obol.org/start/quickstart_group) **Step 1-3** to download Charon, set up the ENR, join a cluster, and run a DKG.  
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

### 2. Configure Charon to use remote Beacon node (Consensus client)  
1. Check the remote Beacon node is reachable  
```
curl http://REMOTE_IP:5052/eth/v1/node/syncing
```  
You should see something like this:  
> {"data":{"head_slot":"XXXXXXXX","sync_distance":"0","is_syncing":false,"is_optimistic":false,"el_offline":false}}

which indicates that the beaconnode is reachable, and `sync_distance` at (0 or 1) and `is_syncing` is (false) which suggest the consensus client is synced.  

2. Set the `CHARON_BEACON_NODE_ENDPOINTS` variable in the `.env` file to remote BN.  
The section should now look like this:  
```
# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
CHARON_BEACON_NODE_ENDPOINTS=http://REMOTE_IP:5052
```
Charon supports connecting to multiple BNs, if you have other BN available you can also add them here.
`CHARON_BEACON_NODE_ENDPOINTS=http://REMOTE_IP1:5052,,http://REMOTE_IP2:5052`
Save and exit.  

### 3. Start Charon  
Start Charon by running  
(Make sure you are running this command under the charon folder, it should be `charon-distributed-validator-node` by default)
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
