# Constant size full node

There have bee many discussions about prunning kernels and rangeproofs. The problem can be approached in many ways, but there is one way that is consistent for prunning all the data which is by aggressively pruning the MMRs. It turns out we can prune everything except for the MMR peaks and still be able to fully validate if we keep some small information around. I believe we can also get the same benefits [Utreexo](https://dci.mit.edu/utreexo) brings to Bitcoin, but without needing to introduce Utreexo accumulator because it is possible to do it with the MMRs - in the exact same way, by pruning everything but the MMR peaks for the Output PMMR and Bitmap MMR. This should allow for a constant size full node regardless of the chain size.

There are a few things that need to be thought out, but nothing insurmountable.

_Note: This does not address the IBD, you still need to download all the data, what changes is the storage required which now becomes really small._

**Discussion:** [Grin forum topic](https://forum.grin.mw/t/constant-size-fully-validating-node/8125)

**Document:** [Constant size full node](https://gist.github.com/phyro/0ad4ccf71e936dd90545648110224e96)