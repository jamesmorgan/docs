# Types Reference


## ClientOptions

Object including the options to instantiate the connext client with.

The object contains the following fields:

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


If the optional values are not provided, they will default to the ones that synchronize with the hub's configuration values. However, you must pass at least one signing option (mnemonic, externalWallet, or channelProvider).

Example:

```javascript
// Instantiate the connext client
const options: ClientOptions = {
  mnemonic: 'Apple Banana ...',
  nodeUrl: 'nats://node.connext.network',
}
```

___


# Deposit

Deposit object, accepted by `connext.deposit`

| Name | Type | Description |
| ------ | ------ | ------ |
| amount | string | Amount to deposit (Wei)  |
| assetId | string | Address of asset (ETH = "0x0") |

```javascript
export type DepositParams = <T=string> = {
  amount: T
  assetId: string
}
```

___

# Transfer

Transfer object, accepted by `connext.transfer`

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | string | Address of recipient |
| meta? | string | (optional) Any transfer metadata |
| amount | string | Amount to deposit (Wei)  |
| assetId | string | Address of asset (ETH = "0x0") |

```javascript
export type TransferParams = <T=string> = {
  recipient: string
  meta?: string
  amount: T
  assetId: string
}
```

___

# Exchange

Exchange object, accepted by `connext.exchange`

| Name | Type | Description |
| ------ | ------ | ------ |
| amount | string | Amount to deposit (Wei)  |
| toAssetId | string | Address of asset you're converting to (ETH = "0x0") |
| fromAssetId | string | Address of asset you're converting from (ETH = "0x0") |


```javascript
export type ExchangeParams = <T=string> = {
  amount: T
  toAssetId: string
  fromAssetId: string
}
```

___

# Withdraw

Withdraw object, accepted by `connext.withdraw`

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | string | Address of recipient of withdrawal |
| amount | string | Amount to withdraw (Wei)  |
| assetId | string | Address of asset (ETH = "0x0") |


```javascript
export type WithdrawParams = <T=string> = {
  recipient?: string // defaults to signer
  amount: T
  assetId: string
}
```

___

# Install App (from CF API)
export type InstallAppParams = <T=string> = {
  recipient: string // counterparty to the application
  appId: string
  abiEncodings?: AppABIEncodings // optional if it's a predefined app within the client
  myDeposit: T
  myAssetId: string
  recipientDeposit: T
  recipientAssetId: string
  timeout: number
  initialState?: AppState // optional if it's a predefined app within the client
}

___

# Uninstall App (from CF API)
export type UninstallAppParams = {
  appInstanceId: string
}

___

# Take Action (from CF API)
export type TakeActionParams = {
  appInstanceId: string
  action: SolidityABIEncoderV2Type
