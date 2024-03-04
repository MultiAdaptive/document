# How to Publish DA data to Domicon

## batcher work flow

### 1. Read the contract on L1, get the broadcast node info

    1.1 read all the info of broadcast  
    ```go
    // get all node address
    bcNodeAddrs := new([]interface{})
    err := l1DomiconNodesContract.Call(&bind.CallOpts{}, bcNodeAddrs, "BROADCAST_NODES")
    ```
    1.2 test the net, and choose the best one
    ```go
    // select best node
    ch := make(chan struct {
    	duration time.Duration
    	url      string
    }, len(nodesAddrRpc))
    for rpcUrl, _ := range nodesAddrRpc {
    	go testRPCLatency(rpcUrl, ch)
    }
    var fastestUrl string
    for i := 0; i < len(nodesAddrRpc); i++ {
    	item := <-ch
    	if item.duration == time.Duration(0) {
    		continue
    	}
    	if item.duration < minDuration {
    		minDuration = item.duration
    		fastestUrl = item.url
    	}
    }
    ```
### 2. build batcher data to Domicon

    2.1 get the index value of the user  
    ```go
    l1DomiconCommitmentContract.Call(&bind.CallOpts{}, results, "indices", userAddr)
    ```
    2.2 call kzg-sdk, get the data commtiment of the batcher data
    ```go
    digest, err := kzgsdk.GenerateDataCommit(candidate.TxData)
    if err != nil {
    	return nil, fmt.Errorf("failed to generate data commit: %w", err)
    }
    rawCD.CM = digest.Bytes()
    ```
    2.3 call kzg-sdk, sign (tx.sender, tx.to, gasPrice, index， the length of tx.data， CM)  
    ```go
    singer := kzgsdk.NewEIP155FdSigner(big.NewInt(chainID))
    sigHash, sigData, err := kzgsdk.SignFd(m.cfg.From, rawCD.To, gasPrice, *m.index, uint64(length), rawCD.CM[:], singer, m.cfg.PrivateKey)
    if err != nil {
    	return nil, fmt.Errorf("failed to sign commitment data : %w", err)
    }
    log.Info("craftCD", "sigHash", sigHash)
    ```
### 3. Publish Data

    call sendDA of rollupClient to publish the data
    ```go
    cCtx, cancel := context.WithTimeout(ctx, m.cfg.NetworkTimeout)
    hash, err := rollupClient.SendDA(cCtx, rawCD.Index, rawCD.Len, rawCD.To, rawCD.From, hexutil.Bytes(rawCD.CM[:]),
    		hexutil.Bytes(rawCD.Sig), hexutil.Bytes(rawCD.Data))
    ```