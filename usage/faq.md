# FAQ

## Technical

## General

### What is Connext?

Connext is a scaling solution for the Ethereum network, leveraging payment channel hubs to power fast and cheap ETH and ERC20 token transactions. Using Connext, developers can enable scalable micropayments in their wallets, applications and protocols.

### What problems does Connext solve?

We believe that there is a global need for digital peer-to-peer micropayments. However, existing payment infrastructure is built around institutions rather than individuals, where institutions have to shoulder the cost burden of preventing fraud. Because of this, the cost of making payments is too high and requires too much coordination, making micropayments below a certain price threshold unfeasible.

We also believe that blockchains offer a solution to some these problems. By minimizing trust in peer-to-peer interactions, blockchains dramatically lower the cost of system-wide regulation (and truly trustless systems may remove the need for regulation altogether). Despite these benefits, blockchains struggle to meet the performance requirements of a micropayment-based economy without either sacrificing security or decentralization.

In other words, p2p micropayments become possible when you can preserve the underlying benefits of a blockchain while also leveraging the cost and speed advantages of being off-chain (on "layer two"). Connext is a layer two network which offers solutions to the following issues:  

**Traditional payment processors like Visa are capable of processing over 50,000 transactions per second (TPS); the Ethereum blockchain averages around 10 TPS.**

With Connext Hubs, there is no theoretical limit to transaction volume or speed; rather, our technology scales with its user volume, and server capacity will dictate maximum throughput. 

**Long transaction confirmation times (15-30 seconds) hinder UX.**

Transactions via a Connext Hub offer instant finality while preserving the security of the base blockchain. Much like settling a tab at a bar, confirmation times are only incurred when a user closes their "channel" with the Connext Hub.

**High transaction (gas) costs limit the ability to do micropayments.**

Because individual transactions using a Connext Hub occur off-chain, users do not incur transaction fees until they close their "channel" and settle their final balance with the blockchain. Because the cost of opening and closing channels (in the form of on-chain transaction fees) is amortized over the volume of payments in that channel, payment channels enable micropayments and small-value transactions for which the on-chain transaction fees would otherwise be too expensive to justify. 

### What is the status of Connext?

Connext is live in production with a custodial version of our Hubs. Noncustodial code is complete but not yet deployed pending testing; networked Hubs (nodes) will be shipped with V2 in mid-late 2019

You can set up your own Hub by following the instructions in the Getting Started Guide. If you have any questions, feel free to reach out to the team directly on our [Discord](https://discordapp.com/invite/yKkzZZm).

### How does Connext compare to other state channel solutions?

Connext is focused specifically on payments, and we've built our system from the ground up with the end user in mind. In addition to usability, Connext is designed for a seamless developer experience.

### How does Connext compare to Plasma/sidechains?

Plasma is a proposed framework for scaling Ethereum capacity by using hierarchical sidechains. While it offers significant speed and latency improvements over Ethereum itself, it cannot offer the near-zero latency and near-free transaction costs that Connext can. Moreover, Connext can be complementary to Plasma sidechains much as it is to Ethereum itself.

### Are there fees?

Our implementation Indra, our client, and our contracts are open source and free to use! Hub operators have the ability to set fees as they desire, but we do not take any fees for using Connext.

### Is there a whitepaper?

Not at the moment. We're seeing such a pressing need for Layer 2 scaling solutions that our time is best spent putting our ideas into action. However, we have written a protocol specification which can be found here.

### Is there a token?

No. At present, we don't see any way to integrate a token into such low level infrastructure without either generating friction or creating usability barriers. Our goal is to be as permissionless as possible.

## Payment Channels

### What happens if two parties disagree about an offchain transaction?

Each state update that is signed by both parties is assigned a "nonce", or a number that uniquely identifies that update. More recent nonces trump older nonces.

As soon as Party A (let's call her Alice) submits a state update, a challenge period starts. During this period, Party B (let's call him Bob) has the opportunity to submit an update with a more recent nonce. When the challenge timer expires, the update with the most recent nonce is used to unlock the blockchain state and distribute funds appropriately.

If the challenge period expires and Bob hasn't submitted a more recent state update than Alice, funds are disbursed according to the most recent double-signed state update (in this case, the one submitted by Alice). In this case, Bob isn't actually cheated out of any money--he misses out on the more recent state update (which is likely beneficial to him). Everyone gets paid the amount that they most recently agreed upon.

Obviously, that's a situation most people would like to avoid, especially since a party who intentionally submits an old state probably lost money in a later state update. There are a few potential solutions to this; one is the challenge timer itself, which at the very least allows Bob some time to submit his state update. Another is a third-party "watchtower" system, which would automatically monitor the channel and (for a small fee) submit the most recent update for Bob even if he's offline.

### Why do you use hubs? 

Single channels (i.e., Party A connected to Party B) work well if you have a financial relationship with some entity or person where you make payments frequently or in metered amounts. Most payments between two specific users, however, are relatively infrequent; few payment paradigms entail repeated payments between the same counterparties. Consider an ecosystem of Parties A, B, C, and D: Party A might not pay any individual counterparty with sufficient frequency to justify a channel, but they might pay Parties B, C, and D often enough for onchain transactions to become prohibitively costly. 

There are several solutions to this: one, as implemented by Bitcoin's Lightning Network, is to find a route from Party A to Party B, C, or D through a network of peers. Our solution is to have those all users connect to a Hub, typically run by a business that needs to facilitate P2P payments. Once connected to the Hub, users are able to pay any other user that is also connected to that Hub.

Here's how it works: the Hub translates a single state transition (Party A pays Party B 1ETH) into two: Party A pays the Hub 1ETH, then the Hub pays Party B 1ETH. Under the hood, this is a simple calculation on the hub's behalf: decrement the balance of Party A by 1ETH and increment the balance of PartyA by 1ETH.



### Doesn't that mean you're centralized?

Sort of! Centralization is different from trusted, and refers to network topology. While we'd like to eventually move towards a decentralized network, our more immediate concern is trust-minimization. That is, we want to ensure that users always have control of their funds and do not have to trust a third party to move money for them.

Our current release is custodial, meaning that the Hub briefly takes ownership of the funds that Party A pays Party B. This means that users currently rely on the Hub to forward the payment; because of demand from the community, we made the decision to ship a usable product and iteratively improve on trustlessness and decentralization. 

In our noncustodial release (code written, audited and deployed but not enabled pending testing), peer-to-peer transactions will be conducted in P2P "threads" rather than channels. In this paradigm, the hub doesn't need to take money from Party A, hold it, and pass it to Party B. The Hub is never actually moving around user money; rather, it just rebalances the funds with which it has collateralized the users' channels. Moreover, it will always be possible to conduct all off-chain state transitions that the hub facilitates, on-chain via the contract. As a result, users will not need to trust the Hub.