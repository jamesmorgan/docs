# Quick Start

The following is a basic end-to-end example of how to connect to the Connext Network and make your first payment.

## Pre-requisites

This guide assumes you have familiary with `nodejs`, `npm` and other common JS development paradigms.

We will connect to the Connext Rinkeby hub hosted at `https://rinkeby.hub.connext.network/api/hub`. If you don't have Rinkeby ETH, we recommend you get some from a [faucet](https://faucet.rinkeby.io/) before continuing with this guide.

## Installation

Installation is simple. In your project directory,

```npm install --save connext@latest```

Then, continue on with: 
  - [Instantiating the Client](../instantiation.md)
  - [Depositing](../deposits.md)
  - [Making Payments and Exchanges](../payments.md)
  - [Withdrawing](../withdrawals.md)

**Warning: Be sure to use only Connext client v3.1.8 and above. Other versions have been deprecated and may be unstable**

## Additional Resources

Further documentation on the client (types, method reference, etc) can be found [here](../develop/client.md).

As an implementer, there are some [system limitations](./limitations.md) to be aware of that may impact user trust minimization.

A live mainnet implementation can be found [here](../usage/daiCard.md).

If you'd like to run your own hub (useful for developers who (i) want to allow users transact in a native token rather than Dai or (ii) want to customize [Hub options](../develop/hub.md)), please see [Running your own hub](../advanced/runHub.md).




