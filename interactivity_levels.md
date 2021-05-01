# Transaction interactivity levels

Let's define two layers of interactivity:
1. **Network/protocol** - Transaction creation between two parties requires exchange of information over the wire
2. **User/UX** - Transaction creation between two parties requires user interaction usually in the form of transaction acceptance confirmation

Bitcoin has noninteractive transactions which don't have Network or User level interactivity (at least not by default).
Mimblewimble comes with interactivity at the protocol level which means that the transaction creation process _MUST_ be interactive on the network layer, but we may be able to avoid interactivity on the UX layer. Let's call this User level _noninteractivity_ **U**ser**NITX**. Similarly, we name user level _interactivity_ **UITX**. Grin performs an automated receive today which makes it a UNITX flow. This document tries to describe the difference between these flows as are understood by some. The table below likely contains mistakes and is probably missing a couple of points. It's a table that will be updated over time.

| Action | UITX | UNITX (Grin) | NITX (Bitcoin) |
| --- | --- | --- | --- |
| Full wallet control | ✓ | X | X |
| Payment proofs (value commitment) | ✓ | ✓ | ✓ |
| Payment proofs (content commitment) | ✓ | X | X |
| Custom payjoins | ✓ | X | X |
| Unified SRS/RSR flow | ✓ | X | / |
| JIT seed decryption (fingerprint/password) | ✓ | X | / |
| Doesn't require scanning the chain upon opening the wallet | ✓ | ✓ | X |
| Wallet level labeling of created outputs | ✓ | X/✓ | X |
| No permanent address spam issue | X | ✓ | ✓ |
| Doesn't require communication between parties | X | X | ✓ |
| Users already understand this | X | ✓ | ✓ |

Explanations:
1. **Full wallet control** - In NITX setting, the receiver can only control which outputs are spent, but has no control over which are received. This opens a lot of unwanted state injections like dusting attacks or "dark/illegal" outputs to the wallet. Controlling the incoming traffic is easier to reason, easier to regulate and eliminates these issues completely at the cost of a click on "Confirm" and a little wait time for the other party. This wait time may require asynchronous transaction building which would mean that the party performing step1 would need to add the Tor address to the slatepack.
2. **Payment proofs (value commitment)** - The sender can prove the value was sent to the receiver by showing the payment proof.
3. **Payment proofs (content commitment)** - UITX allows for a more powerful version of payment proofs which allows the transaction parties to commit to an arbitrary statement/document by signing the hash of the document. Signing documents requires their review prior to signing which makes it only possible in the UITX setting. An example of such a document could be an actual invoice from a store.
4. **Custom payjoins** - The receiver could pick manually which input should be contributed for a payjoin transaction in SRS flow
5. **Unified SRS/RSR flow** - RSR flow _requires_ user-level interactivity. This means that if SRS has UITX flow, the two become very similar.
6. **JIT seed decryption** - The receiver needs to be able to build an output. To minimize the time the seed is in memory, the user could just-in-time decrypt the seed to create the output. This could be done either by entering a password or through a fingerprint on mobile phones.
7. **Doesn't require scanning the chain upon opening the wallet** - We usually need to scan the outputs to see if they belong to us. If our wallet created the output and then sent it off to the chain, then we never require scanning whether the outputs on the chain are ours (unless we reuse the seed on a different device). We can simply check if the outputs we created exist on the chain.
8. **Wallet level labeling of created outputs** - The receiver could label the output on the wallet level at the output creation time. The current UNITX could also achieve this if the label information was added to a one-time grin address when it was generated.
9. **No permanent address spam issue** - Since UITX _requires_ manual confirmation, this means that someone could spam the wallet by repeatedly sending step1. This can be solved with one-time addresses (I believe this does not require multiple Tor addresses).
10. **Doesn't require communication between parties** - Creating a protocol-level ITX transaction requires some form of contact between the sender and the receiver. This can only be avoided through pure NITX.
11. **Users already understand this** - This isn't a flow people are used to in the crypto world because ITX.

ITX are generally thought of as strictly inferior transactions. My point is that it is at least not obvious as to why that would be the case. ITX have their own unique properties that may be worth exploring further.

## Exchanges

UITX is possible even for 3rd party services. The service would need to use the flow such that the user always has step2 which is done manually. An example would be the user selecting which items they want to buy, the exchanges provides the document and memo to sign in step1 which are both manually confirmed and signed by the user at step2.