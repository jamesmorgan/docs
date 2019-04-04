# Introduction

This guide aims to provide just enough information to get started building an application with Connext. If you're already familiar with Connext and how payment channels work, feel free to skip down to the [Components](#Connext-Components) section. These examples are high-level and intended only to get users started; if you're looking for more detailed technical guides please refer to the READMEs of individual components (linked below).

If you're unfamiliar with terms like smart contract and private key, please refer to a more general developer guide such as [this one, compiled by the Ethereum community](https://github.com/ethereum/wiki/wiki/Ethereum-Development-Tutorial), before continuing.

## Payment Channels

Payment channels underpin our architecture. They allow many off-chain transactions to be aggregated into just a few onchain transactions. Here, we described the basic tenets of payment channels. If you're looking for more information, here are a few digestible resources:

* [LearnChannels](https://learnchannels.org/)
* [State Channels for Dummies Series](https://medium.com/blockchannel/counterfactual-for-dummies-part-1-8ff164f78540)
* [State Channels for Babies Series](https://medium.com/connext/state-channels-for-babies-c39a8001d9af)

### How It Works

1. Two users lock the initial blockchain state (i.e., each party's balance) into a smart contract closely resembling a multisig wallet. This ensures that the funds in the wallet can't be used elsewhere or removed until unlocked with an update that both parties have signed.

2. The two parties transact by passing state updates (i.e., balance updates) amongst themselves. If both parties agree on a state update by "signing" it, it could be submitted to the smart contract at any time to unlock funds.

3. When parties have finished transacting, they each submit state updates to the smart contract. If the state updates match, the blockchain state (i.e., each party's balance) is unlocked, typically in a different configuration than the initial state.

### Disputes

Each state update that is signed by both parties is assigned a "nonce", or a number that uniquely identifies that update. More recent nonces trump older nonces.

As soon as Party A (let's call her Alice) submits a state update, a challenge period starts. During this period, Party B (let's call him Bob) has the opportunity to submit an update with a more recent nonce. When the challenge timer expires, the update with the most recent nonce is used to unlock the blockchain state and distribute funds appropriately.

If the challenge period expires and Bob hasn't submitted a more recent state update than Alice, funds are disbursed according to the most recent double-signed state update (in this case, the one submitted by Alice). In this case, Bob isn't actually cheated out of any money--he misses out on the more recent state update (which is likely beneficial to him). Everyone gets paid the amount that they most recently agreed upon.

Obviously, that's a situation most people would like to avoid, especially since a party who intentionally submits an old state probably lost money in a later state update. There are a few potential solutions to this; one is the challenge timer itself, which at the very least allows Bob some time to submit his state update. Another is a third-party "watchtower" system, which would automatically monitor the channel and (for a small fee) submit the most recent update for Bob even if he's offline.

### Hubs

Single channels (i.e., Party A connected to Party B) work well if you have a financial relationship with some entity or person where you make payments frequently or in metered amounts. Most payments between two specific users, however, are relatively infrequent; few payment paradigms entail repeated payments between the same counterparties. Consider an ecosystem of Parties A, B, C, and D: Party A might not pay any individual counterparty with sufficient frequency to justify a channel, but they might pay Parties B, C, and D often enough for onchain transactions to become prohibitively costly. 

There are several solutions to this: one, as implemented by Bitcoin's Lightning Network, is to find a route from Party A to Party B, C, or D through a network of peers. Our solution is to have those all users connect to a Hub, typically run by a business that needs to facilitate P2P payments. Once connected to the Hub, users are able to pay any other user that is also connected to that Hub.

Here's how it works: the Hub translates a single state transition (Party A pays Party B 1ETH) into two: Party A pays the Hub 1ETH, then the Hub pays Party B 1ETH. Under the hood, this is a simple calculation on the hub's behalf: decrement the balance of Party A by 1ETH and increment the balance of PartyA by 1ETH.

### Custodial vs. Noncustodial

Our current release is custodial, meaning that the Hub briefly takes ownership of the funds that Party A pays Party B. This means that users currently rely on the Hub to forward the payment; because of demand from the community, we made the decision to ship a usable product and iteratively improve on trustlessness and decentralization. 

In our noncustodial release (coming in late January 2019), peer-to-peer transactions will be conducted in threads rather than channels. When two parties (who each have a channel open with the Hub) want to open a thread, they each send a signed request to the hub recording the funds that they are committing to the thread. Those funds are locked for use in the thread and cannot be spent elsewhere until they are withdrawn from the thread.

Once the hub has recorded the initial thread state, it no longer needs to observe interactions between the two parties. In a bidirectional paradigm (i.e. payments can flow in either direction), they can pass signed state updates back and forth until they wish to stop transacting and close the thread; at that point, they submit their most recent state update to the hub. The hub compares the balances in that update to the initial state and, after a challenge period and if they make sense, distributes funds between the two parties in accordance with the new state. Then, it decomposes the transaction into two payments as described above.

In this paradigm, the hub doesn't need to take money from Party A, hold it, and pass it to Party B. The Hub is never actually moving around user money; rather, it's just rebalancing the funds with which it has collateralized the users' channels. Moreover, it will be possible to conduct all off-chain state transitions that the hub facilitates, on-chain via the contract. As a result, users will not need to trust the Hub.