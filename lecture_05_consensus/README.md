# Lecture 05 - Consensus

processes propose values and agree on one of them.
output should be the same for all of us. It is not majority or something like that.


* C1. Validity: Any value decided is a value proposed
* C2. Agreement: No two correct processes decide differently (for uniform variant it would be no two processes)
* C3. Termination: Every correct process eventually decides
* C4. Integrity: No process decides twice


## Algorithm 1
Non-uniform
Using perfect failure detector. 
P-based (fail-stop) consensus algorithm.

There are n communcations rounds and each round has a leader.
Leader of a round decides its current proposal and broadcast it to all
Other processes either suspect the leader or wait for the value 


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

## Algorithm 2
How can we change the algorithm to make it uniform?
We do not deliver early. All the processes wait until the end.

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
    if round == N and decided = false:
        trigger <Decide, currentProposal>
        decided = True
    else:
        round += 1

upon event p_round = self and broadcast = false and currentProposal is not None:
    trigger <Decide, currentProposal>
    trigger <bebBroadcast, currentProposal>
    broadcast = True
```

TODO: prove correctness

Notice that perfect failure detector is very important for both of the algorithms. 


## Algorithm 3
We have to assume correct majority and eventually good failure detector <>P failure detector.

Basic idea: the processes alternate in the role of a phase coordinator until one of them
succeeds in imposing a decision.

We do not stop at round N. 


pi is leader in round k if k mod N is i
In such a round pi tries to decide. 

pi succeeds if it is not suspected
if pi succeeds, pi uses a reliable broadcast to send decision to all. Reliable is important: if pi crashes and just some processes deliver. 


