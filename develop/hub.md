# Hub

## Configuration

## API Documentation

Open Endpoints

Channel Endpoints*:

* [Request Deposit](#request-deposit)
* [Request Collateralization](#request-collateralization)
* [Request Exchange](#request-exchange)
* [Request Withdrawal](#request-withdrawal)
* [Update](#update)
* [Sync](#sync)
* [Get Channel](#get-channel)
* [Get Latest Update](#latest-update)
* [Get Latest State No Pending Operations](#latest-state-no-pending-operations)

Thread Endpoints*:

* TODO

*Requires proper authorization

### Request Deposit

Creates a `ProposePendingDeposit` update with the user as the sender of the onchain transaction. Any time a user deposits, the hub may use the opportunity to preemptively collateralize the channel for following state updates, such as in-channel swaps.

* **URL:** *hub-url*/channel/:user/request-deposit

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |
| depositWei | `string` | Amount of eth the user wants to deposit in wei |
| depositToken | `string` | Amount of tokens the user wants to deposit in wei |
| lastChanTx | `number` | The last global nonce of the channel |
| lastThreadUpdateId | `number` | The latest thread update id |
| sigUser | `string` | User's signature of their deposit values |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ChannelRow`](types.html#channelrow) |
| 400 | BAD REQUEST | `{ error : "Received invalid user deposit request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/request-deposit",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    depositWei: "100",
    depositToken: "100",
    sigUser: "0xde4ge23ff...",
    lastChanTx: 3,
    lastThreadUpdateId: 0,
  },
  success : function(r) {
    console.log(r);
  }
});
```

___

### Request Collateralization

Requests collateralization into your channel to handle expected payments.

Uses an [autocollateralization](../usage/coreConcepts.html#autocollateralization) mechanism which determined the amount to collateralize basedj on recent payment volumes and set boundaries.

See the [configuration](#configuration) section of this document for information on how to set these bounds as a hub operator.

* **URL:** *hub-url*/channel/:user/request-collateralization

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |
| lastChanTx | `number` | Global nonce of the user's channel |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `{ error : "Received invalid collateral request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/request-collateralization",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    lastChanTx: 3,
  },
  success : function(r) {
    console.log(r);
  }
});
```

___

### Request Exchange

Initiates an in channel exchange (state update type of `Exchange`) with the hub. User's propose an amount to sell, and the hub will facilitate the exchange if it has collateral.

Note: Failures due to lack of collateral on exchange updates do not trigger the autocollateralization mechanism.

* **URL:** *hub-url*/channel/:user/request-exchange

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |
| weiToSell | `string` | Amount of eth to sell from user's balance in wei units |
| tokensToSell | `string` | Amount of tokens to sell from user's balance in wei units |
| lastChanTx | `number` | Global nonce of the user's channel |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `{ error : "Received invalid exchange request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/request-exchange",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    weiToSell: "1000",
    tokensToSell: "0", // can only sell one type of currency
    lastChanTx: 3,
  },
  success : function(r) {
    console.log(r);
  }
});
```

___

### Request Withdrawal

Creates a `ProposePendingWithdrawal` update, where an onchain tokens for wei exchange is performed. The hub will send the onchain transaction, and the user will not be able to take further action on the channel.

* **URL:** *hub-url*/channel/:user/request-withdrawal

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |
| lastChanTx | `number` | Global nonce of the user's channel |
| tokensToSell | `string` | Tokens to sell from user's balance as part of withdrawal and onchain exchange in wei units |
| weiToSell | `string` | Wei to sell from user's balance as part of withdrawal and onchain exchange in wei units |
| recipient | `string` | Eth address to send withdrawal funds to |
| withdrawalWeiUser | `string` | Eth to withdraw from user's channel balance in wei units |
| withdrawalTokenUser | `string` | Tokens to withdraw from user's channel balance in wei units |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `{ error : "Received invalid withdrawal request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/request-withdrawal",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    lastChanTx: 3,
    tokensToSell: "100",
    weiToSell: "0", // not supported by hub
    recipient: "0xbea7c...",
    withdrawalWeiUser: "100",
    withdrawalTokenUser: "0" // not supported by hub
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Update

Accepts and processes a series of channel updates signed by the user to the hub to be applied to their channel

* **URL:** *hub-url*/channel/:user/update

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |
| lastThreadUpdateId | `number` | Latest thread update id |
| updates | [`UpdateRequest`](types.html#updaterequest) | Array of updates to return to the hub from the client |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ error: string | null, updates:` [`Sync`](types.html#sync) `}` |
| 400 | BAD REQUEST | `{ error : "Received invalid update state request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/update",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    lastThreadUpdateId: 0,
    updates: [{
      reason: "Payment",
      args: { amountWei: "100", amountToken: "0" },
      sigUser: "0xq30gf...",
    }]
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Sync

Returns an array of updates to the user after the specified nonce as a [sync](types.html#sync) JSON object.

* **URL:** *hub-url*/channel/:user/sync?:lastChanTx&:lastThreadUpdate

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |
| lastChanTx | `string` | Channel nonce user is syncing from |
| lastThreadUpdateId | `string` | Thread update user is syncing from |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's authorization token, gotten from `[auth](#auth)` endpoint |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `{ error : "Received invalid sync request. Aborting." }` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47?lastChanTx=3&lastThreadUpdateId=0",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc" // cookie gotten from the `auth` endpoint
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Channel

  Returns a [`ChannelRow`](types.html#channelrow) JSON containing information about the user's channel.

* **URL:** *hub-url*/channel/:user

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ChannelRow`](types.html#channelrow) |
| 400 | BAD REQUEST | `{ error : "Received invalid get channel request. Aborting}` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc" // cookie gotten from the `auth` endpoint
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Latest Update

Returns a [`ChannelStateUpdateRow`](types.html#channelstateupdaterow) JSON the users latest state update.

* **URL:** *hub-url*/channel/:user/latest-update

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ChannelStateUpdateRow`](types.html#channelstateupdaterow) |
| 400 | BAD REQUEST | `{ error : "Received invalid get channel request. Aborting}` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/latest-update",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc" // cookie gotten from the `auth` endpoint
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Latest State No Pending Operations

Returns a [`ChannelState`](types.html#channelstate) JSON which is the latest state in the channel that does not contain any pending operations.

* **URL:** *hub-url*/channel/:user/latest-no-pending

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | User's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ChannelState`](types.html#channelstate) |
| 400 | BAD REQUEST | `{ error : "Received invalid get channel request. Aborting}` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/latest-no-pending",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc" // cookie gotten from the `auth` endpoint
  }
  success : function(r) {
    console.log(r);
  }
});
```