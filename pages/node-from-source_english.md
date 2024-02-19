# Build the Source Code

## Software Dependencies

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

1. Clone the Domicon  

    ```cd ~
    git clone https://github.com/domicon-labs/domicon.git
    ```

2. Enter the Domicon 

    ```bash
    cd domicon
    ```

3. Check out the correct branch

    ```bash
    git checkout domicon
    ```

4. Build the various packages inside of the Domicon

    ```bash
    make op-node
    pnpm build
    ```

### build `op-geth`

1. Clone the op-geth

    ```bash
    cd ~
    git clone https://github.com/domicon-labs/op-geth.git
    ```

2. Enter the op-geth

    ```bash
    cd op-geth
    ```

3. Check out the correct branch

    ```bash
    git checkout develop
    ```

4. build op-geth

    ```bash
    make geth
    ```

## Fill Out Environment Variables

* You'll need to fill out a few environment variables before you can start deploying your chain.

1. Enter the Domicon

    ```bash
    cd ~/domicon
    ```

2. Duplicate the sample environment variable file

    ```bash
    cp .envrc.example .envrc
    ```

3. Fill out the environment variable file

    Open up the environment variable file and fill out the following variables:
    
    | Variable Name | Description                                                                                                                                                                                                  |
    | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | `L1_RPC_URL`  | URL for your L1 node (a Sepolia node in this case).                                                                                                                                                          |
    | `L1_RPC_KIND` | Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: alchemy, quicknode, infura, parity, nethermind, debug_geth, erigon, basic, any. |


## Generate Addresses

* When becoming a broadcasting node, you need an address and its private key.
* `Broadcaster` This address is used for the broadcasting node.

1. Enter the Domicon

    ```bash
    cd ~/domicon
    ```

2. Generate new addresses

   * You should not use the `wallets.sh` tool for production deployments. If you are deploying an OP Stack based chain into production, you should likely be using a combination of hardware security modules and hardware wallets.

    ```bash
    ./packages/contracts-bedrock/scripts/getting-started/broadcast.sh
    ```

3. Check the output

* Make sure that you see output that looks something like the following:
    
     ```text
    Copy the following into your .envrc file:
    
    # Broadcaster account
    export GS_BROADCASTER_ADDRESS=0x9585DD9992F36A32301324F82F67e6EEbF06caE0
    export GS_BROADCASTER_PRIVATE_KEY=0xc75b7b0a1dfeb225a626cf3bbaac3bea2e86f68374a4f2a88f1769127d167fe9
    ```

4. Save the addresses

   * Copy the output from the previous step and paste it into your .envrc file as directed.

5. Fund the addressesoz(In the testing phase, it is not needed for now; you can skip it.)

    You need to provide a certain amount of tokens (DOM) to the address and authorize it for the corresponding contract 0x45a85Ad5F88DD7fFb7419FE445e95Ff48D167F5A.
    
    * `Broadcaster` — 5000 DOM

6. Registrate the broadcaster on L1 contract. The execution of the following script will complete the node registration.

    ```......```

## Load Environment variables

Now that you've filled out the environment variable file, you need to load those variables into your terminal.

1. Enter the Domicon

    ```bash
    cd ~/domicon
    ```

2. Load the variables with direnv
    
    You're about to use direnv to load environment variables from the .envrc file into your terminal. Make sure that you've installed direnv and that you've properly hooked direnv into your shell.
    
    ```bash
    direnv allow
    ```
    
    > **Warning：**
    `direnv` will unload itself whenever your `.envrc` file changes. You must rerun the following command every time you change the .envrc file.


## Initialize `op-geth`

You're almost ready to run your broadcasting node! Now you just need to execute a few commands to initialize op-geth.

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

4. Create an authentication key and copy it to op-node.

    ```bash
    openssl rand -hex 32 > jwt.txt
    cp ./jwt.txt ~/domicon/op-node/
    ```

## Start `op-geth`

Now you'll start op-geth, your Execution Client. Note that you won't start seeing any transactions until you start the Consensus Client in the next step.

1. Open up a new terminal  

    You'll need a terminal window to run op-geth in.

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
    > **Tip：** 
    --bootnodes Official Node Information
    --rollup.sequencerhttp Official Node Information

## Start `op-node`

Once you've got op-geth running you'll need to run op-node. Like Ethereum, the OP Stack has a Consensus Client (op-node) and an Execution Client (op-geth). The Consensus Client "drives" the Execution Client over the Engine API.

1. Open up a new terminal
   You'll need a terminal window to run the `op-node` in.

2. Navigate to the op-node directory

    ```bash
    cd ~/domicon/op-node
    ```

3. Run op-node

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
    After running this command, you will see `op-node` starting to synchronize with the chain."

    ```
    --p2p.static=<nodes> \
    --p2p.listen.ip=0.0.0.0 \
    --p2p.listen.tcp=9003 \
    --p2p.listen.udp=9003 \
    ```

    > **Tips：**
   Because it is developed based on op-stack, you may encounter other useless nodes. To avoid wasting time and network resources, information is specified by the following parameters.
   You can also delete this option, '--p2p.static', but you may encounter failed requests from other chains using the same chain ID.
   In this example, it is the official node information.