# 从源码构建节点

## 环境配置

| Dependency                                                    | Version  | Version Check Command |
|---------------------------------------------------------------|----------|-----------------------|
| [git](https://git-scm.com/)                                   | `^2`     | `git --version`       |
| [go](https://go.dev/)                                         | `^1.21`  | `go version`          |
| [foundry](https://github.com/foundry-rs/foundry#installation) | `^0.2.0` | `forge --version`     |
| [make](https://linux.die.net/man/1/make)                      | `^4`     | `make --version`      |
| [jq](https://github.com/jqlang/jq)                            | `^1.6`   | `jq --version`        |
| [direnv](https://direnv.net/)                                 | `^2`     | `direnv --version`    |

## Build Domicon

### Build the Domicon Monorepo

1. 克隆 Domicon  

```cd ~
git clone https://github.com/domicon-labs/domicon.git
```

2. 进入 Domicon 

```bash
cd domicon
```

3. 切换正确的分支

```bash
git checkout domicon
```

4. 构建domicon中的各种软件包

```bash
make op-node
pnpm build
```

### 构建 `op-geth`

1. 克隆 op-geth

```bash
cd ~
git clone https://github.com/domicon-labs/op-geth.git
```

2. 进入 op-geth

```bash
cd op-geth
```

3.切换正确的分支

```bash
git checkout develop
```

4.构建 op-geth

```bash
make geth
```

## 填写环境变量

* 在开始部署之前，您需要填写一些环境变量

1. 进入Domicon

```bash
cd ~/domicon
```

2. 复制示例环境变量文件

```bash
cp .envrc.example .envrc
```

3. 填写环境变量文件

打开环境变量文件并填写以下内容:

| Variable Name | Description                                                                                                                                                                                                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `L1_RPC_URL`  | L1节点的RPC (教程中是Sepolia).                                                                                                                                                          |
| `L1_RPC_KIND` | 您要连接到的 L1 RPC 类型，用于通知最佳交易收据获取。有效选项: `alchemy`, `quicknode`, `infura`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, `any`. |


## Generate Addresses

成为广播节电时，您需要一个地址及其私钥:

*   `Broadcaster` 用于充当广播节点地址.

1. 进入Domicon

```bash
cd ~/domicon
```

2. 创建新地址

* 不应该在生产部署中使用 wallets.sh 工具。
* 如果你要在生产环境中使用，你可能应该使用硬件安全模块和硬件钱包的组合。

```bash
./packages/contracts-bedrock/scripts/getting-started/broadcast.sh
```

3. 检查输出

* 请确保您看到如下输出内容:

```text
Copy the following into your .envrc file:

# Broadcaster account
export GS_BROADCASTER_ADDRESS=0x9585DD9992F36A32301324F82F67e6EEbF06caE0
export GS_BROADCASTER_PRIVATE_KEY=0xc75b7b0a1dfeb225a626cf3bbaac3bea2e86f68374a4f2a88f1769127d167fe9
```

4. 保存地址

* 将前一步骤的输出复制并粘贴到你的 .envrc 文件中，按照指示进行操作。

5. 为地址提供资金(测试网阶段，暂不需要，可跳过)

您需要为地址提供一定数量的代币(Dom),并授权给相应合约  0x45a85Ad5F88DD7fFb7419FE445e95Ff48D167F5A

* `Broadcaster` — 5000 Domicon

6. 执行以下脚本将完成节点注册

```......```

## 加载环境变量

现在您已经填写了环境变量文件，您需要将这些变量加载到终端中。

1. 进入Domicon

```bash
cd ~/domicon
```

2. 使用 direnv 加载变量

您将使用 `direnv` 将 `.envrc` 文件加载到终端中.
接下来，您需要允许 direnv 读取此文件并使用以下命令将变量加载到终端中。

```bash
direnv allow
```

> **警告：**
`direnv`每当您的`.envrc`文件更改时，都会自行卸载。
每次更改`.envrc`文件时,都必须重新运行以下命令。


## 初始化 `op-geth`

你几乎已经准备好运行你的广播节点！
现在你只需要运行几个命令来初始化`op-geth`。

1. 进入op-geth目录

```bash
cd ~/op-geth
```

2. 创建数据目录文件夹

```bash
mkdir datadir
```

3. 初始化 op-geth

```bash
build/bin/geth init --datadir=datadir genesis.json
```

4. 创建一个认证密钥,并复制到op-node中

```bash
openssl rand -hex 32 > jwt.txt
cp ./jwt.txt ~/domicon/op-node/
```

## 启动 `op-geth`

现在您将启用 `op-geth`, 你的执行客户端.
请注意，在下一步启动共识客户端之前，您不会看到任何交易。

1. 打开一个新的终端
您需要一个新的终端窗口才能运行`op-geth`。

2. 进入op-geth目录

```bash
cd ~/op-geth
```
3. 运行 op-geth

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
--gcmode=archive \
--maxpeers=10 \
--networkid=1988 \
--authrpc.vhosts="*" \
--authrpc.addr=0.0.0.0 \
--authrpc.port=8551 \
--authrpc.jwtsecret=./jwt.txt \
--rollup.disabletxpoolgossip=true \
--bootnodes "enode://dbffc218798fd2febbb1106aa910d336b33bc1b01267a9181b7411af57b37751f9ebcf24e5264dd5fc7fb7572d799d5830882445de76e334590d4662c2a23034@13.212.115.195:30303" \
--rollup.sequencerhttp http://13.212.115.195:8545
```
> **提示：** 
--bootnodes 为官方节点信息
--rollup.sequencerhttp 为官方节点信息

## 启动 `op-node`

一旦你开始运行`op-geth`，你就需要运行`op-node`.
与以太坊一样，Domicon 有一个共识客户端（`op-node`）和一个执行客户端（`op-geth`）。
共识客户端通过API“驱动”执行客户端。

1. 打开一个新的终端
您需要一个新的终端窗口才能运行`op-node`。

2. 进入op-node目录

```bash
cd ~/domicon/op-node
```

3. 运行 op-node

```bash
./bin/op-node \
--l2=http://localhost:8551 \
--l2.jwt-secret=./jwt.txt \
--sequencer.l1-confs=5 \
--verifier.l1-confs=4 \
--rollup.config=./rollup.json \
--rpc.addr=0.0.0.0 \
--rpc.port=8547 \
--rpc.enable-admin \
--l1=$L1_RPC_URL \
--l1.rpckind=$L1_RPC_KIND \
--p2p.static="/ip4/13.212.115.195/tcp/9003/p2p/16Uiu2HAmCeGLkZMk662awQ9KQMeWNqWhn3vZDkRQwZwbh9siRFnP" \
--p2p.listen.ip=0.0.0.0 \
--p2p.listen.tcp=9003 \
--p2p.listen.udp=9003 \
--l1-eth-rpc=$L1_RPC_URL \
--private-key=$GS_BROADCASTER_PRIVATE_KEY
```
运行此命令后，您将看到`op-node`开始进行链同步。

```
--p2p.static=<nodes> \
--p2p.listen.ip=0.0.0.0 \
--p2p.listen.tcp=9003 \
--p2p.listen.udp=9003 \
```

> **提示：**
因为基于op-satck而开发，您可能会看到其他的无用节点，信息通过制定以下参数来避免浪费时间和网络资源
您也可以删除该选项，`--p2p.static`但您可能会看到来自其他链使用相同链 ID 的失败请求。
本例子中为官方节点信息。