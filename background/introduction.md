# Introduction

This guide aims to provide just enough information to get started building an application with Connext. If you're already familiar with Connext and how payment channels work and would like to learn how to develop with our system, feel free to head over to [Getting Started](../usage/gettingStarted.md).

If you're unfamiliar with terms like smart contract and private key, please refer to a more general developer guide such as [this one, compiled by the Ethereum community](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial), before continuing.

## Payment Channels

Payment channels underpin our architecture. They allow many off-chain transactions to be aggregated into just a few onchain transactions. Here, we described the basic tenets of payment channels. If you're looking for more information, here are a few digestible resources:

* [LearnChannels](https://learnchannels.org/)
* [State Channels for Dummies Series](https://medium.com/blockchannel/counterfactual-for-dummies-part-1-8ff164f78540)
* [State Channels for Babies Series](https://medium.com/connext/state-channels-for-babies-c39a8001d9af)

### How It Works

1. Users open a channel by depositing their money into a smart contract closely resembling a multisig wallet. The funds in the wallet can't be used elsewhere or removed until unlocked with an update that both parties have signed.

2. The two parties transact by passing balance updates amongst themselves. If both parties agree on a state update by "signing" it, it could be submitted to the smart contract at any time to unlock funds.

3. When parties have finished transacting, they each submit state updates to the smart contract. If the state updates match, the blockchain state (i.e., each party's balance) is unlocked, typically in a different configuration than the initial state.

4. Connext extends this model to "hubs", which allow users to make instant, trust-minimized, off-chain payments to anyone else that has a channel open with a hub.