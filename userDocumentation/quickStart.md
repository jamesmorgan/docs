# Quick Start
Like [web3.js](https://web3js.readthedocs.io/), the Connext client is a collection of libraries that allow you to interact with a local or remote Connext node.

We will connect to the Connext Rinkeby node hosted at `https://indra-v2.connext.network/api/messaging` using the Connext client. If you don't have any Rinkeby ETH, we recommend you get some from a [faucet](https://faucet.rinkeby.io/) before continuing with this guide.


## Installation
Installing the client is simple. In your project root,
```npm install --save @connext/client;```


## Instantiating the Client


Import the client into your code:
```javascript
import * as connext from '@connext/client';
```

Set up your store (for now, if you're building a web app it's easiest to copy/paste the following code):
```javascript
const store = {
  get: (key) => {
    const raw = localStorage.getItem(`CF_NODE:${key}`)
    if (raw) {
      try {
        return JSON.parse(raw);
      } catch {
        return raw;
      }
    }
    // Handle partial matches so the following line works -.-
    // https://github.com/counterfactual/monorepo/blob/master/packages/node/src/store.ts#L54
    if (key.endsWith("channel") || key.endsWith("appInstanceIdToProposedAppInstance")) {
      const partialMatches = {}
      for (const k of Object.keys(localStorage)) {
        if (k.includes(`${key}/`)) {
          try {
            partialMatches[k.replace('CF_NODE:', '').replace(`${key}/`, '')] = JSON.parse(localStorage.getItem(k))
          } catch {
            partialMatches[k.replace('CF_NODE:', '').replace(`${key}/`, '')] = localStorage.getItem(k)
          }
        }
      }
      return partialMatches;
    }
    return raw;
  },
  set: (pairs, allowDelete) => {
    for (const pair of pairs) {
      localStorage.setItem(
        `CF_NODE:${pair.key}`,
        typeof pair.value === 'string' ? pair.value : JSON.stringify(pair.value),
      );
    }
  }
};

```



Once your store is set up, instantiate the client by passing in an object of type [ConnextOptions](../userDocumentation/types.md).

**For web applications:**
dApps can simple pass a mnemonic and node URL as client options. In Web3-enabled browsers, a `ChannelProvider` will be used in the default Client options.
```javascript
const options: ClientOptions = {
  mnemonic: 'Apple Banana ...',
  nodeUrl: 'wss://rinkeby.indra.connext.network/api/messaging',
  ethProviderUrl: `https://rinkeby.indra.connext.network/api/ethprovider`,
  store
}
```

Note: The node URL defines the transport layer and should be prefixed with `ws://` in most cases. If you're building a native implementation or another non-browser-based implementation, however, you likely need to use `nats://`.

**For wallets:**
Wallet integrations have more optionality in terms of client options, including the ability to pass in a provider; please see [Wallet Integrations](../userDocumentation/walletIntegrations) to read more.

**Setting Up a Channel**

Once you've set your parameters, call `connext.connect()` to establish a connection with your channel. If you're using React, it can be helpful to set up your channel and save the instance to state in `componentDidMount`.

For more information on monitoring you channel, please see [Event Monitoring](../userDocumentation/advanced#event-monitoring)
```javascript
  const channel: Channel = await connext.connect(options)
```


## Depositing
After instantiating and starting Connext, you can deposit into a channel with `channel.deposit`. Our hosted node accepts deposits in ETH and all ERC20 tokens. However, when depositing tokens, ensure the user has sufficient ETH remaining in their wallet to afford the gas of the deposit transaction.

```javascript
// Making a deposit in ETH
const payload: AssetAmount = { 
  amount: '0x3abc', // represented as bignumber
  assetId: AddressZero // Use the AddressZero constant from ethers.js to represent ETH, or enter the token address
}

channel.deposit(payload)
```

## Exchanging
For now, our testnet node is opinionated and only [collateralizes](../userDocumentation/limitations.md#Collateral) each channel with DAI by default. (E.e. only DAI transfers are allowed) If you need ETH transfers, [get in touch](https://discord.gg/raNmNb5) and we'll set you up with Eth-collateralized channels!

Make an in-channel swap with:
```javascript
  // Exchanging Wei for Dai
const payload: ExchangeParams = { 
  amount: "0x3abc" // in Wei, represented as bignumber
  toAssetId: "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359" // Dai
  fromAssetId: AddressZero // ETH
}

await channel.exchange(payload)
```

## Making a Transfer
Making a transfer is simple! Just call `channel.transfer()`. Recipient is identified by the counterparty's xPub identifier, which you can find with `channel.publicIdentifier`. 

```javascript
const payload: TransferParams = { 
  recipient: "xpub1abcdef"  //counterparty's xPub
  meta: "Metadata for transfer"
  amount: "0x3abc" // in Wei, represented as bignumber
  assetId: AddressZero // represents ETH
}

await channel.transfer(payload)
```

## Withdrawing
Users can withdraw funds to any recipient address with `channel.withdraw()`. Right now, the node only supports withdrawals in ETH to mitigate spam. When withdrawing, DAI that is in your channel is automatically exchanged for ETH as part of the withdrawal process. If you need DAI (or DAI + ETH) withdrawals, [get in touch](https://discord.gg/raNmNb5) and we can activate them for your channels.

```javascript

const payload: WithdrawParams = { 
  recipient: // defaults to signer xpub but can be changed to withdraw to any recipient
  amount: "0x3abc" // in Wei, represented as bignumber
  assetId: AddressZero
}

await channel.withdraw(payload)
```


## What's next?

If you've completed this quickstart guide and want to dive deeper into UX optimizations, continue to [Advanced Integrations](../userDocumentation/advanced).


## Additional Resources

Further documentation on the client (types, method reference, etc) can be found [here](../userDocumentation/clientAPI.md).

A live mainnet implementation can be found [here](../userDocumentation/daiCard.md).





