# Getting Started

Integrating Connext and the hosted Dai hub allows you to make instant, trust-minimized, off-chain payments to anyone else connected to the hub. This guide will walk you through how to get a hub up and running locally, some key aspects of integrating the client, and a couple of core concepts. If you are unfamiliar with payment channels, or ethereum basics, make sure you have read the [introduction](introduction.md)

## Setting Up

Before getting started integrating connext, make sure you have the following prerequisites installed:

- Node 9+ and NPM
- Docker
- Make
- jq
- PosgreSQL*
- Redis*
- Yarn*

*Used only if running without using docker.

### Using a Local Hub

When developing locally, you will also need to make sure you have a local hub deployed. Make sure you get the hub up and running, according to the instructions found in the [indra repository](https://github.com/ConnextProject/indra).

After your hub is running locally, install the client in your frontend:

```javascript
npm install connext
```

and configure your environment.

#### With Docker (recommended)

Run the following:

```bash
git clone https://github.com/ConnextProject/indra.git
cd indra
npm start
```

Beware! The first time this is run it will take a very long time (> 10 minutes usually) but have no fear: downloads will be cached & most build steps won't ever need to be repeated again so subsequent npm start runs will go much more quickly.

After your hub is running locally, install the client in your frontend:

```javascript
npm install connext
```

and add the following variables to your `.env`:

```bash
HUB_URL=http://localhost:3000/api/hub
RPC_URL=http://localhost:3000/api/eth
```

To easily send tokens or wei from Metamask to your local network, mport the following private key (hub's key):
```659CBB0E2411A44DB63778987B1E22153C086A95EB6B18BDF89DE078917ABC63```

Additionally, create a custom RPC with the url: `http://localhost:3000/api/eth`.

#### Without Docker

Make sure PostgreSQL, Redis, and Yarn are all properly installed. Start the postgres and redis services. Perform the following steps in order. For each section, use a separate terminal window with the working directory set to the `modules/hub` directory. Closing terminal will stop the process.

- **Ganache**

```bash
yarn install # Install dependencies.
bash development/ganache-reset # Migrates the contracts.
bash development/ganache-run # Runs Ganache (if you put a number after the ganache-run command you can set the blocktime).
```

- **Hub**

```bash
createdb sc-hub # Creates the hub's database (if it already exists, skip this step).
bash development/hub-reset # Resets the hub's database.
bash development/hub-run # Runs hub and chainsaw.
```

After your hub is running locally, install the client in your frontend:

```javascript
npm install connext
```

and add the following variables to your `.env`:

```bash
HUB_URL=http://localhost:8080
RPC_URL=http://localhost:8545
```

To easily send tokens or wei from Metamask to your local network, mport the following private key (hub's key):
```09CD8192C4AD4DD3B023A8EF381A24D29266EBD4AF88ECDAC92EC874E1C2FED8```

Your Metamask should use the traditional ganache RPC url: `http://localhost:8545`

### Connecting to Other Networks

If you are ready to test on rinkeby or on mainnet, update your frontend `.env` variables to use the hosted hubs:

```bash
# Rinkeby Hub
HUB_URL=https://daicard.io/api/rinkeby/hub
RPC_URL=https://eth-rinkeby.alchemyapi.io/jsonrpc/SU-VoQIQnzxwTrccH4tfjrQRTCrNiX6w
# Mainnet Hub
HUB_URL=https://daicard.io/api/mainnet/hub
RPC_URL=https://eth-mainnet.alchemyapi.io/jsonrpc/rHT6GXtmGtMxV66Bvv8aXLOUc6lp0m_-
```

Interested in hosting your own hub? Check out the [indra repository](https://github.com/ConnextProject/indra) for more information on running, deploying, and hosting your own hub.

## Integrating the Client

### Basic Example

To start using a channel, just deposit from the signing wallet into the ChannelManger contract and start making payments!

```javascript
// Import the client and environment
require('dotenv').config()
import { getConnextClient } from "connext/dist/Connext.js";
const Web3 = require("web3");

// instantiate web3, get signer
const web3 = new Web3(process.env.RPC_URL)
const accounts = await web3.eth.getAccounts()

// set the client options
const connextOptions = {
  web3,
  hubUrl: process.env.HUB_URL,
  user: accounts[0],
  origin: 'localhost' // the host url of your app
}

// instantiate a new instance of the client
const connext = await getConnext(connextOptions)

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
```

Further documentation on the client can be found [here](../modules/client). Check out the [Dai Card](https://daicard.io) live, and its [source](https://github.com/ConnextProject/card) for a more detailed example.

### Implementer Considerations

There are a few core concepts to be aware of as an implementer.

### Signing State Updates

The signing wallet is the ethereum address that is used to sign the state updates within the channel. The signing wallet can be any ethereum account you have access to (such as Metamask), however, it is important to understand how signing affects user experience.

When using a Metamask account as the signer, for instance, your users will have to explicitly approve any signature requested of them. The resultant signature pop-ups must be resolved before the channel state can be advanced, and often significantly detract from UX.

To avoid this, you can implement a custom provider that defaults to approving and signing messages without requiring explicit user input (i.e. a signing pop-up). An example autosigner implmentation can be found [here](https://github.com/ConnextProject/card/blob/master/src/utils/ProviderOptions.ts). Error checks against malformed state updates are performed before signing, and any application-specific errors should be checked before calling Connext functions to avoid an improper state update.

### Collateral

If you are using trust-minimized payments, you will only be able to make payments if the hub at least that amount in the payee's channel. For example, if Alice pays Bob 10 DAI through the hub, the hub must have at least 10 DAI in its channel with Bob to facilitate the payment. If Alice and Chris want to pay Bob 10 DAI each, as tips during videogame streaming for example, the hub must have at least 20 DAI in it's channel with Bob.

#### Custodial payments don't need collateral

Collateral requirements only apply if you are using non-custodial payments. To bypass these requirements completely, use trusted hub payments by inserting the payment type `PT_CUSTODIAL`. This allows the hub to forward along payments it receives, without having the collateral in the recipients channel.

#### Autocollateralization

The hub handles these requirements by using an autocollateralization mechanism that is triggered by any payment made, whether or not the payment was successful. Hubs determine amount of collateral needed in a channel based on the number and value of recent payments made to the recipient. Additionally, there are floors and ceilings implemented by hub operators to minimize the amount of collateral that is locked in hub channels, as well as set a minimum amount of collateral to be maintained in each channel.

Typically, hub balances below 10 DAI will trigger recollateralization. Hubs will put up to 170 DAI in any one channel. These values are configurable, so contact your hub operator for more details.

#### Implementer considerations

As an implementer, this can have several important consequences. First, you should expect payments to fail if there is not sufficient collateral, and retry them after monitoring the hubs collateral in the recipients channel via `connext.recipientNeedsCollateral`. Additionally, you should expect to see payments fail if they are uncharacteristically large, occur right after a withdrawal or dispute, or are the first payments in a long time.

This means if you do not send a failing payment to trigger collateraliztion, it will not account for that payment amount when recollateralizing. Deposits where the hub is recollateralizing a channel who's balance is below the floor should not be blocking updates, so long as the hub still has sufficient `balanceTokenHub` in the recipient's channel.

The hub can reclaim collateral by disputing the channel, or by the client submitting a withdrawal request with 0 value to allow the hub to withdraw excess collateral from the channel. By minimizing the hub collateral, you also reduce the amount of user channels the hub disputes (lowering user gas costs and wait times) and help payments pass more smoothly through the network.

### Availability

The wallet must acknowledge every state update by cosigning that state. This means in order to update your channel, the user must be online.

Again, availability requirements only apply if you are using non-custodial payments and using the payment type `PT_CUSTODIAL` will relax these, and rely on a trusted hub.

### Trust Assumptions

While the underlying protocol is completely noncustodial, there are trust assumptions to the Dai Card implementation which we want to make explicit. We plan to address these assumptions over the next few months.

#### Hub can intercept payments

For now, the payments themselves are trusted. In other words, a hub can steal your payment value while the transaction is in-flight. You would usually notice if this happened and leave the system with the rest of your funds, so the risk is limited to your payment itself.

The code for fully noncustodial payments is already in our codebase and is functional. We have left it deactivated for now to reduce complexity while we work on improving logic for channel collateralization. We plan on reactivating it within the next couple of weeks.

#### You don’t always have access to your state

Right now, if you go offline, the hub is the only entity that persists your channel’s state. From a usability perspective, this is great because it allows for easy cross-device support and keeps state in sync. Obviously, this is bad in the event that the hub goes down and you need to recover your funds from the contract directly using your latest state.

“Data availability” is a known problem for channels. To solve it, we need access to a highly reliable/available decentralized data store. Within the next 2–4 weeks, we plan to backup user state on IPFS as a temporary fix. We’re still researching a solution that would be effective long term and at scale.

#### Link payments and other async payments are trusted

Link payments and other async/offline payments are currently done by having the hub hold the payment until some condition for the payment can be proven to be resolved by the recipient.

This can be partly trust-minimized by allowing for conditional resolution of payments (i.e. “generalized state channels”) in our contract. The specification to do this has already been completed and will only require a change to <10 lines of the ChannelManager code (est. 4–6 weeks to ship). However, some service provider may need to be available to “receive” the state here, which we’re still researching how to resolve — everything in time, we’re working quickly.

#### Hub is centralized and can censor payments

Our hub is currently operated by us (Connext) and is centralized off-chain similar to how 0x relayers like Radar Relay work. This means that our hub could be censored, DDoS’d or shut down, which would mean that our payment service could go offline.

Hosting a hub is an interim solution to solve cheap, fast payments now while we work on building out the remaining pieces needed for a decentralized state channel network. We can fix our own centralization by allowing other Connext hubs (at that point, nodes) to be networked together, by making it easy for anyone to run a node, and by moving to sending state updates over some P2P messaging protocol rather than over https. These changes will require some quality of life improvements to our contract functions, however, so we expect to only start work on them after our next contract iteration is deployed.