# Reliable Distributed Storage


Cloud is first of all distributed storage.
You are not going to store your information in one place, you replicate it to make it more reliable.


## Register
Register contains integer, initially 0

Every value written is uniquely identified 

Assume a register is a local to a process, i.e., accesd only by one process.


### Regular Register

Assume only one writer
Provides strong guarantees when there is no concurrent operation.

Otherwise we have minimal guarantess:

Read() returns:
* the last value written if there is no concurrent or failed operations
* otherwise the last value written or any value concurrently written (i.e. some parameter of some Write())


## Implementing a register

Implementing Read() and Write() operations at every process
Processes have to communicate with each other.


We assume a fail-stop model: processes can fail by crashing, channels are reliable, failure detection is perfect


### Simple Algorithm

We implement a regular register
Every process pi has a local copy of the register value vi
Every process reads locally
Every writer writes globally (i.e. at all processes)


```
def Write(v) at pi:
    send [W, v] to all
    for every pj wait until:
        recieve[ack] or detect_fail[pj]

    return OK

def when recieve [W, v] from pj:
    vi = v
    send[ack] to pj


def Read() at pi:
    return vi
```

Correctness (liveness):
Read() is local and eventually returns
Write() eventually returns by the completness of failure detector or reliability of the channels


Correctness (Safety):
a) In the absence of concurrent or failed operations, Read() returns the last value written

Proof: Assume a Write(x) terminates and there is no other write invoked. By the accuracy 
of the failure detector, the value of the register at all processes that are still alive will be x.
Any subsequent Read() invocation by some process pj returns x which is the last value written.

b) If there is concurrency Read() returns the value concuirrently written or the last value written

let x be the value returned by Read(): by the properties of the channels, x is the value of the register
at some process. This value does necessarily come from the last or a concurrent Write()



### What if failure detection is not perfect?

Can we devise an algorithms that implements a regular register and tolarates arbitrary number of crash failures?

No, we need at least majority.


Any wait-free asynchronous implementation of regular register requires a majorty of correct processes.

Proof: Idea: partition the set into n/2 and n/2 which believe that each other is crashed.

Assume Write(v) is performed and n / 2 processes crash (or so we believed), then Read() is performed and the other
n/2 are up: The read() cannot see the value v.



### The Majority Algorithm
P1 is the writer and any process can be reader
A majority of processes is correct
Channels are reliable.

Every process pi maintans a local copy of the register vi, as well as sequence number sni and read timestamp rsi
Process p1 in addition has timestamp ts1


```
Write(v) at p1 :
    ts1++
    send [W, ts1, v] to all
    when recieve [W, ts1, ack] from majority
        return OK


at pi when recieve [W, ts1, v] from p1:
    if ts1 > sni:
        vi = v
        sni = ts1
        send [W, ts1, ack] to p1


Read() at pi:
    rsi++
    send [R, rsi] to all
    when recieve [R, rsi, snj, vj] from majority
    v = vj with largest snj
    return v

at pi when recieve [R,  rsj] from pj:
    send [R, rsj, sni, vi] to pj

```


## Atomic Register

Atomic register has strong guarantees even with concurrent writes.
There is linearization.


Every failed (write) operations appears to be either complete or not have been invoked at all

and

Every complete operations appears to be executed at some instant between its invocation and reply time events




### Fail-Stop Algorithm (simple one)

Idea:
```
Read() at pi:
    send [W, vi] to all
    for every pj, wait until
        recieve[ack] or detect[pj]
    return vi
```

But this may overwrite newer write operations, so we need timestamps

1 Writer N readers algorithm:
```
Write(v) at p1:
    ts1++
    send [W, ts1, v] to all
    for every pi wait until:
        recieved [ack] or detect[pi]
    return OK

Read() at pi:
    send [W, sni, vi] to all
    for every pj wait until:
        recieve[ack] or suspect[pj]
    return vi

at pi when pi recieve [W, ts, v] from pj:
    if ts > sni:
        vi = v
        sni = ts
    send[ack] to pj

```

N writers N readers algorithm:
```
Write(v) at pi:
    send [W] to all
    for every pj wait until:
        recieve[W, snj] or fail pj

    (sn, id) = (highest snj + 1, i)

    send [W, (sn, id), v] to all
    for every pj wait until
        recieve [W, (sn, id),ack] or fail pj
    return OK

when recieve [W] from pj:
    send [W, sn] to pj

when recieve [W, (snj, idj), v] from pj:
    if (snj, idj) > (sn, id):
        vi = v
        (sn, id) = (snj, idj)
    send [W, (snj, idj), ack] to pj

```
















