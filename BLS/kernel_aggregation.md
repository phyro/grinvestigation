# Possible Kernel aggregation with BLS

We knew that with BLS kernel signatures could be aggregated, but lately there's been a [discussion](https://forum.grin.mw/t/grin2020-roadmap-calling-for-blog-posts-with-ideas/6327/67?u=oryhp)
whether we can also aggregate kernel public keys. The assumption here is that Mimblewimble is immune to Rogue Key Attacks by providing KOSK protection from the bulletproofs.

This could potentially allow Grin to substantially reduce the size of the kernel by getting rid of almost 100 bytes leaving the kernel only with the feature byte, lock height and the fee.
If this is possible, it would be worth researching also if we could move the remaining kernel data to the UTXO itself. If we managed to do this, it might allows us to have the "simpler
test" version of verification test which is described under (3) in [this BLS research](https://crypto.stanford.edu/~dabo/pubs/papers/BLSmultisig.html#mjx-eqn-eqaggsame) from the BLS authors.
On top of that, it would bring more privacy because it would not be possible to tell how many transactions were aggregated (note that even with public key and signature aggregation, the number
of transactions could be derived from the number of messages that have been signed). A side effect of it would also be that one could use [Obscuro](./Obscuro/README.md) patterns in a bloat-free way.