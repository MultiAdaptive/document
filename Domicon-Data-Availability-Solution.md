# Domicon: Data Availability Solution

[TOC]

Domicon provides two sets of data availability solutions for Ethereum and Bitcoin, which are currently independent, and we will consider how to integrate these two solutions into one in the future.

All the solution is based on KZG polynomial commitment. A brief introduction to KZG can be found [here](https://github.com/domicon-labs/document/blob/main/KZG-Polynomial-Commitments.md). All the data will be encoded as a polynomial.

## Chapter 1：Domicon on Ethereum

Domicon nodes are classified into broadcast nodes and storage nodes. Broadcast nodes keep all data within seven days, and storage nodes keep only tenant data and store it for a long time based on order requirements.

The user uses the KZG scheme to make a commitment to the data, store the commitment on Ethereum, and send both the data and commitment to Domicon nodes for broadcast and storage.

The Ethereum solution consists of three major parts:

### 1. Contract

1. **Governance Contract**

By pledging tokens, initiate voting for governance.

2. **Node Management Contract**

A node stakes token to become a broadcast or storage node. The information to be registered includes the node URL, home location, and maximum storage space.

The home location of the node can be modified by the governance contract.

3. **Storage Service Management Contract**

If long-term storage is required, the user should select storage nodes and the storage period first. A storage node synchronizes this information to find and store the required data.

4. **Commitment Management Contract**

Domicon's main contract. Manage commitments of user data.

Commitments submitted to this contract must be signed by at least one node.

The contract maintains a global state: Calculate the hash $r_i=H(cm_i||addr_i||r_{i-1})$ for the whole state, so that nodes can check whether all commitments have been scanned.

5. **Data Detention Management Contract**

Any node, if it can not obtain the data corresponding to the commitment, can initiate a "data detention" record in the contract according to the commitment, and if more than 50% of the nodes agree to the record, it will slash the node attached to the commitment when it is submitted.

6.  **Challenge Contract** 

**Broadcast challenge**: Anyone (whitelist can be set in the early stage) can challenge any broadcast node, specify a commitment within a validity period and a random number, and ask the broadcast node to use the polynomial (data) corresponding to the commitment to open at the random number and show proof.

**Storage Challenge/Data Audit**: A user can challenge a storage node that stores its data. The challenge needs to specify the commitment range, such as the data within the first commitment to the 1000th commitment submitted by the user.

When each time the project submits data to Domicon's storage contract, the contract records the commitments $cm_i$ corresponding to each data submission and the corresponding polynomials $f_i(x)$. Assuming data has been submitted a total of $t$ times when the project needs to initiate a full audit, they must generate $t$ random numbers $r_i$ and a random value $v$. The storage provider is then required to open the polynomial $F(x) = \sum r_i \cdot f_i(x)$ at the value $v$.

To facilitate calculations within the contract, you can set $r_i = \text{Hash}(seed||i)$, where $seed$ is a seed selected by the project.

The commitment corresponding to $F(x)$ is $CM = \sum r_i \cdot cm_i$. When the value of $t$ is large, it can be challenging to calculate this commitment within the contract. Therefore, an interactive challenge protocol is designed to ensure that the project and the storage provider can reach a consensus on $CM$.

First, the project submits $CM_{0,t} = \sum r_i \cdot cm_i$. If the storage provider disagrees, the project submits $CM_{0,t/2}$ in an interactive binary manner, allowing both parties to find a pair ${CM_{i,j}, CM_{i,j+1}}$ that they agree upon $CM_{i,j}$ but not on $CM_{i,j+1}$.

At this point, the contract only needs to verify whether $CM_{i,j+1} = CM_{i,j} + r_{j+1} \cdot cm_{j+1}$. This determination can reveal who is being dishonest, leading to penalties.

Once consensus is reached, the storage node must generate $F(v)$ and its corresponding proof for contract verification.

### 2. Nodes

There are broadcast nodes and  storage nodes in the Domicon network. All nodes are connected through a P2P network.

**Broadcast node:**

The broadcast node can receive user data and commitment, check the relationship between the data and commitment, sign the commitment and return it to the user, and save the data in the memory (up to 3 hours).

The broadcast nodes scan Ethereum, read contract events, parse out data commitments, and store them. If the local data corresponding to the commitment is stored, the data is solidified, the time is recorded, and the data is broadcast to other nodes.

If no data corresponding to the commitment is available locally, data is requested in the P2P network. This request is for full data.

If the data cannot be obtained, a data sampling request is made. This request is a data sharding request, and the full data is restored through data sharding (the number of shardings is set according to the network condition).

If it is not obtained in the P2P network, data requests will be sent to these nodes via API according to the node information attached to the commitment submission.

If the data is not available, a data sampling request is made to these nodes via the API (set the number of shards based on 2/3 of the number of nodes).

If the data is not available, the broadcast node submits a "data detention" record.

**Storage node:**

The storage node can receive tenant data and commitments, check the relationship between the data and commitments, sign the commitments and return them to the tenant, and save the data in the memory (for a maximum of 3 hours).

Storage nodes scan Ethereum, read contract events, parse the tenant's data commitments, and store them. If the local data corresponding to the commitment is stored, the data is solidified, the time is recorded, and the data is broadcast to other nodes.

If no data corresponding to the commitment is available locally, the system requests data from the P2P network. The request process is the same as that of the broadcast node.

### 3. SDK

The user encodes the data locally and generates the KZG commitment. The data and commitments are then sent to the specified node (the number can be set), signatures are requested, and when enough signatures are received, the commitments are submitted to the Domicon Commitment management contract.

### 4. Work Flow

1. The node is registered in the Domicon Node management contract.

2. User selects storage nodes in the storage service management contract (optional).

3. The user sends the data and commitment to the specified node through the local SDK. After obtaining enough signatures, the commitment and signature are submitted to the commitment management contract.

4. All nodes scan the contract and start to broadcast and read data in the P2P network after reading the data commitment.

5. Check whether data is detained and report it to the contract.

6. Launch broadcast and storage challenges anytime.

## Chapter 2：Domicon on Bitcoin

Domicon nodes are classified into broadcast nodes and storage nodes. Broadcast nodes keep all data within seven days, and storage nodes keep only tenant data and store it for a long time based on order requirements.

The user uses the KZG scheme to make a commitment to the data, store the commitment on Bitcoin, and send both the data and commitment to Domicon nodes for broadcast and storage.

### Domicon Network

We will build the Domicon network using Avalanche's subnet, and at the same time require all nodes to monitor the Bitcoin network. Users' data commitments will be stored on the Bitcoin network, and the data itself will be broadcast and stored on the subnet network.

![img](images/subnet.png)

When a node discovers a Commitment transaction that complies with Domicon rules on the Bitcoin network, it will write the commitment to the Domicon network store the Commitment locally, and request the corresponding data in the p2p network.

All the contracts like Ethereum, will be deployed on this subnet.

### Commitment to Bitcoin

The commitment submission process is similar to the Ordinals NFT protocol and requires two Bitcoin transactions. The multi-signed addresses of multiple Domicon nodes will be treated as the recipient of the first transaction and the sender of the second transaction. The Domicon node is confirmed to have received the user's data by signing multiple addresses.

A brief overview is as follows:

1. The user creates an Ordinals NFT with a commitment to data and sends the transaction to multiple sign-on addresses of multiple Domicon nodes
2. The user sends the data and commitment to the selected Domicon node and requests the transfer NFT signature
3. After the user collects enough signatures, the NFT is transferred back to the user

This completes the submission of the commitment.

![img](images/bitcoin.png)



## Appendix: Data Sampling

Because each data is encoded into a polynomial $f_i(x)$, and their degree is at most $n$, you only need to know the values of $f_i(x)$ at up to $n+1$ points to reconstruct $f_i(x)$.

So when sampling data, for each data, the sampling party randomly generates up to $(n+1)/16$ values $v_j$ and then requests broadcast nodes to open them at these $(n+1)/16$ points. They provide a commitment Merkle proof, 

$$
[cm_i, [{v_j}, \pi], proof]
$$

for that piece of data. The sampling party first verifies the legality of $cm_i$ based on the Merkle proof and then performs KZG verification.

Because the random space has a size of $2^{256}$, we can assume that the sampling party will not randomly choose the same value twice. Therefore, with just 16 sampling parties, you can obtain the values of $f_i(x)$ at $n+1$ points, ensuring 100% reconstruction of $f_i(x)$.
