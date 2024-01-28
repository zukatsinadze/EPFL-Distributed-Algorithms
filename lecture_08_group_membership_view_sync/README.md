# Group Membership


Composition of the network changes over time.
So we need to keep track of who is involved. Who are the members.


we can have crashing, joining and leaving (even without crashes)


Unlike perfect failure detector, information about failures are coordinated.


We only consider the case of system shrinking (fails and leaves):

Memb1. Local Monotonicity: If a process installs view (j, M) after installing (k, N), then j > k and M < N

Memb2. Agreement: Not two processes install views (j, M) and (j, M') such that M != M'

Memb3. Completness: If a process p crashes, then there is a integer j such that every correct
process enventually installs view (j, M) such that p is not in M.

Memb4. Accuracy: If some process installs a view (i, M) and p not in M, then p has crashed.


Events: <membView, V> (install view, no joins and leaves)

Uses: PerfectFailureDetector (P), UniformConsensus
```
upon event <init>
  view = (0, S)
  correct = S
  wait = false

upon event <crash, p>
  correct = correct \ {p}

upon event (correct is strict subset of view.memb) and (wait = False)
  wait = True
  trigger <uconsPropose, (view.id + 1, correct)>

upon event <uconsDecide, (id, memb)>
  view = (id, memb)
  wait = False
  trigger <membView, view>

```
Group Membership implies Perfect Failure Detector.




# View Synchrony

View synchronous broadcast is combination of group membership and reliable broadcast

Delivry of messages are coordinated with installation of views.

Besides Memb1-4 and RB1-4 we have
VS: A message is vsDelivered in the view where it is vsBroadcast


<vsBroadcast, m>
<vsDeliver, src, m>
<vsView, V>


We need to add specific event  to block the application from vsBroadcaasting messages.
Otherwise if application keeps broadcastins the view syncrhony might never be able to vsInstall new view.


So we add <vsBlock, ok> and <vsBlock>


Uses: GroupMembership (gmp), TerminatingReliableBroadcast (trb), BestEffortBroadcast (beb)
```
upon event <init>
  view = (0, S)
  nextView = None
  pending = set()
  delivered = set()
  trbDone = set()
  flushing = false
  blocked = false

upon event <vsBroadcast, m> and blocked = False
  delivered.add(m)
  trigger <vsDeliver, self, m>
  trigger <bebBroadcast, [Data, view.id, m]>

upon event <bebDeliver, src, [Data, vid, m]>
  if (view.id == vid) and m not in delivered and blocked = False:
    delivered.add(m)
    trigger <vsDeliver, src, m>

upon event <membView, V>
  addtoTail(pending, V)

upon (pending != emtpy) and flushing == False:
  nextView = removeFromHead(pending)
  flushing = true
  trigger <vsBlock>

upon <vsBlock, ok>
  blocked = True
  trbDone = empty
  trigger <trbBroadcast, self, (view.id, delivered)>

upon <trbDeliver, p, (vid, del)>
  trbDone.add(p)
  for all m in del:
    if m not in delivered:
      delivered.add(m)
      trigger <vsDeliver, src, m>

upon trbDone == view.memb and blocked = True:
  view = nextView
  flushing = False
  blocked = False
  delivered = empty
  trigger <vsView, view>
```



## Consensus based VS:

Uses: UniformConsensus, BestEffortBroadcast, Perfect P
```
upon event <init>
  view = (0, S)
  correct = S
  flushing = False
  blocked = False
  delivered = set()
  dset = set()

upon event <vsBroadcast, m> and blocked = False:
  delivered.add(m)
  trigger <vsDeliver, self, m>
  trigger <bebBroadcast, [Data, view.id, m]>

upon event <bebDeliver, src, [Data, vid, m]>
  if (view.id == vid) and (m not in delivered) and (blocked = False)
    delivered.add(m)
    trigger <vsDeliver, src, m>

upon event <crash, p>
  correct \= {p}
  if flushing == False:
    flushing = true
    trigger <vsBlock>

upon event <vsBlock, ok>
  blocked = True
  trigger <bebBroadcast, [DSET, view.id, delivered]>

upon event <bebDeliver, src, [DSET, vid, del]>
  dset.add([src, del])
  if forall correct p we have (p, mset) in dset:
    trigger <uconsPropose, view.id + 1, (correct, dset)>

upon event <uconsDecide, id, memb, vsdset>
  for (p, mset) in vsdset:
    if p in memb:
      for (src, m) in mset:
        if m not in delivered:
          delivered.add(m)
          trigger <vsDeliver, src, m>
  view = (id, memb)
  flushing = False
  blocked = False
  delivered = empty
  dset = empty
  trigger <vsView, view>
```



## Uniform View Synchrony:

RB4 property is with uniform. Everything else is same.

```
upon event <init>
  view = (0, S)
  correct = S
  flushing = False
  blocked = False
  udelivered = set()
  delivered = set()
  dset = set()
  for all m: ack[m] = set()



upon event <vsBroadcast, m> and blocked = False
  delivered.add(m)
  trigger <bebBroadcast, [Data, view.id, m]>

upon event <bebDeliver, src, [Data, vid, m]>
  if view.id == vid:
    ack[m].add(src)

  if m not in delivered:
    delivered.add(m)
    trigger <bebBroadcast, [Data, view.id, m]>

upon event (view.memb <= ack[m] and m not in udelivered):
  udelivered.add(m)
  trigger <vsDeliver, src(m), m>

upon event <crash, p>
  correct = correct \ {p}
  if flushing = False:
    flushing = True
    trigger <vsBlock>

upon event <vsBlock, ok>
  block = True
  trigger <bebBroadcast, [DSET, view.id, delivered]>

upon <bebDeliver, src, [DSET, vid, del]>
  dset.add([src, del])
  if forall correct p we have (p, mset) in dset:
    trigger <uconsPropose, view.id + 1, (correct, dset)>

upon event <uconsDecide, id, memb, vsdset>
  for (p, mset) in vsdset:
    if p in memb:
      for (src, m) in mset:
        if m not in delivered:
          delivered.add(m)
          trigger <vsDeliver, src, m>
  view = (id, memb)
  flushing = False
  blocked = False
  delivered = empty
  dset = empty
  trigger <vsView, view>
```