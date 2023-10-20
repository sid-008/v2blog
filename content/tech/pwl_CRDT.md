---
title: Notes from the PWL October meetup
date: 2023-10-19
description: Keep CALM and CRDT on!
---

These are my notes from the October Papers We Love meetup hosted by Madhav Jivrajani.
The paper discussed was "Keep CALM and CRDT on!"

# What is coordination

- "Knowledge is the dual of possibility"
- There's also message re-ordering, network partitions and all other flavours of why dist sys is hard.
- Other mechanisms like 2PC(2 phase commit) exist.
- coordination mechanisms are a way to sync access to a shared memory of some sort. Probably the most well studied class of algos in dist sys literature
- Coord mechanims have massive performance costs attached to them.
- Intuition from universal scalability law(USL)
- Linear scalability is a scam
- As work is done to achieve data consistency, it starts to bottleneck your system's throughput.

# Downside of coordination

- Coordination avoidance in db systems [paper]
- Anna: a kvs for any scale [paper]

# Can we avoid coordination?

- A significant amount of non-determinism exists in dist sys.
- uncoordinated parallel exec on unreliable machines, message order delivery, network partitions etc
- In an attempt to tame non-determinism we try and Coordinate and accumulate as much Knowledge about what the global system might look like
- Coordination done in hope of some sort of guarantees.

  - Recency guarantees
  - Ordering guarantees

- One way to avoid coordination in txn dbs is using invariants. If a local transaction can be shown to not violate a golabl invariant we can avoid coodinating on this txn
- Invarant confluence
- But this is just for transactional dbs, how do we generalise more?
- Ultimately coordination is to achieve memory consistency
- Mem consistency is violated by all the non-determinism
- What if we shift our focus from memory consistency to something called application level consistency?
- Can our program produce deterministic outputs despite non-determinism in the distributed runtime?
- This is: program confluence
- Clarification: avoiding coordination does not mean machines do not talk to each other
- Machines communicate periodically, kind of like gossip
- For each request, a blocking potentially sequential, throughtput reducing operations are not done.

### Example-1 - distributed deadlock detection

- Refer CALM paper
- As and when local cycles develop, can a local deadlock detector confidently declare a deadlock?
- Turns out, it can! But what about race conditions? Do you need to coodinate with other nodes before declaring a deadlock?
- No need to coordinate, any decision based on partial/local state is still valid. Partial information is kind of an under approx of the global state.

### Example-2 distributed garbage collection

- Goal is to find objects that are disconnected from the "root".
- May be that on another node the object is connected to the root transitively.
- The local case in this case is not an under approximation of the global state.

# CALM

- consistency as logical monotonicity.
- A program has a consistent, coordination free distributed implementation if and only if it is monotonic.
- True from a philosophical pov, (see: non-monotonic logic)
- Formal definition: -DO-
- Need for coordination arises from an intrinsic need to gather missing info.
- As a result monotonic programs are safe in the face of missing info. The oppposite is not true however.
- non-monotonic programs on the other hand tend to "change their mind" in the face of new info. They need to ensure that they know the global state before taking any decisions.

# CRDTs

- conflict free replicated datatypes.
- Replicated structures that provide guarantees to be eventually consistent without the need for coordination
- These gossip(either data or logs) among each other
- Keyword: gossip
- ACI: associative, commutative, Idempotent
- Look into hasse diagrams- this shows how CRDTs can be monotonic, it can only grow up/down(see: cardinality counter)
- Helps deadl with non-determinism that comes with eventually consistent systems: re-ordering, duplication, late-arriving updates - ACI merge function handles that!
- A promise of formal safety guarantees(eventual consistency)

## A few gotchas

"Guaranteed to converge, all replicas are eventually consistent"

- This is a storage guarantee, state readers might get stale reads
- Cannot achieve sequential consistency without coordination
- Deletions from a set violate monotonicity, so how about a grow only set for just deletions?
- NO guarantees for safety when reading state. It gives guarantees for liveness.
- CRDTs provide schrodinger consistency guarantees
- Reads usually do not commute with other operations
- The reason reads do not commute is because the output of our read is not stable
- Because we need to sync access again, we end up back in coordination land
- In other words, without coordination we output not just stale but false info to our state readers.

# Keep CALM and CRDT on

- Can we develop a query model that makes it possible to precisely define when execution on a single replica yields consistent results?
- What all do we want?
  - Safety - sequential consistency
  - Efficiency
  - Simplicity
- Executing the query on local replica will always produce a sequential consistent result
- The true value of a query can never be changed once observed.
- Local state + some updates = global state
- Most importantly: you might read stale info but never wrong info.
- CALM was originally framed for logic programs.
- It applies perfectly well to CRDTs as well.
- We can deifine a monotone query as any whose output is monotone with respect to ordering of the CRDT.
- Monotone queries are exactly the queries that only need a local view of the system to be correct.
- CALMS says that only monotone queries that can satisfy this criteria of coordination avoidance.
- Monotone queries meet all criteria of our good query model.
- (look into PBS{probabistic Bounded staleness})

## what about non monotone queries?

- Not all business logic can be expressed monotonically
- Answer is simple- Coordinate!
- Here as well we can improve on coordination
- ALl update operations commute, you need to order sets of updates not sequences of them
- Contrast with paxos or raft, which enforces everyone, everywhere sees the same order no matter what.

#### TLDR; safe if monotone, unsafe if not monotone.

# What is the next step?

- Need to map query model to a practical language
- Need a rich expression that can manipulate CRDT state(lattice structure)
- Syntax that is easily understood
- Something along the lines of SQL
- With a query model and language, queries are just interfaces to the actual db
- Paper proposes a shift in perspective from an object oriented view of CRDTs to a db view of them;
- We break them up into a query model and a data store.

# what if we want a non-monotonic query?

- "pre-fetch" and "pre-coordinate"
- Sometimes you might just want weakly consistent systems, don't bother with coordination then
- TODO
- Look into "last write wins"
