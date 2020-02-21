# A possible way to improve MW transaction privacy

I'll start by saying that this is just an idea and nothing any MimbleWimble protocol needs to implement. Below, I'll explain different concepts that can be done during a MimbleWimble interaction. They build one on top of another, so it's important to go in the order they are presented. It's also worth mentioning that they work best when there is no transaction kernel overhead created when a transaction is created. By that I mean
that if it is possible to aggregate all the kernels into a single one, these schemes work best because they create multiple transactions and it's best if this does not come at a cost of blockchain bloat.

1. [ObscuroTX](./ObscuroTX.md)
2. [ObscuroDance](./ObscuroDance.md)
3. [ObscuroJoin](./ObscuroJoin.md)

Hopefully there can be many more interesting flows we can do or ideas we can join.