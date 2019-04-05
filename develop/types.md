# Types

## Index

Methods:

Types:

* [ConnextClientOptions](#connextclientoptions)
* [PurchaseRequest](#purchaserequest)
* [PurchasePaymentRequest](#purchasepaymentrequest)
* [MetadataType](#metadatatype)
* [Payment](#payment)
* [WithdrawalParameters](#withdrawalparameters)

Classes:

* [CurrencyConvertable](#currencyconvertable)

### ConnextClientOptions

Object including the options to instantiate the connext client with.

The object contains the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| web3 | Web3 | the Web3 object used by the client |
| hubUrl | String | url of the hub |
| user | Address | address of the signing wallet |
| contractAddress? | Address | (optional) address of the [ChannelManager](contracts.md) contract |
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

### PurchaseRequest

A purchase is considered to be a set of grouped payments (i.e. a tip and a fee).

A `PurchaseRequest` object has the following fields:

| Name | Type | Description |
| ------ | ------ | ------ |
| meta | *[`MetadataType`](#metadatatype)* | Any metadata associated with the purchase request. Stored by the hub. |
| meta | *[`PurchasePaymentRequest`](#purchasepaymentrequest)[]* | Any metadata associated with the purchase request. Stored by the hub. |

<!-- **● meta**: *`MetadataType`*

*Defined in [types.ts:953](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/types.ts#L953)*

**● payments**: *[PurchasePaymentRequest](../#purchasepaymentrequest)<`PaymentMetadataType`>[]*

*Defined in [types.ts:954](https://github.com/ConnextProject/indra/blob/5961649/modules/client/src/types.ts#L954)* -->

___

### PurchasePaymentRequest

A `PurchasePaymentRequest` object has the following fields:
| Name | Type | Description |
| ------ | ------ | ------ |
| meta | *[`MetadataType`](#metadatatype)* | Any metadata associated with the purchase request. Stored by the hub. |
| meta | *[`PurchasePaymentRequest`](#purchasepaymentrequest)[]* | Any metadata associated with the purchase request. Stored by the hub. |

This type is used in conjunction with the [`connext.buy`](client.html#buy) method and the [`PurchaseRequest`](#purchaserequest) type.

___

### MetadataType

___

### Payment

___

### WithdrawalParameters