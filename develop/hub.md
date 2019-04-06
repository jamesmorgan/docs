# Hub

## Configuration

## Authorization

The authorization for client implementers is performed on [`connext.start()`](client.html#start) through an internal `auth` method.

The authorization flow is implemented as follows:

```typescript
const rpcUrl = "localhost:8545"
const hubUrl = "localhost:8080"
// Imagine the `hub` is an object with an interface corresponding to the
// methods outlined below

// first get the nonce
const nonce = await hub.authChallenge()

// create hash and sign
const preamble = "SpankWallet authentication message:";
const web3 = new Web3(rpcUrl)
const hash = web3.utils.sha3(`${preamble} ${web3.utils.sha3(nonce)} ${web3.utils.sha3(origin)}`);
const signature = await web3.eth.personal.sign(hash, this.opts.user);

// return to hub
const auth = await hub.authResponse(nonce, this.opts.user, origin, signature)
// save auth token, e.g
document.cookie = `hub.sid=${auth}`;
```

## API Documentation

Open Endpoints:

* [Get Config](#get-config)
* [Auth Challenge](#auth-challenge)
* [Auth Response](#auth-response)
* [Auth Status](#auth-status)

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

* [Get Thread](#get-thread)
* [Get Threads](#get-threads)
* [Get Initial Thread States](#get-initial-thread-states)
* [Get Incoming Threads](#get-incoming-threads)
* [Get Active Threads](#get-active-threads)
* [Get Last Thread Update Id](#get-last-thread-update-id)
* [Update Thread](#update-thread)

*Requires proper authorization

### Get Config

Returns configuration information from the hub as a JSON. Configuration information returned includes:

| Name | Type | Description |
| ------ | ------ | ------ |
| channelManagerAddress | `string` | The address of the [`ChannelManager`](contracts.html) contract |
| hotWalletAddress | `string` | The eth address of the hub's signing wallet |
| tokenContractAddress | `string` | The approved token contract address |
| ethRpcUrl | `string` | The url of the eth provider used by the hub |
| ethNetworkId | `string` | Network id of the eth network the hub is connected to |
| beiMaxCollateralization | `string` | Token autocollateralization ceiling |

* **URL:** *hub-url*/config

* **Method:** `GET`

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ channelManagerAddress: string, hotWalletAddress: string, tokenContractAddress: string, ethRpcUrl: string, ethNetworkId: string, beiMaxCollateralization: string }` |

* **Example:**

```typescript
$.ajax({
  url: "config",
  type : "GET",
  success : function(r) {
    console.log(r);
  }
});
```

___

### Auth Challenge

Returns a nonce for the user to sign. See the [authorization](#authorization) section for more details.

* **URL:** *hub-url*/auth/challenge

* **Method:** `POST`

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ nonce: string }` |

* **Example:**

```typescript
$.ajax({
  url: "auth/challenge",
  type : "POST",
  data: {},
  success : function(r) {
    console.log(r);
  }
});
```

___

### Auth Response

Handles a response to the users authorization challenge. See the [authorization](#authorization) section for more details.

* **URL:** *hub-url*/auth/response

* **Method:** `POST`

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| address | `string` | Eth address of user requesting auth |
| nonce | `string` | Value retrieved from [auth challenge](#auth-challenge) endpoint |
| origin | `string` | Origin of domain requests |
| signature | `string` | Users signature of the challenge response |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ nonce: string }` |
| 400 | BAD REQUEST | `Received invalid challenge request. Aborting.` |
| 400 | BAD REQUEST | `Received auth challenge from invalid origin` |

* **Example:**

```typescript
$.ajax({
  url: "auth/response",
  type : "POST",
  data: {
    address: "0x22fab...",
    nonce: "82basdh",
    origin: "localhost",
    signature: "0xdan38...",
  },
  success : function(r) {
    console.log(r);
  }
});
```

___

### Auth Status

Returns the authorization status of the request session address as a JSON object with the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| success | `boolean` | Returns true if the session is authorized |
| address | `string` | The address with the valid auth status. Field is ommitted if success is `false` |

* **URL:** *hub-url*/auth/status

* **Method:** `GET`

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ success: true, address: string }` |
| 200 | SUCCESS | `{ success: false }` |

* **Example:**

```typescript
$.ajax({
  url: "auth/status",
  type : "GET",
  success : function(r) {
    console.log(r);
  }
});
```

___

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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |
| depositWei | `string` | Amount of eth the user wants to deposit in wei |
| depositToken | `string` | Amount of tokens the user wants to deposit in wei |
| lastChanTx | `number` | The last global nonce of the channel |
| lastThreadUpdateId | `number` | The latest thread update id |
| sigUser | `string` | User's signature of their deposit values |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ChannelRow`](types.html#channelrow) |
| 400 | BAD REQUEST | `Received invalid user deposit request. Aborting.` |
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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |
| lastChanTx | `number` | Global nonce of the user's channel |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `Received invalid collateral request. Aborting.` |
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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |
| weiToSell | `string` | Amount of eth to sell from user's balance in wei units |
| tokensToSell | `string` | Amount of tokens to sell from user's balance in wei units |
| lastChanTx | `number` | Global nonce of the user's channel |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `Received invalid exchange request. Aborting.` |
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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |
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
| 400 | BAD REQUEST | `Received invalid withdrawal request. Aborting.` |
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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |
| lastThreadUpdateId | `number` | Latest thread update id |
| updates | [`UpdateRequest`](types.html#updaterequest) | Array of updates to return to the hub from the client |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ error: string | null, updates:` [`Sync`](types.html#sync) `}` |
| 400 | BAD REQUEST | `Received invalid update state request. Aborting.` |
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
| authToken | `string` | User's authorization token, gotten from `[auth](#authorization)` flow |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`Sync`](types.html#sync) |
| 400 | BAD REQUEST | `"Received invalid sync request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47?lastChanTx=3&lastThreadUpdateId=0",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
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
| 400 | BAD REQUEST | `Received invalid get channel request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
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
| 400 | BAD REQUEST | `Received invalid get channel request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/latest-update",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
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
| 400 | BAD REQUEST | `Received invalid get channel request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "channel/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/latest-no-pending",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Thread

Returns a [`ThreadRow`](types.html#threadrow) JSON representing thread details of thread with provided sender and receiver.

* **URL:** *hub-url*/thread/:sender/to/:receiver

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| sender | `string` | Sender's eth address |
| receiver | `string` | Receiver's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadRow`](types.html#threadrow) |
| 400 | BAD REQUEST | `Received invalid get thread request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/to/0x2DA565caa7037Eb198393181089e92181ef5Fb53",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Threads

Returns a [`ThreadState[]`](types.html#threadstate), containing threads for the provided user. Including the inactive threads.

* **URL:** *hub-url*/thread/:user/all

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadState[]`](types.html#threadstate) |
| 400 | BAD REQUEST | `Received invalid get threads request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/all",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Initial Thread States

Returns a [`ThreadState[]`](types.html#threadstate), containing the initial thread states for the provided user.

This is used to regenerate the merkle root of the channel as threads open and close.

* **URL:** *hub-url*/thread/:user/initial-states

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadState[]`](types.html#threadstate) |
| 400 | BAD REQUEST | `Received invalid get thread initial states request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/initial-states",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Incoming Threads

Returns a [`ThreadRow[]`](types.html#threadrow), containing the incoming thread (i.e. user is receiver) states for the provided user.

* **URL:** *hub-url*/thread/:user/incoming

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadRow[]`](types.html#threadrow) |
| 400 | BAD REQUEST | `Received invalid get incoming threads request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/incoming",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Active Threads

Returns a [`ThreadState[]`](types.html#threadstate), containing the active threads for the provided user.

* **URL:** *hub-url*/thread/:user/active

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadState[]`](types.html#threadstate) |
| 400 | BAD REQUEST | `Received invalid get active threads request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/active",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Get Last Thread Update Id

Returns the latest thread update id for the provided user

* **URL:** *hub-url*/thread/:user/last-update-id

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| sender | `string` | Sender's eth address |
| receiver | `string` | Receiver's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | `{ latestThreadUpdateId: number }` |
| 400 | BAD REQUEST | `Received invalid get last update id request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/last-update-id",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc"
  }
  success : function(r) {
    console.log(r);
  }
});
```

___

### Update Thread

Accepts and stores sender-signed thread updates. Returns the updated thread as a [`ThreadStateUpdateRow`](types.html#threadstateupdaterow)

* **URL:** *hub-url*/thread/:sender/to/:receiver/update

* **Method:** `POST`
  
* **URL Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| user | `string` | User's eth address |

* **Data Params**

| Name | Type | Description |
| ------ | ------ | ------ |
| authToken | `string` | Request sender's current auth token |
| update | [`ThreadState`](types.html#threadstate) | The sender-signed thread state update |

* **Responses**

| Code | Status | Content |
| ------ | ------ | ------ |
| 200 | SUCCESS | [`ThreadState`](types.html#threadstate) |
| 400 | BAD REQUEST | `Received invalid update thread request. Aborting.` |
| 404 | NOT FOUND | `null` |

* **Example:**

```typescript
$.ajax({
  url: "thread/0x30FdA91A43B5B9eA7c5027619259f8dB2a68aA47/to/0x2DA565caa7037Eb198393181089e92181ef5Fb53/update",
  dataType: "json",
  type : "POST",
  data: {
    authToken: "Qgf7AM5WFc",
    update: {
      contractAddress: "0xj38cw..",
      sender: "0xad43fw..",
      receiver: "0x22asd..",
      threadId: 1,
      balanceWeiSender: "1000",
      balanceWeiReceiver: "0",
      balanceTokenSender: "100",
      balanceTokenReceiver: "0",
      txCount: 0,
      sigA: "0xf7am5..."
    }
  }
  success : function(r) {
    console.log(r);
  }
});
```