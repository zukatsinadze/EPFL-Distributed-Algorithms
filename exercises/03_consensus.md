1) Consider our fail-stop consensus algorithms (Consensus Algorithm I and
Consensus Algorithm II). Explain why none of those algorithms would be correct if
the failure detector turned out not to be perfect.


2) Explain why any fail-noisy consensus algorithm (one that uses an eventually
perfect failure detector ◇P) actually solves uniform consensus (and not only the
non-uniform variant).


Assume we have algorithm that solves non-uniform consensus but not uniform one.


Consider two runs:

Run 1: p1 is faulty and p1 and p2 decide differently. Assume p2 decided at time t.
Run 2: At exactly same time p2 suspects p1. Algorithm goes on as in run 1 so p1 and p2 decide differently. But after t+ time p1 is restored. Now we have two correct processes that decided differently. So contradiction.



3) Explain why any fail-noisy consensus algorithm (one that uses an eventually
perfect failure detector ◇P) requires a majority of the processes to be correct.
More precisely, provide a “bad run” in the case where the majority of processes is
faulty


Assume we have algorithm that works with non-majority correct processes.
Assume N is even. Let A and B be N/2 size subsets of processes. A-s propose A and B-s propose B.


Run 1: All of A-s fail. B-s decide B.
Run 2: All of B-s fail. A-s decide A.
Run 3: A-s and B-s falsly suspect each other at exactly same times as in run 1 and run 2. So
algorithm will play out in the similar way and As will decide A and Bs will decide B.
