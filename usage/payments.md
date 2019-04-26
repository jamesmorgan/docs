# Payments and In-channel Swaps

**Note: The hub is responsible for "collateralizing" all payments. If the hub does not have funds to forward the payment in the payment recipient's channel, the payment will fail until the hub is able to deposit more tokens into the channel. To add collateral to the hub, simply send tokens (Dai if you're using our hosted hub) to the contract address. See the [Collateral section](./coreConcepts.md#collateral) to learn more.**

Now that you've deposited, you have a payment channel open with the hub. You're ready to start transacting by updating your channel state. Connext facilitates in-channel exchanges, token payments, and channel withdrawals in Wei.

Our hosted hub is collateralized with tokens, so prior to making a transaction you'll want to exchange your Wei for the Hub's Dai (denominated in Wei).


```javascript
  // Exchanging Wei for Dai
  await connext.exchange("1000", "wei");
```

Once you've exchanged, making a payment is simple: call `connext.buy()` with a payment object containing the recipient, payment type, payment amounts, and any metadata associated with the payment.

```javascript
  // Payments made can be retrieved using the returned purchase id,
  // which ties together all payment channel updates initiated in the array.
  const purchaseId = await connext.buy({
    meta: {
      // Any metadata associated with the payment
    },
    payments: [
      {
        recipient: "0x7fab....", // Payee  address
        amount: {
          amountToken: "10",
          amountWei: "0" // Only token payments are facilitated
        },
        type: "PT_CHANNEL", // The payment type, see the client docs for more
      },
    ]
  })
  ```

Now that you've made payments, you may want to [withdraw funds from your channel](./withdrawals.html).