# How to create your own domicon op rollup  
&emsp;&emsp; This document demonstrates how to create your own rollup that uses domicon as the Data Availability Layer.
## Software Dependencies
| Dependency                                                    | Version  | Version Check Command |
| ------------------------------------------------------------- | -------- | --------------------- |
| [git](https://git-scm.com/)                                   | `^2`     | `git --version`       |
| [go](https://go.dev/)                                         | `^1.21`  | `go version`          |
| [node](https://nodejs.org/en/)                                | `^20`    | `node --version`      |
| [pnpm](https://pnpm.io/installation)                          | `^8`     | `pnpm --version`      |
| [foundry](https://github.com/foundry-rs/foundry#installation) | `^0.2.0` | `forge --version`     |
| [make](https://linux.die.net/man/1/make)                      | `^3`     | `make --version`      |
| [jq](https://github.com/jqlang/jq)                            | `^1.6`   | `jq --version`        |
| [direnv](https://direnv.net)                                  | `^2`     | `direnv --version`    |
## Build the Source Code
### domicon optimism
1. download domicon optimism
```bash
cd ~
git clone https://github.com/domicon-labs/optimism.git
```
2. Enter the Optimism Monorepo and check out develop-node branch
```bash
cd optimism
git checkout develop-node
```
3. Check your dependencies
```bash
./packages/contracts-bedrock/scripts/getting-started/versions.sh
```
4. Install dependencies
```bash
pnpm install
```
5. build code
```bash
make op-node op-batcher
pnpm build
```
### op-geth
1. download op-geth
```bash
cd ~
git clone https://github.com/ethereum-optimism/op-geth.git
```
2. Enter the op-geth Monorepo
```bash
cd op-geth
```
3. build op-geth
```bash
make op-geth
```
## Fill Out Environment Variables
You'll need to fill out a few environment variables before you can start deploying your chain.  
1. Enter the optimism Monorepo
```bash
cd ~/optimism
```
2. Duplicate the sample environment variable file
```bash
cp .envrc.example .envrc
```
3. Open up the environment variable file and fill out the following variables  

| Variable Name | Description                                                                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `L1_RPC_URL`  | URL for your L1 node (a Sepolia node in this case).                                                                                                                                                          |
| `L1_RPC_KIND` | Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: `alchemy`, `quicknode`, `infura`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, `any`. |

## Generate Addresses
You'll need four addresses and their private keys when setting up the chain:  
The `Admin` address has the ability to upgrade contracts.
The `Batcher` address publishes Sequencer transaction data to L1.
The `Proposer` address publishes L2 transaction results (state roots) to L1.
The `Sequencer` address signs blocks on the p2p network.
1. Enter the Optimism Monorepo
```bash
cd ~/optimism
```
2. Generate new addresses
```bash
./packages/contracts-bedrock/scripts/getting-started/wallets.sh
```
3. Make sure that you see output that looks something like the following:
```text
Copy the following into your .envrc file:
  
# Admin address
export GS_ADMIN_ADDRESS=0x9625B9aF7C42b4Ab7f2C437dbc4ee749d52E19FC
export GS_ADMIN_PRIVATE_KEY=0xbb93a75f64c57c6f464fd259ea37c2d4694110df57b2e293db8226a502b30a34

# Batcher address
export GS_BATCHER_ADDRESS=0xa1AEF4C07AB21E39c37F05466b872094edcf9cB1
export GS_BATCHER_PRIVATE_KEY=0xe4d9cd91a3e53853b7ea0dad275efdb5173666720b1100866fb2d89757ca9c5a
  
# Proposer address
export GS_PROPOSER_ADDRESS=0x40E805e252D0Ee3D587b68736544dEfB419F351b
export GS_PROPOSER_PRIVATE_KEY=0x2d1f265683ebe37d960c67df03a378f79a7859038c6d634a61e40776d561f8a2
  
# Sequencer address
export GS_SEQUENCER_ADDRESS=0xC06566E8Ec6cF81B4B26376880dB620d83d50Dfb
export GS_SEQUENCER_PRIVATE_KEY=0x2a0290473f3838dbd083a5e17783e3cc33c905539c0121f9c76614dda8a38dca
```
4. Copy the output from the previous step and paste it into your .envrc file as directed.  
5. Fund `admin` address with 2.5 Sepolia ETH.
  It is recommended to obtain it through the faucet: `https://sepolia-faucet.pk910.de/#/`
6. Fund `batcher` address Dom tokens  
 6.1 Visit `https://sepolia.etherscan.io/address/0x2DE928B6494A6fd9194dfE33CE0Cf111E2b8Ac04#writeContract` and connect the MetaMask wallet.  
 6.2 Fund Dom tokens to the `batcher` address in `mint`. The recommended amount is `10000000000000000000000`.  
 6.3 Switch MetaMask to the batcher account, and authorize the Dom token of `batcher` to `0x2BbECa3a09d75baBDc9A7F6c0022293d5A14B175` through `approve`. The recommended amount is `10000000000000000000000`
## Load Environment variables
1. Enter the Optimism Monorepo 
```bash
cd ~/optimism
```
2. Load the variables with direnv
```bash
direnv allow
```
If it doesn't take effect, you can hook it and try again.   
&emsp;&emsp;if your terminal is bash
```bash
 eval "$(direnv hook bash)"
```
&emsp;&emsp;if your terminal is zsh
```bash
eval "$(direnv hook zsh)"
```
3. Confirm that the variables were loaded
After running direnv allow you should see output that looks something like the following  
```bash
direnv: loading ~/optimism/.envrc                                                            
direnv: export +DEPLOYMENT_CONTEXT +ETHERSCAN_API_KEY +GS_ADMIN_ADDRESS +GS_ADMIN_PRIVATE_KEY +GS_BATCHER_ADDRESS +GS_BATCHER_PRIVATE_KEY +GS_PROPOSER_ADDRESS +GS_PROPOSER_PRIVATE_KEY +GS_SEQUENCER_ADDRESS +GS_SEQUENCER_PRIVATE_KEY +IMPL_SALT +L1_RPC_KIND +L1_RPC_URL +PRIVATE_KEY +TENDERLY_PROJECT +TENDERLY_USERNAME
```
## Configure your network
1. Enter the Optimism Monorepo
```bash
cd ~/optimism
```
2. Move into the contracts-bedrock package
```bash
cd packages/contracts-bedrock
```
3. Generate the configuration file
Run the following script to generate the getting-started.json configuration file inside of the deploy-config directory.
```bash
./scripts/getting-started/config.sh
```
## Deploy the op-satck L1 contracts 
1. Deploy the L1 contracts
```bash
forge script scripts/Deploy.s.sol:Deploy --private-key $GS_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL
```
2. Generate contract artifacts
```bash
forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --rpc-url $L1_RPC_URL
```
## Generate the L2 config files
You need to generate three important files  
 `genesis.json` includes the genesis state of the chain for the Execution Client.  
 `rollup.json`  includes configuration information for the Consensus Client.  
`jwt.txt` is a JSON Web Token that allows the Consensus Client and the Execution Client to communicate securely (the same mechanism is used in Ethereum clients).  
1. Navigate to the op-node package
```bash
cd ~/optimism/op-node
```
2. Create genesis files  
Now you'll generate the genesis.json and rollup.json files within the op-node folder
```bash
go run cmd/main.go genesis l2 \
  --deploy-config ../packages/contracts-bedrock/deploy-config/getting-started.json \
  --deployment-dir ../packages/contracts-bedrock/deployments/getting-started/ \
  --outfile.l2 genesis.json \
  --outfile.rollup rollup.json \
  --l1-rpc $L1_RPC_URL
```
3. Create an authentication key
```bash
openssl rand -hex 32 > jwt.txt
```
4. Copy genesis files into the op-geth directory  
```bash
cp genesis.json ~/op-geth
cp jwt.txt ~/op-geth
```
## Initialize op-geth
1. Navigate to the op-geth directory
```bash
cd ~/op-geth
```
2. Create a data directory folder
```bash
mkdir datadir
```
3. Initialize op-geth
```bash
build/bin/geth init --datadir=datadir genesis.json
```
## Start op-geth
1. You'll need a new terminal window to run op-geth in.
2. Navigate to the op-geth directory
```bash
cd ~/op-geth
```
3. Run op-geth
```bash
./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=8546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=42069 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```
## Start op-node
1. You'll need a new terminal window to run the op-node in.
2. Navigate to the op-node directory
```bash
cd ~/optimism/op-node
```
3. Run op-node  
`l1.domicon-nodes-contract` is the contract address that records domicon broadcast node information.
```bash
$ ./bin/op-node \
  --l2=http://localhost:8551 \
  --l2.jwt-secret=./jwt.txt \
  --sequencer.enabled \
  --sequencer.l1-confs=5 \
  --verifier.l1-confs=4 \
  --rollup.config=./rollup.json \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8547 \
  --p2p.disable \
  --rpc.enable-admin \
  --p2p.sequencer.key=$GS_SEQUENCER_PRIVATE_KEY \
  --l1=$L1_RPC_URL \
  --l1.rpckind=$L1_RPC_KIND \
  --l1.domicon-nodes-contract=0x76F90b92119E677C7C1a697216Ba6662436b7404
```
## Start op-batcher
1. You'll need a new terminal window to run the op-batcher in.
2. Navigate to the op-batcher directory
```bash
cd ~/optimism/op-batcher
```
3. Run op-batcher  
`l1-domicon-nodes-contract` is the contract that records domicon broadcast node information.   
`l1-domicon-commitment-contract` is the contract for batcher users to query index information.
```bash
./bin/op-batcher \
  --l2-eth-rpc=http://localhost:8545 \
  --rollup-rpc=http://localhost:8547 \
  --poll-interval=1s \
  --sub-safety-margin=6 \
  --num-confirmations=1 \
  --safe-abort-nonce-too-low-count=3 \
  --resubmission-timeout=30s \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8548 \
  --rpc.enable-admin \
  --max-channel-duration=1 \
  --l1-eth-rpc=$L1_RPC_URL \
  --private-key=$GS_BATCHER_PRIVATE_KEY \
  --network-timeout="40s" \
  --kzg-srs=./srs \
  --l1-domicon-nodes-contract=0x76F90b92119E677C7C1a697216Ba6662436b7404 \
  --l1-domicon-commitment-contract=cm:0x2BbECa3a09d75baBDc9A7F6c0022293d5A14B175
```
## Send transaction to rollup  
We have pre-created a test account and a script tool for sending transactions for you. You can use this account to simulate transactions in rollup. The default transaction sending frequency is every 50 milliseconds.
1. You'll need a terminal window to run the op-batcher in. 
2. download code
```bash
git clone https://github.com/HONGYI-SD/nodejs-tools.git
git checkout dev
cd nodejs-tools 
```
3. run this tool
```bash
node sendtx.mjs
```
4. If the previous step is executed successfully, you can see transaction submission records in op-geth and op-node.
## Recover DA data from domicon  
When op-node starts, it will read domicon's smart contract to obtain domicon node information. When DA data needs to be restored, the op-node will request DA data from the domicon node. If the current node cannot return data, it will try the next node.  
The core code is located in `op-node/node/dasource.go`  
1. When op-node is initialized, obtain dominode information from the contract
```go
func NewDaSource(ctx context.Context, log log.Logger, cfg *DaSourceConfig) (*DaSource, error) {
    ...
}
```
2. Query domicon for DA information based on the transaction hash
```go
func (d *DaSource) FileDataByHash(ctx context.Context, hash common.Hash) ([]byte, error){
    ...
}
```
3. try next node  
```go
func (d *DaSource) TryNextNode(ctx context.Context, rpc string) (*dial.StaticL2RollupProvider, error) {
    ...
}
```