# Multiple Charon nodes on one machine

TLDR: Docker compose makes this setup very easy. Clone the repo to a new directory and they will run as two isolated instances.

## 0 Important note
1. [Disclaimer] Not intended for running for the same cluster.
2. [Hardware] Make sure the machine is capable for running multiple nodes.
3. [Ports] Different ports are needed for different nodes, DO NOT use the same ports. Instruction on how to change the port can be found below.  
4. [Note on Port forwarding]

## 1. Clone the charon repo into separate directories  
Specify a directory when cloning the repo, for example `charon-distributed-validator-node-1`:  
```
# Clone the repo
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git charon-distributed-validator-node-1`
# Change directory
cd charon-distributed-validator-node-1/
```

If you want to creat another charon node, change the name of the folder to, for example `charon-distributed-validator-node-2`  
```
# Clone the repo
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git charon-distributed-validator-node-2`
# Change directory
cd charon-distributed-validator-node-2/
```
Repeat the above step if you want to run more than 2 Charon nodes.  

The configuration of the node can be done at each folder.  
The rest of the setup are the same as descibed in the [official guide] (https://docs.obol.org/run/start/quickstart_group) or other [guide here](https://github.com/atomicwhale/obol-guides)  

## 2. Adjust Charon ports  
Each Charon need a different P2P ports (3610 by default)
(If you are configuring Charon for the first time, start with the template by running `cp .env.sample.mainnet .env`(mainnet) or `cp .env.sample.holesky .env` (holesky))  
`nano .env`
Uncomment the P2P port line and add your port here (3611 for example), it shoud look like this after:  
```
# Charon host exposed ports
CHARON_PORT_P2P_TCP=3611
```
* (You will need to do portforwarding for each Charon port if you are running behind a NAT. Google is your friend, search for port forwarding guide for the specific router/gateway model you have)  

## 3. Start Charon  
Start Charon by running  
`docker compose up -d`  
(Make sure you are running this command under the correct charon folder. You will need to run this again for each charon node under their correspding folders to bring up the corresponding nodes)  
  
**Look further down for some tips on different configurations.**  
-----------
*[Running one cdvn as full node, adding more cdvn]
*[Running with external BN]
