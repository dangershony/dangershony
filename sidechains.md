# [This is still WIP]

## Sidechains Overview

The purpose of this post is to describe sidechains and how we intend to implement sidechains on the Stratis Node.  

Sidechains are a way of transferring coins (more accurately transfer value) between blockchains.  
In various papers this is also referred to as 'pegged sidechains'.  

There are two types of sidechains: one-way and two-way peg. One-way means coins are burned on the chain from which they're sent and cannot be transferred back. With two-way peg sidechains, you guessed it, coins can be transferred back and forth between chains.

### Sidechain Consensus Rules
Sidechains are essentially blockchains that allow unlocking of predefined coins, the same concepts of a distributed consensus model apply in a sidechain.  

The consensus rules of a sidechain can be 'adapted' to be fit to purpose, the rules will be built into the consensus model of the sidechain.  

Some examples of possible rules are:  
- Transfer between blockchains can have built-in limitations for example '*up to a total of X amount are allowed to be transferred into the sidechain*'.  
- Transfers can be time-locked (i.e. the pegging window will only be open between block N -> N+1000).  
- Transfers can only come from predefined blockchains by hardcoding the genesis of allowed blockchains either in the genesis block of the sidechain or in the consensus rules (this can be hard-forked to add more genesis chains).  
- Transferred coins may have a different denominator (not necessarily a 1:1 scale but 1:N)    

### Coin Distinction
In sidechains we make a distinction between native coins and transferred coins (we call it external [is this the best name?] coins). This is decided at the genesis block and enforced in the consensus rules.  

**Native coins** are mined (or pre-mined) and cannot be transferred out to the parent chain. However, two sidechains that are both two-way peg can transfer native coins to each other.   

**External coins** are created at the genesis block but are locked, and can only be unlocked when a proof is presented that some coins have been locked on another sidechain.  So at creation time, the sidechain needs to define which other sidechains coins are allowed to be transferred in. 


## Examples
An example of a sidechain coins distribution:  
The genesis block may have two outputs.  
- An output of 30 million native coins (unlocked, pre-mined)
- An output of 50 million transferrable coins (locked)
- Allow mining/staking of additional 20 million native coins.

## Mining/Staking
The processes of mining/staking are no different in sidechains.  

However, sidechains that don’t have native coins may not have an efficient reward program, in that case miners will only collect transaction fees.   

This might discourage miners and add to the risk of centralization of mining.  
Some ways to overcome that:  
- Sidechains that support merged mining for POW.
- In POS parentchain, validators that prove ownership of high-value outputs can be sidechain validators.
- Predefined validators with signing keys.

Additional options are described in the sidechain white paper (demurrage, time-shifted fees etc..)  

It's very unlikely that in the Stratis ecosystem there will be sidechains without a native coin, most likely options will be sidechains with native and external coins. We'll refer to these sidechains as 'hybrid sidechains'. 



## How Does It Work  
[Make a nice graphic]  
Transferring coins between a parentchain and a sidechain:  
- On the parentchain (say Stratis)  
   - A user creates a transaction that locks coins into a special output. This is called the *Withdraw Lock*.  
   - The user then waits for that transaction to be buried under enough blocks. This is called the *Confirmation Period*.  
   - The user can now create an SPV Proof.  
- On the sidechain (say StratisS)  
   - The user can then create a transaction to withdraw coins to the sidechain. This is called a *Withdraw Transaction*.  
   - This transaction contains an SPV Proof and an input that spends the equivalent amount of coins (either 1:1 or 1:N, the denominator will be fixed in the sidechain consensus rules).   
   - Then the user waits for that transaction to be buried under enough blocks before it can be spent. This is called the *Contest Period*.  
   - The transaction can now be spent.  
- To send coins back to the parentchain  
   - The same process is repeated, starting from the sidechain.   
   
   
## Outside World Events
POW is an important part in the processes of withdrawing coins to a sidechain. The withdraw transaction needs to provide an SPV Proof which contains, among other things, a proof that a certain amount of work was done on the parentchain. However, this can still be faked if a miner has enough hash power.  

With POS systems there is no work and a sidechain can't verify the Withdraw Lock was in fact minted by a valid staking block, as the staking coins are not present in the sidechain.  

