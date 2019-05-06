# Quick Start
Like [web3.js](https://web3js.readthedocs.io/), the Connext client is a collection of libraries that allow you to interact with a local or remote Connext hub over HTTP.

We will connect to the Connext Rinkeby hub hosted at `https://rinkeby.hub.connext.network/api/hub` using the Connext client. If you don't have Rinkeby ETH, we recommend you get some from a [faucet](https://faucet.rinkeby.io/) before continuing with this guide.

## Installation
Installing the client is simple. In your project root,
```npm install --save connext@latest```

**Warning: Be sure to use only Connext client v3.1.8 and above. Other versions have been deprecated and may be unstable**

## Instantiating the Client
Import the client into your file:
```javascript
import Connext from `connext`;
```

The client is instantiated by passing in an object of type [ConnextOptions](../develop/types.md#connextclientoptions). There are three ways to set up options:

**For web applications:**
An Ethereum compatible browser will have a `window.connext` or `context.channelProvider` available. In this case,
```javascript
  const connextOptions: ConnextOptions = {
    connextProvider: window.connext,
    web3: window.web3
  }
```
Passing in a hubUrl here is *not needed* because it is contained within the connextProvider's context.

**For wallets:**
The client accepts a hooked signer function. This function is used exclusively for signing *safe* messages (i.e. messages associated with receiving payments or confirming onchain transactions which can safely be signed without prompting the user). We strongly recommend that wallets pass in a custom signer here which does not prompt the user for signatures here.

For more information on integrating into wallets, see [Wallet Integrations] -- (coming soon)
```javascript
  //rinkeby: hubUrl=https://rinkeby.hub.connext.network/api/hub
  //mainnet: hubUrl=https://hub.connext.network/api/hub

  const connextOptions: ConnextOptions = {
    hubUrl,
    web3,
    safeSignHook
  }
```

**For burner-style applications:** 
The client can also be instantiated by directly passing in a private key or mnemonic. The storing the key safely is up to the implementer's discretion.
```javascript
  //rinkeby: hubUrl=https://rinkeby.hub.connext.network/api/hub
  //mainnet: hubUrl=https://hub.connext.network/api/hub

  const connextOptions: ConnextOptions = {
    hubUrl,
    privateKey, //OR mnemonic
  }
```

**After setting up your options:**
```javascript
  const client = await Connext.create(connextOptions)
  client.start() //starts polling for updates
```

## Listening for Events
You can listen for updates to the channel using `connext.on`.
```javascript
  // register listener
  // the only event the client emits is `onStateChange`, which is triggered
  // each time the connexts internal store is updated.

  connext.on('onStateChange', state => {
    console.log('Connext state changed:', state);
  })
  await connext.start() // start polling
```

## Depositing
After instantiation, you can deposit into a channel with `connext.deposit`. Our hosted Hub accepts deposits in both ETH and DAI. However, when depositing tokens, ensure the user has sufficient ETH remaining in their wallet to afford the gas of the deposit transaction.
```javascript
  // Making a deposit in ETH
  await connext.deposit({
    amountWei: "1500",
    amountToken: "0", // assumed to be in wei units
  })
```

## Exchanging
For now, our hosted hub is opinionated and only [collateralizes](../usage/limitations.md#Collateral) each channel with DAI by default. (I.e. only DAI payments are allowed) If you need ETH payments, [get in touch](https://discord.gg/raNmNb5) and we'll set you up with Eth-collateralized channels!

Make an in-channel swap with,
```javascript
  // Exchanging Wei for Dai
  await connext.exchange("1000", "wei");
```

## Making a Payment
Making a payment is simple! Just call `connext.buy()`
```javascript
  await connext.buy({ 
    recipient: "0x8cef....",
    amount: {
      amountToken: "100"
    }
  })
```

## Withdrawing
Users can withdraw funds to any recipient address with `connext.withdraw()`. Right now, the Hub only supports withdrawals in ETH to mitigate spam. When withdrawing, DAI that is in your channel is automatically exchanged for ETH as part of the withdrawal process. If you need DAI (or DAI + ETH) withdrawals, [get in touch](https://discord.gg/raNmNb5) and we can activate them for your channels.
```javascript
  await connext.withdraw({
    // Address to receive withdrawal funds
    // Does not need to have a channel with Connext to receive funds
    recipient: "0x8cef....",
    // Amount to withdraw, note:
    // 1. Dai withdrawals are not enabled by default
    // 2. Withdrawing more Eth than is available will automatically exchange and withdraw Dai instead
    amount: {
      amountWei: "15000"
    }
  })
```

## Additional Resources

Further documentation on the client (types, method reference, etc) can be found [here](../develop/client.md).

As an implementer, there are some [system limitations](./limitations.md) to be aware of that may impact user trust minimization.

A live mainnet implementation can be found [here](../usage/daiCard.md).

If you'd like to run your own hub (useful for developers who (i) want to allow users transact in a native token rather than Dai or (ii) want to customize [Hub options](../develop/hub.md)), please see [Running your own hub](../advanced/runHub.md).




