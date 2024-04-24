# Building Orbit Anytrust Chain with Ethereum Sepolia as L1 and Domicon as DA Storage Layer

## Environment Setup

- Host Configuration: Recommended `t2.xlarge`&`disk > 128GB`
- Ubuntu 22.04
- Docker
- Docker Compose
- Node.js v18.20.1

## Code Preparation

- nitro 

```sh
git clone --recurse-submodules https://github.com/domicon-labs/nitro.git
```

- arbitrum-orbit-sdk

```sh
git clone https://github.com/HONGYI-SD/arbitrum-orbit-sdk.git
```

## Preparatory Work

- Deployer Account: Fund with 2 Sepolia ETH

```txt
addr:
    0x25ee6b8f826319ed6ab37ee658ef7eabfd25d77c
privatekey:
    0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c
```

- Batch Poster: Prepare 1 Sepolia ETH

```txt
addr:
    0xee196d61ecded5a6d04dfd86e7f473d05753ef46
privatekey:
    0xccd7971bbef031c46932000dda77243359d217c73bc150f4b84594d753878a69
```

- Validator, Prepare 1 sepolia eth

```txt
addr:
    0x33c11307b671c8375bc4f2a9663cc4c90a902fd2
privatekey:
    0xfa58f8005275254f89e1d10c82437eb9059d6957194e9df2ac33de746d684def
```

- Multiple Wallet Addresses: Prepare 1 Sepolia ETH for the 0th address

```txt
mnemonic:
    novel silent someone couch shock frame purse life rich large sibling crop
addr:
    0: 0x700BAAcc25b1977b274d832E22D2FEa8055A2Ba8
    1: 0x53BEA5Eecca70Ab3DeBC9389963C46D5fb5Ec7CB
```

## Prepare node-config.json

Navigate to the `arbitrum-orbit-sdk` directory you just downloaded, and go into the `examples` directory.

### Run `create-rollup-eth`

- Set environment variables

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

- Check whether `dac` in `index.ts` is turned on `DataAvailabilityCommittee: true`

- Run `yarn dev` to get the transaction hash

```txt
Deployment transaction hash is 0xfdc29e52b3baf83a9bcd10ed05bc83a162f46bab4c14214feeadb21b7865e213
```

### Run `prepare-node-config`

- Set environment variables

```sh 
cp .env.example .env
```

 where `ORBIT_DEPLOYMENT_TRANSACTION_HASH` is `txhash` of the previous step

```.env
# Required
ORBIT_DEPLOYMENT_TRANSACTION_HASH=0xfdc29e52b3baf83a9bcd10ed05bc83a162f46bab4c14214feeadb21b7865e213
BATCH_POSTER_PRIVATE_KEY=0xccd7971bbef031c46932000dda77243359d217c73bc150f4b84594d753878a69
VALIDATOR_PRIVATE_KEY=0xfa58f8005275254f89e1d10c82437eb9059d6957194e9df2ac33de746d684def

```

- Run `yarn dev` to get `node-config.json`, and record `sequencerInbox` and `upgradeExecutor` from the logs:

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

## Configuring Nitro-related Settings

- Locate the `nitro` repo
- Navigate to  `nitro/nitro-testnode`
- Edit `scripts/config.js`, find the `writeConfigs` interface, and modify the value of `nodeConfig`

```go
    // Assign the content of the previous node-config.json to the nodeConfig variable.
    const nodeConfig = {
        // Specific content, omitted for brevity 
    }

```

- Modify the `l1mnemonic` in `scripts/consts.ts` to the mnemonic of the newly applied multiple-address wallet mentioned earlier. My mnemonic is `novel silent someone couch shock frame purse life rich large sibling crop`.

- Run the `script ./test-node.bash --init --dev`

## Setting Keyset Information

- Find the downloaded `arbitrum-orbit-sdk` and go into `set-valid-keyset`, then modify these two addresses:

    ```ts
    upgradeExecutor: '0x2fa3f939799fBb4C25c89f0e200F8805C17BA4da',
    sequencerInbox: '0x2d6ec173b84145a079288d6DF7F85Caca78E198D',
    ```

- Modify the environment variable

```.env
# Required
DEPLOYER_PRIVATE_KEY=0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c
```

- Run `yarn dev`

    If this command prompts that the `DEPLOYER_PRIVATE_KEY` environment variable is missing, you can first execute `export DEPLOYER_PRIVATE_KEY=0xa7ec84c67598b8cf6a3dde9d0d2ff4274f8e242c36d53e1824e0ae830af9928c`, then run `yarn dev`.

## Others

- Monitor running containers: `docker ps`

- View relevant logs: `docker logs `