This raises an issue: How can the sidechain verify an event outside the blockchain actually happened?  

One solution is that a sidechain will track the Block Header chain of another sidechain, although this is not a desired scenario. Having chain tracking another chain adds complexity, and what about a multi sidechains scenario? This approach means every sidechain will have to track all other sidechains.  

One of the bigger challenges in blockchain networks is representing events that happened outside of blockchain consensus.
Creating a block/transaction/smart-contract are events that are contained to the blockchain environment, all the nodes on the network must have the same outcome when validating this events.  

A possible way of 'bringing' outside events to the blockchain (or proofs that an event happened) is using the miners as they extend the consensus history, and selfishly act in the best interest of the chain they are invested in.  

To overcome this limitation of verifying SPV Proofs are not fake we propose to use a Ownership Withdraw Proof by miners, which we'll described later.  


## What are SPV proofs
SPV proof is a collection of metadata that can be used to prove a transaction exists on a certain chain of headers.  
This contains the following:  
- The transaction that needs to be proved.  
- A list of headers where the transaction was mined.  
- A Merkle branch of the transaction.  

## Fraud Proof
A fraud proof is essentially just another SPV proof with a longer chain showing more hash power was used than a previous SPV Proof.  
Fraud Proofs are a way of invalidating fake SPV Proofs or invalidating a Withdraw Transaction that has been reorg'ed on the parent chain. 
    

## Ownership Withdraw Proof
WIP

## Drivechains
The drivechain model uses a voting system signalled by miners during a very long withdraw voting period. Miners verify the Withdraw Transactions by incrementing a counter (or decrementing).  
By the end of the withdraw voting period, if a withdraw transaction got sufficient votes the transaction can be included in the chain.  

## Federated pegs
The current Stratis blockchain is a descendent clone of Bitcoin and as a result has no built-in support for sidechain withdraws. To overcome this, a centralized solution is available.  

Coins that are transferred to a sidechain will be locked on the Stratis chain with a multi-sig output where the private keys will be kept by custodians, a collection of members that can prove ownership of a large stake, as well as by the Stratis Foundation.  

## Multi sidechain scenario 
Transferring native coins between sidechains.  
Stratis plans to offer sidechain networks to enterprises, communities and companies, some of which may decide to create a hybrid sidechain with native and external coins.  
A possible scenario is in the near future there will be many sidechains linked to the Stratis token, the token will be transferable to any sidechain (that allows two way pegs).  

The economic implications of this are still being determined. One outcome that comes to mind are chains more popular may draw large portions of the economy into them and by doing this drain other chains, making less popular chains diluted and result in gaining more value.  

## Links (credit)
Sidechain white paper - [https://blockstream.com/sidechains.pdf](https://blockstream.com/sidechains.pdf)  
Example on elements blog -  [https://elementsproject.org/posts/the-federated-peg-in-elements-alpha/](https://elementsproject.org/posts/the-federated-peg-in-elements-alpha/)  
Stack exchange comment - [https://bitcoin.stackexchange.com/questions/43445/how-would-spv-proofs-be-verified-when-moving-assets-back-in-2-way-pegged-sidecha](https://bitcoin.stackexchange.com/questions/43445/how-would-spv-proofs-be-verified-when-moving-assets-back-in-2-way-pegged-sidecha])  
SPV and reorg proofs [http://popeller.io/index.php/2016/08/30/spv-proofs-in-sidechains/](http://popeller.io/index.php/2016/08/30/spv-proofs-in-sidechains/)  
Rsk - [http://www.rsk.co/blog/sidechains-drivechains-and-rsk-2-way-peg-design](http://www.rsk.co/blog/sidechains-drivechains-and-rsk-2-way-peg-design)  
Bitcoin.com - [https://news.bitcoin.com/5-ways-bitcoins-transferred-sidechain/](https://news.bitcoin.com/5-ways-bitcoins-transferred-sidechain/)  
Drivechain - [http://www.truthcoin.info/blog/drivechain/](http://www.truthcoin.info/blog/drivechain/)  
Adam Beck - [https://www.mail-archive.com/bitcoin-development@lists.sourceforge.net/msg04315.html](https://www.mail-archive.com/bitcoin-development@lists.sourceforge.net/msg04315.html)  


