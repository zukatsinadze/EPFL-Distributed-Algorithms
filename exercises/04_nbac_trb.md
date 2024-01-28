1) Devise an algorithm that, without consensus, implements a weaker specification of
NBAC by replacing the termination property with

Weak termination: Let p be a distinguished process, known to all other
processes. If p does not crash, then all correct processes eventually decide.


Your algorithm may use a perfect failure detector.


NBAC1. Uniform Agreement: No two processes decide differently
NBAC3. Commit-Validity: 1 can only be decided if all processes propose 1
NBAC4. Abort-Validity: 0 can only be decided if some process crashes or votes 0



2) Devise an algorithm that, without consensus, implements a weaker specification of
NBAC by replacing the termination property with

Very weak termination: If no process crashes, then all processes decide.

Is a failure detector needed to implement this algorithm?


NBAC1. Uniform Agreement: No two processes decide differently
NBAC3. Commit-Validity: 1 can only be decided if all processes propose 1
NBAC4. Abort-Validity: 0 can only be decided if some process crashes or votes 0



3) Can we implement TRB with an eventually perfect failure detector ◇P, under the
assumption that at least one process can crash?


Assume we have implemented such algorithm.

we have src, m and some correct process p

Run 1: src crashes before sending message. p delivers phi at time t.
Run 2: p suspects src which is correct, but all messages are delayed until time t+.

A and B are indistinguishable for p so it delivers phi in B as well. By validity src should deliver m, but by agreement we have contradiction.




4) Design an algorithm that implements consensus using multiple TRB instances.

- Every process uses TRB to broadcast its proposal.
- Let p be any process, eventually every correct process either delivers p’s proposal or ⊥ (if p fails).
- Eventually, every correct process has the same set of proposals (at least one is not ⊥, since not every process crashes).
- Processes use a shared but arbitrary function to extract a decision out of the set of proposals (e.g., sort alphabetically and pick the first)


upon event <init>
  proposals = set()
  delivered = set()
  correct = S

upon event <crash, p>
  correct = correct \ {p}

upon event <propose, v>
  trigger <trbBroadcast, v>

upon event <trbDeliver, src, v>
  if src not in delivered:
    delivered.add(src)
    proposals.add(v)

upon delivered is superset of correct:
  trigger<decide, sort(proposals)[0]>

