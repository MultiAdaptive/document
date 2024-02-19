# 如何创建自己的domicon op rollup  
&emsp;&emsp; 该文档演示如何创建一个自己的rollup，该rollup以domicon为Data Availability Layer。
## 程序依赖环境准备
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
## 代码准备
### domicon optimism部分
1. 下载domicon optimism代码
```bash
cd ~
git clone https://github.com/domicon-labs/optimism.git
```
2. 进入内部并选择develop-node分支
```bash
cd optimism
git checkout develop-node
```
3. 检查一下你的环境依赖
```bash
./packages/contracts-bedrock/scripts/getting-started/versions.sh
```
4. 安装依赖
```bash
pnpm install
```
5. 编译
```bash
make op-node op-batcher
pnpm build
```
### op-geth部分
1. 下载op-geth代码
```bash
cd ~
git clone https://github.com/ethereum-optimism/op-geth.git
```
2. 进入内部
```bash
cd op-geth
```
3. 编译op-geth
```bash
make op-geth
```
## 环境变量配置
在部署rollup之前需要配置一些环境变量。  
1. 进入optimism目录
```bash
cd ~/optimism
```
2. 复制一份环境变量的文件
```bash
cp .envrc.example .envrc
```
3. 打开.envrc文件，填写如下环境变量  

| Variable Name | Description                                                                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `L1_RPC_URL`  | URL for your L1 node (a Sepolia node in this case).                                                                                                                                                          |
| `L1_RPC_KIND` | Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: `alchemy`, `quicknode`, `infura`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, `any`. |

## 生成地址
你需要4个地址以及它们的私钥来设置你的链  
`admin` 用于更新、部署合约  
`batcher` 用于发送sequencer的交易数据到domicon
`proposer` 发送L2的交易结果（状态根）到L1
`sequencer`签名p2p网络中的区块  
1. 进入optimism
```bash
cd ~/optimism
```
2. 生成地址
```bash
./packages/contracts-bedrock/scripts/getting-started/wallets.sh
```
3. 上一步的输出类似下面这样
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
4. 复制上述输出到`.envrc`文件中
5. 为`admin`地址充值L1测试网ETH（建议ETH大于2.5）  
  建议通过水龙头：`https://sepolia-faucet.pk910.de/#/` 获得
6. 为`batcher`地址充值dom代币  
6.1 访问`https://sepolia.etherscan.io/address/0x2DE928B6494A6fd9194dfE33CE0Cf111E2b8Ac04#writeContract`并连接MetaMask钱包。  
6.2 在`mint`中为`batcher`地址充值Dom代币，建议数量为`10000000000000000000000`。   
6.3 将MetaMask切换到batcher账户，通过`approve`将`batcher`的Dom代币授权给`0x2BbECa3a09d75baBDc9A7F6c0022293d5A14B175`, 建议数量为`10000000000000000000000`
## 加载环境变量
您需要加载一些环境变量到您的terminal中。
1. 进入optimism
```bash
cd ~/optimism
```
2. 使用direnv加载环境变量
```bash
direnv allow
```
如果没有生效的话，可以先hook再重试一下。  
&emsp;&emsp;如果terminal是bash
```bash
 eval "$(direnv hook bash)"
```
&emsp;&emsp;如果terminal是zsh
```bash
eval "$(direnv hook zsh)"
```
3. 确认环境变量被正确加载
再次执行 `direnv allow`，如果得到如下输出，则是加载成功
```bash
direnv: loading ~/optimism/.envrc                                                            
direnv: export +DEPLOYMENT_CONTEXT +ETHERSCAN_API_KEY +GS_ADMIN_ADDRESS +GS_ADMIN_PRIVATE_KEY +GS_BATCHER_ADDRESS +GS_BATCHER_PRIVATE_KEY +GS_PROPOSER_ADDRESS +GS_PROPOSER_PRIVATE_KEY +GS_SEQUENCER_ADDRESS +GS_SEQUENCER_PRIVATE_KEY +IMPL_SALT +L1_RPC_KIND +L1_RPC_URL +PRIVATE_KEY +TENDERLY_PROJECT +TENDERLY_USERNAME
```
## 配置网络
1. 进入optimsim
```bash
cd ~/optimism
```
2. 进入contracts-bedrock
```bash
cd packages/contracts-bedrock
```
3. 生成配置文件
执行下列命令，会在目录`deploy-config`下生成配置文件`getting-started.json`
```bash
./scripts/getting-started/config.sh
```
## 部署op-satck L1智能合约
1. 部署合约
```bash
forge script scripts/Deploy.s.sol:Deploy --private-key $GS_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL
```
2. 生成合约的artifacts
```bash
forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --rpc-url $L1_RPC_URL
```
## 生成L2配置文件
&emsp;&emsp;有3个重要的文件需要生成  
 `genesis.json` includes the genesis state of the chain for the Execution Client.  
 `rollup.json`  includes configuration information for the Consensus Client.  
