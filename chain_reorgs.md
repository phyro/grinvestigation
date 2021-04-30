# Chain reorgs and graph merging

When we talk about chain reorganizations, people usually have in mind a chain reorganization that attacks a single target which is double-spent and loses the holdings they believed were secure. This is just the most common scenario, there are much worse scenarios that can happen and this document attempts to focus and describe these.

## Bitcoin

Suppose Alice sends some money to Bob in a 1-2 transaction (1 input, 2 outputs) that lands on the chain. The receiver of the transaction Bob believes it got enough confirmations, but then the chain reorgs. The one input that was used in this transaction was double spent. But let's now focus on the outputs instead and observe what happens with the 2 outputs. Every output in the UTXO model is a graph continuation, so it starts building its own subgraph. If Bob spent his output and sent it to Charlie, then it's not only Bob that is losing the money, but also Charlie because the money he got no longer exists because _the whole subgraph is removed_. This can have some impact but the graph is spreading relatively slowly and the subgraphs are somewhat encapsulated.

What is a worse scenario in Bitcoin is when outputs share a common graph. Consider now that instead of making a
simple 1-2 transaction, Alice and Bob construct this transaction in a CoinJoin with 100 other inputs and outputs.
We now have a 101-102 transaction. Every single one of the created outputs that belongs to random people continues
building its own subgraph in parallel. If Alice performs a 51% attack and double-spends her input, then the whole 101-102
transaction becomes invalid and the whole subgraph collapses and can no longer exist. This is a much greater damage
because it's no longer true that only a few people got their money taken away. Everyone that was a receiver of a utxo
in the joined subgraph now lost their money. Merging UTXO graphs can be dangerous if done at scale.

## Monero

Let's see now what is happening in Monero. Monero uses ring-signatures where the spender proves that it is spending one
of the 11 outputs. Again, suppose Alice sends some money to Bob in a 11-2 transaction (11 outputs in a ring-sig) that
lands on the chain. The way Monero works is that everyone else will now automatically start using both Alice's and Bob's
outputs that were created meaning that a lot of new transactions will use these two outputs in their ring. Now, when
Alice reorgs the chain to double-spend her output that was spent in the 11-2 transaction she sent to Bob, this also
invalidates every single transaction that used either of the outputs that were created _AND_ everything that used the
outputs that were created in these transactions in their ring (and so on and so on...). The damage is huge because the
graphs start overlapping.


## Grin

The [Fan-out](fan_out.md) approach has the same problem. It injects the outputs to other graphs as inputs, so if nobody
has the transaction without the input, they can't replay it. On the other hand, the [Mimblewimble Coinswap Proposal](https://forum.grin.mw/t/mimblewimble-coinswap-proposal/8322) holds some history of the coinswaps so it should in theory be able to reconstruct a transaction without the double-spent outputs. It might not be easy, but it should be doable.