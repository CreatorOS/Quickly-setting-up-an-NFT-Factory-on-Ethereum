# Quickly setting up an NFT Factory on Ethereum
This is the first quest in a sequence of quests that will give you a primer on things that are possible with web3. We’ll quickly checkin to see what is possible using Ethereum and Solidity before we dive deeper.

Here we’ll be using Austin Griffith’s project scaffold eth. Scaffold eth is a great way to get things to work quickly. By getting things to work quickly, we’ll get a feel of what blockchain development feels like and we can build on top of it using the follow on quests. In this series we’ll look at the things that are possible using the technology and open up your imagination to what might be the killer app in crypto. We’re still waiting for one.
## setting up scaffold eth to create our own NFT
Scaffold eth is an open sourced project on github where people have contributed to over 200 projects that we can look at to get started with learning about Ethereum & dapps. 

It comes with a UI in React and a toy blockchain implementation called hardhat. We will be writing some code to deploy an application to the blockchain and see it in action.

So first up, let’s clone the github repository. 

```

git clone [https://github.com/austintgriffith/scaffold-eth.git](https://github.com/austintgriffith/scaffold-eth.git)

cd scaffold-eth

```

This repository has 100s of branches, each representing one project. Let’s jump to the project we’re going to be following - that is to create an NFT and send it to our friends. 

```

git checkout simple-nft-example

```

Here you’ll see all the files here under a directory named packages.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/4b17fe1a-97e2-4549-942a-9ec4c804078c.jpg)

We’ll be concerned with hardhat and react-app for the first few tutorials. Hardhat is the package that contains all the files pertaining to interacting with the blockchain - the backend if you will. React app is where all the front end code resides.

Let’s quickly set things into action. Open 3 terminals. If you’re on windows, please use a WSL2 terminal. 

```

~/scaffold-eth $ yarn install

~/scaffold-eth $ yarn start

```

This will install all the dependencies including react and the UI components created by Austin and others. We’ve also started the React server - so, now we can start accessing the UI from our browser at localhost:3000.

But the this UI is just the front end. We’ve not yet booted up our backend.

```

~/scaffold-eth $ yarn chain

```

This starts a toy version of a blockchain on your own computer that is identical to the Ethereum blockchain. But we’ve only spinned up the blockchain, we’ve not deployed any program to it. Spinning up a blockchain is the equivalent of starting an EC2 instance on AWS. Now we need to deploy the actual code.

To deploy, 

```

~/scaffold-eth $ yarn deploy

```

This will compile the programs that have already been written in this repository and deploy it to the blockchain we started in the previous step. The beauty of scaffold-eth is that all of the heavy lifting happens under the hood to get something running quickly. But at the same time we can go and modify all the code and customize our application.

Great, with that let’s checkout the app at localhost:3000
## Deconstructing the d-app
When you open the application on your browser, you see many things on the screen. You don’t have any NFTs yet, so “Your Collectibles” is empty. An NFT is a collectible. 

But before we get into that, we’ll look at the various moving parts on this screen. 

First up on the top right you’d see “0x…”

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/1858f1cd-e25f-4b61-b1cf-73c2e85d21f9.jpg)

This is your account. Every account has two things. Firstly the address, a hexadecimal string, which identifies the account. Think of this as an account number. Second, an amount that is stored in this account. 

In the above screenshot 0x1ABf is the abbreviated account address. Copy the account address using the copy button next to it and paste it to see an address in all its glory.

The account can hold money in ETH. Currently there is no money in the account. 

To get money into the account, we must buy some ETH. But since we’re using a toy blockchain, we can use what is called a faucet. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/2e20626e-3a9a-4465-afc4-64d906968844.jpg)

Tapping the “grab funds from the faucet” will give this account some money in toy ETH. 

It also reads “localhost” right below the account balance - suggesting that this is a toy Ethereum deployment. This app is interacting with the blockchain we `deploy`ed on one of our terminals.

We’ll ignore the `connect` button for now.
## Creating NFTs
Before NFTs can be transferred from one person to another, they need to be brought into existence. The process of bringing an NFT into existence is called `minting`.

Let us look at how we can bring NFTs into existence. 

For this, jump to the code at `scaffold-eth/packages/hardhat/mint.js`. This is the code that will create new NFTs. Let us go over this line by line to see how. 

In the function `main()`, the first variable is the `toAddress`. 

When an NFT is created it has a default owner. `toAddress` will allocate the NFT to that owner when it is created. 

We’ll come back to the following 2 lines in just a bit

```

  const { deployer } = await getNamedAccounts();

  const yourCollectible = await ethers.getContract("YourCollectible", deployer);

```

But let’s look at how an NFT is created first. 

