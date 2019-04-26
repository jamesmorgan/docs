# Making a deposit to a channel

Now that the client is started, you can make a deposit into a channel. Channels can accept deposits in both ETH and tokens. However, when depositing tokens, ensure the user has sufficient ETH remaining in their wallet to afford the gas of the deposit transaction.

```javascript
  // Making a deposit in ETH
  await connext.deposit({
    amountWei: "1500",
    amountToken: "0", // assumed to be in wei units
  })
```

After you've deposited, you can [make a payment](./payments.html)!