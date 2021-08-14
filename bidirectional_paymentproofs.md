# Bidirectional payment proofs for Mimblewimble

<span style="color:red;font-weight:bold;">NOTE: The cryptography below has not been verified and it could be INSECURE! It needs to be proven correct before being used.</span>

The [existing payment proofs](https://github.com/mimblewimble/grin-rfcs/blob/master/text/0006-payment-proofs.md) only work for the SRS flow and additionally, only allow the sender to show the payment proof.

An improvement over this is [early payment proofs RFC](https://github.com/tromp/grin-rfcs/blob/early-payment-proofs/text/0000-early-payment-proofs.md) which allows us to create payment proofs also for the invoice (RSR) flow.

I think both of them only allow the sender to show the payment proof. So if we think of payment proofs as:

```
            sent to
sender ----------------> receiver

         received from 
sender <---------------- receiver

```
We see that we only get the payment proofs from the sender to the receiver. There's no way for the receiver to prove they received something from the sender. They only work in one direction.

Every payment in store comes with an invoice. The invoice serves as a proof for both the payer and the payee which allows them to prove the payer paid payee for some specific goods. It would be nice to have a payment proof that is simple and works in both ways.

## Bidirectional payment proofs

In order for both parties to end up with a payment proof, we likely want to produce the payment proof at step 2 so that when a party produces a signature, they already have the payment proof information.
Regardless of the flow, at step 2 we have both the sender's public nonce <code>R<sub>s</sub></code> and the receiver's public nonce <code>R<sub>r</sub></code>. This allows us to do the following.

#### Commitment
Suppose our payment proof has the following fields:
```
type
amount
timestamp
memo
```

To commit to this payment proof, we can compute a challenge for the nonces as <code>f = H(M' | R<sub>s</sub> | R<sub>r</sub>)</code> where `M'` is our new hidden message commitment which is our payment proof.

Instead of using <code>R<sub>s</sub></code> as the sender nonce, we use <code>R<sub>s</sub>' = f * R<sub>s</sub></code>. Similarly, instead of using <code>R<sub>r</sub></code>, we use <code>R<sub>r</sub>' = f * R<sub>r</sub></code> and sign for the final kernel nonce <code>R<sub>s</sub>' + R<sub>r</sub>'</code>.

We also need a proof that `Rs` is owned by the `sender_addr` and `Rr` by `receiver_addr`. This can be done by providing a simple signature with their addresses with nonce as the message. This is why we don't need the `sender_addr` and `receiver_addr` in the payment proof as these will have to be our witnesses.

#### Verification

Given <code>(M', X, R<sub>s</sub>, sig(R<sub>s</sub>, sender_addr), R<sub>r</sub>, sig(R<sub>r</sub>, receiver_addr))</code> where `M'` is the payment proof, `X` is the kernel commitment of some kernel on chain and the rest is the original nonces with signatures as proofs of nonce ownership, any party can prove this kernel committed to payment proof `M'` by computing <code>f = H(M' | R<sub>s</sub> | R<sub>r</sub>)</code> and showing that <code>f * R<sub>s</sub> + f * R<sub>r</sub> = R</code> where `R` is a part of the signature `(s, R)` for that kernel.