# Client

## Best Practices

### Configuration

The best way to get a new instance of the connext client is using the `getConnextClient` function, and supplying the `hubUrl` and `rpcUrl` as environment variables.

### Making Payments

## Methods

* [getConnextClient](#getConnextClient)
* [buy](#buy)
* [deposit](#deposit)
* [exchange](#exchange)
* [recipientNeedsCollateral](#recipientneedscollateral)
* [redeem](#redeem)
* [start](#start)
* [stop](#stop)
* [withdraw](#withdraw)

### getConnextClient

Retrieve a new instance of the connext client initialized with the provided parameters.

The `getConnextClient` method takes in a [ConnextClientOptions](types.md#ConnextClientOptions) object.

```javascript
const client = getConnextClient({...})
// register listener
client.on('onStateChange', state => {
  console.log('Connext state changed:', state);
})
await client.start() // start polling
```

### buy

▸ **buy**(purchase: *[PurchaseRequest](../interfaces/purchaserequest.md)*): `Promise`<`object`>

*Defined in [Connext.ts:958](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L958)*

Make a purchase within the channel. A purchase is defined as a bundle of related payments.

Additionally, purchases can have meta objects associated with them. These objects are stored as JSONs by the hub.

**Parameters:**

| Name | Type | Description |
| ------ | ------ | ------ |
| purchase | [PurchaseRequest](types.md#purchaserequest) |  the \`PurchaseRequest\` sent to the hub. |

**Returns:** `Promise`<`object`>

___

### deposit

▸ **deposit**(payment: *[Payment](../#channelupdatereasons.payment)*): `Promise`<`void`>

*Defined in [Connext.ts:935](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L935)*

This method allows users to deposit into their own channels.

It generates a `ProposePendingDeposit` state update, that is then cosigned by the hub, and sent to chain.

If there is a problem with the onchain transaction, the client will automatically send an `Invalidation` update to the hub and revert any pending deposits into their channel.

While users have an onchain transaction in flight, they are not able to make further updates to their state until the transaction is either confirmed onchain, or it is invalidated.

**Parameters:**

| Name | Type | Description |
| ------ | ------ | ------ |
| payment | [Payment](../#channelupdatereasons.payment) |  the deposit amount as a \`Payment\`(#payment) type |

**Returns:** `Promise`<`void`>

___

### exchange

▸ **exchange**(toSell: *`string`*, currency: *"wei" \| "token"*): `Promise`<`void`>

*Defined in [Connext.ts:947](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L947)*

Allows a user to initiate an in-channel exchange with the hub by requesting an exchange and proposing an `Exchange` update type.

Exchange rates used are supplied by the hub, and verified to be within a reasonable percentage before moving forward with the exchange.

**Parameters:**

| Name | Type | Description |
| ------ | ------ | ------ |
| toSell | `string` |  the amount of eth/tokens to sell, in wei units |
| currency | "wei" \| "token" |  can either be "wei" or "token" to specify which type of currency you are selling |

**Returns:** `Promise`<`void`>

___

### recipientNeedsCollateral

▸ **recipientNeedsCollateral**(recipient: *[Address](../#address)*, amount: *[Payment](../#channelupdatereasons.payment)*): `Promise`<`string` \| `null`>

*Defined in [Connext.ts:1010](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L1010)*

Returns null if the hub has enough collateral in the recipient's channel to facilitate a payment, and a descriptive string if it cannot.

Insufficiently collateralized payments will fail, but they will trigger the hub's autocollateralization mechanism.

Implementers can use this function to check the status of a recipient's collateral, and intentionally send a failing payment to start the autocollateralization, then use this function to monitor the status of that collateralization.

**Parameters:**

| Name | Type | Description |
| ------ | ------ | ------ |
| recipient | [Address](../#address) |  the signing wallet address of the payment recipient |
| amount | [Payment](../#channelupdatereasons.payment) |  the amount of the payment, as a \`Payment\` type. |

**Returns:** `Promise`<`string` \| `null`>

___

### redeem

▸ **redeem**(secret: *`string`*): `Promise`<`object`>

*Defined in [Connext.ts:997](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L997)*

Withdraw funds from a channel.

Partial withdrawals, full withdrawals, and withdrawals with an onchain exchange (i.e. the hub uses contract reserves instead of channel balance to fund an exchange at withdrawal) are supported by the hub.

Currently, token withdrawals are not supported as the hub is trying to maintain its token collateral.

**Parameters:**

| Name | Type |
| ------ | ------ |
| secret | `string` |

**Returns:** `Promise`<`object`>

___

### requestCollateral

▸ **requestCollateral**(): `Promise`<`void`>

*Defined in [Connext.ts:984](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L984)*

Withdraw funds from a channel.

Partial withdrawals, full withdrawals, and withdrawals with an onchain exchange (i.e. the hub uses contract reserves instead of channel balance to fund an exchange at withdrawal) are supported by the hub.

Currently, token withdrawals are not supported as the hub is trying to maintain its token collateral.

**Returns:** `Promise`<`void`>

___

### start

▸ **start**(): `Promise`<`void`>

*Defined in [Connext.ts:913](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L913)*

Starts the stateful portions of the Connext client.

Note: the full implementation lives in ConnextInternal.

**Returns:** `Promise`<`void`>

___

### stop

▸ **stop**(): `Promise`<`void`>

*Defined in [Connext.ts:921](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L921)*

Stops the stateful portions of the Connext client.

Note: the full implementation lives in ConnextInternal.

**Returns:** `Promise`<`void`>

___

### withdraw

▸ **withdraw**(withdrawal: *[WithdrawalParameters](../#withdrawalparameters)*): `Promise`<`void`>

*Defined in [Connext.ts:971](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/Connext.ts#L971)*

Withdraw funds from a channel.

Partial withdrawals, full withdrawals, and withdrawals with an onchain exchange (i.e. the hub uses contract reserves instead of channel balance to fund an exchange at withdrawal) are supported by the hub.

Currently, token withdrawals are not supported as the hub is trying to maintain its token collateral.

**Parameters:**

| Name | Type | Description |
| ------ | ------ | ------ |
| withdrawal | [WithdrawalParameters](../#withdrawalparameters) |  the \`WithdrawalParameters\` sent to the hub. |

**Returns:** `Promise`<`void`>