`jwt.txt` is a JSON Web Token that allows the Consensus Client and the Execution Client to communicate securely (the same mechanism is used in Ethereum clients).  
1. 进入op-node
```bash
cd ~/optimism/op-node
```
2. 创建genesis.json  
以下命令会在op-node目录下生成`genesis.json`和`rollup.json`
```bash
go run cmd/main.go genesis l2 \
  --deploy-config ../packages/contracts-bedrock/deploy-config/getting-started.json \
  --deployment-dir ../packages/contracts-bedrock/deployments/getting-started/ \
  --outfile.l2 genesis.json \
  --outfile.rollup rollup.json \
  --l1-rpc $L1_RPC_URL
```
3. 创建authentication key
```bash
openssl rand -hex 32 > jwt.txt
```
4. 拷贝创世相关文件到op-geth  
```bash
cp genesis.json ~/op-geth
cp jwt.txt ~/op-geth
```
## 初始化op-geth
1. 进入op-geth
```bash
cd ~/op-geth
```
2. 创建数据存储目录
```bash
mkdir datadir
```
3. 初始化op-geth
```bash
build/bin/geth init --datadir=datadir genesis.json
```
## 启动op-geth
1. 打卡一个新的terminal窗口
2. 进入op-geth
```bash
cd ~/op-geth
```
3. 运行op-geth
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
## 启动op-node
1. 打开一个新的terminal 窗口
2. 进入op-node目录
```bash
cd ~/optimism/op-node
```
3. 运行op-node  
`l1.domicon-nodes-contract`为记录domicon广播节点信息的合约地址。
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
## 启动op-batcher
1. 开启一个新的terminal窗口
2. 进入op-batcher目录
```bash
cd ~/optimism/op-batcher
```
3. 运行batcher  
`l1-domicon-nodes-contract`为记录domicon广播节点信息的合约。  
`l1-domicon-commitment-contract`为batcher用户查询index信息的合约。
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
  --l1-domicon-commitment-contract=0x2BbECa3a09d75baBDc9A7F6c0022293d5A14B175
```
## 向rollup发送交易  
我们为您预创建了一个测试账户以及一个发送交易的脚本工具，可以使用该账户来模拟rollup中的交易。 默认交易发送频率为每50毫秒一次。
1. 打开一个新的terminal窗口 
2. 下载测试脚本
```bash
git clone https://github.com/HONGYI-SD/nodejs-tools.git
git checkout dev
cd nodejs-tools 
```
3. 运行脚本
```bash
node sendtx.mjs
```
4. 上一步执行成功的话，可以在op-geth和op-node中看到有交易提交记录
## 从domicon恢复DA数据  
op-node在启动时，会读取domicon的智能合约以获取domicon node信息。当需要恢复DA数据时，op-node会向domicon node请求DA数据，如果当前node不能返回数据，则会尝试下一个node。  
核心代码位于`op-node/node/dasource.go`  
1. op-node初始化时，从合约中获取domiconnode信息
```go
func NewDaSource(ctx context.Context, log log.Logger, cfg *DaSourceConfig) (*DaSource, error) {
    ...
}
```
2. 根据交易哈希，向domicon查询DA信息
```go
func (d *DaSource) FileDataByHash(ctx context.Context, hash common.Hash) ([]byte, error){
    ...
}
```
3. 尝试下一个node  
```go
func (d *DaSource) TryNextNode(ctx context.Context, rpc string) (*dial.StaticL2RollupProvider, error) {
    ...
}
```