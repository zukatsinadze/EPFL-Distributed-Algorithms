1) Explain why any fail-noisy consensus algorithm (one that uses an eventually
perfect failure detector ◇P) actually solves uniform consensus (and not only the
non-uniform variant).



By contradiction assume we have algorithm that solves non-uniform variant but not uniform.

Let's consider two processes p1 and p2. And two runs of algorithm A and B.

A: p1 is faulty and p1 and p2 decide differently. Let t be time when p2 decides.
B: p1 and p2 are both correct, but our <>P wrongly suspects p1 at the same point in time it crashed during A. Also messages from p1 are delayed until time t. and p1 is not restored until t.

In B p2 recives exactly same messages and indifactions as in A, so it will decide differently from p1 as well in B.
However, in B p1 never failed. So non-uniform agreement is violated.





2) Explain why any fail-noisy consensus algorithm (one that uses an eventually
perfect failure detector ◇P) requires a majority of the processes to be correct.
More precisely, provide a “bad run” in the case where the majority of processes is
faulty.


Consider even number N processes. A and B are distincs subsets of N/2 processes and proposa A and B values respectively.
By contradiction assume there is an algorithm that achieves consesnsus when N/2 processes fail.


Execution 1: All processes in A crash at the beginning without issuing any message.
All processes in B still achieve consensus and by validity decide B. Let T_b denote time when last process decided.

Execution 2: All processes in B crash at the beginning without issuing any message. All processes in A
still achieve consensus and decide A. Let T_a be time when last process decided.

Execution 3: All processes in A are suspected by each process in B and vice-versa (at exactly same times as in Executions 1 and 2)
No message between a process in A and process in B is delivered before max(T_a, T_b). No process restores any process in another group before time max(T_a, T_b).

Processes in A can't distinguish between executions 2 and 3 and decide A.
Processes in B can't distignuish between executions 1 and 3 and decide B.
Agreement is violated.



3) <>P is not enough for NBAC
Proof:
Run 1:
p1 propose 1 and crash
p2, p3 propose 1 but decide 0

Run 2:
p1 propose 1
p2, p3 propose 1

p2, p3 falsy suspect p1 at the exact same time as in run 1 and decide 0.
but p1 is actually not crashed, so everyone proposed 1, no one crashed but we decided 0.
