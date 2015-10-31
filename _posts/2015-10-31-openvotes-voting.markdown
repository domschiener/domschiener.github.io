---
layout: post
title:  "PublicVotes: Ethereum-based Voting Application"
date:   2015-10-31 19:34:14
categories: projects
---


As part of my research for Reposium, I have started working on a [simple voting application](http://publicvotes.org) built with Meteor that utilizes the Ethereum Blockchain to create a provably fair and transparent voting system. This is one of a few applications I want to create as part of my research (decentralized decision making or liquid democracy is most likely next). Before explaining the underlying structure of this application, let me explain which problems this application solves and which it doesn’t.


## What this is and isn’t

What PublicVotes essentially is, is an easy to use, provably fair voting application. All votes of participants are recorded (by proxy) into the Blockchain for the world to verify. The application is not fully decentralized, since my design goal from the beginning was to create an application that is easy to use for people outside the Ethereum space. People should be able to participate in the voting process without even knowing what the underlying protocols are — which has lead to some functionality moving to the server (i.e. centralization). I will gradually work on making this more trustless. Right now you should treat PublicVotes as an **application “empowered by Ethereum”**, instead of a DApp powered by Ethereum.


# How it works


The entire platform is built on Meteor, there is currently one smart contract coded in Solidity that is used for placing a poll into the Blockchain and for casting the votes. For display purposes on the website and making it easy to vote, I’m also storing poll information, votes and the related Ethereum accounts in MongoDB collections.
There are basically 2 processes that make up the system. The first is the poll creation, the second is actual voting.

## Poll Creation

Anyone with a small amount of Ether can create a poll. At PublicVotes, the creator of the poll pays for the creation of the poll and for all votes. This way anyone can vote by simply clicking their preferred option, without even knowing what the underlying technology (Ethereum) is.

A poll consists of the following information:

* Title (required): Mostly a question that indicates what the users are voting about. Maxlength: 48 chars.
* Description (not required): A more comprehensive description that explains to the users what the vote is exactly about. Maxlength: 145 chars.
* Options (min 2 required): The actual voting options for your poll. Minimum is 2, max is 10 options. Maxlength: 20 chars.

**Settings:**

* Public Poll: The user can choose if the poll should be public or not. If the poll is private, only people with the link can participate in the vote.
* Multioption (coming soon): Makes it possible for users to choose more than 1 poll option.
* Vote Limit: Limits the number of people that can participate in the poll. (not required).
* Time Limit: A timelimit for the poll. Right now this is a requirement as the account will eventually run out of Ether. But once we have setup a faucet I will remove the timelimit.

Once the creator has entered this information, he/she is required to send a specified amount (0.2 Ether to be exact) of Ether to an address. All of the accounts are generated on the client, thanks to silentcicero’s ethereumjs library. This account is then stored in a local MongoDB collection and will be used for all future votes.

Once the Ether have been received at the specified address, the poll is ready to go live and be deployed onto the Ethereum Blockchain. This is the smart contract that is deployed onto the Blockchain: [Github Link](https://github.com/domschiener/publicvotes/blob/master/contracts/contract.sol) . Once the contract has been mined, the poll will go live and people can start voting.


## Casting a Vote

Anyone with a link can participate in the poll and cast their vote. Casting a vote is as simple as clicking a button. The application does most of the rest, such as preparing and signing the transaction which will be sent to the smart contract. Once a vote has been received, the smart contract will record the vote into the Blockchain’s event log.

After voting, the user is redirected to /voted where there are statistics about the poll and the people who have voted. The table at the bottom of the page has been purely generated through filters that get the event logs of the Ethereum Blockchain.


# Where to go from here

This is a Minimum Viable Product and the system currently has many flaws. The major flaw being the IP-based redirect system. I will implement OAuth over the coming days so that users can participate in the voting process through social media accounts.

It would be very interesting to work on a sybil-attack resistant system, but it seems that so far there is no viable solution to the problem at hand that offers both security and ease of use.
Overall though, I will work on more improvements over the coming weeks and also on a follow-up application that will take this voting application to the next level (mostly based on Liquid Democracy concepts).
