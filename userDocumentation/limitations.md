# Additional Considerations

There are a few considerations to be aware of as an implementer.

## Availability

The wallet must acknowledge every state update by cosigning that state. This means in order to update your channel, the user must be online. We've built several different payment types to help accomodate user availability constraints. 


## Payment Types and UX Impacts

### Trust-minimized

If you are using trust-minimized payments, you will only be able to make payments if the hub has at least that amount in the payee's channel. For example, if Alice pays Bob 10 DAI through the hub, the hub must have at least 10 DAI in its channel with Bob to facilitate the payment. If Alice and Chris want to pay Bob 10 DAI each, as tips during videogame streaming for example, the hub must have at least 20 DAI in it's channel with Bob.

In practice, this means that when a user *receives* their first payment, the hub will need to collateralize the recipient's channel. This involves an onchain transaction, and will take 15 - 30 seconds. See [Autocollateralization](#autocollateralization) for a more detailed description of node collateralization behavior.

To collateralize the hub, send funds to the contract address.


### Custodial

Collateral requirements only apply if you are using trust-minimized payments. To bypass these requirements completely, use trusted hub payments by inserting the payment type `PT_CUSTODIAL`. This allows the hub to forward along payments it receives, without having the collateral in the recipients channel.

### Link

Payment type `PT_LINK` allows you to create a preloaded link that can be redeemed for a given amount of funds. When you create a link, the amount of the link is deducted from your channel balance and the funds are locked by the hub pending the receipt of a secret. Next, a link is generated with that secret. It cannot be regenerated, so don't lose it or you'll lose those funds! This link (and only this link) can be used to unlock those funds.

Link payments are a useful tool for onboarding new users and/or creating prepaid accounts.


### Optimistic

Optimistic payments (`PT_OPTIMISTIC`) combine elements of custodial and noncustodial payments: if a user is online, the payment will be sent as a noncustodial transfer; if not, it will be sent as a custodial transfer. This mitigates UX issues caused by availability requirements, so long as you're okay with some custodial payments.



## Autocollateralization

The hub uses an autocollateralization mechanism that is triggered by any payment made, whether or not the payment was successful. Hubs determine amount of collateral needed in a channel based on the number and value of recent payments made to the recipient. Additionally, there are floors and ceilings implemented by hub operators to minimize the amount of collateral that is locked in hub channels, as well as set a minimum amount of collateral to be maintained in each channel.

Typically, hub balances below 10 DAI will trigger recollateralization. Hubs will put up to 170 DAI in any one channel. These values are configurable, so contact your hub operator for more details. Additionally, check out the [hub](../nodeDocumentation/node.md) documentation for additional configurable parameters.

### Implementer considerations

As an implementer, this can have several important consequences. First, you should expect payments to fail if there is not sufficient collateral, and retry them after monitoring the hubs collateral in the recipients channel via `connext.recipientNeedsCollateral`. Additionally, you should expect to see payments fail if they are uncharacteristically large, occur right after a withdrawal or dispute, or are the first payments in a long time.

This means if you do not send a failing payment to trigger collateralization, it will not account for that payment amount when recollateralizing. Deposits where the hub is recollateralizing a channel who's balance is below the floor should not be blocking updates, so long as the hub still has sufficient `balanceTokenHub` in the recipient's channel.

The hub can reclaim collateral by disputing the channel, or by the client submitting a withdrawal request with 0 value to allow the hub to withdraw excess collateral from the channel. By minimizing the hub collateral, you also reduce the amount of user channels the hub disputes (lowering user gas costs and wait times) and help payments pass more smoothly through the network.


## Current Trust Assumptions

While the underlying protocol is completely noncustodial, there are trust assumptions to the Dai Card implementation which we want to make explicit. We are actively addressing these assumptions, so expect this section to change over the next few months.

### Hub can intercept payments

For now, the payments themselves are trusted. In other words, a hub can steal your payment value while the transaction is in-flight. You would usually notice if this happened and leave the system with the rest of your funds, so the risk is limited to your payment itself.

The code for fully noncustodial payments is already in our codebase and is functional. We have left it deactivated for now to reduce complexity while we work on improving logic for channel collateralization.

### You don’t always have access to your state

Right now, if you go offline, the hub is the only entity that persists your channel’s state. From a usability perspective, this is great because it allows for easy cross-device support and keeps state in sync. Obviously, this is bad in the event that the hub goes down and you need to recover your funds from the contract directly using your latest state.

“Data availability” is a known problem for channels. To solve it, we need access to a highly reliable/available decentralized data store. We’re still researching a solution that would be effective long term and at scale, while in the short to medium term we hope to make use of IPFS for storing latest state while clients are offline.

### Link payments and other async payments are trusted

Link payments and other async/offline payments are currently done by having the hub hold the payment until some condition for the payment can be proven to be resolved by the recipient.

This can be partly trust-minimized by allowing for conditional resolution of payments (i.e. “generalized state channels”) in our contract. The specification to do this has already been completed and will only require a change to <10 lines of the ChannelManager code (est. 4–6 weeks to ship). However, some service provider may need to be available to “receive” the state here, which we’re still researching how to resolve — everything in time, we’re working quickly.

### Hub is centralized and can censor payments

Our hosted hub is currently operated by us (Connext) and is centralized off-chain similar to how 0x relayers like Radar Relay work. This means that our hub could be censored, DDoS’d or shut down, which would mean that our payment service could go offline. 

Hosting a hub is an interim solution to solve cheap, fast payments now while we work on building out the remaining pieces needed for a decentralized state channel network. We can fix our own centralization by allowing other Connext hubs (at that point, nodes) to be networked together, by making it easy for anyone to run a node, and by moving to sending state updates over some P2P messaging protocol rather than over https. These changes will require some quality of life improvements to our contract functions, however, so we expect to only start work on them after our next contract iteration is deployed.