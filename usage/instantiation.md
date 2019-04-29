# Instantiating the Client


First, install the client package:

```javascript
npm install connext
```


Next, we'll need to start up Connext. The constructor accepts a Web3 object, the URL of the hub you're connecting to (in most cases, https://rinkeby.hub.connext.network/api/hub), a user address, and the host URL of your application. We'll use `getConnextClient` to spin up an instance of Connext and assign it to a variable. 
 
```javascript
import { getConnextClient } from "connext/dist/Connext.js";
const Web3 = require("web3");

// URLs for a live rinkeby hub
hubUrl=https://rinkeby.hub.connext.network/api/hub
ethUrl=https://rinkeby.hub.connext.network/api/eth

// URLs for if you're developing using a local hub (not recommended)
hubUrl=http://localhost:3000/api/eth
ethUrl=http://localhost:3000/api/hub

// Instantiate web3, get signer
const web3 = new Web3(new Web3.providers.HttpProvider(ethUrl))

async function() {

  const accounts = await web3.eth.getAccounts()

// Set the client options
  const connextOptions = {
    web3,
    hubUrl,
    user: accounts[0],
    origin: 'localhost' // the host url of your app
  }

// Instantiate a new instance of the client
  const connext = await getConnextClient(connextOptions)
```

Now that you've instantiated the client, you'll want to register a listener for the events that it emits and then start Connext!

```javascript
// Start the app, and register a listener
  connext.on('onStateChange', connext => {
    console.log('Connext:', connext)
  }
// Start Connext
  await connext.start()
}
```

Once you've instantiated the client, [the next step is to make a deposit to a channel](../deposits.md).