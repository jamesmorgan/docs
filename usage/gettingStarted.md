# Getting Started

Integrating Connext and the hosted Dai hub allows you to make instant, trust-minimized, off-chain payments to anyone else connected to the hub. This guide will walk you through how to get a hub up and running locally and some key aspects of integrating the client. If you are unfamiliar with payment channels, or ethereum basics, make sure you have read the [introduction](../background/introduction.md).

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

Alternatively, point metamask to a Rinkeby rpc url (eg use our hub's eth endpoint: `https://rinkeby.hub.connext.network/api/eth`) and get some Rinkeby eth from a [faucet](https://faucet.rinkeby.io/).

Once all the components are up and running, navigate to `http://localhost/` to checkout the sandbox. To implement the client on your own frontend, checkout the [implementing the client](#implementing-the-client) section.

### Additional Resources

For more detailed information about our hubs, check out the following:

- [API Docs](../develop/hub.md)
- Deploying locally [with docker](https://github.com/ConnextProject/indra#to-deploy-using-docker) and [without docker](https://github.com/ConnextProject/indra#to-deploy-locally)
- [Deploying to Production](https://github.com/ConnextProject/indra#deploying-to-production)
- [Interacting with a Hub](https://github.com/ConnextProject/indra#how-to-interact-with-an-indra-hub)

## Implementing the Client

### Configuration

First, install the client package:

```javascript
npm install connext
```

### Basic Example

The following is a basic example of how to use the client to make payments. As an implementer, there are some [core concepts](./coreConcepts.md) to be aware of that will impact user experience based on your client implementation.

To start using a channel, just deposit from the signing wallet into the ChannelManger contract and start making payments:

```javascript
import { getConnextClient } from "connext/dist/Connext.js";
const Web3 = require("web3");

hubUrl=http://localhost:3000/api/eth
ethUrl=http://localhost:3000/api/hub

// Or, use the following to connect to a live rinekby hub
hubUrl=https://rinkeby.hub.connext.network/api/hub
ethUrl=https://rinkeby.hub.connext.network/api/eth

// instantiate web3, get signer
const web3 = new Web3(new Web3.providers.HttpProvider(ethUrl))

(async function() {

  const accounts = await web3.eth.getAccounts()

  // set the client options
  const connextOptions = {
    web3,
    hubUrl,
    user: accounts[0],
    origin: 'localhost' // the host url of your app
  }

  // instantiate a new instance of the client
  const connext = await getConnextClient(connextOptions)

  // the connext client is an event emitter
  // start the app, and register a listener
  connext.on('onStateChange', connext => {
    console.log('Connext:', connext)
  }
  // start connext
  await connext.start()

  // Now that the client is started, you can make a deposit into the channel.
  // Channels can accept deposits in both ETH and tokens. However, when depositing tokens,
  // ensure the user has sufficient ETH remaining in their wallet to afford the gas
  // of the deposit transaction.

  // make a deposit in ETH
  await connext.deposit({
    amountWei: "1500",
    amountToken: "0", // assumed to be in wei units
  })

  // Congratulations! You have now opened a payment channel with the hub,
  // and you are ready to start updating your state. Connext facilitates
  // in channel exchanges, token payments, and channel withdrawals in wei.

  // exchange wei for dai
  await connext.exchange("1000", "wei");

  // make a dai payment
  // payments made can be retrieved using the returned purchase id,
  // which ties together all payment channel updates initiated in the array.
  const purchaseId = await connext.buy({
    meta: {

    },
    payments: [
      {
        recipient: "0x7fab....", // payee  address
        amount: {
          amountToken: "10",
          amountWei: "0" // only token payments are facilitated
        },
        type: "PT_CHANNEL", // the payment type, see the client docs for more
      },
    ]
  })

  // Note: The hub is responsible for collateralizing all payments
  // If the hub does not have funds to forward the payment in the
  // payment recipient's channel, the payment will fail until the
  // hub is able to deposit more tokens into the channel.
  // See the Collateral section to learn more

  // withdraw funds from channel as wei
  // the token funds that are in your channel are exchanged for
  // wei on chain as part of the withdrawal
  await connext.withdraw({
    // address to receive withdrawal funds
    // does not need to have a channel with connext to receive funds
    recipient: "0x8cef....",
    // USD price if using dai
    exchangeRate: "139.35",
    // wei to transfer from the user's balance to 'recipient'
    withdrawalWeiUser: "500",
    // tokens from channel balance to sell back to hub
    tokensToSell: "990",
  })

})()
```

Further documentation on the client can be found [here](../develop/client.md).

Check out the [Dai Card](https://daicard.io) live, and its [source](https://github.com/ConnextProject/card) for a more detailed implementation example.
