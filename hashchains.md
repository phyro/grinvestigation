# Hash-based blockchain discussion in 2010

There’s a rather popular [bitcointalk comment by Satoshi](https://bitcointalk.org/index.php?topic=770.msg8637#msg8637) that has been interpreted in different ways.
Specifically, what gathered interest was the following statement
> If a solution was found, a much better, easier, more convenient implementation of Bitcoin would be possible. - Satoshi

Here are some of the interpretations that have been floating around:
- [Interpretation by Zooko](https://twitter.com/zooko/status/1499478874375163904) from ZCash
- [Interpretation by fluffypony](https://twitter.com/fluffypony/status/1534709478368714753) from Monero


While they're right that privacy improvements along with stealth addresses and ring signatures were a part of the discussion, this wasn't what the discussion was about and what Satoshi's comment referred to. The comment was aimed at a specific hash-based construction of a blockchain that Red described in the post. What about the construction triggered Satoshi's interest enough to call it “a much better, easier, more convenient implementation of Bitcoin”? Here’s what was really discussed.

![Bitcointalk discussion](https://i.imgur.com/7aIlYvF.png)

The main thing Red wanted to achieve was to modify the construction such that a block holds only information about hash commitments rather than all transaction data. This would have brought some interesting properties. Since commitments are by definition random, they can't leak information about their links in a block. At least not without additional data. Having only commitments would also no longer reveal information about amounts or owners/addresses. We can see why Satoshi thought this would've been a better implementation of Bitcoin. There was a big problem with the solution Red described which Satoshi described as

> It seems a node must know about all transactions to be able to verify that.  If it only knows the hash of the in/outpoints, it can't check the signatures to see if an outpoint has been spent before.  Do you have any ideas on this?

If we are only left with the commitments, it seems impossible to verify they were created by following the rules of the network. Further down, Satoshi shares his view on the commitment information

> If we're willing to have clients keep the history for their own money, then some of the information may not need to be stored by the network, such as:
> - the value
> - the association of inpoints and outpoints in one transaction

He correctly claimed that the construction described would mean the clients would have to keep the history to prove to the receiver the amounts they hold. This would also mean the network itself doesn't show information about the amounts or the links between the commitments. He further describes the construction as

> The network would track a bunch of independent outpoints.  It doesn't know what transactions or amounts they belong to.  A client can find out if an outpoint has been spent, and it can submit a satisfying inpoint to mark it spent.  The network keeps the outpoint and the first valid inpoint that proves it spent.  The inpoint signs a hash of its associated next outpoint and a salt, so it can privately be shown that the signature signs a particular next outpoint if you know the salt, but publicly the network doesn't know what the next outpoint is.

Essentially, what Satoshi described is that the chain could keep track of uniformly random commitments and use these as inputs and outputs. The inputs would sign a salted newly created commitment which would prevent people from knowing which input signed which output. 

![Commitment chain](https://i.imgur.com/KIg7gZQ.png)

The general idea is brilliant, we have a construction that detaches the inputs from outputs.

He then elaborates a bit more on it

> I was talking about in the hypothetical system I was describing, if the network doesn't know the values and lineage of the transactions, then it can't verify them and vouch for them, so the clients would have to keep the history all the way back.

This is the problem with the construction above. The network can't guarantee the transactions and thus the state of commitments is valid and an offchain proof is required for validation/trust.

> If a client wasn't present until recently, the two ways to convince it that a transaction has a valid past is:
> 1) Show it the entire history back to the original generated coin.
> 2) Show it a history back to a thoroughly deep block, then trust that if so many nodes all said the history up to then was correct then it must be true.

This was a reasonable assumption. If we have only hash commitments on the chain, we could either prove the whole history of a transaction or rely on a weaker trust model described in 2. which at some point skips validation and relies on PoW hardness.

Even though the idea was very interesting, there were unfortunately quite a few issues with this construction in the way it was presented. It moved the whole notion of a transaction offchain which meant the trust and validation moved offchain as well. The chain was just a transition of commitments.




