# v2.0 Architecture Overview



## Indra

[Indra](https://github.com/ConnextProject/indra-v2) is the core implementation repository for Connext. Indra contains ready-for-deployment code for our core contracts, client, node, as well as scripts needed to deploy and operate a node in production. When deploying a node to production, the most recent docker images will be used.

For detailed instructions on how to run the code contained within the repository see the [Running your own node](../nodeDocumentation/runNode.md) guide.

There are several modules contained within the indra repository:

### Client

The Connext Client package is a Typescript interface which is used to connect to a node and communicate over the Connext Network. The client package is available through [NPM](https://www.npmjs.com/package/connext).

Clients are ideally integrated directly at the wallet layer. This is because they contain validator code which determines, on behalf of the user, whether a given state is safe to sign. Clients can also be used via a `connextProvider` in a frontend environment to make calls to a hooked client that is hosted in a more trusted environment such as a wallet.

Clients contain the following functionality:

* Depositing to a channel
* Opening a thread to any counterparty
* Closing a thread and automatically submitting the latest available mutually agreed update.
* Withdrawing from a channel and automatically submitting the latest available mutually agreed update.
* Handling a dispute.
* Generating/signing/sending and validating/receiving state updates over NATS. The Client takes in the address of the server that is being used to pass messages in the constructor.

Payment channel implementations need a communication layer where users can pass signed state updates to each other. The initial implementation of Connext does this through traditional server-client HTTPS requests. While this is the simplest and most effective mechanism for now, we plan to move to a synchronous message passing layer that doesn't depend on a centralized server as soon as possible.

Further documentation on the client can be found [here](../userDocumentation/clientAPI.md).

### node

nodes can be thought of as an automated implementation of the client. nodes have the same functionality outlined above, and forward payments between clients.

### Contracts

Our payment channel contracts. Our implementation relies on a combination of the research done by a variety of organizations, including Spankchain, Finality, Althea, Magmo and CounterFactual. Comprehensive documentation are fully [open source](https://github.com/ConnextProject/indra/modules/contracts) and are available [here](../develop/contracts.md).

The contracts repository should only be used for development purposes. The latest stable version of the contracts which works with node and Client will always be kept in Indra. **Do not modify the contracts themselves before deploying - this could break the security model of the entire protocol**

### Dashboard

This is a simple admin UI for easily analyzing recent payment and user information as a node operator. The dashboard server is accessible at `localhost:3000/api/dashboard` and the UI is served from `localhost:3000/dashboard/` (if running on docker).

### Database

Contains all the database migration information to be used by the docker images. Make sure that the migrations in the database module, and in the `node/migrations` folder are consistent if you are making changes here.

### Proxy

Routing proxy and related configurations.

## Card

The [card](https://github.com/ConnextProject/card/) is a sample implementation of the client designed to help you bootstrap an application that integrates Connext. It contains a simple inpage wallet and payment interface, as well as a custom Web3 injection that automatically signs transactions using the inpage wallet. For developers just beginning to build their application, the card is a great way to get started; for developers looking to integrate with existing an existing app, it's a good instructive resource for implementation and includes some components that you can easily copy over.