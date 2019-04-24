# Core Concepts

There are a few core concepts to be aware of as an implementer.

## Signing State Updates

The signing wallet is the ethereum address that is used to sign the state updates within the channel. The signing wallet can be any ethereum account you have access to (such as Metamask), however, it is important to understand how signing affects user experience.

When using a Metamask account as the signer, for instance, your users will have to explicitly approve any signature requested of them. The resultant signature pop-ups must be resolved before the channel state can be advanced, and often significantly detract from UX.

To avoid this, you can implement a custom provider that defaults to approving and signing messages without requiring explicit user input (i.e. a signing pop-up). An example autosigner implmentation can be found [here](https://github.com/ConnextProject/card/blob/master/src/utils/ProviderOptions.ts). Error checks against malformed state updates are performed before signing, and any application-specific errors should be checked before calling Connext functions to avoid an improper state update.

## Collateral

If you are using trust-minimized payments, you will only be able to make payments if the hub has at least that amount in the payee's channel. For example, if Alice pays Bob 10 DAI through the hub, the hub must have at least 10 DAI in its channel with Bob to facilitate the payment. If Alice and Chris want to pay Bob 10 DAI each, as tips during videogame streaming for example, the hub must have at least 20 DAI in it's channel with Bob.

To collateralize the hub, send funds to the contract address.

### Custodial payments don't need collateral

Collateral requirements only apply if you are using non-custodial payments. To bypass these requirements completely, use trusted hub payments by inserting the payment type `PT_CUSTODIAL`. This allows the hub to forward along payments it receives, without having the collateral in the recipients channel.

### Autocollateralization

The hub handles these requirements by using an autocollateralization mechanism that is triggered by any payment made, whether or not the payment was successful. Hubs determine amount of collateral needed in a channel based on the number and value of recent payments made to the recipient. Additionally, there are floors and ceilings implemented by hub operators to minimize the amount of collateral that is locked in hub channels, as well as set a minimum amount of collateral to be maintained in each channel.

Typically, hub balances below 10 DAI will trigger recollateralization. Hubs will put up to 170 DAI in any one channel. These values are configurable, so contact your hub operator for more details. Additionally, check out the [hub](../advanced/hub.md) documentation for additional configurable parameters.

### Implementer considerations

As an implementer, this can have several important consequences. First, you should expect payments to fail if there is not sufficient collateral, and retry them after monitoring the hubs collateral in the recipients channel via `connext.recipientNeedsCollateral`. Additionally, you should expect to see payments fail if they are uncharacteristically large, occur right after a withdrawal or dispute, or are the first payments in a long time.

This means if you do not send a failing payment to trigger collateraliztion, it will not account for that payment amount when recollateralizing. Deposits where the hub is recollateralizing a channel who's balance is below the floor should not be blocking updates, so long as the hub still has sufficient `balanceTokenHub` in the recipient's channel.

The hub can reclaim collateral by disputing the channel, or by the client submitting a withdrawal request with 0 value to allow the hub to withdraw excess collateral from the channel. By minimizing the hub collateral, you also reduce the amount of user channels the hub disputes (lowering user gas costs and wait times) and help payments pass more smoothly through the network.

## Availability

The wallet must acknowledge every state update by cosigning that state. This means in order to update your channel, the user must be online.

Again, availability requirements only apply if you are using non-custodial payments and using the payment type `PT_CUSTODIAL` will relax these, and rely on a trusted hub.

## Trust Assumptions

While the underlying protocol is completely noncustodial, there are trust assumptions to the Dai Card implementation which we want to make explicit. We plan to address these assumptions over the next few months.

### Hub can intercept payments

For now, the payments themselves are trusted. In other words, a hub can steal your payment value while the transaction is in-flight. You would usually notice if this happened and leave the system with the rest of your funds, so the risk is limited to your payment itself.

The code for fully noncustodial payments is already in our codebase and is functional. We have left it deactivated for now to reduce complexity while we work on improving logic for channel collateralization. We plan on reactivating it within the next couple of weeks.

### You don’t always have access to your state

Right now, if you go offline, the hub is the only entity that persists your channel’s state. From a usability perspective, this is great because it allows for easy cross-device support and keeps state in sync. Obviously, this is bad in the event that the hub goes down and you need to recover your funds from the contract directly using your latest state.

“Data availability” is a known problem for channels. To solve it, we need access to a highly reliable/available decentralized data store. Within the next 2–4 weeks, we plan to backup user state on IPFS as a temporary fix. We’re still researching a solution that would be effective long term and at scale.

### Link payments and other async payments are trusted

Link payments and other async/offline payments are currently done by having the hub hold the payment until some condition for the payment can be proven to be resolved by the recipient.

This can be partly trust-minimized by allowing for conditional resolution of payments (i.e. “generalized state channels”) in our contract. The specification to do this has already been completed and will only require a change to <10 lines of the ChannelManager code (est. 4–6 weeks to ship). However, some service provider may need to be available to “receive” the state here, which we’re still researching how to resolve — everything in time, we’re working quickly.

### Hub is centralized and can censor payments

Our hosted hub is currently operated by us (Connext) and is centralized off-chain similar to how 0x relayers like Radar Relay work. This means that our hub could be censored, DDoS’d or shut down, which would mean that our payment service could go offline. 

Hosting a hub is an interim solution to solve cheap, fast payments now while we work on building out the remaining pieces needed for a decentralized state channel network. We can fix our own centralization by allowing other Connext hubs (at that point, nodes) to be networked together, by making it easy for anyone to run a node, and by moving to sending state updates over some P2P messaging protocol rather than over https. These changes will require some quality of life improvements to our contract functions, however, so we expect to only start work on them after our next contract iteration is deployed.