# Introduction

Connext is an infrastructure layer on top of Ethereum that lets projects do instant, high volume transactions with arbitrarily complex conditions for settlement. The goal of the Connext Network is to help developers build applications on top of Ethereum that can scale to large consumer bases without affecting usability or cost.

Connext does this by batching signed Ethereum interactions which can all be settled later on the blockchain without needing to trust intermediaries. Users deposit assets into a "state channel" with a counterparty and cryptographically sign, send and receive conditional updates to the asset balance which can be used at any time to make a finalized transaction on the base blockchain. This trasnaction batching process can further extended to multiple counterparties by linking channels together into a network. 

Connext can be used to dramatically lower the speed and cost of transacting on the Ethereum blockchain. Projects in the space are already using Connext to enable instant wallet payments, monetize content with microtransactions, and power marketplaces on the Ethereum mainnet!

If you're already familiar with Connext and how state channels work and would like to learn how to develop with our system, check out the [Quick Start](../usage/gettingStarted.md).

If you're unfamiliar with terms like smart contract and private key, please refer to a more general developer guide such as [this one, compiled by the Ethereum community](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial), before continuing.

## Status

V1.0 of Connext is *live* on the Ethereum mainnet and already being used by prominent projects in the space to scale their transactions.

This iteration of Connext features basic payment transaction support limited to Ether and the [Dai stablecoin](https://makerdao.com). Transactions are routed through a centralized Hub that is hosted by the Connext team which can be connected to with any [Connext client](../develop/client.md). User's funds are completely noncustodial, though there are instances where payments themselves, while in-flight, may place a trust burden on the Hub. For a detailed overview of the trust assumptions that exist at present, please read [Core Concepts](../usage/coreConcepts.md).

Development of v2.0 of Connext is currently underway. The coming update will move Connext onto the [CounterFactual Framework](https://counterfactual.io) and enable more abstract conditionality for settling payments, much stronger trust-minimization, and the ability to route transactions over multiple decentralized nodes. 

## How State Channels Work

1. A user opens a channel by depositing their money into our Channel Manager smart contract with the Hub.

2. The user transacts to any counterparty by sending balance updates. The balance update is valid once both the payer and the payee have digitally signed it. 

3.  When parties are done transacting, they each submit their latest balance update to the contract; if the updates match, each party's new balance (reflecting the agreed-upon balance updates) is unlocked onchain. If the updates don't match, the update with the highest transaction nonce takes precedence.

State channels allow many off-chain transactions to be aggregated into just a few onchain transactions. If you're looking for more information, here are a few digestible resources on how they work:

* [EthHub's Layer Two Scaling Page](https://docs.ethhub.io/ethereum-roadmap/layer-2-scaling/state-channels/)
* [State Channels for Dummies Series](https://medium.com/blockchannel/counterfactual-for-dummies-part-1-8ff164f78540)
* [State Channels for Babies Series](https://medium.com/connext/state-channels-for-babies-c39a8001d9af)

