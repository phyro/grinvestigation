# Why Grin

Below is a comparison of Grin/Mimblewimble with some other technologies. This isn't to say Grin is better than others, it simply makes different tradeoffs.

| Feature | Bitcoin | Monero | Grin |
| --- | --- | --- | --- |
| Chain growth rate | 4x | 20x | 1x |
| Spend index size | \|UTXO\| | \|TXO\| (orders larger) | \|UTXO\| |
| Fully auditable supply | ✓ | X | X |
| Hides amounts | X | ✓ | ✓ |
| Hides addresses | X | ✓ | ✓ |
| Hides the transaction graph (untraceability) | X | ✓(mostly) | X |
| Possible constant-size full node | ✓ | X | ✓ |
| Scriptless scripts | ✓ | X | ✓ |
| Emission curve | exponential | exponential + linear | linear </br>(1 per second) |
| Total supply | 21M | ∞ | ∞ |
| Soft total supply </br>(reached when yearly inflation rate <1%) | 19.7M | 18M | 3150M |
| Time to reach soft total supply | 16 years | 8 years | 100 years |
| Fraction of soft total supply emitted in first year alone | 13.3% | 40% | 1% |
| Block time | 10 minutes | 2 minutes | 1 minute |
| PoW complexity | simple | very complex | simple |
| PoW verification | trivial | very expensive | trivial |
| ASIC resistant PoW | X | ✓ | X |
| Protocol complexity | complex | complex | simple |
| Codebase | original | cloned from Bytecoin | original |
| Multiple implementations | ✓ | X | ✓ |
| Formal process for protocol improvements | ✓ | X | ✓ |
| (relative) time locks (for Layer 2) | ✓ | X | ✓ |
| Possible atomic swaps | ✓ | ✓ | ✓ |
| Full wallet control | X | X | ✓ |
| Payment proofs (value commitment) | ✓ | ✓ | ✓ |
| Stronger payment proofs (content commitment) | X | X | ✓ |
| Non-interactive transactions | ✓ | ✓ | X |
| Doesn't require scanning the chain upon opening the wallet | X | X | ✓ |


