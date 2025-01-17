# Running Charon with Existing Execution/Consensus Clients

Charon is a middleware built by Obol to enable any existing Ethereum validator clients to operate together as part of a distributed validator.  
Users are highly recommended to go through [Obol's official documetation](https://docs.obol.org/run/start/quickstart_overview) first to have a basic understanding of how a Charon DVT cluster works.

## Overview
These guides are created for pepole who want to run Charon with their existing execution/consensus clients.  
Obol provides an official docker-based package called [Charon's Distributed Validator Node (CDVN)](https://github.com/ObolNetwork/charon-distributed-validator-node). These guides assume you will be running this package **using docker**.  
The basic principles are the same across all these guides. These guides aims to cover tips and tricks that are included in the standard setup described in the official documents.

## Minimum/Recommended Hardware (local machine or VPS)
- **Node (with/without Charon)**  
 CPU: 4+ core  
 RAM: 24+ GB  
 SSD: 2/4 TB NVMe (TLC with DRAM)  
  *Please refer to the excellent guides from [ETH-Staker](https://ethstaker.cc/staking-hardware) and ETH-Docker [SSD guide](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038).*
  
- **Charon (without EC/BN)**  
 CPU: 1/2+ core  
 RAM: 1/2 GB (While 1 GB works, 2+ GB is highly recommended) each Charon node  
 Disk: A few GBs  

## Guides
1. [Running Charon with Native Execution/Consensus Clients](https://github.com/atomicwhale/obol-guides/blob/main/charon_local-native.md)  
This guide covers running Charon with local EC/BN running as system service (systemd). For example, if you have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar) to set up your node.  
  
2. [Running Charon with Docker Execution/Consensus Clients](https://github.com/atomicwhale/obol-guides/blob/main/charon_local-docker.md)  
This guide covers running Charon with local EC/BN managed by a docker-based stack (ETH-Docker/Rocket Pool Smart Node/Sedge).

3. [Running Charon with remote EC/BN](https://github.com/atomicwhale/obol-guides/blob/main/charon_remote.md)  
This guide covers running Charon with remote EC/BN. For people who already have EC/BN running on a remote/differnt machine, and want to run Charon separately (for example a small VPS).  

Other guides  
 4. [Running Multiple Charon instances on one machine](https://github.com/atomicwhale/obol-guides/blob/main/charon_multiple.md)  
  This guide covers steps to run multiple Charon instaces on one machine.  
