# A way to prove existence with timestamp on Mimblewimble

Bitcoin allows a transaction to commit to a hash and hence prove existence of a data at a certain point in time.
It's not obvious how to do that in Mimblewimble. The main issue is that everything that a transaction leaves forever on
the chain is a Kernel which consists of
`features (1 byte) | fee (8 bytes) | lock_height (8 bytes) | excess (32 bytes) | signature (64 bytes)`

As we can see, there's no place for a hash. One solution would be to add a hash and make sure the signature
also signs that hash but this would mean we are making the kernel even bigger.

## Commiting to data with no additional bloat
We can commit to a hash without adding any additional data to the kernel. The trick is to somehow commit to a hash
with the excess public key which is 32 bytes. Let's say I want to create a transaction that uses UTXO `v1*H + r1*G` and
creates just one new UTXO `v2*H + r2*G`. We know that the excess of the transaction will be `r2*G - r1*G`. Let's say we
want to commit to a hash `X` (32 bytes). We convert the hash to a number by converting every single character to ascii
integer and represent each as 1 byte (8 bits). The sequence of bits forms our number `K`. Note that we can derive the
hash from the number `K` by doing the reverse which is converting to bits and then taking each byte and converting it
to ascii character. The trick we do now is that we choose `r2` such that `r2 - r1 = K`. This means that the excess
we end up with will be `K*G`. Since the excess will stay on the chain forever, we can prove the hash commitment at a
certain point in time by showing the excess public key `K*G` and then proving we know `K` and that this `K` derives
to the hash `X`.
We are not done yet. The kernel itself does not have a timestamp. This means we have to prove that the kernel existed
at a block `M` which has a timestamp. Kernels have their own Merkle Mountain Range structure so we also need to provide
the inclusion proof of our transaction kernel for the MMR root at block `M`.
So the data we need to prove hash `X` existed at block `M` is:
1. Kernel which holds our commitment to hash `X`
2. `K` private key for the kernel excess. `K` when converted to ascii gives us `X`
3. Inclusion proof which proves that Kernel was included in the MMR root at block `M`


### Update:

After talking to John Tromp about this, he was not only aware of the above solution (he also mentioned
it in the public already), but he also showed me a couple different ways to do timestamping on Grin:

1. The hash can be encoded in the `r` blinding factor of the UTXO itself. We can prove hash existence even after the UTXO was spent, since the block
that added the UTXO contains it's Pedersen commitment in the `output_MMR_root` hash, so if we know the Pedersen commitment of the UTXO `v*H + r*G` along with the inclusion
proof for the `output_MMR_root` at block `M`, we are able to prove that the Pedersen commitment encoded a hash by revealing `v` and `r`
and showing that `r` encodes our hash. Since we are showing `v`, we could avoid showing relevant values by creating an
extra Pedersen commitment with `0.001*H + r_hash*G` which would prevent giving useful information of our `v` value. This extra commitment
can later be spent since we don't really need it around for our proof.

2. If we have full nodes available to query, it can make the first idea that encodes the hash to the kernel
excess even simpler. Instead of showing the inclusion proof, we can take advantage of full nodes storing all the
kernels as a sequence and just remember the kernel's index. We can ask any full node to give us the kernel at this
position and we don't need the inclusion proof. In order to prove it existed at block `M` we can check the `kernel_mmr_size` at block `M` and since kernels are added sequentially,
this can prove that our kernel was already a part of the kernel MMR at that block. If we don't know the block `M` we can find it by doing a binary search on `kernel_mmr_size`.
This approach has a much smaller proof since we only need to store the kernel index (32 bit) instead of the whole inclusion proof.
Note that in the case od node returning a fake kernel for our position, we could ask some other node or just sync a new
node and get all the kernels.