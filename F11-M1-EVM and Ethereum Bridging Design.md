# EVM and Ethereum Bridging Design

This document states Rosen Bridge base requirements to support new blockchains and architectural design on Ethereum and other EVM chains for them.


## Contents
- [Rosen Bridge Requirements](#rosen-bridge-requirements)
- [Solutions for Ethereum](#solutions-for-ethereum)
    - [Writing Data](#writing-data)
    - [Multi Signer Addresses](#multi-signer-addresses)
    - [Tokens](#tokens)
    - [dApp Connector](#dapp-connector)
    - [Transaction Chaining](#transaction-chaining)
    - [Event Distinguishability](#event-distinguishability)
    - [Transaction Fee](#transaction-fee)
    - [Data Source](#data-source)
- [Other EVM Chains](#other-evm-chains)


## Rosen Bridge Requirements

To support a blockchain in Rosen Bridge, two base requirements should be satisfied:

1. Writing Data: Various data are required to be written on the chain, such as the target chain and destination address. The chain should support writing arbitrary data on transactions
2. Multi-Signer Addresses: As the transfer is done by guards, all assets should be in an address that requires multiple signers (guards) to transfer the assets. Currently, two approaches are used by Rosen; Threshold Signature Scheme (TSS) and Multi-Signature (usually handled on the contract side)

Based on the chain architecture, some requirements are trivial or may be omitted:

1. Support Tokens
2. dApp Connector
3. Transaction Chaining
4. Event Distinguishability
5. Handling Transaction Fee


## Solutions for Ethereum

After in-depth research and testing on Ethereum, the Rosen Bridge approach for each of the requirements is as follows:

### Writing Data
Ethereum transactions have a data field that makes it possible to write arbitrary data. It is also used for contract calls, such as transferring tokens, but it is possible to write arbitrary data after contract data without interfering with the contract call. Rosen data use less than 100 Bytes, therefore, the data length is not a limitation.

### Multi-Signer Addresses
Ethereum uses ECDSA to sign the transactions and TSS supports it. Also, Multi-Signature wallets such as Safe exist and can be used.

There are two usages for multiple signer addresses in Rosen. The first one is the lock address (aka Hot address) that the guards use to transfer assets and pay for the events. The second one is the Cold address that holds the rest of the assets. Rosen utilizes TSS for the hot and Multisig wallet for the cold address.

### Tokens
As for supporting tokens, Rosen utilizes the ERC-20 standard for wrapped tokens. For transferring tokens, only the `transfer` function of the contract is supported and if a token does not have the `transfer` function (i.e. not an ERC-20 token) it cannot be supported by Rosen.

### dApp Connector
Ethereum Provider JavaScript API (EIP-1193) introduces a specification for the wallets API. The provider APIs also are available with the `window.ethereum` which can be used.

### Transaction Chaining
Since Ethereum is account-based, chaining transactions is not necessary. However, generating multiple transactions with a single nonce results only in one valid transaction. Handling chaining for multiple nonces is complex since even if the first item of the chain becomes invalid, other items are still valid. Therefore, Ethereum transactions will not chained in Rosen.

### Event Distinguishability
Normally, event transaction distinguishability is trivial, but in Ethereum, the payment transaction for two parallel identical events is the same, resulting in a single unsigned hash and on the other hand, the transaction ID is the signed hash of the transaction (i.e. not recognizable before signing) and if a guard is not present in the signing process, there is no way for him to verify if the transaction was successful or not. This will open the way to other attacks, such as double payments for a single event while holding the transaction signature. To prevent this issue, event transactions should be distinguishable and for this purpose, Rosen writes the event ID (i.e. Blake2B hash of the lock transaction) in the payment transaction.

### Transaction Fee
Ethereum fees may fluctuate based on network activity, as a busy network may result in higher fees for transactions (it is possible to see a 700% increase in fees in just one or two hours). In such cases, it is necessary to handle fee fluctuation as much as possible.

There are two parameters in the transaction fee. The first one is the gas limit which specifies the maximum amount of gas that can be spent in the transaction. Guards fetch this value from the network and use multiplication of it in their transactions since the excess amount won't be spent. The second one is the gas price which specifies the amount of burned ETH per gas. Guards use the exact value that is fetched from the network in their transactions. The idea is fee cannot just increase for multiple hours, as it would stop or decrease too and since transactions won't be chained ([Transaction [Chaining section](#transaction-chaining)) if fees increase, new transactions will be created with higher fees and old transactions become invalid.

The fees are important in three stages.

The first stage is when the user is requesting the transfer. The network fee should be paid by the user which is the required fee for a transaction that will be created around 1-2 hours later. Due to network fluctuations, it is not possible to estimate the exact value of the fee at the time of the transaction. Therefore, we take the current fee of the network and increase it by a small percentage to use it as the network fee.

The second stage is when the request reaches guards and they want to consensus on the payment transaction. Based on how busy the guards are, signing and sending this transaction may take 3 minutes or multiple hours. Usually, this process won't take much time. Thus, no estimation is required and the fee is calculated based on the parameters which are described above.

The third stage is when the transaction is ready to be sent. Before sending a transaction to the network, the required fee is calculated. If the transaction gas limit is less than the required amount, the transaction won't be sent to prevent failure. The current gas price of the network is irrelevant since low gas price won't cause failure and the transaction just won't be mined. The guards will wait until the fee is sufficient or another transaction is submitted which makes this transaction invalid and a new transaction with a higher fee will be created for it.

Note that although the network fee that is paid in the first stage is based on the required fee in the second and third stages, The fee is selected independently; i.e. if fees decrease and the paid fee by the user is more than the required fee, the excess amount goes to guard treasury and if fees increase and the paid fee by the user is less than the required fee, the guards have to pay it from the treasury.

### Data Source
Other than the requirements, Rosen needs to interact with the blockchain in different stages. Watchers need to scan the network to capture transfer requests. Guards need to verify Watchers reports and also generate and send payment transactions. The interface to fetch the required data is called Data Source and usually RPC and Explorer APIs are used.

For the watcher side, using a personal node is highly recommended and RPC API is supported using the TypeScript `ethers-js` library. For the guard side, supporting multiple data sources is more important. For the start, RPC API is supported and other sources such as explorers and GraphQL APIs will be added along the way. Unfortunately, no explorer nor the RPC supported all the required functionality of the guards and unlike other networks, a scanner is required. Note that this is just a design principle and the scanner requirement of the guards may be solved as the implementation proceeds.

## Other EVM Chains
The research on the above requirements is done with attention to other EVM networks. The provided solutions apply to other EVM networks, though some of them can be omitted on other networks. For example, Transaction fees don't fluctuate so much in other chains such as Binance and a complex solution for handling fee fluctuation is not necessary and a fixed fee can be used as it's used for Cardano and Ergo.