1. **Chain growth rate** - Thanks to transaction cut-through, it is possible to prune every spent input/output pair in the history of the chain. This leaves the chain with only 100 bytes of data per transaction which makes it 4x smaller than an average Bitcoin transaction which is around 400 bytes and 20x smaller than an average Monero transaction which is ~2kb.
2. **Spend index size** - Describes the size of the index which is needed to verify whether an output has been already spent. While Bitcoin and Grin have a set of unspent outputs which makes the index much smaller, Monero can't distinguish between a spent and unspent output and needs to keep a "key image" entry in the index for every spent output. 
3. **Fully auditable supply** - Both Grin and Monero rely on the soundness of their supply on a cryptographic assumption that the discrete log is hard to find. On top of that, the users also rely that the implementation of validation methods don't have bugs which could be exploited to print coins out of thin air. This approach is expected to gain confidence over time as more people attempt to find bugs - assuming the validation code doesn't drastically change. Bitcoin has a simpler way to audit the supply because the amounts are public. In case quantum computers start becoming a reality which may provide inflation threat, Grin can "flip a switch" to make the chain secure against quantum computer attacks, but at the cost of showing the amounts.
4. **Hides amounts** - Both Grin and Monero use Confidential Transactions to blind the amounts.
5. **Hides addresses** - Grin doesn't have addresses. The outputs are just a random point on the elliptic curve and the secret to spending this output is encoded in that curve point. Monero uses stealth addresses.
6. **Hides the transaction graph (untraceability)** - Bitcoin and Grin don't obfuscate the transaction graph today. Monero adds some decoys to the input side of the transaction making the links less clear, but it's not a perfect solution. Grin may have some options to improve the transaction graph obfuscation through its ability to noninteractively aggregate transactions, but the options need to be researched.
7. **Possible constant-size full node** - Bitcoin node can prune historical data and still fully validate. This leaves it only with the UTXO set being needed to fully validate a new block, but even this can be reduced to a constant with Utreexo. Grin has a very similar ability to prune everything away and get a constant-size full node with the existing accumulators it uses. It's unclear whether Monero can achieve this, there have been no solutions for that AFAIK.
8. **Scriptless scripts** - Grin supports some scripting through scriptless-scripts. Bitcoin has a more expressive scripting language Bitcoin script - this comes with a lot of additional complexity though.
9. **Emission curve** - Grin and Monero have a linear emission model which may be more sustainable in the long run.
10. **Total supply** - Bitcoin has a hardcap of 21 million Bitcoins while both Monero and Grin have an uncapped supply.
11. **Soft total supply (reached when yearly inflation rate <1%)** - Shows total supply points at which the soft total supply is reached.
12. **Time to reach soft total supply** - Both Bitcoin and Monero reach the soft total supply in the first two decades while it takes 100 years for Grin to reach it.
13. **Fraction of soft total supply emitted in first year alone** - Grin only emitted 1% of the [soft total supply](https://john-tromp.medium.com/a-case-for-using-soft-total-supply-1169a188d153) in its first year and will continue to add a percent per year until we reach 100%. This may serve as one possible measure of wealth concentration. 
14. **Block time** - Not much to add here.
15. **PoW complexity** - Grin uses Cuckoo cycle which is simpler than computing a sha256 used by Bitcoin and much simpler than RandomX which is what Monero uses.
16. **Cheaply verifiable PoW** - The solutions of both Cuckoo cycle and sha256 are much simpler to verify compared to a solution of a RandomX PoW.
17. **ASIC resistant PoW** - Grin and Bitcoin have an ASIC friendly PoW while Monero's RandomX is ASIC resistant.
18. **Protocol complexity** - Out of all three, Grin has the simplest protocol design.
19. **Codebase** - Both Bitcoin and Grin were written from scratch. Monero was forked from a project called Bytecoin.
20. **Multiple implementations** - Grin has two implementations, the Rust node and the Grin++ which is written in C++. This is very useful for catching bugs when a network split happens. Monero has a single node implementation.
21. **Formal process for protocol improvements** - Changes to the Bitcoin protocol are proposed through BIPs. Grin has a similar formal way of proposing changes through RFCs. I'm not sure Monero has a formal process for the changes.
22. **(relative) time locks (for Layer 2)** - Grin and Bitcoin have an option to express bidirectional payment channels. There is no known way to achieve this on Monero.
23. **Possible atomic swaps** - It's possible to do these on all three.
24. **Full wallet control** - In a NITX (noninteractive transactions) setting, the receiver can only control which outputs are spent, but has no control over which are received. This opens a lot of unwanted state injections like dusting attacks or “dark/illegal” outputs to the wallet. Controlling the incoming traffic is easier to reason, easier to regulate and eliminates these issues completely at the cost of a click on “Confirm” and a little wait time for the other party. This is not possible when there is an option to receive an output you have not signed.
25. **Payment proofs (value commitment)** - The sender can prove the value was sent to the receiver by showing the payment proof.
26. **Stronger payment proofs (content commitment)** - ITX allows for a more powerful version of payment proofs which allows the transaction parties to commit to an arbitrary statement/document by signing the hash of the document. Signing documents requires their review prior to signing which makes it only possible in the ITX setting. An example of such a document could be an actual invoice from a store. It may be possible to do that on Bitcoin as well, but I'm not aware of any approaches.
27. **Non-interactive transactions** - Grin does not support non-interactive transactions. This makes it a bit more challenging to make donations or send to an offline wallet.
28. **Doesn't require scanning the chain upon opening the wallet** - We usually need to scan the outputs to see if they belong to us. If our wallet created the output and then sent it off to the chain, then we never require scanning whether the outputs on the chain are ours (unless we reuse the seed on a different device). We can simply check if the outputs we created exist on the chain.