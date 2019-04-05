# Architecture Overview

Connext is composed of three components that interoperate. At the core of the platform is our [ChannelManager](https://github.com/ConnextProject/contracts), which handle the onchain complexities of depositing to and withdrawing from payment channels, as well as disputing outcomes.

The [Client package](https://github.com/ConnextProject/connext-client) enables easy interaction with onchain functionality, in addition to allowing end users to make offchain payments amongst themselves. Specifically, it creates and validates offchain state updates, and helps users interact with our ChannelManager contract and the Hub.

Our [Hubs](https://github.com/ConnextProject/indra) can be thought of as an automated implementation of the Client package: once launched, their logic allows the Hub operator to manage many open channels and threads and facilitate payments between the users that are connected to that Hub.

[Architecture diagram](https://github.com/ConnextProject/docs/blob/master/ConnextArchitecture.png)

## Indra

[Indra](https://github.com/ConnextProject/indra) is the core implementation repository for Connext. Indra contains ready-for-deployment code for our core contracts, client, Hub, as well as scripts needed to deploy and operate a hub in production. When deploying a hub to production, the most recent docker images will be used.

For detailed instructions on how to run the code contained within the repository see the [Quickstart](../gettingStarted.md) and [Troubleshooting](../troubleshooting.md) guides.

There are several modules contained within the indra repository:

### Client

The Connext Client package is a Typescript interface which is used to communicate with deployed Connext contracts and with other clients. The client package is available through [NPM](https://www.npmjs.com/package/connext).

Clients are typically integrated into client-side code: either the frontend of your application or directly into the wallet layer (see [Wallet](#Wallet) for more). While you can use a combination of the Client and the contracts as a hot wallet, we recommend that you use an inpage wallet to autosign transactions. This allows you to abstract away many of the complexities of payment channels for your users and reduce UX friction.

Clients contain the following functionality:

* Depositing to a channel
* Opening a thread to any counterparty
* Closing a thread and automatically submitting the latest available mutually agreed update.
* Withdrawing from a channel and automatically submitting the latest available mutually agreed update.
* Handling a dispute.
* Generating/signing/sending and validating/receiving state updates over HTTPs. The Client takes in the address of the server that is being used to pass messages in the constructor.

Payment channel implementations need a communication layer where users can pass signed state updates to each other. The initial implementation of Connext does this through traditional server-client HTTPS requests. While this is the simplest and most effective mechanism for now, we plan to move to a synchronous message passing layer that doesn't depend on a centralized server as soon as possible.

Further documentation on the client can be found [here](../develop/client.md).

### Hub

Hubs can be thought of as an automated implementation of the client. Hubs have the same functionality outlined above, and forward payments between client users.

### Contracts

Our payment channel contracts. Our implementation relies on a combination of the research done by a variety of organizations, including Spankchain, Finality, Althea, Magmo and CounterFactual. Comprehensive documentation are fully [open source](https://github.com/ConnextProject/indra/modules/contracts) and are available [here](../develop/contracts.md).

The contracts repository should only be used for development purposes. The latest stable version of the contracts which works with Hub and Client will always be kept in Indra. For contributor documentation, check the repository.

### Wallet

The wallet is a sandbox-style UI for testing the functionality of the hub and client. This interface is designed primarily for testing, best practice implementations should refer to the [card](#Card) component. When running with docker, this should be available at `localhost:3000`.

### Dashboard

This is a simple admin UI for easily analyzing recent payment and user information as a hub operator. The dashboard server is accessible at `localhost:3000/api/dashboard` and the UI is served from `localhost:3000/dashboard/` (if running on docker).

### Database

Contains all the database migration information to be used by the docker images. Make sure that the migrations in the database module, and in the `hub/migrations` folder are consistent if you are making changes here.

### Proxy

Routing proxy and related configurations.

## Card

The [card](https://github.com/ConnextProject/card/) is a sample implementation of the client designed to help you bootstrap an application that integrates Connext. It contains a simple inpage wallet and payment interface, as well as a custom Web3 injection that automatically signs transactions using the inpage wallet. For developers just beginning to build their application, the wallet is a great way to get started; for developers looking to integrate with existing an existing app, it's a good instructive resource for implementation and includes some components that you can easily copy over.