# A way to commit to data with a Pedersen commitment

Let's say we want to commit to more data than a single Pedersen commitment `v*H + r*G`. David Burkett introduced an idea
that uses Bulletproofs to inject a hash in the 20 bytes that are available and it seems to work. This however makes the
protocol dependent on the specific range proof implementation called Bulletproofs.

## Commiting inside the Pedersen commitment itself

Let's say we expand the `H` side of the Pedersen commitment to be `v*H - j*H` and the `G` side of the Pedersen commitment
to be `r*G - k*G` forming a Pedersen commitment `v*H - j*H + r*G - k*G`. Along with this Pedersen commitment we also show
these values:

1. `j` which is the private key for `j*H`
2. `k` which is the private key for `k*G`
3. `data` which is the data we want to commit to

What must hold is that `j`'s bytes converted to ascii give us the first half of `RIPEMD-128` hash and the `k`'s bytes
converted to ascii give us the second half of the hash which together form a hash `X` where `RIPEMD128(data) = X`.
This means that `j` and `k` together commit to the hash `X`.

When verifying the MW inflation formula `sum(outputs) - sum(inputs)` we need to add both Pedersen commitment
offsets `j*H` and `k*G` creating the original Pedersen commitment `v*H + r*G` which is what the kernel signs.

This uses additional 128 bits (16 bytes) for the integers `j` and `k` on the UTXO compared to the idea that injects
a hash into a Bulletproof. The benefit here is that if an alternative to Bulletproofs comes, we could safely switch the
range proof implementation of the new Pedersen commitments without changing the underlying logic.

*The idea above was partly influenced by the ideas from the Gandalf the Pink's paper.*