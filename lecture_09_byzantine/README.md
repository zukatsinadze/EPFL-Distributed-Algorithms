Processes are random...


True information: Recieved from at least f + 1 processes

Decision: Recieved from at least 2f+1 processes

2f+1 and 2f+1 always intersect with f+1 if we have 3f+1 in total

# Asynch Byzantine Consistent Broadcast

broadcast(m)
deliver(m)

BCB1. If sender src is correct and broadcast m, then all correct deliver m
BCB2. No two correct processes deliver different messages
BCB3. No correct process delivers more than one message
BCB4. If correct process delivers message m and src is correct then it was broadcasted by src



We can tolerate at most 33% byzantine processes

Let's say we devise an algorithm that works with f byzantine processes and we have 3f processes in total.


Lets partition in three sets P1, P2, P3.

Lets say P1 and P3 are correct and P2 are byzantine. Lets assume P1 and P3 messages are delayed. and processes in P2 are showing correct behaviour to P1 and P3.

P1-s will deliver m
and P3-s will deliver m'.



Algorithm 1:
Uses: AuthenticatedPerfectLinks (apl)


upon event <init>
  sentecho = False
  delivered = False
  echos[p] = [None]

upon event <bcbBroadcast, m>
  for q in S:
    trigger <aplSend, q, m>

upon event <aplDeliver, p, m> and p == src and sentEcho == False:
  sentecho = True
  for q in S:
    trigger <aplSend, [ECHO, m]>

upon event <aplDeliver, p, [ECHO, m]>
  if echos[p] == None:
    echos[p] = m

upon exists message m s.t. #(p in S | echos[p] = m) > (N + f) / 2
  if not delivered:
    delivered = True
    trigger <bcbDeliver, src, m>



Algorithm 2:

Sender sends message to all and waits for verifications.
Once it recieved enough verifications it sends out message together with witnesses.






# Async Byzantine Reliable Broadcast

BRB1. If S is correct and broadcasts message m, S eventually delivers m
BRB2. No two correct processes deliver different messages
BRB3. No correct process delivers more than one message
BRB4. If a correct process delivers message m and S is correct, then m was broadcast by S
BRB5. If a correct process delivers a message, then every correct process eventually delivers a message