Every NFT consists of a name, a description and an image. This is what is displayed on the NFT marketplaces like OpenSea. [https://opensea.io/assets/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7/164](https://opensea.io/assets/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7/164)

There are also `external_url` which pertains to where the user can find more information about this NFT.

Lastly the attributes define what properties this NFT has. These attributes are however, optional.

Now that we’ve defined what our NFT looks like by defining the name, description, image and attributes, we can now publish this.

```

 const uploaded = await ipfs.add(JSON.stringify(buffalo))

```

This statement uploads the NFT object to a distributed cloud storage called IPFS. IPFS is like amazon s3, but free and decentralized.

Lastly, now that the content is ready, we can actually mint the NFT. 

```

  await yourCollectible.mintItem(toAddress,uploaded.path,{gasLimit:400000})

```

But to understand this we need to revisit something we skipped over earlier
## Minting NFTs
We had earlier skipped over the following 2 lines : 

```

  const { deployer } = await getNamedAccounts();

  const yourCollectible = await ethers.getContract("YourCollectible", deployer);

```

`getNamedAccounts()` looks for a hardhat config file and looks for configured accounts to deploy using. We have to deploy a program called a smart contract before we can start minting the NFTs. This is a program that will run on your local blockchain. 

This config resides in `scaffold-eth/packages/hardhat/hardhat.config.js`

Line 294 tells hardhat that there is an account called deployer. Each program that is deployed to the blockchain is deployed by some account. It’s like a user deploying to AWS. 

`default : 0` tells the blockchain to use a default account that hardhat has already setup.

We’ll define a program that will be deployed to the blockchain. The filename is `YourCollectible.sol` 

You can find this in `scaffold-eth/hardhat/contracts/YourCollectible.sol`

You don’t really need to understand too much here, but you just need to know that there exists a function in this sol file (aka contract) called `mintToken()`.

This `mintToken` function enables us to create NFTs on the blockchain. 

So, first, we need to deploy this smart contract to the blockchain and then we’ll be able to call the `mintToken` function.

```

  const yourCollectible = await ethers.getContract("YourCollectible", deployer);

```

Will deploy the contract on the blockchain and set the owner as the account `deployer`

```

  const uploaded = await ipfs.add(JSON.stringify(buffalo))

  console.log("Minting buffalo with IPFS hash ("\+uploaded.path\+")")

  await yourCollectible.mintItem(toAddress,uploaded.path,{gasLimit:400000})

```

Lastly once the NFT data has been uploaded to the IPFS decentralized cloud storage, we can mint the NFT. The NFT (token) will be sent to `toAddress` and can be transferred or sold to anyone else thereafter.
## Selling NFTs
Lastly let’s run minting by calling 

```

~/scaffold-eth $ yarn mint

```

This will call the `main` function in `mint.js`

It will create all the NFTs (buffalo, zebra, godzilla etc) and assign it to you. 

Go back to your browser where the app was running and refresh the page. You should see all the NFTs in your “YourCollectibles” tab.

You can transfer these NFTs to any other account address you want :)

At any point of time only one person can be the owner of this NFT. This NFT is a great Programmable Proof Of Ownership (PPO). You can ask your friend to pay you before you transfer the NFT to them.

To do so, open your app (localhost:3000) on a private browser. On the top right you’ll notice that this is a different account address indicating that this is mimicking a different user. You’ll also notice that this user doesn’t have any NFTs yet. 

To send the NFT from your main account to the private-browser-account, copy the address of the account from the private browser. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/3da814b5-7c85-46cb-b0b7-8dcb686eba5d.jpg)

Now open the app in your regular browser window where all the NFTs are listed. Paste the address against the NFT you want to transfer and hit `transfer`. Go to your private browser and see  the NFT under YourCollectibles.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/af05b58b-5f53-4bff-acdf-6f9d0d6c5509.jpg)

What just happened here is we transfered ownership. Every NFT has an owner - exactly one. When we minted the token, it was automatically assigned the owner as defined by `deployer`. We can transfer the ownership of the NFT from one person (or one account) to another. There is no way an NFT is owned by multiple people at the same time. This is a neat property that enables applications to be built on top of the Programmable Proof of ownership - like entry to community based on NFT ownership, ticketing, collectibles (think collecting trading cards) and many more!
## Next steps
Explore the file structure a little more and get familiar with it. We’ll be using a lot of Scaffold eth in our learning process from now on. 

It’s ok if you don’t know enough to create this entire project from scratch - you need to understand the basic terms we’ve introduced in this quest. 

*Try adding a new animal NFT and mint it to yourself.*

In the next quest on, we’ll build applications ourselves from scratch!