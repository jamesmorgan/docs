# Introduction

Connext is a plug-and-play payment channel solution for applications built on Ethereum.

This guide aims to help you get started building an application with Connext. If you're already familiar with Connext and how payment channels work and would like to learn how to develop with our system, feel free to head over to [Getting Started](../usage/gettingStarted.md).

If you're unfamiliar with terms like smart contract and private key, please refer to a more general developer guide such as [this one, compiled by the Ethereum community](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial), before continuing.

### How Payment Channels Work

1. A user opens a channel by depositing their money into our smart contract.

2. The user pays a counterparty by sending them balance updates. The balance update is valid once both the payer and the payee have digitally signed it. 

3.  When the parties are done transacting, they each submit their latest balance update to the contract; if the updates match, each party's new balance (reflecting the agreed-upon balance updates) is unlocked onchain. If the updates don't match, the most recent update takes precedence.

## Get Started

- [The Basics](../../usage/gettingStarted.md)
- [Real-world example](../../usage/daiCard.md)
- [Running your own hub (Advanced)](../../advanced/runHub.md)



## More on Payment Channels

Payment channels underpin our architecture. They allow many off-chain transactions to be aggregated into just a few onchain transactions. Here, we described the basic tenets of payment channels. If you're looking for more information, here are a few digestible resources:

* [LearnChannels](https://learnchannels.org/)
* [State Channels for Dummies Series](https://medium.com/blockchannel/counterfactual-for-dummies-part-1-8ff164f78540)
* [State Channels for Babies Series](https://medium.com/connext/state-channels-for-babies-c39a8001d9af)

