# Types

## Best Practices

### Conversion Functions

The client provides helpful conversion functions for converting numeric fields of [`ChannelState`](#convertchannelstate), [`ThreadState`](#convertthreadstate), and [`Args`](#convertargs) objects between [`BN`](https://github.com/indutny/bn.js/), [`BigNumber`](https://mikemcl.github.io/bignumber.js/), and `string` types.

Example:

```typescript
const channelState = connext.store.getState().persistent.channel
// as a string
console.log(convertChannelState('str', channelState))
// as a bn
console.log(convertChannelState('bn', channelState))
// as a bignumber
console.log(convertChannelState('bignumber', channelState))
```

### Using Currency Types

It is recommended that client implementers, especially those using Connext hosted hubs, use the `CurrencyConvertable` and `Currency` classes to easily format and convert between different currencies.

Example helper function:

```typescript
import CurrencyConvertable from "connext/dist/lib/currency/CurrencyConvertable";
import { CurrencyType } from "connext/dist/state/ConnextState/CurrencyTypes";
import getExchangeRates from "connext/dist/lib/getExchangeRates";
import Currency from "connext/dist/lib/currency/Currency";

export function getChannelBalanceInUSD(channelState, connext) {
  const connextState = connext.store.getState()
  const channelState = connextState.persistent.channel

  const convertableTokens = new CurrencyConvertable(CurrencyType.BEI, channelState.balanceTokenUser, () => getExchangeRates(connextState))

  const convertableWei = new CurrencyConvertable(CurrencyType.WEI, channelState.balanceWeiUser, () => getExchangeRates(connextState))

  const total = new CurrencyConvertable(
    CurrencyType.BEI,
    convertableTokens.toBEI().amountBigNumber.plus(convertableWei.toBEI().amountBigNumber),
    () => getExchangeRates(connextState)
  ).toUSD().amountBigNumber

  return  Currency.USD(total).format({})
}
```

See the detailed [`CurrencyConvertable`](#currencyconvertable) and [`Currency`](#currency) documentation sections.

## Index

Methods:

* [convertChannelState](#convertchannelstate)
* [convertThreadState](#convertthreadstate)
* [convertArgs](#convertargs)

Types:

* [ConnextClientOptions](#connextclientoptions)
* [PurchaseRequest](#purchaserequest)
* [PurchasePaymentRequest](#purchasepaymentrequest)
* [MetadataType](#metadatatype)
* [Payment](#payment)
* [WithdrawalParameters](#withdrawalparameters)
* [ChannelState](#channelstate)
* [UpdateRequest](#updaterequest)
* [ExchangeArgs](#exchangeargs)
* [PaymentArgs](#paymentargs)
* [DepositArgs](#depositargs)
* [WithdrawalArgs](#withdrawalargs)
* [ConfirmPendingArgs](#confirmpendingargs)
* [InvalidationArgs](#invalidationargs)
* [EmptyChannelArgs](#emptychannelargs)
* [ThreadState](#threadstate)
* [ThreadHistoryItem](#threadhistoryitem)
* [ExchangeRateState](#exchangeratestate)
* [SyncResult](#syncresult)
* [ChannelStatus](#channelstatus)
* [ChannelRow](#channelrow)
* [Sync](#sync)
* [ChannelStateUpdateRow](#channelstateupdaterow)
* [ThreadRow](#threadrow)
* [ThreadStateUpdateRow](#threadstateupdaterow)
* [CurrencyType](#currencytype)

Classes:

* [SyncControllerState](#synccontrollerstate)
* [CurrencyConvertable](#currencyconvertable)
* [Currency](#currency)

### ConnextClientOptions

Object including the options to instantiate the connext client with.

The object contains the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| web3 | Web3 | the Web3 object used by the client |
| hubUrl | String | url of the hub |
| user | Address | address of the signing wallet |
| contractAddress? | Address | (optional) address of the [ChannelManager](contracts.html) contract |
| hubAddress? | Address | (optional) address of the hub's signing wallet |
| tokenAddress? | Address | (optional) address of the channel's token |
| origin? | String | (optional) origin of the hub calls |
| ethNetworkId? | String | (optional) the ETH network id |
| tokenName? | String | (optional) name of the channel token |
| gasMultiple? | Number | (optional) the multiple of the `gasEstimate` used to calculate gas for any contract calls sent by the client. Defaults to 1.5 |

If the optional values are not provided, they will default to the ones that synchronize with the hub's configuration values.

Example:

```javascript
// *** Instantiate the connext client ***
const connext = getConnextClient({
  web3: new Web3("http://localhost:8545"),
  hubUrl: "http://localhost:8080",
  user: "http://localhost:8080",
})
```

___

### PurchaseRequest

A purchase is considered to be a set of grouped payments (i.e. a tip and a fee).

A `PurchaseRequest` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| meta | *[`MetadataType`](#metadatatype)* | Any metadata associated with the purchase request. Stored by the hub. |
| meta | *[`PurchaseRequest`](#purchasepaymentrequest)[]* | Any metadata associated with the purchase request. Stored by the hub. |

___

### PurchasePaymentRequest

A `PurchasePaymentRequest` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| purchaseId | String | A unique ID for this purchase, generated by the Hub |
| meta |  *[`MetadataType`](#metadatatype)*  | Metadata related to the purchase. |
| amount | *[`Payment`](#payment)* | A convenience field summarizing the total amount of this purchase |
| payments | *[`PurchasePayment`](#purchasepayment)[]* | The payments comprising the purchase |

___

### PurchasePayment

A `PurchasePayment` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | String | A unique ID for this purchase, generated by the Hub |
| amount |  *[`Payment`](#payment)*  | A convenience field summarizing the change in balance of the underlying channel or thread |
| meta | *[`MetadataType`](#metadatatype)* | Metadata related to the payment. For linked payments, the secret must be included in the metadata|
| type | String | Payment type. Options: 'PT_CHANNEL', 'PT_CUSTODIAL', 'PT_THREAD', 'PT_LINK'|

___

### MetadataType

A `MetadataType` object accepts any JSON.

___

### Payment

A `Payment` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | Address | Address of recipient |
| amountToken | String | Amount of token to send |
| amountWei | String | Amount of wei to send |

___

### WithdrawalParameters

A `WithdrawalParameters` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | Address | Address for withdrawal |
| exchangeRate | String | The exchange rate shown to the user at the time of the withdrawal |
| withdrawalWeiUser | String | Amount of wei to transfer from the user's balance to 'recipient' |
| tokensToSell | String | Amount of tokens to sell and transfer equivalent wei to 'recipient' |
| weiToSell? | String | Amount of wei to sell and transfer equivalent tokens to 'recipient' |
| withdrawalTokenUser | String | Amount of tokens to transfer from the user's balance to 'recipient' |

___

### ChannelState

A `ChannelState` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| contractAddress | Address | [`ChannelManager`](contracts.html) contract address |
| user | Address | Eth address of signing wallet of channel user |
| recipient | Address | Eth address of recipient of outside funds, defaults to user in all cases but `ProposePendingWithdrawal` states |
| balanceWeiHub | string | The eth balance of the hub in the channel in wei units |
| balanceWeiUser | string | The eth balance of the user in the channel in wei units  |
| balanceTokenHub | string | The token balance of the hub in the channel in wei units  |
| balanceTokenUser | string | The token balance of the user in the channel in wei units |
| pendingDepositWeiHub | string | Any pending hub eth deposits in wei units, set in `ProposePending*` type updates |
| pendingDepositWeiUser | string | Any pending user eth deposits in wei units, set in `ProposePending*` type updates |
| pendingDepositTokenHub | string | Any pending hub token deposits in wei units, set in `ProposePending*` type updates |
| pendingDepositTokenUser | string | Any pending user token deposits in wei units, set in `ProposePending*` type updates |
| pendingWithdrawalWeiHub | string | Any pending hub eth withdrawals in wei units, set in `ProposePendingWithdrawal` type updates |
| pendingWithdrawalWeiUser | string | Any pending user eth withdrawals in wei units, set in `ProposePendingWithdrawal` type updates |
| pendingWithdrawalTokenHub | string | Any pending hub token withdrawals in wei units, set in `ProposePendingWithdrawal` type updates |
| pendingWithdrawalTokenUser | string | Any pending user token withdrawals in wei units, set in `ProposePendingWithdrawal` type updates |
| txCountGlobal | number | The global nonce of the channel |
| txCountChain | number | The onchain nonce of the channel |
| threadRoot | string | The thread root of the channel; this is generated by taking the root of a merkle tree with leaves containing the hash of the initial thread state |
| threadCount | number | The number of open threads associated with the channel |
| timeout | number | The expiry of the given channel update |
| sigUser? | string | The users signature on the channel state |
| sigHub? | string | The hubs signature on the channel state |

___

### UpdateRequest

___

### ExchangeArgs

___

### PaymentArgs

___

### DepositArgs

___

### WithdrawalArgs

___

### ConfirmPendingArgs

___

### InvalidationArgs

___

### EmptyChannelArgs

___

### ThreadState

___

### ThreadHistoryItem

___

### ExchangeRateState

___

### SyncResult

___

### ChannelStatus

___

### SyncControllerState

___

### ChannelRow

___

### Sync

___

### ChannelStateUpdateRow

___

### ThreadRow

___

### ThreadStateUpdateRow

___

### convertChannelState

___

### convertThreadState

___

### convertArgs

___

### CurrencyConvertable

___

### Currency