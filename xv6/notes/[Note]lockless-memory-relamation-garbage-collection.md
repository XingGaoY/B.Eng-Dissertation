# Lockless Memory Reclamation and Garbage Collection

## Background
- **read/reclaim races**: two threads hold references to element `a` of a linked list, and T1 trys to remove a when T2 visit `a`'s field.
 `a` must be relaimed, but it's unsafe when T2 continue to reference it.
- Reclamation is subsumed into **automatic garbage collectors** such as Java, providing it thread-safe.

## Quiescent-state-based reclamation
- QSBR and EBR relaim memory once a **grace period** has passed.
- **grace period**: is a time interval [a, b] such that, after time `b`, all elements removed before time `a` can safely be reclaimed.
- QSBR uses **quiescent states** to detect grace periods. A quiescent state for thread T is a state in which T holds no reference to shared elements--in particular
, T holds no references to any shared elements which have been removed from a lockless data structure. Any interval of time in which **each thread passes through at
 least one quiescent state** is thus a grace period for QSBR.
> That is, all threads abandon their ref, the remove before the first one can be safe after the last one does that.
- No requirement that QSBR implementations find the shortest grace periods possible.
- **Fuzzy barrier**: barrier protects access to some code which no thread should execute before all other threads finish some prior stage of computation.
 A thread in a fuzzy barrier skips the protected code and continues executing, instead of blocking, if some other thread has not yet entered the battier.
 The thread will again attempt to execute the protected code upon subsequent fuzzy barrier entries.
- For implementation QSBR, a thread can enter the barrier when passing through a quiescent state, the protected code performs the memory reclamation.
- Some failure conditions.

## Epoch-based reclamation
- EBR uses grace period. But while QSBR relies on the programmer to annotate the program with quiescent states, EBR hides this bookkeeping within the impletation of lockless operations.
- The body of lockless operation is termed a **critical region**. Each thread sets a per-thread flag upon entry into a critical region. The thread clears this flag at the end of the lockless operation.
- No thread is allowed to access an EBR-protected object outside of a critical region, before the threshold of entries is met.
- Each thread executes in one of the three logical epochs and may lag at most one epoch behind the global epoch. Each epoch has an associated lombo list for elements awaiting reclamation.
- When a thread enters a critical region, it updates its local epoch to match the global epoch. After some predetermined number of critical region entries since changing its local epoch,
 a thread will attempt to increment the global epoch. This attempt will succeed only if the local epoch of each thread in a critical region is equal to the global epoch.

## sv6 gc
The mechanism is quite similar with EBR, and it is used as a substitute of RCU, quite similar with what rcu `reference` and `dereference` does right?

# Reference
- http://csng.cs.toronto.edu/publication_files/0000/0159/jpdc07.pdf