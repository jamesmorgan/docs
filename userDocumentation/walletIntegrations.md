# Wallet Integrations

The Connext client needs to run in a trusted execution environment because it contains functionality to automatically sign on behalf of an end user. Decentralized applications then need to be able to make hooked calls to a user’s client similarly to how existing hooked Ethereum RPC calls occur.

In other words, the Connext client behaves similar to web3: it can be instantiated both within the wallet by passing in signer functionality or instantiated within decentralized applications using a ChannelProvider.

