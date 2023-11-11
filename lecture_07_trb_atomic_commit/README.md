# Terminating Reliable Broadcast

There is no consesus in completly assynchronous system.
We need failure detector.


TRB is strictly stronger than uniform reliable broadcast.

With (Uniform) Reliable Broadcast if sender fails there are no guarantees.
Recievers might or might not get the message.


TRB guarantees that if message doesn't get through, then recievers deliver special Phi
So broadcast terminates somehow. You don't have to wait forever anymore.


The problem is defined for a specific broadcaster source pi (known by all)
Process src is supposed to broadcast message m (other than phi)
The other processes need to deliver m if src is correct but may deliver phi if src crashes


* TRB1. Integrity: If a process delivers m, then either m is phi or m was broadcasted by src
* TRB2. Validity: If the sender src is correct and broadcasts m, then src eventually delivers m
* TRB3. (Uniform) Agreement: For any message m, if a correct (any) process delivers m, then every correct 
    process delivers m.
* TRB4. Termination: Every correct process eventually delivers exactly one message.


Events
 Request: <trbBroacast, m>
 Indication: <trbDeliver, p, m>

## Algorithm (trb)
```
Implements: trbBroadcast 
Uses: BestEffortBroadcast(beb), PerfectFaiureDetector (P), Consesus (cons)

upon event <Init> do
    prop = None
    correct = S

upon event <trbBroadcast, m> do
    trigger <bebBroadcast, m>

upon event <crash, src> and prop = None do
    prop = phi

upon event <bebDeliver, src, m> and prop = None 
    prop = m

upon event prop != None do
    trigger <Propose, prop>

upon event <Decide, decision> do
    trigger <trbDeliver, src, decision>
```

Is Perfect Failure Detector neccessary? 

Is there an algorithm that uses TRB to implement P? (i.e. are they equivalent problems)

Every process pi starts broadcasting infinite number of instances of TRB where it is sender
If a process pk delivers Phi_i pk suspects pi 

By TRB properties we know that pk will deliver Phi_i only if pi failed (strong accuracy)
Furthermore if pi fails pk will deliver Phi_i (soundness)

so we got perfect failure detector.

This proves that we need perfect failure detector for TRB. 





# Non-Blocking Atomic Commit: An agreement problem

Set of processes need to decide on 0 or 1.

Transaction is an atomic program describing a sequence of accesses to shared and distributed information.

A transaction can be terminated either by commiting or aborting.



As in consesus, every process has initial value 0 (no) or 1 (yes) and must decide on final value.


The proposition means the ability to commit the transaction

The decision reflects the contract with the user (I promise that the operation has been executed on me is going to be)


Unlike consesus, the processes here seek to decide 1 but every process has a veto right.



* NBAC1. Agreement: No two processes decide differently
* NBAC2. Termination: Every correct process eventually decides
* NBAC3. Commit-Validity: 1 can only be decided if all processes propose 1
* NBAC4. Abort-Validity: 0 can only be decided if some process crashes or votes 0


We can do 2-Phase commit. There will be an initiator which will collect all votes, decide and send decision to everyone. 
BUT, this will be blocking if initiator crashes.


## Non-blocking algorithm
Events
    Request <Propose, v>
    Indication <Decide, v'>

```
Implements: NonBlockingAtomicCommit(nbac)
Uses: PerfectFaiureDetector, BestEffortBroadcast, UniformConsesus

upon event <Init> do
    prop = 1
    delivered = set()
    correct = S

upon event <crash, pi> do
    correct.remove(pi)

upon event <Propose, v> do
    trigger <bebBroadcast, v>;

upon event <bebDeliver, pi, v> do
    delivered.add(pi)
    prop = prop * v (if it was zero keep zero)

upon event correct \ delivered == empty do
    if correct != S
        prop = 0
    trigger <uncPropose, prop>

upon evemt <uncDecide, decision> do
    trigger <Decide, decision>
```


Do we need perfect failure detector P?

1. Eventually perfect <>P is not enough
Need to build counter example.

If we have <>P, then it might happen that before it becomes perfect, all processes propose 1, but some of them
are tricket into suspecting p1, so they will decide 0. 


2. P is needed if one process can crash




