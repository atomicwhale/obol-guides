# Run Charon with Existing Execution/Consensus Clients

Charon is a middleware built by Obol to enable any existing Ethereum validator clients to operate together as part of a distributed validator.  
Users are highly reccomaneded to go through [Obol's official documetation](https://docs.obol.org/int/Overview) to have a basic understanding of how a Charon DVT cluster work and the standard setup.

## Overview
These guides are created for pepole who want to run Charon with their existing execution/consensus clients.
The basic principle are the same across all these guides. These guides aims to cover tips and tricks that are not included in the standard setup decibed in the official documents.

## Reccommended Hardware (local machine or VPS)
- **Node (with/without Charon)**  
 CPU: 4+ core  
 RAM: 32 GB (16 GB works with some client combinations)  
 SSD: 4 TB NVMe (TLC wiht DRAM)  
  *Please refer to the excellent hardware guide from [ETH-Staker](https://ethstaker.cc/staking-hardware) and ETH-Docker [SSD guide](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038).*
  
- **Charon (without EC/BN)**  
 CPU: 1+ core  
 RAM: 2 GB  
 Disk: Few GB  

## The guides
Please pick one of the guides below

1. [Running Charon with Native Execution/Consensus Clients](Link)  
This guide is for people who have local EC/BN as system service (systemd), for example, if you have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar).  
  
2. [Running Charon with Docker Execution/Consensus Clients](Link)  
This guide is for people who have local EC/BN manged as a docker-based stack (ETH-Docker/Rocket Pool Smart Node)

3. [Running Charon with remote EC/BN](Link)  
This guide is for people who want to run only Charon locally with remote execution/consensus clients, for example, if you have followed one of these guides (Somer Esat Guides / CoinCashew Guides / EthPilar).  


