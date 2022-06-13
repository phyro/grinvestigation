# Commitment-based blockchain discussion in 2010

There’s a rather popular [bitcointalk comment by Satoshi](https://bitcointalk.org/index.php?topic=770.msg8637#msg8637) that has been interpreted in different ways.
Specifically, what gathered interest was the following statement
> If a solution was found, a much better, easier, more convenient implementation of Bitcoin would be possible. - Satoshi

Here are some of the interpretations that have been floating around:
- [Interpretation by Zooko](https://twitter.com/zooko/status/1499478874375163904) from ZCash
- [Interpretation by fluffypony](https://twitter.com/fluffypony/status/1534709478368714753) from Monero


While they're right that privacy improvements like stealth addresses and ring signatures were a part of the discussion, this wasn't what the discussion was about and what Satoshi's comment referred to. The comment was aimed at a specific commitment-based blockchain construction that Red described in the post. What about this construction triggered Satoshi's interest enough to call it “a much better, easier, more convenient implementation of Bitcoin”? Here’s what was really discussed.

## Dissecting the bitcointalk discussion

#### Red's idea

> The general question is, could the block list be/have been implemented in a way that didn't store the full transactions in the list? Specifically, *perhaps* it would be possible to store only hashes of the in-points, out-points in the block list.

Can we store only commitments rather than all the data?

> If everything validated correctly, the additional in/out-point hashes would be added to the block. This closes the transaction's in-points, and marks the new out-point hashes as unspent.

We'd like to validate input commitments were correctly spent and create output commitments.

> Once a node completes the block (by winning the hashing contest), he then broadcasts the block of hashes and the related transactions+plus antecedents to the other nodes for confirmation and acceptance.

As an example, Red gave the following structure:
```
block-9: {hash-a, hash-b, hash-c, hash-x}
block-12 {hash-a, hash-y, hash-c, hash-d}
block-17: {hash-b, hash-d, hash-e, hash-z, hash-f}

Transaction:
 in-points: {hash-x, hash-y, hash-z}
 tx_data: {address, signature and other transactions stuff}
 out-points: {hash-payed, hash-change}
 
generating-block: {hash-x, hash-y, hash-z, hash-payed, hash-change}
```
From this, we can see that a block consists only of commitments. Transactions are sent separately from the block and the online nodes would verify these.

> So basically, if the i/o-point hash existed twice in the block list, it has been spent. If it exists only once it has not been spent.

If a commitment appears twice, it is considered spent.

> The goal is to provide all the same security of the existing system, but to avoid creating a public graph of every transaction that is easily correlated. In this case, the hashes don't even have to associate in the block. The block could simply sort all hashes in ascending order.

We'd like to keep the same security model without revealing association links between the commitments. A block is just a sorted list of commitments (a set of commitments).

> I guess my real question is: "What is the earliest you can remove the transactions?"

Could we remove transaction data as soon as it is spent?

> You could argue that nodes could remember everything anyway (the web never forgets). But if you structured the protocol so that new nodes would only receive a block list of hashes, they could only remember from this moment forward. That would give a little additional privacy. (Maybe)

We'd like to structure the protocol such that a new node only sees the commitments rather than transaction data.

Identifying the UTXO set is central to this approach, and something both ZCash and Monero forsake.

#### Satoshi's first response

> This is a very interesting topic.  If a solution was found, a much better, easier, more convenient implementation of Bitcoin would be possible.

It would also be more scalable and could be slightly more private due to nodes dropping data.

> It seems a node must know about all transactions to be able to verify that.  If it only knows the hash of the in/outpoints, it can't check the signatures to see if an outpoint has been spent before.  Do you have any ideas on this?

There was no known solution to this problem at the time.

A screenshot of the discussion with highlights is available [here](https://i.imgur.com/7aIlYvF.png).

Let's summarize some of the properties of Red's hypothetical system:
- a block is a a set of commitments
- transaction data (address, amount) and links between the commitments in a block would ideally not be visible to a new node
- spent commitments can be identified by their previous appearance which makes it possible to construct the UTXO set regardless when you joined the network

<hr/>

The main thing Red wanted to achieve was to modify the construction such that a block holds only information about commitments rather than all transaction data. This would have brought some interesting properties. Since commitments are by definition random, they can't leak information about their links in a block. At least not without additional data. Having only commitments would also no longer reveal information about amounts or owners/addresses to a new node. We can see why Satoshi thought this would've been a better implementation of Bitcoin. There was a big problem with the idea Red described which Satoshi described as

> It seems a node must know about all transactions to be able to verify that.  If it only knows the hash of the in/outpoints, it can't check the signatures to see if an outpoint has been spent before.  Do you have any ideas on this?

If we are only left with the commitments, it seems impossible to verify they were created by following the rules of the network. Further down, Satoshi shares his view on the commitment information

> If we're willing to have clients keep the history for their own money, then some of the information may not need to be stored by the network, such as:
> - the value
> - the association of inpoints and outpoints in one transaction

He correctly claimed that the construction described would mean the clients would have to keep the history to prove to the receiver the amounts they hold. This would also mean the network itself doesn't show information about the amounts or the links between the commitments. He further describes the construction as

> The network would track a bunch of independent outpoints.  It doesn't know what transactions or amounts they belong to.  A client can find out if an outpoint has been spent, and it can submit a satisfying inpoint to mark it spent.  The network keeps the outpoint and the first valid inpoint that proves it spent.  The inpoint signs a hash of its associated next outpoint and a salt, so it can privately be shown that the signature signs a particular next outpoint if you know the salt, but publicly the network doesn't know what the next outpoint is.

From what I understand, Satoshi described a chain that would keep track of uniformly random commitments and uses these as inputs and outputs. The inputs would come with a proof that it was spent and would sign a hash of the next commitment and a salt which would prevent people from knowing which output it signed.

![Commitment chain](https://i.imgur.com/Tu28hpr.png)

In the picture, we assume the signature (somehow) proves the input was spent and signs its new output in a blinded way. The construction in the image above may not accurately describe what Satoshi had in mind since it's rather hard to understand the design as many of the details were never described sufficiently. The key idea seems to have been to have commitments rather than data in an attempt to hide the value and associations of input and output commitments inside a block. What's perhaps less obvious and was not discussed is that such a construction already supports coinjoin of commitments by design by taking the set unions which preserves the validity. As we can see from the image above, the block is just two commitment sets where input commitments come with a signature. We no longer have separate transactions with clear boundaries, the whole block can be seen as a joint transaction with unknown links.

He then elaborates a bit more on it

> I was talking about in the hypothetical system I was describing, if the network doesn't know the values and lineage of the transactions, then it can't verify them and vouch for them, so the clients would have to keep the history all the way back.

This is the problem with the construction above. The network can't guarantee the transactions and thus the state of commitments is valid and an offchain proof is required for validation/trust. It's also not clear how a verifier could audit its supply in this case.

> If a client wasn't present until recently, the two ways to convince it that a transaction has a valid past is:

> 1) Show it the entire history back to the original generated coin.

> 2) Show it a history back to a thoroughly deep block, then trust that if so many nodes all said the history up to then was correct then it must be true.

Sounds reasonable. If we have only hashes on the chain, we could either prove the whole history of a transaction by showing it offchain or rely on a weaker trust model described in 2. which at some point skips validation and relies on PoW hardness.

Even though the idea was very interesting, there were unfortunately quite a few issues with it the way it was presented. One big issue was that it moved the whole notion of a transaction offchain which meant the trust and validation moved offchain as well. The chain had no rules regarding transactions, it was just a verifiable transition of commitments through transition proofs. At that time, it was unclear how to construct a commitment-based blockchain and whether it was even possible to create one while preserving a Bitcoin-like model of verifying transactions and the total supply.


## From discussion to a working construction

In the next few years many new faces joined the Bitcoin community and started researching new ways of improving Bitcoin. One way to improve privacy is to use not a hash commitment, but a homomorphic commitment called a Pedersen commitment which, because of its algebraic nature, allows us to verify a transaction preserves the balance of amounts. This was described in a Confidential Transactions research paper and was a key ingredient needed to bring the construction Satoshi described to life. In 2016, someone under a pseudonym of Tom Elvis Jedusor dropped a text file describing a way to construct a commitment-based chain which solved the issues Satoshi encountered. The main idea was to use the blinding factor in a Pedersen commitment as a spending secret. From there, everything followed rather naturally to form a simple and elegant chain construction.

![commitment-chain](https://i.imgur.com/6yYwWCr.png)

[The construction that came out](https://github.com/mimblewimble/docs/wiki/MimbleWimble-Origin) is one of the most elegant (potentially the most elegant) blockchain designs we have today and it seems incredibly hard to simplify it any further. The signature that serves as a spending proof and the binding to the newly created commitments is detached not just from the outputs but they are no longer a part of the input either. There are not direct links between the spent commitments, created commitments and spending proofs. The construction also comes with a simple formula which enables anyone to verify the transaction is well balanced (no inflation) and that the owners of spent commitments have in fact signed transition to the newly created commitments. As already mentioned above, a block now has a set of spent and a set of created commitments which allows for a natural coinjoin of these and since a block is no different than a transaction, we can verify the transfers were correct the same way we verify a transaction.

The author further improved on the design Satoshi and Red discussed. Not only did design make it possible to verify both the preservation of amount balances and spending proofs to new commitments but it also allowed us to forget every single spent commitment completely (which includes its creation) while retaining the same security model as Bitcoin. It's hard to believe we could prune every spent commitment as if it never existed and still be able to verify the chain history had only valid transactions.

![chain-cut-through](https://i.imgur.com/mwTuSGq.png)

It turns out that this detail also reduces the whole chain to a single transaction creating the entire UTXO set. Since the chain is now also a transaction, the same formula that allows us to verify a transaction is well balanced can also be used as a total supply audit as well as a proof that every single transaction in history came with a valid signature from its owner. Perhaps the most surprising thing in this design is that a transaction, a block and the chain itself are the same construction. The chain is just a very large transaction whose created commitments are exactly the UTXO set. Beautiful.

If you're interested in more details about how the commitment-based chain construction achieves this, you can [read more about it here](https://phyro.github.io/what-is-grin/mimblewimble.html).