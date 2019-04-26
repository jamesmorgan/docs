# Withdrawing from a channel

Right now, Connext supports withdrawals of funds from channel as Wei. The token funds that are in your channel are exchanged for Wei onchain as part of the withdrawal. The code to support token withdrawals is complete, but is currently disabled; if this presents a problem for your use case, please reach out to us.

```javascript
  await connext.withdraw({
    // Address to receive withdrawal funds
    // Does not need to have a channel with Connext to receive funds
    recipient: "0x8cef....",
    // USD price if using dai
    exchangeRate: "139.35",
    // Wei to transfer from the user's balance to 'recipient'
    withdrawalWeiUser: "500",
    // Tokens from channel balance to "sell" back to hub
    tokensToSell: "990",
  })

})()
```