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
