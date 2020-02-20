# BLS Non-interactive txs

This builds on the idea from David Burkett and the main thing is that we try to avoid the need to keep the signatures
for noninteractive txs around for `X` blocks and instead have them around forever but aggregated.

Let's say we have BLS on MW and can aggregate both public keys and signatures.

Along with kernel aggregated public key and aggregated signature, we have another pair `A_TX` and `AS_TX` in the
block header that hold aggregated public key and aggregated signature for the noninteractive txs.

1. When a pedersen commitment is CREATED and commits to the receiver's public key `x*G` we add `x*G` to `A_TX`.
2. When a pedersen commitment is SPENT that commits to a receiver's public key `x*G` we must provide a `x*G`signature
of message `M` which is always the same to enable the use of fast verification for `x*G` and we add this
signature to `A_TX`.


To validate the chain state, we perform the regular kernel validation but also do the noninteractive tx validation which
means that we test whether `A_TX` aggregated public key and `AS_TX` aggregated signature are valid. The issue is that
the UTXOs that have not been spent yet have not added their signature part to the `AS_TX` so we need to subtract their
public keys from the aggregated public key `TEST_A_TX = A_TX - UTXO_PUB_KEYS`. After this subtraction the verification
should pass.

# Issues

## Stealing coins

The creator of the UTXO know the blinding factor `r` of the UTXO because he created the pedersen commitment. He could
spy for the UTXO to be spent and could see the signature, do a 1 block reorg and spend use the same signature to
spend it.

David's solution:
To resolve this, we can have an additional commitment in the input (`commit_i`), and change the signature to be `sig(receiver_pubkey, input_commitment, commit_i)`, and then change `commit_k` to be `sum(output_pubkeys) - sum(commit_i) - commit_k_offset`. This should look familiar, since it's basically just the mimblewimble formula again, except this time it's only used to validate per-block, not across the entire chain.

## Rogue key attack

Is this possible and what could one gain from doing it?

# Additional resources

There's been a discussion of BLS signature by Gandalf the Pink on [a github issue](https://github.com/mimblewimble/grin/issues/2504#issuecomment-467446197) where he attached a
[document](https://github.com/mimblewimble/grin/files/2905763/MWpp.pdf) which might have some mistakes, but has some interesting ideas inside.