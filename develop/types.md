# Types Reference

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
* [Args](#args)
* [ExchangeArgs](#exchangeargs)
* [PaymentArgs](#paymentargs)
* [DepositArgs](#depositargs)
* [WithdrawalArgs](#withdrawalargs)
* [ConfirmPendingArgs](#confirmpendingargs)
* [InvalidationArgs](#invalidationargs)
* [InvalidationReason](#invalidationreason)
* [EmptyChannelArgs](#emptychannelargs)
* [ThreadState](#threadstate)
* [ThreadHistoryItem](#threadhistoryitem)
* [ExchangeRateState](#exchangeratestate)
* [ExchangeRates](#exchangerates)
* [SyncResult](#syncresult)
* [ChannelStatus](#channelstatus)
* [ChannelRow](#channelrow)
* [Sync](#sync)
* [ChannelStateUpdate](#channelstateupdate)
* [ChannelUpdateReason](#channelupdatereason)
* [ChannelStateUpdateRow](#channelstateupdaterow)
* [ThreadRow](#threadrow)
* [ThreadStateUpdate](#threadstateupdate)
* [ThreadStateUpdateRow](#threadstateupdaterow)
* [CurrencyType](#currencytype)
* [CurrencyFormatOptions](#currencyformatoptions)

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
// Instantiate the connext client
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
| meta | [`MetadataType`](#metadatatype) | Any metadata associated with the purchase request. Stored by the hub. |
| payments | [`PurchaseRequest[]`](#purchasepaymentrequest) | The payments associated with a given purchase |

___

### PurchasePaymentRequest

A `PurchasePaymentRequest` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| purchaseId | String | A unique ID for this purchase, generated by the Hub |
| meta |  [`MetadataType`](#metadatatype)  | Metadata related to the purchase. |
| amount | [`Payment`](#payment)* | A convenience field summarizing the total amount of this purchase |
| payments | [`PurchasePayment[]`](#purchasepayment) | The payments comprising the purchase |

___

### PurchasePayment

A `PurchasePayment` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | String | A unique ID for this purchase, generated by the Hub |
| amount |  [`Payment`](#payment) | A convenience field summarizing the change in balance of the underlying channel or thread |
| meta | [`MetadataType`](#metadatatype) | Metadata related to the payment. For linked payments, the secret must be included in the metadata|
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

An `UpdateRequest` is a JSON object containing information to update a channel state. The object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| id? | number | The id number of the channel update. Undefined if the hub has not seen this `UpdateRequest` |
| reason | [ChannelUpdateReason](#channelupdatereason) | The type of state update you are requesting to create |
| args | [Args](#args) | The [Args](#args) to generate the next state from the latest double signed state |
| txCount? | number | `txCountGlobal` of the channel. Will be null if the update is unsigned |
| sigUser? | string | User's signature of the update, if available |
| sigHub? | string | Hub's signature of the update, if available |
| createdOn? | Date | Date the update was added to the database, or `undefined` if the hub has not seen the update request |
| initialThreadStates? | [ThreadState[]](#threadstate) | An array of open initial thread states ysed to generate the next update (if needed) |

___

### Args

An `Args` object is a JSON object representing the args used in a given state transition. There are 8 types of argument objects: [`ExchangeArgs`](#exchangeargs), [`PaymentArgs`](#paymentargs), [`DepositArgs`](#depositargs), [`WithdrawalArgs`](#withdrawalargs), [`ConfirmPendingArgs`](#confirmpendingargs), [`InvalidationArgs`](#invalidationargs), [`EmptyChannelArgs`](#emptychannelargs), and [`ThreadState`](#threadstate)
___

### ExchangeArgs

`ExchangeArgs` is a JSON object used to create an `Exchange` type state update. with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| exchangeRate | string | ERC20 / ETH exchange rate |
| seller | `"user"` \| `"hub"` | Exchange initiator |
| tokensToSell | string | Amount of tokens to exchange with channel counterparty in wei units |
| weiToSell | string | Amount of wei to exchange with channel counterparty in wei units |

___

### PaymentArgs

`PaymentArgs` is a JSON object used to create an `Payment` type state update. with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | `"user"` \| `"hub"` | Payment recipient |
| amountWei | string | Amount of wei to pay recipient in wei units |
| amountToken | string | Amount of tokens to pay recipient in wei units |

___

### DepositArgs

`DepositArgs` is a JSON object used to create a `ProposePendingDeposit` type state update. with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| depositWeiHub | string | Amount of wei hub wants to deposit in wei units |
| depositWeiUser | string | Amount of wei user wants to deposit in wei units |
| depositTokenHub | string | Amount of tokens hub wants to deposit in wei units |
| depositTokenUser | string | Amount of tokens user wants to deposit in wei units |
| timeout | number | The time the `ProposePending` update will be invalidated |
| sigUser? | string | The user's signature on their deposit parameters. Not required for hub deposits |
| reason? | any | Metadata decribing why this deposit was made for hub to track |

___

### WithdrawalArgs

`WithdrawalArgs` is a JSON object used to create a `ProposePendingWithdrawal` type state update. with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| seller | `'user'` \| `'hub'` | Who is initiating onchain exchange |
| exchangeRate | string | ERC20 / ETH exchange rate |
| tokensToSell | string | Amount of tokens sold for exchange in wei units |
| weiToSell | string | Amount of wei sold for exchange in wei units |
| recipient | number | The address which should receive the transfer of user funds |
| tagetWeiUser? | string | The final `balanceWeiUser` in channel after the withdrawal. If not supplied, defaults to `balanceWei - weiToSell` |
| targetTokenUser? | any | The final `balanceTokenUser` in channel after the withdrawal. If not supplied, defaults to `balanceWei - tokensToSell` |
| targetWeiHub? | string | The final `balanceWeiHub` in channel after the withdrawal. If not supplied, defaults the previous state's `balanceWeiHub` |
| targetTokenHub? | any | The final `balanceTokenHub` in channel after the withdrawal. If not supplied, defaults the previous state's `balanceTokenHub` |
| additionalWeiHubToUser? | string | Any out of channel wei balance for hub to send to user as part of withdrawal from contract reserves |
| additionalTokenHubToUser? | any | Any out of channel token balance for hub to send to user as part of withdrawal from contract reserves |
| timeout | number | The time the `ProposePending` update will be invalidated |

It is important to note that this type is used internally by the hub and client, and should not be used by client implementers. Instead, use the `connext.withdraw()` method, which accepts [`WithdrawalParameters`](#withdrawalparameters)

___

### ConfirmPendingArgs

`ConfirmPendingArgs` is a JSON object used to verify a proposed onchain transaction occurred, and has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| transactionHash | string | The transaction hash that corresponds to the previous `ProposePending*` type update |

___

### InvalidationArgs

`InvalidationArgs` is a JSON object used to invalidate any pending updates if the onchain transaction failed or if the update has timed out. Invalidating updates will unwind any updates between the `previousValidTxCount` and the `lastInvalidTxCount`.

 Has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| previousValidTxCount | number | the previously valid transaction count |
| lastInvalidTxCount | number | the last invalid transaction count |
| reason | [`InvalidationReason`](#invalidationreason) |  |
| message? | string | An optional descriptive string for why the update is being invalidated |

___

### InvalidationReason

An enum containing different reasons a state update may be invalidated. The options include:

| Name | Description |
| ------ | ------ |
| `CU_INVALID_TIMEOUT` | The state has timed out |
| `CU_INVALID_REJECTED` | The state is being rejected |
| `CU_INVALID_ERROR` | The state is being invalidated due to some other error |

___

### EmptyChannelArgs

`EmptyChannelArgs` is a JSON object used to reset a channel to open after it has been disputed onchain, and has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| transactionHash | string | The transaction hash that corresponds to the `EmptyChannel` event |

___

### ThreadState

`ThreadState` is a JSON object used to reset a channel to open after it has been disputed onchain, and has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| contractAddress | Address | [`ChannelManager`](contracts.html) address of channel |
| sender | Address | Payor's eth address |
| receiver | Address | Payee's eth address |
| threadId | number | Global threadId between sender and receiver |
| balanceWeiSender | string | Eth balance of sender in thread in wei units |
| balanceWeiReceiver | string | Eth balance of receiver in thread in wei units |
| balanceTokenSender | string | Token balance of sender in thread in wei units |
| balanceTokenReceiver | string | Token balance of receiver in thread in wei units |
| txCount | number | The thread nonce |
___

### ThreadHistoryItem

`ThreadHistoryItem` is a JSON object used to track the appropriate threadID by the clients [`PersistentState`](client.html#persistentstate), and has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| sender | Address | Eth address of payor |
| receiver | Address | Eth address of receiver |
| threadId | number | Latest thread id for the sender/receiver combo |

___

### ExchangeRateState

An `ExchangeRateState` is an object that displays the [`ExchangeRates`](#exchangerates) for various [`CurrencyTypes`](#currencytype). Fields

| Key or Value | Type | Description |
| ------ | ------ | ------ |
| lastUpdated | Date | When exchange rates were last updated |
| rates | [`ExchangeRates`](#exchangerates) | Key : USD exchange rate |

___

### ExchangeRates

An `ExchangeRates` is an object that displays the currency to USD exchange rate as a `string` or `BigNumber` for each [`CurrencyType`](#currencytype). Structure:

| Key or Value | Type | Description |
| ------ | ------ | ------ |
| key | a [`CurrencyType`](#currencytype) | Type of currency |
| value | string \| BigNumber | Key : USD exchange rate |

___

### SyncResult

An `SyncResult` is a JSON object returned by the hub containing information about a proposed thread or channel state update. It has the following form:

| Name | Type | Description |
| ------ | ------ | ------ |
| type | `"thread"` or `"channel"` | The type of state update, whether it is for a thread or for a channel |
| update | [`ThreadStateUpdate`](#threadstateupdate) or [`UpdateRequest`](#updaterequest) | Update information. It is an `UpdateRequest` type for a `channel` state updae, and `ThreadStateUpdate` for a `thread` state change |

___

### ChannelStatus

An enum containing various channel statuses. There are the following `ChannelStatus` options

| Name | Description |
| ------ | ------ |
| `CS_OPEN` | Channel is open and able to accept offchain state updates |
| `CS_CHANNEL_DISPUTE` | Channel is being disputed onchain |
| `CS_THREAD_DISPUTE` | One or more of the threads with your channel is being disputed onchain |
| `CS_CHAINSAW_ERROR` | Chainsaw, the chain event listener component of the hub, cannot process a channel event |

___

### SyncControllerState

The `SyncControllerState` is an internal client class, used to store updates that need to be sent back to the hub so the client and hub channel states are in sync. The class does not have any class methods, and has the following properties:

| Name | Type | Description |
| ------ | ------ | ------ |
| updatesToSync | [`SyncResult[]`](#syncresult) | Updates received from the hub, to sync with client's local state |

___

### ChannelRow

The `SyncControllerState` is an internal client class, used to store updates that need to be sent back to the hub so the client and hub channel states are in sync. The class does not have any class methods, and has the following properties:

| Name | Type | Description |
| ------ | ------ | ------ |
| updatesToSync | [`SyncResult[]`](#syncresult) | Updates to return to the hub from the client |

___

### Sync

The hub returns this type from most endpoints. It is an object containing information about the channel status, as well as any updates the client has requested, or needs to sync:

| Name | Type | Description |
| ------ | ------ | ------ |
| status| [`ChannelStatus`](#channelstatus) | The channel status |
| updates| [`SyncResult[]`](#syncresult) | Updates received from the hub, to sync with client's local state |

___

### ChannelStateUpdate

A `ChannelStateUpdate` object contains all information regarding a proposed or accepted state update with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| id? | `number` | The database id of the update |
| reason | [`ChannelUpdateReason`](#channelupdatereason) | The reason for the update |
| state | [`ChannelState`](#channelstate) | The channel state after applying args |
| args | [`Args`](#args) | args for state update  |
| metadata | `Object` | any associated metadata |

___

### ChannelUpdateReason

The reason for this state update. This is an enum type with the following values:

| Name | Description |
| ------ | ------ |
| `Payment` | Payment state update |
| `Exchange` | Exchange state update |
| `ProposePendingDeposit` | Deposit state update |
| `ProposePendingWithdrawal` | Withdrawal state update |
| `ConfirmPending` | Confirm an onchain transaction or pending operations on state |
| `Invalidation` | Invalidate pending operations |
| `OpenThread` | Open a new thread in channel |
| `CloseThread` | Close existing thread in channel |
| `EmptyChannel` | Return to operating channel from dispute |

___

### ChannelStateUpdateRow

A `ChannelStateUpdateRow` is used by the hub to store information about channel state updates (eg. args, sigs, etc) in its database. In addition to extending the [`ChannelStateUpdate`](#channelstateupdate) type, it has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| id | `number` | The channel update id in the hub's database |
| createdOn| `Date` | Timestamp of when entry inserted into db |
| channelId?| `number` | Associated channel id |
| chainsawId?| `number` | Associated chain event id (if possible) |
| invalid? | [`InvalidationReason`](#invalidationreason) | If the update has been invalidated, the reason why |
| onchainTxLogicalId? | `number` | The id of the entry in the onchain transactions raw table in the hub's database |

___

### ThreadRow

A `ThreadRow` is used by the hub to store information about threads in its database. It is an object with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| id | `number` | The thread id in the hub's database |
| state| [`ThreadState`](#threadstate) | Current thread state |

___

### ThreadStateUpdateRow

A `ThreadStateUpdateRow` is used by the hub to store information about thread updates in its database. It is an object with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| id | `number` | The thread id in the hub's database |
| createdOn | `Date` | Timestamp of update being added to hub's database |
| state| [`ThreadState`](#threadstate) | Current thread state |

___

### convertChannelState

Converts the numeric fields of a [`ChannelState`](#channelstate) object between `string`, [`BigNumber`](https://mikemcl.github.io/bignumber.js/), and [`BN`](https://github.com/indutny/bn.js/) types based on the input.

| Parameter Name | Type | Description |
| ------ | ------ | ------ |
| conversionType | `"bn"`, `"str"`, `"bignumber"` | The type to convert numeric fields to |
| obj | [`ChannelState`](#channelstate) | Object to convert |

Example usage:

```typescript
// this function has the call signature:
// convertChannelState(toType, channelState)

// imagine threadState is a previously created thread obj
const channelStateBN = convertChannelState("bn", channelState)
const channelStateBigNum = convertChannelState("bignumber", channelState)
const channelStateStr = convertChannelState("str", channelStateBN)
```

___

### convertThreadState

Converts the numeric fields of a [`ThreadState`](#threadstate) object between `string`, [`BigNumber`](https://mikemcl.github.io/bignumber.js/), and [`BN`](https://github.com/indutny/bn.js/) types based on the input.

| Parameter Name | Type | Description |
| ------ | ------ | ------ |
| conversionType | `"bn"`, `"str"`, `"bignumber"` | The type to convert numeric fields to |
| obj | [`ThreadState`](#threadstate) | Object to convert |

Example usage:

```typescript
// this function has the call signature:
// convertThreadState(toType, threadState)

// imagine threadState is a previously created thread obj
const threadStateBN = convertThreadState("bn", threadState)
const threadStateBigNum = convertThreadState("bignumber", threadState)
const threadStateStr = convertThreadState("str", threadStateBN)
```

___

### convertArgs

Converts the numeric fields of any [`Args`](#args) object between `string`, [`BigNumber`](https://mikemcl.github.io/bignumber.js/), and [`BN`](https://github.com/indutny/bn.js/) types based on the input.

| Parameter Name | Type | Description |
| ------ | ------ | ------ |
| to | `"bn"`, `"str"`, `"bignumber"` | The type to convert numeric fields to |
| reason | `"Payment"`, `"Exchange"`, `"ProposePendingDeposit"`, `"ProposePendingWithdrawal"`, `"ConfirmPending"`, `"Invalidation"`, `"EmptyChannel"`, `"OpenThread"`, `"CloseThread"` | The specific type of args |
| args | [`Args`](#args) | Object to convert |

Example usage:

```typescript
// this function has the call signature:
// convertArgs(to, reason, args)

// eg. convert payment args
const pay: PaymentArgs = {
  amountToken: ,
  amountWei: ,
  recipient: "hub",
}
const payBN = convertArgs("bn", "Payment", pay)
const payBigNum = convertArgs("bignumber", "Payment", pay)
const payStr = convertArgs("str", "Payment", pay)

// eg. convert exchange args
const exchange: ExchangeArgs = {
  exchangeRate: "5.0",
  seller: "hub",
  tokensToSell: "0",
  weiToSell: "10",
}
const exchangeBN = convertArgs("bn", "Exchange", exchange)
const exchangeBigNum = convertArgs("bignumber", "Exchange", exchange)
const exchangeStr = convertArgs("str", "Exchange", exchangeBN)
```

___

### CurrencyConvertable

The `CurrencyConvertable` is a class used to convert between various currencies and denominations easily. Client implementers can use this class to easily convert between and display appropriate values.

In addition, class instances have the following class methods for conversion:

```typescript
to(toType: CurrencyType): CurrencyConvertable
toUSD(): CurrencyConvertable
toETH(): CurrencyConvertable
toWEI(): CurrencyConvertable
toFIN(): CurrencyConvertable
toBOOTY(): CurrencyConvertable
toBEI(): CurrencyConvertable
```

___

### Currency

The `Currency` class is used to easily handle and display the currencies within a channel. The currencies supported by this class are defined by the [`CurrencyType`](#currencytype) enum.

Constructor:

```typescript
constructor(type: CurrencyType, amount: BN | BigNumber | string | number): Currency
```

`static` methods:

```typescript
// Returns a new Currency object with the indicated type and given
Currency.ETH(amount: BN | BigNumber | string | number): Currency
Currency.USD(amount: BN | BigNumber | string | number): Currency
Currency.WEI(amount: BN | BigNumber | string | number): Currency
Currency.FIN(amount: BN | BigNumber | string | number): Currency
Currency.BOOTY(amount: BN | BigNumber | string | number): Currency
Currency.BEI(amount: BN | BigNumber | string | number): Currency

// returns true if the currencies have the same type and amount
Currency.equals(c1: Currency, c2: Currency): boolean
```

Getters:

```typescript
type(): CurrencyType
symbol(): string
currency(): { type: CurrencyType, amount: BigNumber }
amount(): string
amountBigNumber(): BigNumber
amountBN(): BN
```

Methods:

```typescript
format(options?: CurrencyFormatOptions): string
floor(): Currency
toFIN(): CurrencyConvertable
toBOOTY(): CurrencyConvertable
toBEI(): CurrencyConvertable
```

Example usage:

```typescript
const curr1 = new Currency(CurrencyType.USD, 105.70362)
const curr2 = Currency.USD(105.70362)

console.log(Currency.equals(curr1, curr2)) // true
console.log(curr.type()) // USD
console.log(curr.symbol()) // $
console.log(curr.currency()) // { type: 'USD', amount: BigNumber(105.70362) }

// set the format options
let opts: CurrencyFormatOptions = {
  decimals: 2,
  withSymbol: true,
  showTrailingZeros: false,
}
console.log(curr1.format(opts)) // $105.70
opts = {
  decimals: 0,
  withSymbol: false,
  showTrailingZeros: false,
}
console.log(curr1.format(opts)) // 105
```

See [`CurrencyType`](#currencytype) for the different currencies supported.

___

### CurrencyFormatOptions

`CurrencyFormatOptions` is an object used to define the how a currency is displayed as a string, with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| decimals? | `number` | Number of decimals to display |
| withSymbol? | `boolean` | Display currency with it's default symbol, which is a `$` for USD, and the first three characters of the token or denomination name otherwise (eg. FIN for finney) |
| showTrailingZeros| `boolean` | Display any trailing 0s |

___

### CurrencyType

An enum containing various currencies to convert between in [`CurrencyConvertable`](#currencyconvertables). Currency types include:

| Name | Description |
| ------ | ------ |
| `USD` | USD |
| `ETH` | Eth |
| `WEI` | Eth in wei units |
| `FINNEY` | Eth in fin units |
| `BOOTY` | BOOTY currency (should be same as USD) |
| `BEI` | BOOTY in wei units |