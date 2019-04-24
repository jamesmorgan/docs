## Running a Local Hub

To sample the hub in action on rinkeby or mainnet check out the [Dai Card](https://daicard.io). To get started locally, make sure you have the following prerequisites installed:

- [Node 9+ and NPM](https://nodejs.org/en/)
- [Docker](https://www.docker.com/)
- Make: (probably already installed) Install w ```brew install make``` or ```apt install make``` or similar.
- jq: (probably not installed yet) Install w ```brew install jq``` or ```apt install jq``` or similar.

Then, open a terminal window and run the following:

```bash
git clone https://github.com/ConnextProject/indra.git
cd indra
npm start
```

Starting Indra will take a while the first time, but the build gets cached so it will only take this long the first time. While it's building, configure your metamask to use the rpc `localhost:3000/api/eth` and import the hub's private key (`659CBB0E2411A44DB63778987B1E22153C086A95EB6B18BDF89DE078917ABC63`) so you can easily send money to the signing wallet.

Before trying to make payments, ensure that that hub is collateralized by sending tokens to the contract address.

Once all the components are up and running, navigate to `http://localhost/` to checkout the sandbox. To implement the client on your own frontend, checko ut the [Getting Started guide](../usage/gettingStarted).

### Additional Resources

For more detailed information about our hubs, check out the following:

- [API Docs](./hub.md)
- Deploying locally [with docker](https://github.com/ConnextProject/indra#to-deploy-using-docker) and [without docker](https://github.com/ConnextProject/indra#to-deploy-locally)
- [Deploying to Production](https://github.com/ConnextProject/indra#deploying-to-production)
- [Interacting with a Hub](https://github.com/ConnextProject/indra#how-to-interact-with-an-indra-hub)