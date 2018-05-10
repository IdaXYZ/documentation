# Javacript Library

## Introduction

Welcome to the requestNetwork.js documentation! requestNetwork.js is a Javascript library for interacting with the Request Network protocol. Using the library you can create new requests from your applications, pay them, consult them or update them from your own off-chain applications.

## Installing

### Using NPM
`npm install @requestnetwork/request-network.js --save`

### Using Yarn
`yarn add @requestnetwork/request-network.js`

(We are currently working on retrieving the name of requestnetwork.js)

## Example

```javascript
const RequestNetwork = require('@requestnetwork/request-network.js');
const Web3 = require('Web3');

const web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));
const requestNetwork = new RequestNetwork.default({
  provider: web3.currentProvider,
  ethNetworkId: 10000000000,
  useIpfsPublic: false,
});

const [payeeAddress, payerAddress] = await web3.eth.getAccounts();

// Create a request as payer
const payerInfo = {
  idAddress: payerAddress,
  refundAddress: payerAddress,
};

const payeesInfo = [{
  idAddress: payeeAddress,
  paymentAddress: payeeAddress,
  expectedAmount: web3.utils.toWei('1.5', 'ether'),
}]

const { request } = await requestNetwork.createRequest(
  RequestNetwork.Types.Role.Payee,
  RequestNetwork.Types.Currency.ETH,
  payeesInfo,
  payerInfo,
);

// Pay a request
await request.pay([web3.utils.toWei('1.5', 'ether')], [0], { from: payerAddress });

// The balance is the same amount as the the expected amount: the request is paid
const data = await request.getData();
console.log(data.payee.expectedAmount.toString());
console.log(data.payee.balance.toString());
```

## Importing and Initializing

Using import
```javascript
import RequestNetwork from '@requestnetwork/request-network.js';
const requestNetwork = new RequestNetwork({ provider, ethNetworkId });
```

Using require
```javascript
const RequestNetwork = require('@requestnetwork/request-network.js');
const requestNetwork = new RequestNetwork.default({ provider, ethNetworkId });
```

The parameter for the constructor (all optional) are
 - provider: Web3.js Provider instance an url as string
 - ethNetworkId: the Ethereum network ID (1: main, 2: morden, 3: ropsen, 4: rinkeby, 42: kovan, other: private)
 - bitoinNetworkId: the Bitcoin network ID
 - useIpfsPublic: false to use a private ipfs network

## Features

### Create a Request

```javascript
const { request } = await requestNetwork.createRequest(
    role,
    Types.Currency.ETH,
    [{
        idAddress: accounts[0],
        paymentAddress: accounts[0],
        additional: 5,
        expectedAmount: 100,
    }],
    {
        idAddress: accounts[1],
        refundAddress: accounts[1],
    }
);
```

### Retrieve a Request from its ID

```javascript
const request = await requestNetwork.fromRequestId(requestId);
```

### Pay a Request

```javascript
await request.pay([web3.utils.toWei('1.5', 'ether')]);
```

For example for Requests in ETH, the amounts are in wei. An amount of 100 will be 100 wei. An amount of 0.1 will be rounded down to 0 wei.

### Cancel a Request

```javascript
await request.cancel();
```

### Signed Requests

#### Sign a request

```javascript
const signedRequest = await requestNetwork.createSignedRequest(
    Types.Role.Payee,
    Types.Currency.ETH,
    payeesInfo,
    Date.now() + 3600*1000
);
```

#### Broadcast a signed request

```javascript
const { request } = await requestNetwork.broadcastSignedRequest(signedRequest, payerInfo);
```

The payer is necessary to broadcast the signed request

#### Check a signed request

```javascript
signedRequest.isValid(payerInfo);
```

The payer is necessary to validate the signed request

### Broadcasted events

```javascript
await request.pay([web3.utils.toWei('1.5', 'ether')]).on('broadcasted', () => {});
```

## Accessing the Private API
