---
Categories:
- Development
- Blockchains
- Cryptocurrency
Description: "an overview of etheream vs bitcoin"
Tags:
- blockchain
- bitcoin
- etheream
- cryptocurrency
date: 2017-03-07T14:21:05-05:00
menu: blog
title: Better living through blockchains - Intro
---

2 years ago, I bought 10 bitcoins at $134 each.
A week later they shot up to $240.
Naivly I flipped them and made a tidy thousand dollar profit.
My initial glee had turned out to be one of my biggest regret as you might guess from the current valuation of $1.2k per bitcoin.
Nevertheless, It cemented cryptocurrencies into the horizon of my view.

While Bitcoin gets most of the attention, Its only the tip of the iceburg.
More disruptive is the potential enabled by the blockcahin in creating decentralized transaction.
Bitcoin's blockchain is limited in features and is oriented towards cryptocurrency.
[Etheream](https://www.ethereum.org/) takes the ideas introduced by etheream and delivers a quantum leap in functionality.

<!--more-->

## Blockchains, a primer
A blockchain is a network of nodes running on seperate computers that syncronize data as a set of transformations.
On each node is a chain of cryptographic blocks that start with a publicly agreed upon root number.
that number is used with the data of a transaction to generate the seed number for encrypting the next transaction.

On a single node, lets say we have a function H that takes the seed number and some set of data.
With it we can generate a block, hash a seed number to be used in the next iteration of the chain.


```
H(original_seed, dataA) => Block{dataA, hashA, seedA}
H(seedA, dataB) => Block{dataB, hashB, seedB}
```

What makes the blockchain so powerful is the ability to verify if the data it encodes has been tampered with.
With the previous block's generated seed and the current block's data we can check that the generated hash matches the one in the current block.
Additionaly, with that new seed, we can verify that for the next block for the entirety of the chain.

Suppose some nefarious agent breaks into the server and modifies the contents of `Block{b}`.
The following code would verify that the blockchain has indeed been tampered.

```
tamperfree = true
seed = getSeed()
blockchain = getBlockchain()
while - blockchain.has_more_blocks()
  block = blockchain.getCurrentBlock()
  newBlock = H(block.data, seed)
  if newBlock.hash != block.hash
    tamperfree = false
    break
  else
    seed = newBlock.seed
    blockchain.iterate()

return tamperfree

```

Thats just a single node.
Now say we have a network of these nodes.
When a new piece of data is added to the chain, the data gets sent to the network where each computer verifies their chain and votes on the validity of the new transaction.
If consensus is reached, the change is committed to the entire network.
We now have a decentralized data store for storing transactions while securing against bad behavior by rogue nodes.

