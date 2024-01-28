1) Implement a reliable broadcast algorithm without using any failure detector, i.e., using only BestEffort-Broadcast(BEB).

Uses: BestEffortBroadcast <beb>

upon event <init>
  delivered = set()

upon event <rbBroadcast, m>
  delivered.add(m)
  trigger <rbDeliver, self, m>
  trigger <bebBroadcast, [Data, self, m]>

upon event <bebDeliver, sender, [Data, src, m]>
  if m not in delivered:
    delivered.add(m)
    trigger <rbDeliver, src, m>
    trigger <bebBroadcast, [Data, src, m]>


2) The reliable broadcast algorithm presented in class has the processes
continuously fill their different buffers without emptying them. Modify it to remove (i.e. garbage collect) unnecessary messages from the buffers:
A. from, and
B. delivered


A. from can be cleaned once we relay messages when crash event happens.
B. delivered can't be cleaned.


3) What happens in the reliable broadcast and uniform reliable broadcast algorithms
if the:
A. accuracy, or
B. completeness
property of the failure detector is violated?


Reliable Broadcast:
A. Something was marked as failed, but it is not. It means that we remove it from correct processes and relay all the messages. Which is performance hit but is ok.
B. Something failed but we didn't discover the failure. It means that it is still in correct set. So it might happen that we recieved some message m from this p and other process didn't recieve this message. And since we think that p is correct we won't be relaying the message. Agreement violated.


Uniform Reliable Broadcast:
A. Something was marked as failed, but it is not. It might happen that this process didn't recieve message m, but we don't have it in our correct set anymore so we might deliver the message.
B. Something failed but we didn't discover the failure. We will get blocked since we will wait for ack from failed process.




4) Implement a uniform reliable broadcast algorithm without using any failure
detector, i.e., using only BestEffort-Broadcast(BEB)

We need to assume correct majority. And deliver when number of acks recived is majority.



5) Can we devise a broadcast algorithm that does not ensure the causal delivery
property but only (in) its non-uniform variant:
No correct process pi delivers a message m unless pi has already delivered every
message m1 such that m1 → m2?

Assume we have algorithm. So failing processes are allowed to deliver messages out of order. However process p doesn't know beforehand that it is going to fail. So it can't be treated in a special way from algorithm.



7) Can we devise a Best-effort Broadcast algorithm that satisfies the causal delivery property, without being a causal broadcast algorithm, i.e., without satisfying the agreement property of a reliable broadcast?

BEB + CO5 but no agreement.

This can happen only if p1 delivers some message m and it's not delivered by p2.
So sender of this message was faulty.


Assume p1 after delivering message m broadcasted some m'. p2 will recieve this message and deliver m'. But given that m->m' q must have delivered m as well. A contradiction.


