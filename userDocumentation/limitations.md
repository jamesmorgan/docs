# Additional Considerations

There are a few considerations to be aware of as an implementer.

## Availability

The wallet must acknowledge every state update by cosigning that state. This means in order to update your channel, the user must be online. We've built several different payment types to help accomodate user availability constraints. 


## Payment Types and UX Impacts

### Link

Payment type `PT_LINK` allows you to create a preloaded link that can be redeemed for a given amount of funds. When you create a link, the amount of the link is deducted from your channel balance and the funds are locked by the node pending the receipt of a secret. Next, a link is generated with that secret. It cannot be regenerated, so don't lose it or you'll lose those funds! This link (and only this link) can be used to unlock those funds.

Link payments are a useful tool for onboarding new users and/or creating prepaid accounts.


## Autocollateralization

The node uses an autocollateralization mechanism that is triggered by any payment made, whether or not the payment was successful. Nodes determine amount of collateral needed based on user profiles (e.g., in a retail environment the cashier might be assigned a profile that gives them a large amount of collateral so they can receive many payments). Additionally, there are floors and ceilings implemented by node operators to minimize the amount of collateral that is locked in node channels, as well as set a minimum amount of collateral to be maintained in each channel.

Check out the [node](../nodeDocumentation/node.md) documentation for additional configurable parameters.

### Implementer considerations

As an implementer, this can have several important consequences. First, you should expect payments to fail if there is not sufficient collateral, and retry them after monitoring the node collateral in the recipients channel via `proposeInstallApp`. Additionally, you should expect to see payments fail if they are uncharacteristically large, occur right after a withdrawal or dispute, or are the first payments in a long time.

This means if you do not send a failing payment to trigger collateralization, it will not account for that payment amount when recollateralizing. Deposits where the node is recollateralizing a channel who's balance is below the floor should not be blocking updates, so long as the node still has sufficient `balanceTokennode` in the recipient's channel.

The node can reclaim collateral by disputing the channel, or by the client submitting a withdrawal request with 0 value to allow the node to withdraw excess collateral from the channel. By minimizing the node collateral, you also reduce the amount of user channels the node disputes (lowering user gas costs and wait times) and help payments pass more smoothly through the network.


## Current Trust Assumptions

While the underlying protocol is completely noncustodial, there are trust assumptions which we want to make explicit. We are actively addressing these assumptions, so expect this section to change over the next few months.

### You don’t always have access to your state

Right now, if you go offline, the node is the only entity that persists your channel’s state. From a usability perspective, this is great because it allows for easy cross-device support and keeps state in sync. Obviously, this is bad in the event that the node goes down and you need to recover your funds from the contract directly using your latest state.

“Data availability” is a known problem for channels. To solve it, we need access to a highly reliable/available decentralized data store. We’re still researching a solution that would be effective long term and at scale, while in the short to medium term we hope to make use of IPFS for storing latest state while clients are offline.

### Node is centralized and can censor payments

Our hosted node is currently operated by us (Connext) and is centralized off-chain similar to how 0x relayers like Radar Relay work. This means that our node could be censored, DDoS’d or shut down, which would mean that our transfer service could go offline. 

In practice, this means that the node could refuse to transfer payments; however, the node is noncustodial and *cannot steal user funds*. 

Hosting a node is an interim solution to solve cheap, fast transfers now while we work on building out the remaining pieces needed for a decentralized state channel network. We can fix our own centralization by allowing other Connext nodes to be networked together, and by making it easy for anyone to run a node.