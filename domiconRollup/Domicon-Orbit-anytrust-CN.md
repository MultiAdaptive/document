# 以 etherum sepolia 为 L1, 以 domicon 为 DA存储层 搭建 orbit anytrust chain

## 环境准备

- host 配置，建议 `t2.xlarge`, `disk > 128GB`

- ubuntu 22.04

- docker

- docker compose 

- node v18.20.1

## 代码准备

- nitro 

```sh
git clone --recurse-submodules https://github.com/domicon-labs/nitro.git
```

- arbitrum-orbit-sdk

```sh
git clone https://github.com/domicon-labs/arbitrum-orbit-sdk.git
```

## 准备工作

- deployer 账户, 充点钱进去 2 sepoia eth

```txt
addr:
    0x25ee6b8f826319ed6ab37ee658ef7eabfd25d77c
privatekey:
    0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c
```

- Batch poster, 准备 1 sepolia eth

```txt
addr:
    0xee196d61ecded5a6d04dfd86e7f473d05753ef46
privatekey:
    0xccd7971bbef031c46932000dda77243359d217c73bc150f4b84594d753878a69
```

- validator, 准备 1 sepolia eth

```txt
addr:
    0x33c11307b671c8375bc4f2a9663cc4c90a902fd2
privatekey:
    0xfa58f8005275254f89e1d10c82437eb9059d6957194e9df2ac33de746d684def
```

- 多钱包地址, 给 第 0 个地址准备 1 sepolia eth

```txt
mnemonic:
    novel silent someone couch shock frame purse life rich large sibling crop
addr:
    0: 0x700BAAcc25b1977b274d832E22D2FEa8055A2Ba8
    1: 0x53BEA5Eecca70Ab3DeBC9389963C46D5fb5Ec7CB
```

## 准备 node-config.json

找到刚才下载好的 `arbitrum-orbit-sdk`, 并进入 `examples` 目录

### 运行 `create-rollup-eth`

- 设置环境变量

```sh
cp .env.example .env
```

```.env
# Required
# Required
DEPLOYER_PRIVATE_KEY=0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c

# Optional (these will be randomly generated if not provided)
BATCH_POSTER_PRIVATE_KEY=0xccd7971bbef031c46932000dda77243359d217c73bc150f4b84594d753878a69
VALIDATOR_PRIVATE_KEY=0xfa58f8005275254f89e1d10c82437eb9059d6957194e9df2ac33de746d684def
```

- 检查 `index.ts` 中 dac 是否开启 `DataAvailabilityCommittee: true`

- 执行 `yarn dev` , 得到 txhash

```txt
Deployment transaction hash is 0xfdc29e52b3baf83a9bcd10ed05bc83a162f46bab4c14214feeadb21b7865e213
```

### 运行 `prepare-node-config`

- 设置环境变量
`cp .env.example .env`，其中 `ORBIT_DEPLOYMENT_TRANSACTION_HASH` 为上一步的 `txhash`

```.env
# Required
ORBIT_DEPLOYMENT_TRANSACTION_HASH=0xfdc29e52b3baf83a9bcd10ed05bc83a162f46bab4c14214feeadb21b7865e213
BATCH_POSTER_PRIVATE_KEY=0xccd7971bbef031c46932000dda77243359d217c73bc150f4b84594d753878a69
VALIDATOR_PRIVATE_KEY=0xfa58f8005275254f89e1d10c82437eb9059d6957194e9df2ac33de746d684def

```

- 执行 `yarn dev` 得到 `node-config.json`, 同时会在输出的日志中看到， 请记录下 `sequencerInbox` 和 `upgradeExecutor`, 在后续的操作中会用到他们：

```json
{
  rollup: '0xCFcAbbe44e66E0EC595Ce786fb4ab3482600e9B0',
  inbox: '0x72E2593dfC806Cb36cAF5969aD6b1Ebcf3972113',
  nativeToken: '0x0000000000000000000000000000000000000000',
  outbox: '0x6f77d4b816931828Aa7347aA1BdF743d2ad6d9b4',
  rollupEventInbox: '0xE6DaeC4181c3058c147bcf4230Fcd256C24A152e',
  challengeManager: '0x4D97018eE43e1f1426615B0B3d61a3C0FEdCAB30',
  adminProxy: '0xDF3433E0969712a39353b0B7B31a6502aEB9E6eE',
  sequencerInbox: '0x3f9e006a9E955eC5434154cd091ABC7752D9f730',
  bridge: '0xa80bE8E8144101A8EEf14427c7a91cd0aFFCB11A',
  upgradeExecutor: '0xf283E02A2cD690Ac111D01CEAf4f7Fd7dDF04429',
  validatorUtils: '0xb33Dca7b17c72CFC311D68C543cd4178E0d7ce55',
  validatorWalletCreator: '0x75500812ADC9E51b721BEa31Df322EEc66967DDF',
  deployedAtBlockNumber: 5766137
}
```

## 配置nitro相关

- 找到代码 `nitro`
- 进入 `nitro/nitro-testnode`
- 打开scripts/config.js, 找到接口 `writeConfigs` 修改变量 `nodeConfig` 的值

```go
    // 将前面 node-config.json 的内容赋值给 nodeConfig 变量。
    const nodeConfig = {
        // 具体内容，省略展示 
    }
```

- 修改 `scripts/consts.ts` 中的 `l1mnemonic` 为上文新申请的多地址钱包的助记词，我的助记词为 `novel silent someone couch shock frame purse life rich large sibling crop`

- 执行脚本 `./test-node.bash --init --dev`

## 设置keyset信息

- 找到下载的 `arbitrum-orbit-sdk` 并进入 `set-valid-keyset`，修改这两个地址

    ```ts
    upgradeExecutor: '0x2fa3f939799fBb4C25c89f0e200F8805C17BA4da',
    sequencerInbox: '0x2d6ec173b84145a079288d6DF7F85Caca78E198D',
    ```

- 修改 环境变量

```.env
# Required
DEPLOYER_PRIVATE_KEY=0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c
```

- 运行 `yarn dev`

    执行该命令如果提示没有 DEPLOYER_PRIVATE_KEY 这个环境变量，可以先执行 `export DEPLOYER_PRIVATE_KEY=0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c` ，然后在 `yarn dev`

## 其它

- 观察运行的容器 `docker ps`

- 查看相关日志 `docker logs `