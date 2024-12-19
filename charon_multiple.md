# Multiple Charon nodes on one machine

TLDR: Docker compose makes this setup very easy. Clone this folder into a new directory and they will run as two isolated instance.

## Clone the charon repo into separate directories
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
The configuration of the node can be done at each folder.  
The rest of the setup are the same as descibed in the [official guide] (https://docs.obol.org/run/start/quickstart_group) or other [guide here](https://github.com/atomicwhale/obol-guides)  

### Start Charon  
Start Charon by running  
(Make sure you are running this command under the correct charon folder. You will need to run this again for each charon node under their correspding folders)  
`docker compose up -d`  

## Important note:
1. [Disclaimer] Not intended for running for the same cluster.
2. Make sure the machine is capable for running multiple nodes.
3. Different ports are needed for different nodes, DO NOT use the same ports.
[instruction on how to change ports]
4. [Note on Port forwarding]
5. XX
