# EVM and Ethereum Modules in Rosen Bridge

This document states the chain-specific modules of the Rosen Bridge and describes their approach for Ethereum and other EVM-based blockchains.


## Contents
- [Chain Specific Modules](#chain-specific-modules)
- [EVM Implementation](#evm-implementation)
    - [Scanner](#scanner)
    - [Observation Extractor](#observation-extractor)
    - [Rosen Extractor](#rosen-extractor)
    - [Abstract Chain](#abstract-chain)
    - [Lock Transaction](#lock-transaction)


## Chain Specific Modules
Rosen Bridge core modules are designed on an abstract level, meaning that the supported chain and its traits do not affect it. Therefore, the requirements are split into common and chain-specific modules. For example, the `TxAgreement` module handles the consensus of transactions before sending them to the signing process. This module takes the transaction in the `PaymentTransaction` format and the chain of the transaction doesn't matter at all. For verifying the transaction, this module depends on the `ChainHandler` which contains the chain-specific actions of each chain.

The chain-specific modules of Rosen Bridge mainly are:

1. Scanner: For each blockchain, the watchers should process all the transactions and extract the required data. The scanner module handles fetching transactions and passing them to the extractors.
2. Observation Extractor: It extracts the transfer requests (the full data of the lock transaction with requested Rosen data) from scanner transactions.
3. Rosen Extractor: The inner module of the Observation Extractor, which extracts data from the lock transaction. The structure of the lock transaction directly depends on the implementation of this module. There are two versions of this module:
    - Network-based, which is used on the watcher side and the transaction format is based on the network, e.g. RPC or explorer.
    - Universal, which is used on the guard side and the transaction format is the same as the format used on the chain-specific modules of the guard-service.

    This approach removes data conversion on the watcher side which accelerates the scanning process since all of the transactions are processed.
4. Abstract Chain: The main chain-specific module of the guard service which handles all actions in a blockchain. There is a [separate document](https://github.com/rosen-bridge/rosen-chains/blob/dev/packages/abstract-chain/README.md) that describes the requirements of this module and how to implement them for a new blockchain.

5. Abstract Network: While the Abstract Chain module implements all the necessary actions of the chain, it requires interaction with the blockchain to fetch data or send transactions. As described in the design, many data sources such as RPC APIs or explorers are available for such purposes. The Abstract Network module states these requirements on an abstract level to enable the usage of any data sources.

6. Lock Transaction: Users request the transfer using a transaction aka the lock transaction. Depending on the design, the structure of this transaction is usually unique since the transfer data such as the target chain and destination is written on it. In such cases, a module is required to generate this transaction. For now, this module is implemented as part of the Rosen App in `@rosen-bridge/ui` which will be moved to Rosen SDK in the near future.



## EVM Implementation
Rosen uses the `ethers-js` library for almost every action in the implementation of EVM-based chains as it is a complete and simple library to interact with such blockchains. In the following, the implementation of each module is described alongside references to the codes and release packages.

Note that the base code for most of the modules is implemented for EVM and not Ethereum, meaning that it will be used for all of the EVM-based chains. The data extracted by these modules should be separated for each chain to avoid interference. This is done using the chain name, which is given to the module as an argument or the base code is defined abstractly and inherited in the child class corresponding to each chain.

### Scanner
The general approach is selected for EVM which contains the following actions:
- Fetching block at a certain height: available using the `getBlock` method of RPC
- Fetch all transactions of a block: available using the `getBlock` method of RPC (`prefetchedTransactions` field)
- Get the current height of the blockchain: available using the `getBlockNumber` method of RPC

The scanner gets the current height of the blockchain. If the last recorded height is lower than the current height, the next block will be retrieved along with its transactions and processed by all registered extractors. This process is repeated until the current block is scanned.

Reference:
- Implementation: [GitHub](https://github.com/rosen-bridge/scanner/tree/dev/packages/scanners/evm-rpc-scanner)
- Released: [NpmJs](https://www.npmjs.com/package/@rosen-bridge/evm-rpc-scanner)

### Observation Extractor
The main part of the Observation Extractor is extracting data from the transaction using the Rosen Extractor module. Since all of the data is written on the transaction itself, this module can be implemented completely abstractly as it is now. The only required action is getting the transaction ID from the transaction structure.

Reference:
- Abstract Observation Extractor Class: [GitHub](https://github.com/rosen-bridge/scanner/blob/dev/packages/observation-extractor/lib/extractor/abstract/AbstractObservationExtractor.ts)
- Implementation: [GitHub](https://github.com/rosen-bridge/scanner/tree/dev/packages/observation-extractors/evm-observation-extractor)
- Released: [NpmJs](https://www.npmjs.com/package/@rosen-bridge/evm-observation-extractor)

### Rosen Extractor
There are two types of lock transactions:
1. Lock tokens. Since transferring tokens requires the contract call which is written into the `data` field, the Rosen data is written after the contract call data. Only ERC-20 tokens and transfers using the `transfer` call are supported. The first 68 Bytes are contract call data and the remaining data are all parsed as Rosen data.
2. Lock native token. This type of transaction does not need contract calls or similar data and only the Rosen data is written into the `data` field.

The Rosen data structure is as follows:
- the target chain: 1 Byte
- the bridge fee: 8 Bytes
- the network fee: 8 Bytes
- length of encoded destination address: 1 Byte
- the destination address: custom, though a limitation is applied on the `@rosen-bridge/address-encoder` package which encodes and decodes the address for this field

Usually, transaction format differs based on the data source to accelerate extractors as this was necessary in Ergo and Cardano, but testing on Ethereum showed converting RPC transactions into the `ethers-js` transaction format does not affect speed that much. Therefore, a single implementation was enough and the universal version directly uses the RPC extractor.

Reference:
- Implementation: [GitHub](https://github.com/rosen-bridge/utils/tree/dev/packages/rosen-extractor/lib/getRosenData/evm)
- Rosen-Extractor Package: [NpmJs](https://www.npmjs.com/package/@rosen-bridge/rosen-extractor)

### Abstract Chain
The functionalities of EVM-based chains in Abstract Chain are under four categories:

- Transaction Utility, such as generating, signing and sending transactions.

    As described in the design document, the payment transactions are not chained and a maximum number of parallel transactions are generated. On the other hand, there is a function to generate multiple chained transactions which is used for transferring assets to the cold address.

    The transaction fee is also a direct implementation of the design document. A safe limit is used for the gas limit which is configurable in the `EvmChain` class and the gas price is fetched from the network.

    For the sake of event distinguishability, the event ID is written on the transaction `data` field (similar to how Rosen data is written, if the transaction is a token transfer, the event ID is written after contract call data).

    The transaction fee is checked before submission and if the gas limit is not sufficient, submission is stalled to prevent transaction failure.

    Also, the balance of the lock address is checked before submitting a transaction. Although this check is not required in native token transfers, transferring a token without having enough balance results in a failed transaction and burning fees which should be prevented as much as possible.

- Verification, which consists of multiple functions to verify a transaction generated by other guards and a function to verify an event. The conditions are:

    - Transaction fees cannot be much different from the expected value. The expected value is what the guard would use based on the current network status, and the difference is measured using the slippage config (e.g. if slippage is 10 percent and the guard expected fee is 100, the transaction is verified if the fee is between 90 and 110). This slippage check goes for both the gas limit and gas price.
    - Transaction should have the `to` and `data` field and be of type 2. 
    - Event ID should be present at the end of the `data` field
    - Transaction should not transfer multiple assets (e.g. should not transfer any Ether while transferring a token)
    - Only the `transfer` contract call should be used for token transferring

    Verifying an event is [implemented abstractly](https://github.com/rosen-bridge/rosen-chains/blob/eb591a956b155e7d64e7d4c39c0c025be9d73381/packages/abstract-chain/lib/AbstractChain.ts#L154) and no extra condition is required in EVM chains.

- Serialization, which is almost trivial and only converts transactions of the `ethers-js` library format to the `PaymentTransaction` structure used in the guard service and no additional data is required, unlike Ergo and Cardano which require complete data of input boxes. 

- Validation. The only condition that makes a transaction invalid, is when its nonce is less than the current nonce of the address in the blockchain.

Reference:
- Implementation: [GitHub](https://github.com/rosen-bridge/rosen-chains/tree/dev/packages/chains/evm)
- Released: [NpmJs](https://www.npmjs.com/package/@rosen-chains/evm)

### Lock Transaction
The lock transaction generation module is part of the UI and Rosen App, which will be implemented once the integration of modules into core parts is started. Since data is also written in the payment transactions by the guards, there is nothing new in the structure of this type of transaction as it is identical to the payment transaction with the data structure described in the Rosen Extractor module. This module is part of the frontend/UI and will be implemented for Milestones 4 and 5.
