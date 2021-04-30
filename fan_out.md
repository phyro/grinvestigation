# Fan-out graph obfuscation

Decoy proposals usually require others to provide additional inputs/outputs to obfuscate your transaction making it a model where the user's transaction obfuscation and hence graph obfuscation relies on other people e.g. [Beam's decoy model](https://github.com/BeamMW/beam/wiki/Transaction-graph-obfuscation). In this document, we describe a different obfuscation model where the user obfuscates its own graph making it a trustless obfuscation - the key thing is that we don't need the inputs/outputs of others to make our transaction graph hard to trace forward.

Suppose we have 2 wallet keys, primary and secondary (decoy key). We use the primary key to create our real outputs
and decoy key to create decoy outputs.

We now follow these 2 simple rules:

## 1. Transaction creation

When I'm creating my output in a transaction, I create 1 output with primary key and 49 with secondary key. This means that a transaction will have 50 outputs that belong to me, so we will push a 1-51 transaction to the dandelion phase. During generation of decoy outputs, we "pair" each decoy output with one random output on the chain, favoring the recently created ones. We define a decoy output "ripe" for spending when its paired output has been spent +- random_delta. Of course the decoy output also needs more than 10 confirmations. Having the "ripe" definition depend on the actual outputs on the chain favoring recent ones may stimulate real usage and hence make decoy outputs really hard/impossible to identify.

## 2. Stem transaction

My node holds the decoy key, which means that it can identify and spend the decoy outputs I created. It's safe to keep the decoy keys online all the time because the decoy outputs hold no value. When a node receives a stem transaction, it checks if it holds any "ripe" outputs and adds one of these as an input to the stem transaction and adjust the offset - we don't create any output.

## Transaction graph noise

By following these two rules we end up having the original 1-51 transaction, where even if everyone knew that 50 of these belong to me, nobody can actually trace my actual usage because 49 of these outputs will be added as inputs to random transactions flowing across the network. They would only be traceable if one of the next transactions spent only a single input.

What we are doing is creating a forward noise in the transaction graph. They may know which outputs were ours, but they only know 1 of 50 is our real output, they can't trace the actual spending because they all continue a separate path on the transaction graph.

The downside is clearly more throughput due to more outputs, but every decoy output will bring more privacy to some other random transaction! It's not really a clear downside, more like a tradeoff where a user gains graph obfuscation by obfuscating the graph of others a bit.

_NOTE: We'd need to have transactions that overpay some fee in order to be able to include the inputs._

## Network forward noise

If many users did this, it becomes a graph noise generator where your transaction can get inputs added by other nodes so the benefit for the transaction extends to many more people (including yourself) when they get added input decoys to their transaction.

## Ripe logic

The idea is to have a distribution that corresponds to real spending. The ripe algorithm that pairs an outputs with a random output that landed on the chain is recursive in nature. The chosen output has to be one of the two:
1. a decoy output
2. a real output

If we map to a real output, then it is obvious that our spending pattern will match the real spending pattern +- delta. It's less clear what happens if we choose a decoy output. In this case, we know that the decoy output must have been paired with another output before and that output was again, either a decoy output or a real output so we have a recursive definition here where at the end, we definitely arrive at the real output which was a real spending pattern. Depending on the depth, it may or may not resemble a real spending, because when we have a chain of decoy outputs that depended on one another, every one of them extended the real spending time by some delta, meaning the the last one will have the sum of these deltas added to the real spending pattern. If the sum is small enough, we still end up with a pattern that is very likely to happen with a real spending.

## Comparison

```
Coinswap:
+ noninteractive
+ big anonymity set
+ obfuscates the transaction graph
+ reorg friendly
+ cheap in terms of fees
+ does not substantially lower the chain throughput as every output has only 1 new output

- trust-minimized
- can be attacked
- it takes a bit of time to obfuscate, but that shouldn't be a big issue

Fan-out:
+ noninteractive
+ obfuscates the transaction graph
+ trustless
+ can't be attacked
+ given enough volume can be obfuscated faster

- not reorg friendly (!!!)
- more costly (fee cost depends on the anonymity set)
- can impact chain througput for real transactions if many use it due to many decoys
- possible chain bloat left behind - though it isn't clear how much
- variable anonymity set (but usually smaller)
- it's not completely clear the decoy creation follows natural spending
```

**Discussion:** [Grin forum topic](https://forum.grin.mw/t/fan-out-graph-obfuscation/8843)

**Document:** [Fan-out Graph Obfuscation](https://gist.github.com/phyro/6fce03d2d4c116ba86053e99472d28ba)
