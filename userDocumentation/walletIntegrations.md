# Wallet Integrations

The Connext client needs to run in a trusted execution environment because it contains functionality to automatically sign on behalf of an end user. Decentralized applications then need to be able to make hooked calls to a user’s client similarly to how existing hooked Ethereum RPC calls occur.

In other words, the Connext client behaves similar to web3: it can be instantiated both within the wallet by passing in signer functionality or instantiated within decentralized applications using a ChannelProvider.

The object used to instantiate the Connext client contains the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| rpcProviderUrl | String | the Web3 provider URL used by the client |
| nodeUrl | String | url of the node |
| mnemonic? | String | (optional) Mnemonic of the signing wallet |
| externalWallet? | any | (optional) External wallet address |
| channelProvider? | ChannelProvider | (optional) Injected ChannelProvider |
| keyGen? | () => Promise<string[]> | Function passed in by wallets to generate ephemeral keys|
| store? | object | Maps to set/get from CF. Defaults localStorage |
| logLevel? | number | Depth of logging |
| natsUrl? | String | Initially hardcoded |
| natsClusterId? | String | Initially hardcoded |
| natsToken? | String | Initially hardcoded |

The `keyGen` field allows you to pass in a function to generate ephemeral signing keys.

If the optional values are not provided, they will default to the ones that synchronize with the hub's configuration values. However, you must pass at least one signing option (mnemonic, externalWallet, or channelProvider). 

