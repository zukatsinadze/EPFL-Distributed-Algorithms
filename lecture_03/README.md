# Lecture 03 - Casual Broadcast


So far we didn't care about ordering among messages.
Messages might have context.


Let m1 and m2 be any two messages: m1 -> m2 (m1 cassually precedes m2) if one of the following is true:

* C1 (FIFO Order). Some process pi broadcasts m1 before broadcasting m2
* C2 (Local Order). Some process pi delivers m1 and then delivers m2
* C3 (Transitivity). There is a message m3 such that m1 -> m3 and m3 -> m2

If I am broadcasting two messages that do not have to do anything with each other, 
in that case I as a programmer won't use same casual broadcast primitive. (There  can be different 
primitives for each topic...)


Events
Request: <coBroadcast, m>
Indication: <coDeliver, src, m>

Property:

CO: If any process pi delivers a message m2, then pi must have delivered every 
message m1 such that m1 -> m2.




## Reliable Causal Broadcast

Properties: RB1, RB2, RB3, RB4, CO


## Algorithm 1
```
Implements: ReliableCausalOrderBroadcast (rco)
Uses: ReliableBroadcast (rb)

upon event <Init> do
    delivered = set()
    past = set()

upon event <rcoBroadcast, m>
    trigger <rbBroadcast, [Data, past, m]>
    past.add([self, m])

upon event <rbDeliver, pi, [Data, pastm, m]> do
    if m not in delivered:
        for (sn, n) in pastm do
            if n not in delivered:
                trigger <rcoDeliver, sn, n>
                delivered.add(n)
                past.add([sn, n])

        trigger <rcoDeliver, pi, m>
        delivered.add(m)
        past.add([pi, m])
```
Sending past for each message: inefficient as fuck.

If we replace reliable broadcst with uniform reliable broadcast,
then algorithm will guarantee uniform causal order broadcast.

We can do some garbage collection.

```
Uses: ReliableBroadcast(rb), PerfectFailureDetector(P)

upon event <Init> do
    delivered, past = set()
    correct = S
    ack[m] = set() (for all m)

upon event <crash, pi>
    correct.remove(pi)

upon for some m in delivered: self not in ack[m] do
    ack[m].add(self)
    trigger <rbBroadcast, [ACK, m]>

upon event <rbDeliver, pi, [ACK, m]> do
    ack[m].add(pi)
    if for-all pj in correct: pj in ack[m] do
        past.remove([sm, m])
```

What happens if Failure Detector is faulty.
If correctness is violated (i.e. we do not remove somebody from the correct set)
If accuracy is violated (false positive, i.e. we removed someone from correct and we should not have.)

Does these two violate causality?


## Algorithm 2 

First algo was non blocking. So when I rbDelivered a message there was just some local computation and I always rcoDelivered it right away.
This algorithm does not send past and we do not need perfect failure detector, but it will be blocking.

Idea: somehow encode that we need to wait for x messages from some process before delivering a message.
Uses Vector Clocks.

```
Implements: ReliableCausalOrderBroadcast (rco)
Uses: ReliableBroadcast (rb)

upon event <Init> do
    for all pi in S: VC[pi] = 0
    pending = set()

upon event <rcoBroadcast, m> do
    trigger <rcoDeliver, self, m>
    trigger <rbBroadcast, [Data, VC, m]>
    VC[self] += 1

upon event <rbDeliver, pj, [Data, VCm, m]> do
    if pj != self then
        pending.add(pj, [Data, VCm, m])
        deliver-pending()

function deliver-pending() 
    while (s, [Data, VCm, m]) in pending 
        if for all pk: VC[pk] >= VCm[pk] do
            pending.remove(s, [Data, VCm, m])
            trigger <rcoDeliver, self, m>
            VC[s] += 1
```

Why do we put VC[pk] >= VCm[pk] and why not just ==?
First message will fail.

