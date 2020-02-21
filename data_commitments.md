# A way to commit to additional UTXO data in the scope of a MW transaction

Let's say we want to commit to more data than a single Pedersen commitment `v*H + r*G`. David Burkett introduced an idea
that uses Bulletproofs to inject a hash in the 20 bytes that are available in the Bulletproof. This however makes the
protocol dependent on the specific range proof implementation Bulletproofs. It would be best if there was a way
to commit to additional data without relying on the range proof implementation. The reason for this is that if an alternative
to Bulletproofs comes, a switch to a different range proof solution would not cause us to lose a way to commit
to additional data.

## Commiting to additional data using the transaction kernel

Let's say we want to commit to some `data` with a Mimblewimble UTXO. Along with the Pedersen commitment `P1 = v*H + r*G` we also show the `data` value which is the data we want to commit to.
This means our UTXO now holds two pieces of information `UTXO = [P1, data]`.

With the information from the `UTXO`, we can compute `P2 = v*H + r*G + hash(P1 | data)*G`.
When creating the kernel public key and signature, instead of using `P1` Pedersen commitments we use `P2` which can be computed from the UTXO. This makes the transaction kernel commit to the excess
which is a combination of values of the form `r+hash(P1 | data)*G` instead of just `r*G`.
Only the owner of the Pedersen commitment knows the `r`, but everyone can compute the `hash(P1 | data)` because the `P1` and `data` are public. Swapping the `data` for a different one is
hard because one would need to create a different `data` which would produce the same hash to keep the transaction kernel valid. Finding such data requires finding a hash collision. Similarly, if someone
wants to spend the commitment they need to know the `r + hash(P1 | data)` and since we know they already know the `hash(P1 | data)*G` they really only need to know `r` which is the same
requirement as in the current implementation.

If someone tries to change the `data` after the transaction has been created, then the kernel will be invalid.

This also means that the inflation check formula `sum(outputs) - sum(inputs)` needs to use `P2` commitment.

So we end with a commitment implication chain:

`kernel => (r+hash(P1 | data))*G => UTXO` and hence `kernel => UTXO`

where `=>` reads as `commits to`.

This does not use any more data than the idea that injects a hash into a Bulletproof. The benefit as mentioned above is that it does
not depend on the range proof implementation, but it does require more changes due to the change in the kernel computation which is the
downside of this approach.