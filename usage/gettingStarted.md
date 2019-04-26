# Getting Started

This guide will walk you through how to get up and running with Connext. If you are unfamiliar with payment channels or Ethereum basics, make sure you have read the [introduction](../background/introduction.md).




## Basic Example

The following is a basic end-to-end example of how to use the client to make payments. As an implementer, there are some [core concepts](./coreConcepts.md) to be aware of that will impact user experience based on your client implementation.

The easiest way to get started is to implement the Connext client using our hosted Rinkeby hub. Point Metamask to a Rinkeby RPC url (e.g. use our hub's eth endpoint: `https://rinkeby.hub.connext.network/api/eth`), get some Rinkeby ETH from a [faucet](https://faucet.rinkeby.io/), and follow this guide.

**Warning: Be sure to use the Connext client @ 3.0.19. Other versions may be unstable**
 
  - [Instantiating the Client](../instantiation.md)
  - [Making Deposits](../deposits.md)
  - [Making Payments and Exchanges](../payments.md)
  - [Withdrawals](../withdrawals.md)



## Additional Resources

Further documentation on the client (types, method reference, etc) can be found [here](../develop/client.md).

For information on running your own hub (useful for developers who (i) want to allow users transact in a native token rather than Dai or (ii) want to customize [Hub options](../advanced/hub.md)), please see [Advanced](../advanced/runHub.md).

Check out the [Dai Card](https://daicard.io) live, and its [source](https://github.com/ConnextProject/card) for a more detailed implementation example.


