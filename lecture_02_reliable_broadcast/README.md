# Lecture 02 - Reliable Broadcast


## Best Effort Broadcast

Specification:

Events:
Request: <bebBroadcast, m>
Indication: <bebDeliver, src, m>
Properties: BEB1, BEB2, BEB3


* BEB1. Validity if pi and pj are correct then every message broadcasted from pi is 
eventually delivered by pj.
* BEB2. No duplication: no message is delivered more than once.
* BEB3. No creation: No message is delivered unless it was broadcasted.

BEB1. liveness
BEB2. safety
BEB3. safety


### Algorithm

```
Implements: BestEffortBroadcast
Uses: PerfectLinks (pp2p)

upon event <bebBroadcast, m>  do
    for pi in S do
        trigger <pp2pSend, pi, m>

upon event <pp2pDeliver, pi, m> do
    trigger <bebDeliver, pi, m>

```

### Proof 
BEB1. Validity: by the validity property of perfect links and the fact that 
every message is broadcasted using pp2pSend, we know for sure that every 
correct process will pp2pDeliver m, hence it will bebDeliver it as well.

BEB2. No duplication: By the validity property of perfect links.

BEB3. No creation: By the no creation property of perfect links.


## Reliable Broadcast

Strong form of the best effort broadcast
* RB1 = BEB1
* RB2 = BEB2
* RB3 = BEB3
* RB4. Agreement: for any message m, if a correct process delivers m, then every correct process 
delivers m.

Key difference from BEB: if sender machine failed and at least one reciever delivered message, then
all of them must.

### Algorithm without Failure Detector
``` 
Implements: ReliableBroadcast
Uses: PerfectLinks (pp2p)

upon event <Init> do
    delivered = set()

upon event <rbBroadcast, m> do
    for pi in S do
        trigger <pp2pSend, pi, m>

upon even <pp2pDeliver, pi, m> do
    if m not in delivered:
        delivered.add(m)
        trigger <bebDeliver, pi, m>
        trigger <rbBroadcast, m>
```

### Algorithm with Perfect Failure Detector (very strong assumption...)

```
Implements: ReliableBroadcast
Uses: PerfectLinks (pp2p), PerfectFailureDetector (P)

upon event <Init> do
    delivered = set()
    correct = S (set of all processes, people know each other.)
    for pi in S: do from[pi] = set()

upon event <rbBroadcast, m> do
    delivered.add(m)
    trigger <rbDeliver, self, m>
    trigger <bebBroadcast, [Data, , m] 

upon event <crash, pi> do
    correct.remove(pi)
    for (pj, m) in from[pi] do
        trigger <bebBroadcast, []>

upon event <bebDeliver, pi, [Data,pj, m]>
    if m not in delivered:
        delivered.add(m)
        trigger <rbDeliver, pj, m>
        if pi not in correct:
            trigger <bebBroadcast, [Data, pj, m]>
        else
            from[pi].add([pj, m])

```
There are two cases when I relay the message. First in crash, when failure detector tells us that pi is crashed
and second in bebDeliver. 

Question: Why do we need to do it both times?
Let's say we do not relay in crash, then it might happen that I delivered message from pj, after some time pj crashed and 
there is some other process who didn't deliver the message, since when I checked for correctness during delivery pj was still correct.

Let's say we do not relay during bebDeliver, then it might happen that I pj sent a message to me (it is still not recieved), then it crashed 
and I got crash event, but at that time I don't have any messages to relay and only after that I receive a message from pj. 


### Proof
RB1, RB2, RB3: same as before.

RB4. Agreement. For any message m if correct process deliveres m, then every correct process must deliver m.
Assume correct pi delivered message from pj. If pj is correct, then by BEB1 all correct processes delivered m.
If pj crashes, then by completness property of PerfectFailureDetector, pi will detect the crash and bebBroadcast 
m to all. Then since pi is correct by BEB1 property all correct processes will bebDeliver m, hene rbDeliver m.


Notice that if our failure detector falsly detects some crashes it is still ok, we will just relay some messages
unnecessarily, but all the properties still hold.



## Uniform Broadcast

Even stronger form of broadcast.

* URB1 = BEB1
* URB2 = BEB2
* URB3 = BEB3
* URB4. Uniform Agreement: if a process delivers m, then every correct process delivers m

Key difference: even if crashed process delivered m, then all the other remaning correct
processes should deliver it.

Processes should not deliver messages too early.

### Algorithms

```
Implements: uniformReliableBroadcast
Uses: BestEffortBroadcast (beb), PerfectFailureDetector (P)


upon event <Init> do
    correct = S
    delivered = set()
    forward = set()
    ack[Message] = set


upon event <crash, pi> do
    correct.remove(pi)

upon event <urbBroadcast, m> do
    forward.add([self, m])
    trigger <bebBroadcast, [Data, self, m]>

upon event <bebDeliver, pi, [Data, pj, m]>
    ack[m].add(pi)
    if [pj, m] not in forward:
        forward.add([pj, m])
        trigger <bebBroadcast, [Data, pj, m]>;


upon event <for any [pj, m] in forward: correct is subset of ack[m] and m not in delivered> do
    delivered.add(m)
    trigger <urbDeliver, pj, m>;


```

### Proof
URB2, URB3: follow from BEB2, BEB3

Lemma: if a correct process pi bebDelivers a message m, then pi urbDelivers message m.
Consider the process that delivers m, then it will bebBroadcast m, by the validity property of 
BEB, there is a time at which pi bebDelivers m from every correct process and hence urbDelivers m.


URB1. Validity: If a correct process delivers m, then it bebBroadcasts and bebDelivers, so by the lemma
it urbDelivers m.


URB4. Agreement: Assume some process pi urbDelivers m. By the completness and accuracy properties
of failure detector, every correct process bebDelivers m. By our lemma, every correct process will
urbDeliver m.


