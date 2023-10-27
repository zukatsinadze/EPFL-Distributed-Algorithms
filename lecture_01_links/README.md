# Lecture 01

## Fair Loss links

* FL1. Fair Loss. If a message is sent inf. Many times from Pi to Pj and neither of them crash,
them m is delivered infinitely often by Pj.
* FL2. Finite Duplication. IF a message m is sent a finite number of times from pi to pj, then 
m is delivered finite number of times by pj.
* FL3. No creation. No message is delivered unless it was sent.

FL1. Liveness (I can never observe the violation)
FL2. Liveness (I can never observe that infinite number of messages have been delivered)
FL3. Safety (I can observe the violation. I can see the clear point where it is violated)


## Stubborn Links

* SL1. Stubborn Delivery. If a process pi send message m to correct process pj and pi does not crash,
then pj delivers m infinately many times
* SL2. No creation. No message is delivered unless it was sent


```
Algorithm (sl)
Implements: StubbornLinks (sp2p)
Uses: FairLossLinks (flp2p)

upone even <sp2pSend, dest, m> do
    while (true) do
        trigger <fl2psend, dest, m>;


upon event <flp2pDeliver, src, m> do
    trigger <sp2pDeliver, src, m>;
```


## Perfect Links (Reliable Links)

* PL1. Validity. If pi and pj are correct, then every message sent by pi to pj is eventually delivered by pj.
* PL2. No duplication: No message is delivered more than once to a single process.
* PL3. No creation: No message is delivered unless it was sent.

PL1. Liveness (I can't observe violation, because of 'eventually')
PL2. Safety (I can observe violation)
PL3. Safety (I can observe violation)


```
Algorithm (pl)
Implements: PerfectLinks (pp2p)
Uses: Stubborn Links (sp2p)

upon event <Init> do delivered = set();

upon event <pp2pSend, dest, m> do 
    trigger <sp2pSend, dest, m>

upon event <sp2pDeliver, src, m> do
    if m not in delivered:
        trigger <pp2pDeliver, src, m>
        delivered.add(m)
```

*Notice that for delivery sp2p and pp2p are in reverse order.*



Proof that algorithm is correct:

PL1. If pi and pj are correct, then every message sent by pi to pj is eventually delivered by pj.
By SL1 property we know that this message would have been sent infinitely many times, and hence
sp2pDeliver would be triggered. Only possibility for pp2pDeliver not to be triggered is if 
m was already in delivered set, hence it was delivered.


PL2. No duplication: no message is delivered more than once to a single process.
pp2pDeliver only happens when m is not already in the delivered set, so it is not possible to deliver it more than once.


PL3. No message is delivered unless it was sent:
Delivery only happens if sp2pDeliver is triggered, which means that it was delivered using stubborn links.
But with Stubborn Links' SL2 property we know that it is impossible unless message was sent.



From now on we assume that we have reliable links. They ensure that messages exchanged
between correct processes are not lost.



