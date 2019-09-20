# Advanced

## Monitoring your channel

### Accessing Channel State

Information about channel state can be accessed with `getChannel()`. This includes current node and client balances, availability of channel, and more. 

#### Usage Example

Information about channel state retrieved with `getChannel()` can be used (for example) to stop execution if certain conditions are not met:

```javascript
    var channelAvailable = (await channel.getChannel()).available
    if (!channelAvailable) {
      console.warn(`Channel not available yet.`);
      return;
    }
```

### Event Monitoring

The Connext client is an event emitter. You can trigger actions such as transfer confirmations in your application by listening for events using `connext.on()`. `connext.on()` accepts a string representing the event you'd like to listen for, as well as a callback. Available events are:

```
CHANNEL_CREATED
DEPOSIT_STARTED
DEPOSIT_CONFIRMED
WITHDRAW_STARTED
WITHDRAW_CONFIRMED
APP_INSTALLED
APP_REJECTED
APP_UNINSTALLED
APP_UPDATED
CHALLENGE_INITIATED
CHALLENGE_RESPONDED
CHALLENGE_RESOLVED
```

Example:

```javascript
connext.on("DEPOSIT_STARTED", () => 
            { console.log("Your deposit has begun")
              this.showDepositStarted()
            });
```

### Availability

The wallet must acknowledge every state update by cosigning that state. This means in order to update your channel, the user must be online. We've built several different transaction types to help accomodate user availability constraints. 


## Conditional Transfers

### Installing and Uninstalling Apps

## Predefined Transfer Types

### Link Transfers

### Oracle-based Transfers

### In-flight Swaps

## Writing Custom Apps

### How to Write Custom Contracts

#### Exporting Custom Contract ABI

### Creating a forceMove Update

