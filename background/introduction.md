# Introduction

Connext is an infrastructure layer on top of Ethereum that enables instant, high volume transactions by reducing the amount of data that needs to be committed to the blockchain. The goal of the Connext Network is to help Ethereum applications scale to large consumer bases by improving their usability and cost. Projects in the space are already using Connext to enable instant wallet to wallet payments, monetize content with microtransactions, power marketplaces, and build games on the Ethereum mainnet!

Connext does this using *state channels*. State channels enable users to batch up normal Ethereum transactions without needing to trust intermediaries. State channels do not require any external custodians or add any additional functionality to Ethereum, they simply allow existing Ethereum interactions to occur *more quickly* and at *lower cost* by putting more interactions into each block.

If you're already familiar with Connext and how state channels work and would like to learn how to develop with our system, check out the [Quick Start Guide](../usage/quickStart.md).

If you're unfamiliar with terms like smart contract and private key, please refer to a more general developer guide such as [this one, compiled by the Ethereum community](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial), before continuing.

## State Channel Basics

State channels allow many off-chain transactions to be aggregated into just a few onchain transactions:

1. A user opens a channel by depositing their money into a multisignature smart contract with a counterparty. Note that the smart contract runs entirely on the blockchain and so the user remains *entirely* in custody of their own funds.

2. The user transacts by sending signed settlement instructions for how the counterparty can retrieve funds from the smart contract. Because the instructions give the counterparty irrevocable access to part of the funds, the user can make multiple "updates" to their balances while only paying the fees of the initial deposit.

3. When either party is done transacting, they can take the latest update to the smart contract and unlock the finalized funds.

4. Because there can be arbitrary conditionality to the settlement instructions, the above relationship can be extended to allow users to transact with more parties. For instance, if Alice wants to pay Charlie but only has a channel with Bob, Alice can pay Bob conditionally based on whether he pays Charlie. If the latter part of the transaction is not completed, then Alice's transaction to Bob will not occur either - this makes transactions *atomic* and *noncustodial*.

If you're looking for more information, here are a few digestible resources on how they work:

* [EthHub's Layer Two Scaling Page](https://docs.ethhub.io/ethereum-roadmap/layer-2-scaling/state-channels/)
* [State Channels for Dummies Series](https://medium.com/blockchannel/counterfactual-for-dummies-part-1-8ff164f78540)
* [State Channels for Babies Series](https://medium.com/connext/state-channels-for-babies-c39a8001d9af)

## Status

V1.0 of Connext is *live* on the Ethereum mainnet and already being used by some prominent projects in the space to scale their transactions.

This iteration of Connext features basic transaction support in ETH and the [DAI stablecoin](https://makerdao.com). For safety, there are also limits on the maximum capacity of channels and transactions. Additionally, this iteration of Connext features only one public node - currently hosted by Connext - over which transactions are routed, which can be connected to by any [Connext client](../develop/client.md). Note that Connext is fully open source and many organizations run their own private Connext nodes.

In Connext, users' funds are completely noncustodial, though there are instances where payments themselves, while in-flight, may place a trust burden on the node. For a detailed overview of the trust assumptions and limitations that exist at present, please read [System Limitations](../usage/limitations.md).

V1.0 is an experiment for the Connext development team to collect usage data on the system and make iterative process towards better ensuring that users *always* have custody over their own funds, even when update packets are routed over random and anonymous actors. Development of v2 of Connext is currently underway which will solve most of the above limitations. The coming update will move Connext onto the [CounterFactual Framework](https://www.counterfactual.com/) and enable better state update backups, more abstract conditionality for settling payments, much stronger trust-minimization, and the ability for nodes to communicate with each other.

At that time, we intend to shut down the Connext-hosted node completely and let the network be fully p2p.
