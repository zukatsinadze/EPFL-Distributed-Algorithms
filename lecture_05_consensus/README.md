# Lecture 05 - Consensus



* C1. Validity: Any value decided is a value proposed
* C2. Agreement: No two correct processes decide differently
* C3. Termination: Every correct process eventually decides
* C4. Integrity: No process decides twice


## Algorithm 1
Non-uniform
Using perfect failure detector. 
P-based (fail-stop) consensus algorithm.


```
Implements: Consensus (cons)
Uses: BestEffortBroadcast (beb), PerfectFailureDetector (P)

upon event <Init> do
    suspected = set()
    round = 1
    currentProposal = None
    broadcast = false
    delivered[] = false

upon event <crash, pi> do
    suspected.add(pi)

upon event <Propose, v> do
    if currentProposal = None:
        currentProposal = v

upon event <bebDeliver, p_round, value> do
    currentProposal = value
    delivered[round] = True

upon event delivered[round] = True or p_round in suspected
    round += 1

upon event p_round = self and broadcast = false and currentProposal is not None:
    trigger <Decide, currentProposal>
    trigger <bebBroadcast, currentProposal>
    broadcast = True
```

