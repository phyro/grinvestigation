# Schnorr signatures

**NOTE: This was not checked by a cryptographer so it likely contains flaws!**

## The problem

Alice wants to prove to Bob that she knows the private key of a public key `P`. The simplest way to prove this would be to show scalar `p` such that <code>P = p*G</code>. The proof is sound, but it shares the ownership of `P` with Bob since now also Bob knows the private key. Instead of revealing the private key, it would be much more useful if we managed to provide a zero knowledge proof which would prove we know the private key of `P`. Digital signatures are a zero knowledge proof of knowledge of private key and we'll go through the simplest signature scheme called Schnorr signature.

## Schnorr signatures

Suppose Alice wants to prove she knows the private key of `P` without revealing her private key `p`. Instead of proving `P`, she could prove she knows the private key for `P + R` where `R` would be some random curve point Alice made up. So she could generate a random curve point `R = r*G` and instead of showing the private key `p`, she shows the private key `s = p + r` and `R`. We call this `R` a nonce because it can only be used once - if `R` was reused in signatures, people could compute the private key `p`. Given a pair `<R, s>`, Bob can verify that `P + R = s*G` holds. This would be a super neat proof, but there's a huge vulnerability in this scheme. Adversarial Alice could pick `R` by starting with a curve point `-P`, choose a random scalar `r` and set `R` as `R = -P + r*G`. She then computes `s = r`. What happens now is that when Bob receives `<R, s>`, the `P + R = s*G` becomes
```
   P + (-P + r*G) = s*G
=> P - P + r*G = s*G
=> r*G = s*G  # holds!
```
and since `s = r` the equation holds and Alice didn't need to know the private key of `P` because the `P` curve points cancelled out!

To solve this issue, we make a signature a multi-step process. First Alice picks a random nonce `R = r*G` and sends `R` to Bob. Bob now picks a random scalar `e` and sends it to Alice, telling her he wants a proof not for `P`, but for `P' = e*P` - note that if Alice knows the private key for `P`, she will also know the private key for `e*P`. This means that Alice now needs to compute `s` as `s = e*p + r` and share it with Bob. Note that this still doesn't leak any information about `p` because it is blinded by `r`. Bob upon receiving `s` has a pair `<R, s>` for which it can check that `P' + R = s*G` holds. The key here is that Alice commited to `R` by sending it first to Bob so even if she subtracted `P` from `R`, she'd need to guess which `e` Bob would pick to subtract it `e` times. This scheme is now secure against public key cancellation. However, we have made it a multi-step process which requires an interaction between Alice and Bob. The steps that we introduced require Alice committing to `R` and then getting a random scalar `e`. This can be done without Bob by using hash functions. Alice can commit to `R` by computing `e = H(R)` which can be thought of as a simulation of Bob steps because Alice first commits by adding `R` as hash input and only then receives a random scalar `e`. We now have a way to construct a signature without needing any interactive, but with a new assumption that the hash function used is safe.

Usually the signatures sign a message in which case Alice computes `e = H(R | M)` where `M` is the message. Bob knows `R` and the message `M` and can compute `e` to verify the signature is valid.

## Schnorr multi-sig

What if Alice owned public key `P_a = p_a*G` and Bob owned public key `P_b = p_b*G` and they wanted to create a signature where they both sign a message `M`? They could both construct a signature on their own and we'd have 2 signatures for the same message. We can do better and produce a single signature which proves both parties signed the message. We need to construct a proof that we know the private key for curve point `P = P_a + P_b`. We start off with Alice doing the exact same thing as she did in regular signature which is to pick a random nonce `R_a = r_a*G`, but then she asks Bob for his public nonce `R_b`. Alice can then compute `e = H(R_a + R_b | M)` and `s_a = e*p_a + r_a`. Note that we added a suffix `_a` to the public and private keys created by Alice and similarly suffix `_b` to those created by Bob. She now sends `<R_a + R_b, s_a>` to Bob. This signature is not valid because the equation `P = s*G` doesn't hold for `P = P_a + P_b`. Let's take a look at what is missing from our signature
```
   P_a + P_b + R_a + R_b = e*(p_a+p_b) + r_a + r_b
=> P_a + P_b + R_a + R_b = e*p_a + e*p_b + r_a + r_b
```

We know that Alice produced `s_a = e*p_a + r_a` so we only need to add `e*p_b + r_b` to `s_a` to produce a valid signature. Bob knows the values we lack so he can compute `s_b = e*p_b + r_b` and compute the final signature as `<R_a + R_b, s_a + s_b>`. This pair is a valid multisignature of Alice and Bob for message `M`.

Note that the multisig is indistinguishable from a single-sig. They are both just a pair `<R, s>`.


## Batch verification

By now we know that we can validate Schnorr signature by checking if `e*P + R = s*G`. Consider we have 100 signatures of this form
```
e1*P1 + R1 = s1*G
e2*P2 + R2 = s2*G
...
e100*P100 + R100 = s100*G
```

It seems like we can sum up all the left hand side and all the right hand side and check that the following equation is still well balanced
```
e1*P1 + R1 + e2*P2 + R2 + ... + e100*P100 + R100 = (s1 + s2 + ... + s100)*G
```
However, this is vulnerable to attacks now because signatures can help hide each other's flaws! An example of this would be to construct 2 signatures
```
e1*P1 + R1 + e2*P2 + R2 = (s1 + s2)*G
```
such that the attacker picks as `P1` a point they don't know the private key for and then sets `R2 = -e1*P1 + u*G`. This would expand to
```
   e1*P1 + R1 + e2*P2 -e1*P1 + u*G = (s1 + s2)*G
=> R1 + e2*P2 +u*G = (s1 + s2)*G
```
and they wouldn't need to know the private key of `P1` to validate these two in batch! To solve this we can [pick a random number for every signature equation](https://github.com/mimblewimble/secp256k1-zkp/blob/master/src/modules/schnorrsig/main_impl.h#L301-L304) which prevents us from knowing the relation between the two signatures and makes the above attack impossible. So instead of summing the original equations, we generate a random number for every equation and multiply both sides by it. Then we can sum them up and check that the equality holds.

Bonus: As an example, we can show how this could lead to inflation bug in Mimblewimble if we are not careful. The public key we sign is a Pedersen commitment which should have a form `0*H + x*G`. However, we could make it inflate e.g. `10*H + x*G` and then we could construct two signatures
```
e1*(10*H + x*G) + R1 != s1*G               # not equal! We don't know the private key for 10*H + x*G
e2*(0*H + y*G) + (-e1*(10*H) + y*G) = s2*G   
```
Summing these up would make the inflated `10*H` disappear and I think we could produce a valid combined signature where equality of both signatures equations summed up would hold because we'd know the other private keys needed and could hence compute the correct sum of `s` values.


## More funny business

Commiting to `H(R | M)` leaves some creativity for some people. An attacker could convince someone they know the private key for point `P` when in fact they did not. The attacker first chooses message `M` and some curve point `R`, computes the challenge `e = H(R | M)` and then sets `P` as `P = -e^(-1)*R + u*G`. Now when we compute the `e*P + R` we get
```
   e*(-e^(-1)*R + u*G) + R
=> -R + e*u*G + R
=> e*u*G
```
note that we only need to know the private key `u` so we can set `s = e*u` and it validates. We don't need to know the private key of either P or R!
We can avoid this attack by committing also to `P` in the hash `H(R | M | P)`. Now we can't pick `P` after the message because the message commits to it.

Thanks to @kurt for explaining the attack.
