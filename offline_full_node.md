# Offline full node

Below is a simple description of a node that sync to the blockchain, prunes data and disappears. Such node ends up having only the UTXO set size (range proofs are dropped once validated). It's not healthy for the network
to have a node that only leeches data without giving back, but unfortunately some devices will have a hard time not prunning data due to their limit on storage capacity and/or bandwidth.
Such devices can bootstrap their full node from the beginning and then drop off the network and instead rely on a third party to provide them with new blocks e.g. a website that provides
block data for everyone. There is no trust involved in this because the device will still be able to fully validate everything. A possible approach to achieve this is:

1. Given kernel `A` and `B` once the node validates their signature it can sum together the kernel public keys and forget about anything else. This means the kernels can essentially be reduced
   to a single public key. The reasong for this is that the node has validated every kernel curve point so it knows their sum is also valid and it can 'pretend' it has a sig for it
2. Given a sequence of UTXOs a node validates each rangeproof and throws it away after validation which means it only keeps the Pedersen commitment
3. The node keeps only a single kernel public key (you'd really need all kernels in a given window to handle reorgs/NRD compatibility) and all Pedersen commitments which have been validated.
4. Once the node is at the HEAD of the chain it drops off the Grin network and connects to a third party service to obtain new blocks. Once a new block is received from the third party, it
   assumes the inputs have a valid rangeproof if they are in node's set of Pedersen commitments. The range proofs of outputs still need to be validated which is also a part of the block
   validation. If the block is valid, the node updates its set of pedersen commitments by adding the new outputs (without range proofs) and removing commitments that have been spent (again,
   you'd need to keep a window of these changes to account for reorgs)
5. Since the node is still doing full validation it doesn't require any more trust than a regular node.

# Why

Grin already has a smaller size than Bitcoin and achieves better privacy. Most of the data will be kernels that need to stay on the chain and the range proofs of UTXOs. By prunning both of them
we might be able to make a full node validation available to a wider range of devices e.g. mobile phones.
