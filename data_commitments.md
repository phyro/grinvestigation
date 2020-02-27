# A way to commit to additional UTXO data in the scope of a MW transaction

Let's say we want to commit to more data than a single Pedersen commitment `v*H + r*G`. David Burkett introduced an idea
that uses Bulletproofs to inject a hash in the 20 bytes that are available in the Bulletproof. This however makes the
protocol dependent on the specific range proof implementation Bulletproofs. It would be best if there was a way
to commit to additional data without relying on the range proof implementation. The reason for this is that if an alternative
to Bulletproofs comes, a switch to a different range proof solution would not cause us to lose a way to commit
to additional data.

## What it's trying to solve

This is not about commiting arbitrary data on the chain. This should only be considered when the protocol needs
more data than a single Pedersen commitment. Commiting to arbitrary data can be solved with [timestamping](timestamping.md).
Let's say that the protocol rules requires more than just a single Pedersen commitment for successful validation. For example the Noninteractive
approach that David showed requires some additional fields to be a part of the UTXO. The data commitment approach mentioned above
is one way to commit to additonal data that's needed to validate the UTXO and uses Bulletproofs to achieve this.

## Commiting to additional data using the transaction kernel

Let's say we want to commit to some `data` with a Mimblewimble UTXO. Along with the Pedersen commitment `P1 = v*H + r*G` we also show the `data` value which is the data we want to commit to.
This means our UTXO now holds two pieces of information `UTXO = [P1, data]`.

With the information from the `UTXO`, we can compute `P2 = v*H + r*G + hash(P1 | data)*G`.
When creating the kernel public key and signature, instead of using `P1` Pedersen commitments we use `P2` which can be computed from the UTXO. This makes the transaction kernel commit to the excess
which is a combination of values of the form `(r+hash(P1 | data))*G` instead of just `r*G`.
Only the owner of the Pedersen commitment knows the `r`, but everyone can compute the `hash(P1 | data)` because the `P1` and `data` are public. Swapping the `data` for a different one is
hard because one would need to create a different `data` which would produce the same hash `hash(P1 | data)` to keep the transaction kernel valid. Finding such data requires finding a hash collision. If someone
wants to spend the commitment they need to know the `r + hash(P1 | data)` and since the `P1` and `data` is public, everyone already knows the `hash(P1 | data)` part so the only thing an attacker would need to know is `r` which is the same requirement as in the current implementation.

This also means that the inflation check formula `sum(outputs) - sum(inputs)` needs to use `P2` commitments.

This does not use any more data than the idea that injects a hash into a Bulletproof. The benefit as mentioned above is that it does
not depend on the range proof implementation, but it does require more changes due to the change in the kernel computation which is the
downside of this approach.