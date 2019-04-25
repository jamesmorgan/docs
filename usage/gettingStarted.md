# Getting Started

Integrating Connext and the hosted Dai hub allows you to make instant, trust-minimized, off-chain payments to anyone else connected to the hub. This guide will walk you through how to get up and running with the client and the hosted Hub. If you are unfamiliar with payment channels or Ethereum basics, make sure you have read the [introduction](../background/introduction.md).

The easiest way to get started is to implement the Connext client using our hosted Rinkeby hub. Point Metamask to a Rinkeby RPC url (e.g. use our hub's eth endpoint: `https://rinkeby.hub.connext.network/api/eth`), get some Rinkeby ETH from a [faucet](https://faucet.rinkeby.io/), and follow the guide below.

For information on running your own hub (useful for developers who want to allow users transact in a native token rather than Dai), please see [Advanced](../advanced/runHub.md).

## Implementing the Client

### Configuration

First, install the client package:

```javascript
npm install connext
```

### Basic Example

**Warning: Be sure to use the Connext client @ 3.0.19**

The following is a basic example of how to use the client to make payments. As an implementer, there are some [core concepts](./coreConcepts.md) to be aware of that will impact user experience based on your client implementation.

To start using a channel, just deposit from the signing wallet into the ChannelManager contract and start making payments:


**Instantiate and Start the Client**

First, let's start up Connext. The constructor accepts a Web3 object, the URL of the hub you're connecting to (in most cases, https://rinkeby.hub.connext.network/api/hub), a user address, and the host URL of your application. We'll use `getConnextClient` to spin up an instance of Connext and assign it to a variable. 
 
```javascript
import { getConnextClient } from "connext/dist/Connext.js";
const Web3 = require("web3");

// URLs for a live rinkeby hub
hubUrl=https://rinkeby.hub.connext.network/api/hub
ethUrl=https://rinkeby.hub.connext.network/api/eth

// URLs for local development
hubUrl=http://localhost:3000/api/eth
ethUrl=http://localhost:3000/api/hub

// Instantiate web3, get signer
const web3 = new Web3(new Web3.providers.HttpProvider(ethUrl))

async function() {

  const accounts = await web3.eth.getAccounts()

// Set the client options
  const connextOptions = {
    web3,
    hubUrl,
    user: accounts[0],
    origin: 'localhost' // the host url of your app
  }

// Instantiate a new instance of the client
  const connext = await getConnextClient(connextOptions)
```

Now that you've instantiated the client, you'll want to register a listener for the events that it emits and then start Connext!

```javascript
// Start the app, and register a listener
  connext.on('onStateChange', connext => {
    console.log('Connext:', connext)
  }
// Start Connext
  await connext.start()
}
```

**Making a deposit to a channel**

Now that the client is started, you can make a deposit into a channel. Channels can accept deposits in both ETH and tokens. However, when depositing tokens, ensure the user has sufficient ETH remaining in their wallet to afford the gas of the deposit transaction.

```javascript
  // Making a deposit in ETH
  await connext.deposit({
    amountWei: "1500",
    amountToken: "0", // assumed to be in wei units
  })
```

**Payments and In-channel Swaps**

Now that you've deposited, you have a payment channel open with the hub. You're ready to start transacting by updating your channel state. Connext facilitates in-channel exchanges, token payments, and channel withdrawals in Wei.

Note: The hub is responsible for "collateralizing" all payments. If the hub does not have funds to forward the payment in the payment recipient's channel, the payment will fail until the hub is able to deposit more tokens into the channel. To add collateral to the hub, simply send tokens (Dai if you're using our hosted hub) to the contract address. See the [Collateral section](./coreConcepts.md#collateral) to learn more.

```javascript
  // Exchanging Wei for Dai
  await connext.exchange("1000", "wei");

  // Making a Dai payment
  // Payments made can be retrieved using the returned purchase id,
  // which ties together all payment channel updates initiated in the array.
  const purchaseId = await connext.buy({
    meta: {
      // Any metadata associated with the payment
    },
    payments: [
      {
        recipient: "0x7fab....", // Payee  address
        amount: {
          amountToken: "10",
          amountWei: "0" // Only token payments are facilitated
        },
        type: "PT_CHANNEL", // The payment type, see the client docs for more
      },
    ]
  })
  ```


**Withdrawing from a channel**

Right now, Connext supports withdrawals of funds from channel as Wei. The token funds that are in your channel are exchanged for Wei onchain as part of the withdrawal. The code to support token withdrawals is complete, but is currently disabled; if this presents a problem for your use case, please reach out to us.

```javascript
  await connext.withdraw({
    // Address to receive withdrawal funds
    // Does not need to have a channel with Connext to receive funds
    recipient: "0x8cef....",
    // USD price if using dai
    exchangeRate: "139.35",
    // Wei to transfer from the user's balance to 'recipient'
    withdrawalWeiUser: "500",
    // Tokens from channel balance to "sell" back to hub
    tokensToSell: "990",
  })

})()
```

Further documentation on the client can be found [here](../develop/client.md).

Check out the [Dai Card](https://daicard.io) live, and its [source](https://github.com/ConnextProject/card) for a more detailed implementation example.


