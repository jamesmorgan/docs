# Getting Started

Integrating Connext and the hosted Dai hub allows you to make instant, trust-minimized, off-chain payments to anyone else connected to the hub. This guide will walk you through how to get a hub up and running locally and some key aspects of integrating the client. If you are unfamiliar with payment channels, or ethereum basics, make sure you have read the [introduction](../background/introduction.md).

The easiest way to get started is to implement the Connext client using our hosted Rinkeby hub. Point Metamask to a Rinkeby RPC url (e.g. use our hub's eth endpoint: `https://rinkeby.hub.connext.network/api/eth`), get some Rinkeby ETH from a [faucet](https://faucet.rinkeby.io/), and follow the guide below.

## Implementing the Client

### Configuration

First, install the client package:

```javascript
npm install connext
```

### Basic Example

**Warning: Be sure to use the Connext client @ 3.0.19**

The following is a basic example of how to use the client to make payments. As an implementer, there are some [core concepts](./coreConcepts.md) to be aware of that will impact user experience based on your client implementation.

To start using a channel, just deposit from the signing wallet into the ChannelManger contract and start making payments:


**Instantiate and Start the Client**
 
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

// The Connext client is an event emitter
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

Note: The hub is responsible for collateralizing all payments. If the hub does not have funds to forward the payment in the payment recipient's channel, the payment will fail until the hub is able to deposit more tokens into the channel. To add collateral to the hub, simply send tokens (Dai if you're using our hosted hub) to the contract address. See the Collateral section to learn more.

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


## Running a Local Hub

To sample the hub in action on rinkeby or mainnet check out the [Dai Card](https://daicard.io). To get started locally, make sure you have the following prerequisites installed:

- [Node 9+ and NPM](https://nodejs.org/en/)
- [Docker](https://www.docker.com/)
- Make: (probably already installed) Install w ```brew install make``` or ```apt install make``` or similar.
- jq: (probably not installed yet) Install w ```brew install jq``` or ```apt install jq``` or similar.

Then, open a terminal window and run the following:

```bash
git clone https://github.com/ConnextProject/indra.git
cd indra
npm start
```

You can also skip this step for now if you want to connect to our hosted rinkeby hub instead of running your own locally.

Starting Indra will take a while the first time, but the build gets cached so it will only take this lone the first time. While it's building, configure your metamask to use the rpc `localhost:3000/api/eth` and import the hub's private key (`659CBB0E2411A44DB63778987B1E22153C086A95EB6B18BDF89DE078917ABC63`) so you can easily send money to the signing wallet.

Once all the components are up and running, navigate to `http://localhost/` to checkout the sandbox. To implement the client on your own frontend, checkout the [implementing the client](#implementing-the-client) section above.

### Additional Resources

For more detailed information about our hubs, check out the following:

- [API Docs](../develop/hub.md)
- Deploying locally [with docker](https://github.com/ConnextProject/indra#to-deploy-using-docker) and [without docker](https://github.com/ConnextProject/indra#to-deploy-locally)
- [Deploying to Production](https://github.com/ConnextProject/indra#deploying-to-production)
- [Interacting with a Hub](https://github.com/ConnextProject/indra#how-to-interact-with-an-indra-hub)