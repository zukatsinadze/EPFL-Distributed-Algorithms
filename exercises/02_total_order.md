1) Would it make sense to add the total-order property to the best-effort broadcast?

Weird question.


2) What happens in our "Consensus-Based
Total-Order Broadcast" algorithm, if the
set of messages delivered in a round is
not sorted deterministically after deciding
in the consensus abstraction, but before
it is proposed to consensus?
What happens in that algorithm if the set
of messages decided on by consensus is
not sorted deterministically at all?

Easy.



3) The "Consensus-Based Total-Order Broadcast" algorithm transforms a consensus
abstraction (together with a reliable broadcast abstraction) into a total-order
broadcast abstraction.
Describe a transformation between these two primitives in the other direction, that
is, implement a (uniform) consensus abstraction from a (uniform) total-order
broadcast abstraction.

Use Total-Order to implement Consensus:

upon event <init>
  decided = False

upon event <propose, v>
  trigger <toBroadcast, v>

upon event <toDeliver, src, v>
  if decided == False:
    trigger <decide, v>
    decided = True
