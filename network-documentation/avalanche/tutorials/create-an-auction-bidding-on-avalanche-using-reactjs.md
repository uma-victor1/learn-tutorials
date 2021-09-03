# Introduction

We will learn how to build smart contracts by making an auction on which users can place bids, and deploy the smart contracts on Avalanche. We will then be able to interact with them using ReactJS and Drizzle.

We are going to generate [ReactJS](https://reactjs.org) boilerplate code using `create-react-app`, which we will modify for our auction bidding frontend. React is useful for this task due to its efficiency and user-friendly blockchain interaction. For the smart contracts which provide the on-chain functionality, [Solidity](https://docs.soliditylang.org/en/v0.8.4/) code will be deployed to the Avalanche blockchain using [Truffle](https://www.trufflesuite.com).

# Prerequisites

- Basic knowledge of [ReactJS](https://reactjs.org).
- Basic knowledge of [Avalanche's architecture](https://docs.avax.network/learn/platform-overview) and smart contracts.
- Basic familiarity with the React [context API](https://reactjs.org/docs/context.html).
- You will also need an account on [DataHub](https://datahub.figment.io/sign_up?service=avalanche).

# Requirements

- [NodeJS](https://nodejs.org/en) >= 10.16 and [npm](https://www.npmjs.com/) >= 5.6 installed.
- [Truffle](https://www.trufflesuite.com/truffle), which can be installed globally with `npm install -g truffle`.
- The Metamask wallet software must be added to your browser, which should only be obtained from the official Metamask website: https://metamask.io. **Do not download Metamask from an unofficial source!**

# Project setup

Before starting work on the project, we need to set up a working directory. Follow the steps below to create a directory for the project. Open up a terminal and navigate to the directory where you would like to store this application - typically the user home directory.

Create a new subdirectory with the command `mkdir <directory_name>`. Change the current working directory to this newly created directory using `cd <directory_name>`. For instance: if we name it `bid`, then:

```text
mkdir bid
cd bid
```

## Creating the React project

We can create a new React app using `npx` ([npm's package runner](https://www.npmjs.com/package/npx)). The typical use is to download the files needed to run an npm package. Using `npx` to execute the package binaries for `create-react-app` will generate a new React app scaffold in the specified directory.

```text
npx create-react-app client
```

Now change to the directory "client" using `cd client`, then install the required dependencies using the command:

```text
npm install --save dotenv web3 @truffle/contract @truffle/hdwallet-provider @drizzle/store
```

Open the file `index.js` inside the `src` directory and replace the existing code with the following:

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

import { Drizzle } from "@drizzle/store";
import drizzleOptions from "./drizzleOptions";
import { DrizzleProvider } from "./drizzleContext";

const drizzle = new Drizzle(drizzleOptions);

ReactDOM.render(
  <DrizzleProvider drizzle={drizzle}>
    <App />
  </DrizzleProvider>,
  document.getElementById("root")
);
```

Next, open the `App.js` file present inside the `src` directory and replace the existing code with the following:

```javascript
import Auction from "./Auction";
import { useDrizzleContext } from "./drizzleContext";

function App() {
  const { drizzleVariables } = useDrizzleContext();

  if (!drizzleVariables.initialized) {
    return "Loading...";
  } else {
    return <Auction />;
  }
}

export default App;
```

When this is done, the React project setup is complete. We are now ready to set up Truffle.

## Setup the Truffle project

Truffle can be used to create the boilerplate code needed for the project. Run the following command in the project root directory:

```text
truffle init
```

Now the basis of the project is set up. Solidity code will be stored in the `contracts` subdirectory. Truffle deployment functions written in JavaScript will be stored in the `migrations` folder. By default, the `/build/contracts` folder contains information about the compiled and deployed contract (like the [Application Binary Interface](https://docs.soliditylang.org/en/latest/abi-spec.html)) in JSON format. These meta-files are commonly referred to as `artifacts`.

`truffle-config.js` is the configuration file created by the `truffle init` command. This file contains the information telling Truffle how to deploy contracts, which network to deploy on, and more. We should keep the default file for reference and we can create a copy of this file using the command:

```text
cp truffle-config.js truffle-config-default.js
```

Now that we have a copy of the default configuration, we can update `truffle-config.js` with the information needed to deploy the smart contract on the Fuji testnet. This will help us in connecting to the DataHub Avalanche node, however we will require an Avalanche API key (from DataHub) along with an Avalanche wallet mnemonic used for deploying the contract on the network.
Replace the existing contents of `truffle-config.js` with the following:

```javascript
require("dotenv").config();
const HDWalletProvider = require("@truffle/hdwallet-provider");

// Account credentials from which our contract will be deployed
const mnemonic = process.env.MNEMONIC;

// API key of your Datahub account for Avalanche Fuji test network
const APIKEY = process.env.APIKEY;

const DATAHUB_RPC_URL = `https://avalanche--fuji--rpc.datahub.figment.io/apikey/${process.env.APIKEY}/ext/bc/C/rpc`;

module.exports = {
  contracts_build_directory: "./client/src/build/contracts/",
  networks: {
    fuji: {
      provider: function () {
        return new HDWalletProvider({
          mnemonic,
          providerOrUrl: DATAHUB_RPC_URL,
          chainId: "0xa869",
        });
      },
      network_id: "*",
      gas: 3000000,
      gasPrice: 470000000000,
      skipDryRun: true,
    },
  },
  compilers: {
    solc: {
      version: "0.8.0",
    },
  },
};
```

Here we are setting the `gas` and `gasprice` to appropriate values for the Avalanche C-Chain. You might notice that `contract_build_directory` is being used to change the default location of `artifacts` from the project root directory to the `src` folder. This is because React is unable to access the files present outside the `src` folder.

## Receive Avalanche Credentials

For the deployment of the smart contracts we need to take care of two things:

1. A node connected to the Avalanche network and an account with some AVAX.
2. An Avalanche API key is required to access the DataHub Avalanche node through RPC (Remote Procedure Call). Visit the [Avalanche Services Dashboard](https://datahub.figment.io/services/avalanche) on DataHub to get an Avalanche specific API key.

![](https://imgur.com/60J8ZZ7.png)

Next, we need to create a new Avalanche wallet to make transactions on the network and to deposit our funds. To create an Avalanche wallet, go to https://wallet.avax.network and save all the necessary information -- the wallet address and mnemonic seed phrase.

## Add .env file

Create a new file named `.env` in the project root folder. Copy your Avalanche API key from DataHub and the Avalanche wallet's mnemonic into the `.env` file as shown below. If you have any difficulty in setting up the `.env` file then please refer to the Figment Learn guide on [dotenv and .env](https://docs.figment.io/network-documentation/extra-guides/dotenv-and-.env).

```text
DATAHUB_API_KEY=<your-api-key>
MNEMONIC="<avalanche-wallet-mnemonic>"
```

Now the project setup is completed! Run the command `npm start` in the project root folder to start the development server. This will allow us to build the React interface with immediate visual feedback.

## Create Auction smart contract

Next we will need to create our main Solidity smart contract file named `Auction.sol` inside the `contracts` subdirectory and add the following:

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.0;

contract AuctionManager {
	uint public uId = 0;
	uint public aId = 0;

	// Structure to store user information
	struct User {
		uint userId;
		string name;
		address publicAddress;
	}

	// Structure to store bid information
	struct Bid {
		uint userId;
		uint bidPrice;
	}

	// Structure to store auction information
	struct Auction {
		uint auctionId;
		uint userId;
		string name;
		string description;
		uint msp;
		uint auctionBidId;
	}

	// Structure to store real time analytics of each auction
	struct AuctionAnalytics {
		uint auctionId;
		uint auctionBidId;
		uint latestBid;
		uint lowestBid;
		uint highestBid;
	}

	// List of all auctions
	Auction[] public auctions;

	// Mapping for storing user info, bids and auction analytics
	mapping (uint => User) public users;
	mapping (uint => Bid[]) public bids;
	mapping (uint => AuctionAnalytics) public auctionAnalytics;

	// Public function to check the registration of users (public address)
	function isRegistered(address _publicAddress) public view returns (uint256[2] memory) {
		uint256[2] memory result = [uint256(0), uint256(0)];
		for(uint i = 0; i < uId; i++) {
			if(_publicAddress == users[i].publicAddress) {
				result[0] = 1;
				result[1] = i;
				return result;
			}
		}
		return result;
	}

	// Creating new users
	function createUser(string memory _name) public {
		require((isRegistered(msg.sender))[0] == 0, "User already registered!");
		users[uId] = User(uId, _name, msg.sender);
		uId++;
	}

	// Creating new auctions
	function createAuction(string memory _name, string memory _description, uint _msp) public {
		require((isRegistered(msg.sender))[0] == 1, "User not registered!");
		uint MAX_UINT = 2 ** 256 - 1;
		auctions.push(Auction(aId, isRegistered(msg.sender)[1], _name, _description, _msp, 0));
		auctionAnalytics[aId] = AuctionAnalytics(aId, 0, 0, MAX_UINT, 0);
		aId++;
	}

	// Private function to update auction analytics after the new bids
	function updateAucionAnalytics(uint _aId, uint _latestBid) private {
		auctionAnalytics[_aId].latestBid = _latestBid;
		auctionAnalytics[_aId].auctionBidId = auctions[_aId].auctionBidId;
		if(_latestBid < auctionAnalytics[_aId].lowestBid) {
			auctionAnalytics[_aId].lowestBid = _latestBid;
		}
		if(_latestBid > auctionAnalytics[_aId].highestBid) {
			auctionAnalytics[_aId].highestBid = _latestBid;
		}
	}

	// Creating new bids
	function createBid(uint _aId, uint _bidPrice) public {
		require((isRegistered(msg.sender))[0] == 1, "User not registered!");
		bids[_aId].push(Bid((isRegistered(msg.sender))[1], _bidPrice));
		auctions[_aId].auctionBidId++;
		updateAucionAnalytics(_aId, _bidPrice);
	}

	// Return list of all auctions
	function showAuctions() public view returns (Auction[] memory) {
		return auctions;
	}
}
```

`Auction` is a Solidity contract which enables us to view the auction details and correspondingly its minimum price. We will be accessing the deployed Auction contracts using their deployed address and ABI. Each time a new auction is created, the Solidity code will be deployed to the blockchain.

# Understanding the contract

## Users, bids, auctions and analytics

```javascript
// List of all auctions
Auction[] public auctions;

// Mapping for storing user info, bids and auction analytics
mapping (uint => User) public users;
mapping (uint => Bid[]) public bids;
mapping (uint => AuctionAnalytics) public auctionAnalytics;
```

This code declares public variables for storing user information, their bids, auctions and auction analytics using [Solidity mappings](https://docs.soliditylang.org/en/latest/types.html#mapping-types). Have a look at the struct definitions used in these variables: They contain the data types and fields which make up a User, Bid or AuctionAnalytics.

## Function to check registered user

```javascript
// Public function to check the registration of users (public address)
function isRegistered(address _publicAddress) public view returns (uint256[2] memory) {
	uint256[2] memory result = [uint256(0), uint256(0)];
	for(uint i = 0; i < uId; i++) {
		if(_publicAddress == users[i].publicAddress) {
			result[0] = 1;
			result[1] = i;
			return result;
		}
	}
	return result;
}
```

This function takes the public address as its argument and returns an integer array with 2 elements - **isRegistered** at index **0** and **userId** at index 1. If **0th** index is 1 then the user exists and vice-versa. The **1st index** represents the **userId** of the user. This function iterates over the mapping **users** to check if the required public address exists.

## Auction analytics

We have created a mapping for storing analytics like the latest bid, highest bid, and lowest bid for each auction. This mapping will map **auctionId** to **AuctionAnalytic** struct. When a new auction is created, we initialize its corresponding entry in the **AuctionAnalytics** map.

```javascript
// Private function to update auction analytics after the new bids
function updateAucionAnalytics(uint _aId, uint _latestBid) private {
	auctionAnalytics[_aId].latestBid = _latestBid;
	auctionAnalytics[_aId].auctionBidId = auctions[_aId].auctionBidId;
	if(_latestBid < auctionAnalytics[_aId].lowestBid) {
		auctionAnalytics[_aId].lowestBid = _latestBid;
	}
	if(_latestBid > auctionAnalytics[_aId].highestBid) {
		auctionAnalytics[_aId].highestBid = _latestBid;
	}
}
```

The auction analytics need to be updated every time there is a new bid. This function is called whenever a bid is created. It takes an **auctionId** and the **latest bid amount** as its two arguments, and updates the analytics corresponding to the auction.

The remaining functions are self explanatory, but have been well commented for readers to understand.

# Creating Truffle migrations

Create the file `Migration.sol` inside of the `contracts` directory and add the following:

```javascript
// SPDX-License-Identifier: MIT
pragma solidity >=0.4.22 <0.9.0;

contract Migrations {
	address public owner = msg.sender;
	uint public last_completed_migration;

	modifier restricted() {
		require(
			msg.sender == owner,
			"This function is restricted to the contract's owner"
		);
		_;
	}

	function setCompleted(uint completed) public restricted {
		last_completed_migration = completed;
	}
}
```

This `Migration.sol` smart contract manages the deployment of other contracts that we want to migrate to Avalanche.

Next, create a new file in the `migrations` directory named `2_deploy_contracts.js`, and add the following block of code. This handles deploying the `Auction` smart contract to the blockchain.

```javascript
const AuctionManager = artifacts.require("./Auction.sol");

module.exports = function (deployer) {
  deployer.deploy(AuctionManager);
};
```

# Compiling contracts with Truffle

If we have altered the code within our Solidity source files or made new ones \(like `Auction.sol`\), we need to run `truffle compile` in the terminal, from inside the project root directory.

The expected output would look similar:

```text
Compiling your contracts...
===========================
> Compiling ./contracts/Auction.sol
> Compiling ./contracts/Migrations.sol

> Artifacts written to /home/guest/blockchain/client/build/contracts
> Compiled successfully using:
	 - solc: 0.8.0+commit.c7dfd78e.Emscripten.clang
```

The compiled smart contracts are written as JSON files in the `/src/build/contracts` directory. These are the stored ABI and other necessary metadata - the artifacts.

{% hint style="info" %}
**ABI** refers to Application Binary Interface, which is a standard for interacting with the smart contracts from outside the blockchain as well as contract-to-contract interaction. Please refer to the [Solidity documentation about the ABI spec](https://docs.soliditylang.org/en/latest/abi-spec.html) to learn more.
{% endhint %}

# Run migrations on the C-Chain

During the deployment of the smart contract to the C-Chain, there is a required deployment cost. This can be seen inside `truffle-config.js`. The HDWallet Provider will help us in deploying on Fuji C-Chain and the deployment cost will be supplied by the account whose mnemonic has been stored in the `.env` file. Therefore, we need to fund the account before we are able to deploy.

## Fund your account

We need funds in our C-Chain address, as smart contracts on Avalanche are deployed on the C-Chain (Contract-Chain). This address can easily be found on the [Avalanche Wallet](https://wallet.avax.network) dashboard. Avalanche network has 3 chains: X-Chain, P-Chain, and C-Chain. The address of all these chains can be found by switching tabs at the bottom of the division, where there is a QR code. So, switch to C-Chain and copy the address. Now fund your account using the [Fuji testnet faucet](https://faucet.avax-test.network/) by pasting your C-Chain address in the input field. Refer to the image below, to identify the address section.

![](https://imgur.com/N0YQq1L.png)

{% hint style="info" %}
You'll need to send at least `135422040` **nAVAX** to the account to cover the cost of contract deployments. Here **nAVAX** refers to nano-AVAX i.e. one billionth of an **AVAX**, or simply 1 **nAVAX** = (1/1000,000,000) **AVAX**. Funding through the Fuji faucet will give you enough `AVAX` to run multiple deployments and transactions on the testnet.
{% endhint %}

## Run Migrations

Everything is in place to run the Truffle migrations and now we can easily deploy the `Auction` contract with the command:

```text
truffle migrate --network fuji
```

For rapid development and testing, we can also deploy our contracts on a local network by using [Ganache](https://www.trufflesuite.com/ganache) (Truffle's local blockchain simulation) with the command:

```text
truffle migrate --network development
```

On successful execution of either of the above commands, we should expect to see similar output:

```text
Starting migrations...
======================
> Network name:    'fuji'
> Network id:      1
> Block gas limit: 8000000 (0x7a1200)


1_initial_migration.js
======================

	 Deploying 'Migrations'
	 ----------------------
	 > transaction hash:    0x094a9c0f12ff3158bcb40e266859cb4f34a274ea492707f673b93790af40e9e9
	 > Blocks: 0            Seconds: 0
	 > contract address:    0x0b1f00d0Af6d5c864f86E6b96216e0a2Da111055
	 > block number:        40
	 > block timestamp:     1620393171
	 > account:             0x80599dd7F8c5426096FD189dcC6C40f47e8e3714
	 > balance:             39.71499696
	 > gas used:            173118 (0x2a43e)
	 > gas price:           20 gwei
	 > value sent:          0 ETH
	 > total cost:          0.00346236 ETH


	 > Saving migration to chain.
	 > Saving artifacts
	 -------------------------------------
	 > Total cost:          0.00346236 ETH


2_deploy_contracts.js
=====================

	 Deploying 'AuctionManager'
	 ------------------------
	 > transaction hash:    0xbeb13fc6bbee250eea9151faf02bfe247ec497294acc84c9b8319ed609ced086
	 > Blocks: 0            Seconds: 0
	 > contract address:    0xf30D372A6911CCF6BBa1e84c3CEd51cC0F3D7769
	 > block number:        42
	 > block timestamp:     1620393172
	 > account:             0x80599dd7F8c5426096FD189dcC6C40f47e8e3714
	 > balance:             39.69235442
	 > gas used:            1090212 (0x10a2a4)
	 > gas price:           20 gwei
	 > value sent:          0 ETH
	 > total cost:          0.02180424 ETH


	 > Saving migration to chain.
	 > Saving artifacts
	 -------------------------------------
	 > Total cost:          0.02180424 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.0252666 ETH
```

If you have not created an account on the C-Chain, you'll see this error:

```text
Error: Expected parameter 'from' not passed to function.
```

If you have not funded the account, you'll see this error:

```text
Error:  *** Deployment Unsuccessful***

"Migrations" could not deploy due to insufficient funds
	 * Account:  0x090172CD36e9f4906Af17B2C36D662E69f162282
	 * Balance:  0 wei
	 * Message:  sender doesn't have enough funds to send tx. The upfront cost is: 1410000000000000000 and the sender's account only has: 0
	 * Try:
			+ Using an adequately funded account
```

The information and ABI of the deployed contract are present in the `src/build/contracts` directory as `Auction.json`.

# Building the user interface

Our blockchain code, which will act as the backend for this application, is deployed on the blockchain and now we can make the client-side for interacting with the contracts. We will be using Trufflesuite's **Drizzle** library for connecting our web app with the blockchain. Drizzle makes the integration process very easy and scalable. It also provides a mechanism to **cache** a particular contract-call, so that we can get real-time updates of any changes to the data on the blockchain.

We will be using React's context API to facilitate this integration. The context API makes the use of variables which are declared in the parent component very easy to access in the child components.

It is based upon the **Provider** and **Consumer** concept. The **Provider** component contains necessary logic and variables that need to be passed along. Then this Provider component is wrapped around the components which want to access its variables. Every child component can access these variables. But in order to access them, we use a **Consumer** API. This API will return the variables from the **Provider** component (only when called from its child components). Look at the code below to understand what is happening.

In the `drizzleContext.js` file, **DrizzleProvider** is the provider component and **useDrizzleContext** is the consumer function. Look at the return statements of these functions. One is returning the Context Provider (provider), and the other is returning the values of the Context itself (consumer).

## Drizzle Option component

Create a file `drizzleOption.js` inside the `drizzle-auction/client/src/` directory and paste the following code:

```javascript
import AuctionManager from "./build/contracts/AuctionManager.json";

const drizzleOptions = {
  contracts: [AuctionManager],
};

export default drizzleOptions;
```

The `drizzleOptions` constant contains the configuration like contracts we want to deploy, our custom web3 provider, smart contract events, etc. Here we are just instantiating the `AuctionManager` smart contract.

## Index component

Inside the file `index.js` in the `src` directory, add the following code:

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

import { Drizzle } from "@drizzle/store";
import drizzleOptions from "./drizzleOptions";
import { DrizzleProvider } from "./drizzleContext";

const drizzle = new Drizzle(drizzleOptions);

ReactDOM.render(
  <DrizzleProvider drizzle={drizzle}>
    <App />
  </DrizzleProvider>,
  document.getElementById("root")
);
```

Importing `Drizzle` from the `@drizzle/store` module will help in instantiating drizzle according to our drizzleOptions. The following line is responsible for this action:

```javascript
const drizzle = new Drizzle(drizzleOptions);
```

Then we wrap the `App` component inside the `DrizzleProvider` so that we can use extracted variables (see drizzleContext.js) inside `App`. We pass the `drizzle` object to the provider component because it will be required to extract other information from it.

## Drizzle Context

Create a file `drizzleContext.js` inside the `drizzle-auction/client/src/` directory with the following code:

```javascript
import React, { createContext, useContext, useState } from "react";

const Context = createContext();

export function DrizzleProvider({ drizzle, children }) {
  const [drizzleVariables, setDrizzleVariables] = useState({
    initialized: false,
    state: null,
    web3: null,
    accounts: null,
    AuctionManager: null,
    subscriber: null,
  });
  const unsubscribe = drizzle.store.subscribe(() => {
    const drizzleState = drizzle.store.getState();
    if (drizzleState.drizzleStatus.initialized) {
      const { web3, accounts } = drizzleState;
      const AuctionManager = drizzle.contracts.AuctionManager.methods;
      const subscriber = drizzleState.contracts.AuctionManager;
      setDrizzleVariables({
        state: drizzleState,
        web3,
        accounts,
        AuctionManager,
        subscriber,
        initialized: true,
      });
    }
  });
  drizzleVariables.initialized && unsubscribe();

  return (
    <Context.Provider value={{ drizzle, drizzleVariables }}>
      {children}
    </Context.Provider>
  );
}

export function useDrizzleContext() {
  const context = useContext(Context);
  return context;
}
```

The `DrizzleProvider` function takes `drizzle` as its argument and extracts other information like whether drizzle contracts are initialized or not, web3 info, account info, deployed contract's instance, etc. We need to **subscribe** to the drizzle's **store** for this information because the data is not fetched at once, and since we do not know when will we get the data, we subscribe to the store (where the data resides). Once the drizzle is initialized with our contract data, we **unsubscribe** from the store, so that it will not re-render infinitely!

### Drizzle state

```javascript
const drizzleState = drizzle.store.getState();
```

This variable holds the state of the store. Cached calls are those contract calls for which we want real-time data from the blockchain. Whenever there is some change in our data on the blockchain, it gets notified in the drizzle state variable of the store.

### AuctionManager

```javascript
const AuctionManager = drizzle.contracts.AuctionManager.methods;
```

`drizzle.contracts` is an object containing instances of all the deployed contracts which are added to drizzle (in drizzleOptions or manually). We are simply storing all the **methods** of this contract instance so that whenever we want to call functions or public identifiers from this contract, we can simply use `AuctionManager.method_name().call()`.

## App component

Now open `App.js` inside the `drizzle-auction/client/src/` directory and add the following code:

```javascript
import Auction from "./Auction";
import { useDrizzleContext } from "./drizzleContext";

function App() {
  const { drizzleVariables } = useDrizzleContext();

  if (!drizzleVariables.initialized) {
    return "Loading...";
  } else {
    return <Auction />;
  }
}

export default App;
```

`drizzleVariables.initialized` would ensure that, `Loading...` state is visible until Drizzle is ready for interaction.

## Auction Component

Create a file `Auction.js` inside the `drizzle-auction/client/src/` directory and add the following code. This component deals with the entry-point of our application, where all the data like `userInfo`, `AuctionLists`, `AuctionDetails` etc. get generated.

```javascript
import React, { useState, useEffect } from "react";
import { useDrizzleContext } from "./drizzleContext";
import FetchAuctions from "./AuctionList";
import CreateAuction from "./CreateAuction";

function Auction() {
  // Importing drizzle variables from drizzle context
  const { drizzleVariables } = useDrizzleContext();
  const { AuctionManager, subscriber, accounts } = drizzleVariables;

  // Setting up cache keys corresponding to cache calls
  const [cacheKeys, setCacheKey] = useState({
    uId: null,
    aId: null,
    showAuctions: null,
    isRegistered: null,
    auctionAnalytics: [null],
  });
  const [auctionAnalyticsCacheKey, setAuctionAnalyticsCacheKey] =
    useState(null);

  // Setting up cache calls for required functions
  const cacheCalls = {
    isRegistered: subscriber?.isRegistered[cacheKeys?.isRegistered]?.value,
    user: subscriber?.users[cacheKeys?.uId]?.value,
    aId: subscriber?.aId[cacheKeys?.aId]?.value,
    showAuctions: subscriber?.showAuctions[cacheKeys?.showAuctions]?.value,
    auctionAnalytics: [],
  };

  for (var i = 0; i < cacheCalls.aId; i++) {
    cacheCalls.auctionAnalytics.push(
      subscriber?.auctionAnalytics[auctionAnalyticsCacheKey[i]]?.value
    );
  }

  const [isRegistered, setIsRegistered] = useState(false);
  const [userInfo, setUserInfo] = useState({
    id: null,
    name: null,
  });

  // Initializing cache keys
  useEffect(() => {
    const _auctionCacheKey = AuctionManager?.showAuctions?.cacheCall();
    const _aIdCacheKey = AuctionManager?.aId?.cacheCall();
    const _isRegistered = AuctionManager?.isRegistered?.cacheCall(accounts[0]);
    setCacheKey({
      ...cacheKeys,
      showAuctions: _auctionCacheKey,
      aId: _aIdCacheKey,
      isRegistered: _isRegistered,
    });
  }, []);

  useEffect(() => {
    var _auctionAnalyticsCacheKey = [];
    for (var i = 0; i < cacheCalls.aId; i++) {
      _auctionAnalyticsCacheKey.push(
        AuctionManager?.auctionAnalytics?.cacheCall(i)
      );
    }
    setAuctionAnalyticsCacheKey(_auctionAnalyticsCacheKey);
  }, [cacheCalls.aId]);

  useEffect(() => {
    if (
      cacheCalls.isRegistered !== undefined &&
      cacheCalls.isRegistered[0] == 1
    ) {
      setIsRegistered(true);
      (async () => {
        const userInfo = await AuctionManager.users(
          cacheCalls.isRegistered[1]
        ).call();
        setUserInfo({
          id: userInfo.userId,
          name: userInfo.name,
        });
      })();
    } else {
      setIsRegistered(false);
    }
  }, [cacheCalls.isRegistered]);

  const createUser = async (name) => {
    await AuctionManager?.createUser(name)?.send({ from: accounts[0] });
  };

  const [userName, setUserName] = useState("");

  const handleUserNameChange = (event) => {
    setUserName(event.target.value);
  };

  const submitLogin = (event) => {
    event.preventDefault();
    createUser(userName);
  };

  const UserInfo = () => {
    return (
      <div>
        <label style={{ color: "red" }}>ID: </label> {userInfo.id}
        <label style={{ marginLeft: "50px", color: "green" }}>
          Name:{" "}
        </label> {userInfo.name} <br />
        <br />
      </div>
    );
  };

  return (
    <div>
      <h1>Auctions</h1>
      {isRegistered ? (
        <>
          <UserInfo />
          <FetchAuctions cacheCalls={cacheCalls} userInfo={userInfo} /> <br />
          <br />
          <CreateAuction />
        </>
      ) : (
        <form onSubmit={submitLogin}>
          <font color="red" font="2">
            This address is not yet registered!
          </font>
          <br />
          <br />
          <label>Address: </label>
          <input disabled value={accounts[0]} />
          <br />
          <br />
          <label>Name: </label>
          <input
            key="1"
            value={userName}
            required
            onChange={handleUserNameChange}
            placeholder="Enter your name"
          />
          <br />
          <br />
          <input type="submit" value="Register" />
        </form>
      )}
    </div>
  );
}

export default Auction;
```

In order to keep our data fresh from the blockchain, Drizzle uses the caching mechanism. On our behalf, Drizzle keeps track of every change on the blockchain. If there is any transaction involving our smart contracts, then it will notify our dApp.

We need to define the calls which we want to monitor. Caching a particular method will provide cache keys (hash) to us. Each cached method is associated with a particular unique hash. Using this key, we can get live data from the blockchain. The component will re-render anytime there is some new value associated with this call.

For example, in the above code, we used the following cache keys

```javascript
const [cacheKeys, setCacheKey] = useState({
  uId: null,
  aId: null,
  showAuctions: null,
  isRegistered: null,
  auctionAnalytics: [null],
});
```

Suppose we want to cache the `isRegistered` method. This can be done using:

```javascript
const _isRegistered = AuctionManager?.isRegistered?.cacheCall(accounts[0]);
setCacheKey({
  isRegistered: _isRegistered,
});
```

Once a method is cached, the Drizzle `store` creates a **key-value pair** representing hash-key and the real-time data associated with this call. In the above program, this data is accessed using the `subscriber` variable:

```javascript
const realTimeIsRegistered =
  subscriber?.isRegistered[cacheKeys?.isRegistered]?.value;
```

In this component, we made a simple object of cached call variables named `cacheCall`, which implements the above code snippet. The cached version of `isRegistered` can be accessed as `cacheCalls.isRegistered`.

## Auction List

Create a file `Auctionlist.js` inside the `drizzle-auction/client/src/` directory, adding the following code. This component deals with the management of the auction like creating a new bid, displaying the real-time auction analytics, etc. All the data is passed by its parent component i.e. `Auction.js` which manages the cache keys and calls.

```javascript
import React, { useState } from "react";
import { useDrizzleContext } from "./drizzleContext";

function FetchAuctions({ cacheCalls, userInfo }) {
  const { drizzleVariables } = useDrizzleContext();
  const { AuctionManager, accounts } = drizzleVariables;

  const [bidPrices, setBidPrices] = useState(new Map([]));

  const createBid = async (id, bidPrice) => {
    await AuctionManager?.createBid(id, bidPrice).send({ from: accounts[0] });
    clearBidPriceInput(id);
  };

  const submitNewBid = (event, id) => {
    event.preventDefault();
    createBid(id, bidPrices.get(id));
  };

  const handleBidPriceChange = (event, id) => {
    let _bidPrices = bidPrices;
    _bidPrices.set(id, event.target.value);
    setBidPrices(_bidPrices);
  };

  const clearBidPriceInput = (id) => {
    let _bidPrices = bidPrices;
    _bidPrices[id] = "";
    setBidPrices(_bidPrices);
  };

  const getAuctionAnalytics = () => {
    var auctionAnalytics = [];
    for (var i = 0; i < cacheCalls.aId; i++) {
      auctionAnalytics.push(cacheCalls.auctionAnalytics[i]);
    }
    return auctionAnalytics;
  };

  const allAuctions = cacheCalls.showAuctions;
  const auctionAnalytics = getAuctionAnalytics();
  return (
    <table border="1" style={{ maxWidth: "800px", width: "90%" }}>
      <tr>
        <td>Auction ID</td>
        <td>Auction Details</td>
        <td>Minimum Price</td>
        <td>Bid</td>
      </tr>
      {allAuctions !== undefined &&
        allAuctions.map((auction, index) => (
          <tr>
            <td>{auction.auctionId}</td>
            <td>
              <b>
                {auction.name}{" "}
                <font size="2" color="green">
                  {auction.userId == userInfo.id && "(created by you)"}
                </font>
              </b>
              <br />
              <font size="2">{auction.description}</font>
              <br />
              <tr>
                <td>Total Bids</td>
                <td>Latest Bid</td>
                <td>Highest Bid</td>
                <td>Lowest Bid</td>
              </tr>
              <tr>
                <td>{auctionAnalytics[index]?.auctionBidId}</td>
                <td>₹{auctionAnalytics[index]?.latestBid}</td>
                <td>₹{auctionAnalytics[index]?.highestBid}</td>
                <td>
                  ₹
                  {auctionAnalytics[index]?.auctionBidId == 0
                    ? 0
                    : auctionAnalytics[index]?.lowestBid}
                </td>
              </tr>
            </td>
            <td>₹{auction.msp}</td>
            <td>
              <form
                onSubmit={(event) => submitNewBid(event, auction.auctionId)}
                style={{ margin: "10px" }}
              >
                <input
                  required
                  type="number"
                  min={auction.msp}
                  onChange={(event) =>
                    handleBidPriceChange(event, auction.auctionId)
                  }
                  placeholder="Enter your bid price"
                />
                <br />
                <br />
                <input type="submit" value="Make Bid" />
                <br />
                <br />
              </form>
            </td>
          </tr>
        ))}
    </table>
  );
}

export default FetchAuctions;
```

## Creating new Auctions

Create a file `CreateAuction.js` inside the `drizzle-auction/client/src/` directory and add the following code. This component deals with creation of new Auctions, by submitting transactions on the network.

```javascript
import React, { useState } from "react";
import { useDrizzleContext } from "./drizzleContext";

function CreateAuction() {
  // Importing drizzle variables from drizzle context
  const { drizzleVariables } = useDrizzleContext();
  const { AuctionManager, accounts } = drizzleVariables;

  const createAuction = async ({ title, description, msp }) => {
    await AuctionManager?.createAuction(title, description, msp)?.send({
      from: accounts[0],
    });
    setAuctionDetails({
      title: "",
      description: "",
      msp: "",
    });
  };

  const [auctionDetails, setAuctionDetails] = useState({
    title: "",
    description: "",
    msp: "",
  });

  const handleAuctionTitleChange = (event) => {
    setAuctionDetails({
      ...auctionDetails,
      title: event.target.value,
    });
  };

  const handleAuctionDescriptionChange = (event) => {
    setAuctionDetails({
      ...auctionDetails,
      description: event.target.value,
    });
  };

  const handleAuctionMspChange = (event) => {
    setAuctionDetails({
      ...auctionDetails,
      msp: event.target.value,
    });
  };

  const submitNewAuction = (event) => {
    event.preventDefault();
    createAuction(auctionDetails);
  };

  return (
    <form
      onSubmit={submitNewAuction}
      style={{ border: "1px black solid", maxWidth: "400px", padding: "10px" }}
    >
      <label>Title: </label>
      <br />
      <input value={auctionDetails.title} onChange={handleAuctionTitleChange} />
      <br />
      <br />
      <label>Description: </label>
      <br />
      <textarea
        rows="4"
        value={auctionDetails.description}
        onChange={handleAuctionDescriptionChange}
      />
      <br />
      <br />
      <label>MSP: </label>
      <br />
      <input value={auctionDetails.msp} onChange={handleAuctionMspChange} />
      <br />
      <br />
      <input type="submit" value="Create Auction" />
    </form>
  );
}

export default CreateAuction;
```

Now go to the project root directory, i.e. `avalanche-voting`, and run the command `npm start`. The React development server will start automatically. Visit http://localhost:3000 in a web browser to interact with the dApp frontend.

Don't forget to set up Metamask with the Fuji testnet and also fund the account with Fuji C-Chain test tokens to be able to vote. In the Metamask extension, add a custom RPC endpoint by clicking the network dropdown in the centre of the extension. Fill in the details as shown in the table below.

| Info               | Value          |
| :----------------- | :------------- |
| Network Name       | Avalanche Fuji |
| New RPC URL        |                |
| Chain ID           | 43113          |
| Currency Symbol    | AVAX-C         |
| Block Explorer URL |                |

# Conclusion

We have successfully built a dApp through which we can organize auctions, bid in them and declare results, with both frontend and smart contracts. We have used the Drizzle library from Trufflesuite for integrating our frontend with the blockchain and to keep our data updated in real-time.

![](https://imgur.com/QAtm67Y.gif)

# Next Steps

Our dApp currently has very minimalistic designs. We can use Consensys' Rimble UI library for adding modals for each transaction, add links to drip Avalanche's test tokens etc. which can help users to navigate through our dApp.

# About the authors

This tutorial was created by [Raj Ranjan](https://www.linkedin.com/in/iamrajranjan), [Rakesh Singh](https://github.com/iamrakeshsingh) and [Naman Gera](https://www.linkedin.com/in/namannimmo).

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!