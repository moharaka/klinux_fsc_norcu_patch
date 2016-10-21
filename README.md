This patch removes the direct use of "RCU" from the path_lookup 
operation. The goal is to avoid the potential memory over-consumption 
of RCU. This can happen since RCU delays the liberation (freeing) of 
data structures.

The new solution is based on a lock-free algorithm that has been
described in a French paper [1], that suggests using generation counters
(GC) in place of RCU, to gain in memory consumption. However, rather than
using GC we reuses seqlock to implement the algorithm. This latter
check at each node (inodes or dentry) that the seqlock has not changed
value to confirm that walk was safe.

The implementation was aided by the fact the idea behind the algorithm
was already implemented, but for another reason: to detect the
concurrent rename operations (see Documentation/filesystem/path_lookup.txt).
There is only some keys points that has been changed, like when allocating
and deallocating nodes.

Still the algorithm requires the use of RCU at the SLAB allocator layer.
This should not be a problem since changing the memory type is less
frequent than the removal of item from the cache.

First experiments on Qemu with Tmpfs indicates no real differences in memory
consumption nor performance. More experiments are needed, maybe on real
time Linux where the RCU stalling seems to be more frequent.

[1]: Mohamed Lamine Karaoui, Quentin L. Meunier, Franck Wajsbürt, Alain
Greiner. Mécanisme de synchronisation scalable à plusieurs lecteurs et
un écrivain. Conférence en Parallélisme, Architecture et Systèmes,
ComPAS 2014, Apr 2014, Neuchâtel, Suisse
