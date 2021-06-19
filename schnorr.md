# Schnorr signatures

## The problem

Alice wants to prove to Bob that she knows the private key of the public key `P`. The simplest way to prove this would to be to show scalar `p` such that <code>P = p*G</code>. The proof is sound, but it shares the ownership of `P` with Bob since now also Bob knows the private key. Instead of revealing the private key, it would be much more useful if we managed to provide a zero knowledge proof which would prove we know the private key of `P`. Digital signatures are a zero knowledge proof of knowledge of private key and we'll go through the simplest signature scheme called Schnorr signature.

## Schnorr signatures

Suppose Alice wants to prove she knows the private key of `P` without revealing her private key `p`. Instead of proving `P`, she could prove she knows the private key for `P+R` where `R` would be some random curve point Alice made up. So she could generate a random curve point `R = r*G` and instead of showing the private key `p`, she shows the private key `s = p + r` and `R`. We call this `R` a nonce because it can only be used once - if `R` was reused in signatures, people could compute the private key `p`. Given a pair `<R, s>`, Bob can verify that `P+R = s*G` holds. This would be a super neat proof, but there's a huge vulnerability in this scheme. Adversarial Alice could pick `R` by starting with a curve point `-P`, choose a random scalar `r` and show the `R` as `R = -P + r*G`. She then computes `s = r`. What happens now is that when Bob receives `<R, s>`, the `P + R = s*G` becomes
```
   P + (-P + r*G) = s*G
=> P - P + r*G = s*G
=> r*G = s*G  # holds!
```
and since `s = r` the equation holds and Alice didn't need to know the private key of `P` because the `P` curve points cancelled out!

To solve this issue, we make a signature a multi-step process. First Alice picks a random nonce `R = r*G`. Then, she sends `R` to Bob. Bob now picks a random scalar `e` and sends it to Alice, telling her he wants a proof not for `P`, but for `P' = e*P` - note that if Alice knows the private key for `P`, she will also know the private key for `e*P`. This means that Alice now needs to compute `s` as `s = e*p + r` and share it with Bob. Note that this still doesn't leak any information about `p` because it is blinded by `r`. Bob upon receiving `s` has a pair `<R, s>` for which it can check that `P' + R = s*G` holds. The key here is that Alice commited to `R` by sending it first to Bob so even if she subtracted `P` from `R`, she'd need to guess which `e` Bob would pick to subtract it `e` times. This scheme is now secure against public key cancellation. However, we have made it a multi-step process which requires an interaction between Alice and Bob. The steps that we introduced require Alice committing to `R` and then getting a random scalar `e`. This can be done without Bob by using hash functions. Alice can commit to `R` by computing `e = H(R)` which can be thought of as a simulation of Bob steps because Alice first commits by adding `R` as hash input and only then receives a random scalar `e`. We now have a way to construct a signature without needing any interactive, but with a new assumption that the hash function used is safe.

Usually the signatures sign a message in which case Alice computes `e = H(R | M)` where `M` is the message. Bob knows `R` and the message `M` and can compute `e` to verify the signature is valid.