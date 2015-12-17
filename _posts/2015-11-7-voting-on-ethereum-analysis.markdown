---
layout: post
title:  "Voting on the Ethereum Blockchain: An Analysis"
date:   2015-11-7 11:34:14
categories: research
---


The main reason why I’ve created [PublicVotes](http://publicvotes.org) (apart from the intellectual challenge) is to determine the feasibility of utilizing Ethereum as a decision making tool for organizations. After a little under 2 weeks of operation I have acquired some data to draw early conclusions and answer the questions I had prior to this experiment. This will be an attempt at offering a more in-depth analysis so that current and future voting applications that utilize Ethereum or other Blockchains can benefit from it.

**TL;DR:** Using Ethereum as a decision making tool costs money and all entries are public. I do not think that Ethereum should be used as a platform (or rather a tool) for the constant decision-making inside organizations, mainly because the advantages offered are not what organizations require. BUT, for decision-making that requires trust and a publicly verifiable and transparent system (such as elections of any kind), using Ethereum makes a lot of sense.  The 2010 General Election in the UK cost some $127Million. In a hypothetical scenario on Ethereum, it would cost $275,546.776 to register the same amount of votes. At the present time, on PublicVotes it costs about $2.4 to cast some 1368 votes (including polls). On average, it costs $0.0015 to cast a vote and $0.024 to create a poll.
<br>

# What actually costs money?

For many outsiders that have not used or heard of Ethereum before (this poll shows it’s quite a few ["Have you heard of Ethereum?"](http://publicvotes.org/vote/PuwexrCddeyTWh2Q2)), the question of *“Why charge money for voting online?”* comes up quite quickly. After all, voting is simply adding a new field into a database, that doesn’t cost money right? Therefore the only reason why PublicVotes costs money is because Dominik is a greedy developer.

Well not quite (I’m not that greedy). To understand what PublicVotes is actually doing, we need to move away from this notion of having a single computer holding a database and entering new entries, to an entire Peer-to-Peer Network of computers collectively holding a database and validating the entries that are written to it.

That is what PublicVotes essentially is: a voting system that utilizes the Ethereum network that holds a database (a Blockchain to be more specific) that validates new entries (votes in our case) written to that Blockchain.

Obviously this requires computer power, because after all, this network ensures the integrity of the Blockchain by securing it so that no adversary can alter entries. These computers need to be incentivized to do all of this incredibly important work, which leads us back to the question of this paragraph *“What actually costs money?”*. So now you should know what costs money: whenever you create a poll or you vote, you are sending Ether to the Ethereum network that collectively verifies your vote/poll and inserts it into this globally shared Blockchain.

___
<br>

## The Analysis

The two major processes to analyse with PublicVotes are for one the *poll creation* and for another the actual *voting* on a poll. Both of these processes cost money since we are writing directly to the Blockchain.

For those interested, here is the smart contract that shows exactly what happens when we create a poll and vote. The contract, developed in Solidity, is super simple and should give you a good feel what one can do with Ethereum: [Smart Contract](https://github.com/domschiener/publicvotes/blob/master/contracts/contract.sol)

Since this analysis involves getting quite a lot of data, I wrote a simple program in Python that contacts several API’s from service providers to get the required data (such as specific transaction details, total transaction cost, Ethereum USD price and so on). You can check out the program here [Github Link](https://github.com/domschiener/ethereum-tx-details)

Lastly, I have published all of the data on Google Drive as Spreadsheets. Therefore, feel free to create your own analysis with the data :) [Link to individual poll analysis](https://docs.google.com/spreadsheets/d/1NLhTspcIsfIcKJpfJcvrbDZs3809q_o8VuiP58YUVAs/edit?usp=sharing) and [Link to general analysis](https://docs.google.com/spreadsheets/d/1ig_4-Xkuf2hAM7gXE84Br-6r4e4oZwmIDEF2s3wSo_8/edit?usp=sharing)
<br>

## Poll Creation: Cost Specifics

Creating a poll on PublicVotes means deploying a smart contract to the Ethereum network and writing information (provided by the client) about the poll to the contracts internal storage. More specifically, we are storing the following information to a public variable p of type Poll (basically a struct):

* **p.owner.** This is the address (20 bytes) of the creator of the poll
* **p.options.** This is all of the poll options concatenated to a single string. The maximum length here is 200 chars
* **p.title** The title of the poll, stored as a string with max length of 48chars
* **p.votelimit** The vote limit stored as an unsigned integer.
* **p.deadline** This is the deadline of the poll, which is a UNIX formatted date, also stored as an unsigned integer.
* **p.status** Simple boolean value of the poll, on instantiation it is set to true.
* **p.numVotes** And lastly, we set the number of votes to 0

As you can see, we do quite a lot when we start a poll and this comes at a cost. On average, it cost about **$0.024399**. The highest cost for just creating a poll was **$0.0295448**. In the following diagram you can see that the cost for creating a poll vary a lot and are dependant on the respective Ether price.

![alt tag](/assets/images/eth_analysis/1.png)


Speaking strictly in Ether terms, the gas which the contracts required for deployment also fluctuated a lot. The most expensive contract (**590955 gas** was required) costing about 20% more than the least expensive (**491295 gas** was required) one. At a price of 50 shannon (which was the same for all contracts that were deployed), we can see the cost difference here.

![alt tag](/assets/images/eth_analysis/2.png)

As you can see, the creation of a polls varies quite a bit. But this is to be expected since the information which a poll stores into the contracts storage can differ greatly in size.
<br>

# Ether Price Development vs. Poll Creation

As you may realize by now, the cost for starting a poll is not only dependant on the size of what gets stored, but also on the price of Ether at the time of the contract’s deployment. Therefore, the higher the Ether price, the more expensive it is to actually create a poll. The below diagram shows this in more detail. It clearly shows how the price of Ether dictates the cost for deploying a contract.

![alt tag](/assets/images/eth_analysis/3.png)

___
<br>

## Voting: Cost Specifics

Now as for voting costs, we no longer need to actually deploy the contract to the network. Instead when someone votes we do two things: For one, we actually *write to the contracts storage* by increasing p.numVotes by 1. And for another, *we create an event*, which is basically storing information into the Blockchains client log and making it available for the outside world to read (which is exactly what we need for voting). We also have some logic implemented, namely checking if the sender is the owner and if a votecount is equal than the votelimit, but for this analysis this is not relevant.

What is mostly important about the costs when someone votes is that we do **very little writing to the contracts storage**. If we were to store additional information in the contract, such as storing all of the votes in a public array, thus making it possible for other contracts to access this storage, I think it would cost at least twice to three times as much as voting currently costs. Therefore, keep in mind that the contract currently is very lightweight with the intention of keeping costs at a minimum.

To jump right in, overall, it cost about **$2.05 to cast 1350 votes**. On average it cost **$0.001519**, with a high of **$0.00261** and a low of **$0.001049**, to cast votes. As you can see, these numbers differ a lot. To visualize this, here is the diagram:

![alt tag](/assets/images/eth_analysis/4.png)

To put the price in perspective:

![alt tag](/assets/images/eth_analysis/5.png)

As you can see, there is a clear correlation between the Ether price and the cost of voting (who would have thought?). To get a better picture as to how much voting actually costs, we need to leave the Ether<->USD market out of the picture and instead focus on gas usage and price.

On average, voting requires **30000 gas**, with a **high of 45485 gas and a low of 21000**. What is interesting is that the price of gas fluctuated during the time the data was collected. Usually the price of gas is at a constant of **50 Shannon**, but it reached a **max of 57 Shannon** on one instance. This is because the demand of computation increased, thus the computational power itself became scarcer, which meant that the price of the gas increased. Market dynamics at its best. If we multiply the gas used with the gas price, we get the following diagram:

![alt tag](/assets/images/eth_analysis/6.png)

As you can see, overall the costs were relatively stable, only on few instances the costs were abnormally high or low. This is a combination of high gas costs, high gas spending and mostly because it was the first vote in a poll (which is more expensive, which we are going to discuss soon).

* * *
<br>

## Interesting Insights into specific polls

There were some interesting insights that were revealed while collecting all of the data from PublicVotes. First of all, it is very interesting that the first vote that is being cast is nearly always in the range of 45000 gas spent, which is a lot more than all subsequent votes. This is  unexpected behavior and there is no  good explanation yet as to why the first vote that is being cast is more expensive than the others. So if someone reading this can explain this in more detail, please let me know.

### Here are some interesting polls:

![alt tag](/assets/images/eth_analysis/7.png)
![alt tag](/assets/images/eth_analysis/8.png)
![alt tag](/assets/images/eth_analysis/9.png)
![alt tag](/assets/images/eth_analysis/10.png)
![alt tag](/assets/images/eth_analysis/11.png)
![alt tag](/assets/images/eth_analysis/12.png)
<br>

As you can see, the recent sudden increase of the Ether price had a major impact in the overall costs of voting.

* * *
<br>

## Ethereum-based Voting for Decision-Making?

As the intention of this analysis was to offer to the reader a definitive answer to the question whether Ethereum should be used for decision-making inside organizations, here is the definitive answer: **yes and no**. (exactly the definitive answer you wanted)

Organizations of any type, whether small or large, have to make constant decisions on virtually anything, including which pizza to order for the team. Therefore I do not think that it makes sense to use such an Ethereum-based (or Blockchain-based for that matter) voting application that publicly records and verifies all votes that had been cast. **Organizations want to keep the information about polls and votes private and hidden from the public, therefore recording it into an immutable public ledger is not acceptable.** The costs and advantages offered by a blockchain-based voting system do not meet the requirements of today’s organizations.

**BUT,** I do think for decisions, that require trust and a publicly verifiable system, that Ethereum-based voting is very powerful. It basically ensures the integrity of the poll (nobody can alter votes once they have been cast), it is incorruptible, publicly verifiable and transparent (everyone can see all votes on the Blockchain) and a voter can be ensured that her/his vote has been counted. This means that such a voting system is **perfect for elections inside organizations, but also for governments, where trust is of utmost importance.**

Since this is a such a huge topic, let’s dive deeper into a cost comparison with current voting systems for elections.
<br>

## Comparison with current voting systems for elections

To compare Ethereum-based voting systems to the current voting system that is used for national and state elections, we first need to describe how this voting system even works. For this specific example I’m using the UK as an example since they had published some useful numbers to base our comparison on.

As you may have guessed, **holding elections today is expensive, tedious and slow**. The system is still based off of physical booths, paper ballots and requiring humans to count the ballots. (in some cases there are machines counting the ballots, but these machines are even more expensive than humans on a per voter cost basis.[1])

For the *2010 General Elections in the UK*[2], the vote turnout was at 65.1%. Out of 45,597,461 registered voters, some 29,991,471 actually voted[3]. Mark Harper, the Cabinet Office minister, estimated the costs of the general election to be **£113,255,271**[4]. This figure consists of £28,655,271 for the cost of distributing candidates' mailings and a further £84.6 million for the conduct of the poll. That means on average, **it cost about £3.77 to register and count a vote**. If we only take the costs for conducting the poll (£84.6 million), it would cost £2.82 to cast a single vote.

To draw a comparison with Ethereum based voting, remember that on average, **it costs $0.0015 to cast a vote** and **$0.024 to create a poll**. Therefore, for holding the 2010 General Election on the Blockchain it would cost $0.024 (to create the poll) and $0.0015 * 29,991,471 = **$44,987.23**. Now this number is not completely correct, because it presumes that the price of Ether will stay the same (around $0.90) and the price of Gas will also stay at 50 Shannon. To get a more exact figure, we need to more accurately describe a hypothetical scenario in the future where a government holds elections on the Ethereum Blockchain.
<br>

## Creating a hypothetical scenario

First of all, the most important assumption of all is that the Ethereum network can even handle the amount of transaction volume an election will bring. At the current gas limit of 3,141,592, at the cost of 30000 gas per vote, we could only register some **104 votes every 12 seconds**. That is about 748800 in a day. Therefore, holding the General Elections on the Ethereum Blockchain would take some **40 days** until it’s finished. That of course is not acceptable. We therefore assume that the Ethereum Network has no such gas limit at the time the election is being held.

Secondly, we assume that the price of Ether will increase to at least **$3.50**. A government holding elections on the Blockchain is a huge deal and speculators love something like that. Because of that, we can safely assume that $3.50 is a good price of Ether (and I actually think it is on the lower end of the spectrum, as the price could definitely increase much more).

Lastly, we need to more accurately predict the gas price. Right now the Gas price is stable at about 50 Shannon. With a government holding elections and potentially some 29,991,471 votes being registered on the Blockchain, the price of gas would certainly increase. Since predicting market behavior is impossible, we simply assume that the gas price will increase 50% to **75 Shannon**. (let’s assume that the demand created by the government makes more miners want to participate, since it’s quite lucrative).

With this hypothetical scenario we can now calculate how much it would (hypothetically) cost to hold an election. To create the poll, some 520000 in gas will be required at a cost of 75 Shannon, that is 0.039 Ether or $0.1365. At a gas used of 35000 and a price of 75 Shannon per gas, the total transaction cost in Ether will be 0.002625. In USD terms that is $0.0091875. In sum, the entire election would cost $0.1365 + $0.0091875 * 29,991,471 = **$275,546.776**.

To compare, the current voting system is **at least 46205% more expensive than a Blockchain based voting system**. Casting a vote in the current system costs **$4.24, compared to $0.0091875** on the Ethereum based voting system.

Most importantly though apart from price, a Blockchain-based voting system would get rid off all of the issues which current systems have[5]. Issues such as not being able to vote due to technical difficulties is not acceptable in a democracy, where supposedly “every vote counts”.

* * *
<br>

## Conclusion

An Ethereum-based voting system offers a lot of benefits that current voting systems simply lack, or achieve at a very expensive price. **Transparency, public verifiability, corruption-resistance and assurance that a vote has been cast comes at a price of $0.0091875 per vote**. Even though using such a voting system for the constant decision-making inside an organization may be too much of what is required, holding elections on the Blockchain makes a lot of sense.

I will continue to work on more applications that offer different usecases for voting on the Blockchain. Liquid democracy is up next as I think that smart contracts are the perfect solution to making such a system viable.
<br>

# Sources
[1] http://www.theguardian.com/technology/2009/sep/30/electronic-vote-counting

[2] https://en.wikipedia.org/wiki/United_Kingdom_general_election,_2010

[3] http://www.idea.int/vt/countryview.cfm?CountryCode=GB

[4] http://www.bbc.com/news/uk-politics-24842147

[5] https://en.wikipedia.org/wiki/United_Kingdom_general_election,_2010#Voting_problems
