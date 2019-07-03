# FAQ

## General

### What Connext is:
- A scalability solution for EVM-compatible blockchains.
- A software company that builds open source technology infrastructure for Ethereum applications.
- A non-custodial batching or "compression" layer on top of Ethereum which retains the security and self-sovereignty qualities of the base blockchain.

### What Connext is not:
- A payment company.
- A custodial wallet or custody solution of any sort.
- An exchange or other financial service provider.
- A blockchain of its own.

**Connext DOES NOT in any way take custody of user funds. We don't even run the code, you do.**

### What problems does Connext solve?
Connext makes it possible to build scalable payment and transaction-related applications on the Ethereum blockchain. 

The Ethereum blockchain already enables trust-minimized payments between peers, but these payments have high costs and slow confirmation times. This makes the Ethereum blockchain very suitable as a settlement layer, but not usable for many day-to-day usecases. Connext reduces the number of transactions that need to be put onto Ethereum without changing the core decentralization and trust-minimization properties of Ethereum itself.

### Does Connext have a token or native currency?
No. We have no token.

v2.0 of Connext allows transactions to be made in any Ethereum ERC20 token.

### How does Connext compare to other state channels solutions?
Connext is live on the Ethereum mainnet, leverages existing industry standards rather than implementing custom solutions, does not have a token, and focuses largely on use cases related to conditional and simple transfers.

Connext is also very easy to use as compared to other solutions! Check out [Quick Start](../userDocumentation/quickStart.md) to learn more.

### How does Connext compare to Plasma/sidechains?
Plasma is a framework for scaling Ethereum capacity by using hierarchical sidechains. Plasma helps Ethereum scale *out* (i.e. allowing for more parallel processing) rather than scaling *up* (i.e. allowing for more transactions per block). 

The way that plasma is theorized right now still places a minor amount of trust on the plasma operator as the plasma chain custodies your assets entirely which can be lost if the chain is shut down. With Connext, your assets are always yours - they are stored in a contract which *you* have the keys to.

Even in a multichain world, an overlay clearance network like Connext is still needed for seamless UX.

### Are there fees?
Connext itself does not collect any fees from the network. Nodes providing transaction routing and storage services within the network may collect fees if they choose to do so.

### How does Connext sustain itself?
For now, Connext is backed by VCs and grants from the Ethereum Foundation. Long term, the Connext team expects to monetize through selling ancillary services for reducing the operating costs of running nodes.

### Is there a whitepaper?
No. A protocol specification for Connext v2.0 can be found in the [Counterfactual framework documentation](https://specs.counterfactual.com/en/latest/).

## State Channels

### What happens if two parties disagree about an offchain transaction?
In Connext, each state update must be signed by all channel parties and contain an update nonce. In the event that a disagreement occurs, either party can resolve the dispute on the blockchain by submitting their highest-nonce update to the channel dispute resolution contract.

The dispute resolution contract contains instructions on how a transaction should be resolved, based on the contents of the state update and/or external factors such as time or onchain events.

### Why do you host a single node for v2.0?
One of the most difficult challenges of channelized networks is ensuring that there is enough *capacity* (or collateral) in a given channel to receive transactions without interrupting the flow of payments. In a relatively simple hub and spoke system, if Alice wants to pay Bob 1 Eth through the node, Alice would first pay the node 1 Eth in her channel conditionally based on if the node would pay 1 Eth to Bob in his channel. To successfully complete this transaction, the node would need to *already* have had 1 Eth in Bob's channel, which it could only have done if it knew beforehand that Bob would be the receiver of funds.

It turns out, this is largely a data science problem related to payment behavior. Our goal with running a  node ourselves for v1.0 (and 2.0) is to prioritize usability. By collecting data and coming up with a rebalancing/recollateralization protocol beforehand, we can improve the efficiency of collateral allocation for decentralized nodes in the future.

### Doesn't that mean you're centralized?
Yes! This version of Connext is centralized. We fully acknowledge that this is the case and have been transparent about it from first deployment. V2.x of Connext, shipping in late Q3 2019, will enable support for transactions routed over multiple nodes, and will make it much simpler for any orgazation or individual to run their own node. At that point, we expect the network to become more and more decentralized.

Note that centralization here is completely different from being custodial. While we are currently the only entity routing and processing transactions, this *does not* mean that we hold user funds in any way. The primary risk here is censorship of payments themselves. Also note that our node implementation is fully open source, so anyone can come run their own node if they wanted to - in fact, many companies already do!


## Dai Card

### Why is my payment taking so long?
Some payments can seem to take forever, and it can be difficult to figure out when it happens. This has to do with the collateral requirements of passing on noncustodial payments, and it means that the Connext node is depositing into the receivers channel before agreeing to forward on the payment. This could appear as if your payment has failed, but don't worry, the node is just autocollateralizing onchain!

Check out the [System Limitations](../userDocumentation/limitations.md) section for more detailed information on the collateral requirements of the system.

### What happened to my linked payment?
If you did not save the link, it is lost for now. To get your payment refunded, or to regenerate a link, [contact us](https://discordapp.com/invite/yKkzZZm).

### My channel says its not open?
There are a couple of reasons this may be the case. First, nodes can freely dispute any channels with the latest state to reclaim excess collateral from channels. This could throw a channel into a `CS_CHANNEL_DISPUTE` state. Channels may also be thrown into a `CS_CHAINSAW_ERROR` if there are errors processing events from web3.

In both of these cases, users are free to reclaim funds by initiating onchain dispute processes. However, these can cost gas and take a while to resolve onchain, so we would reccommend [reaching out to us](https://discordapp.com/invite/yKkzZZm)

### I deposited tokens into the signing wallet, but they are not appearing in the channel?
Users depositing into their own channel is the *only* user submitted ethereum transaction, and that means it needs gas to complete. If you only send tokens to the signing wallet, it will not have enough eth to send the transaction as well. 

If this is a problem for your UX, we recommend using eth only deposits, and utilizing in-channel exchanges for a token of the nodes choosing to avoid the two onchain transactions funding a browser wallet.