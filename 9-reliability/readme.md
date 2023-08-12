# ECE 350 - Reliability

"The best option for resiliency is if the system can correct the problem and carry on!"

May not be possible

* "If the magic smoke has come out of the hard drive, no amount of rebotting will fix it"

System **CAN** probably keep running at lower capacity though...

## "Continue at Reduced Capacity"

If the system cannot be restored to full capacity automatically, the next best thing is if the system can continue as best it can, likely at reduced capacity.

Example:

* If a system has 4 cores and one of them dies, keep running with the 3 healthy cores

<br>

**Possible strategy:** keep running at reduced capacity until external force can come to "repair" the system.

<br>

In an real-time system, loss of capacity probably means the workload becomes no longer schedulable.

* The system is **stable** if it will always meet the deadlines of its most critical tasks, even if lower-priority tasks don't get completed

## Graceful Shutdown

Instead of throwing the *emergency stop* (e-stop) when things go wrong, maybe it's possible to do a graceful shutdown.

Example: Traditional UNIX

* If the kernel detects data corruption, will write contents of memory to disk for anlysis, then stop execution

A good solution for *problems we can't live with, but also cannot fix*.

## Emergency Stop (E-Stop)

Stop all execution immediately, ASAP.

Example:

* MCU controller industrial machinery
* Better to stop everything so machinery stops moving, instead of killing someone

"*Downtime is bad, but not as bad as the system killing someone.*"

# Faults

See [ECE 455 notes](https://github.com/RyanJMah/ece455-notes/blob/master/4-reliability/readme.md) for more detailed explanations (for everything in this section tbh).

* **Failure**
  * When the delivered service no longer complies with the specification
  * Remember: "Fail" to deliver service within specification

* **Error**
  * System state that leads to a subsequent failure
  * Remember: "Error STATE", always think of "state" for errors

* **Fault**
  * The cause of an error
  * Remember: "Who's FAULT is the error?" - the fault, lol

```
Fault --> Error --> Failure
```

## Types of Faults

**Permanent**

* Fault is always present after it occurs
* Can only be fixed by actualy replacing or repairing the faulty component
* Example: MOSFETs get overvolted and catch fire (lol)

**Intermittent**

* Fault recurrs at random, unpredictable times
* Example: faulty chip, HW bug

**Transient**

* Fault occurs once but does not recur
* Example: cosmic rays causing bit flips

## Fault Prevention

1. Make sure you understand the physical environment to protect system from physical damage
2. HW solutions for reliability are better than SW, use HW when you can
3. Within SW:
    * Have a sufficiently detailed and rigorous specification
    * Use appropriate design and test methodologies
    * Use the right programming lanuage? (Zarnett is a Rust shill, lol)
    * Use analysis tools
4. Verify that components and tools work as they advertise

## Fault Tolerance

**Process Isolation**

* Example: OS prevents one process from accessing memory of another process

**Dual-Mode Operation**

* Monopolize access to HW in OS to prevent threads from interfereing

**Preemptive, Priority-Based Scheduling**

* Makes sure the highest priority tasks get to run first
* Also makes sure that lowest-priority tasks are tardy first

**Checkpoints, Transactions, Rollback**

* Look at [ECE 455 notes](https://github.com/RyanJMah/ece455-notes/blob/master/4-reliability/readme.md)

<br>

**Types of Redundancy**

* **Information** Redundancy
    * Example: RAID

* **Physical** Redundancy
    * Example: Run same algorithm on two computers concurrently

* **Temporal** Redundancy
    * Example: Repeat a calculation if an error is detected

<br>

Moral of the story: *there should be no single point of failure*

## Consensus in Distributed Systems

Again, look at [ECE 455 notes](https://github.com/RyanJMah/ece455-notes/blob/master/4-reliability/readme.md).

## Clock Synchronization in Distributed Systems

How do we know what time it is? How do get get everyone to know what time it is, and agree on it?

In non-real-time systems, not a huge deal. Big deal in real-time systems though.

* Use an **RTC** (real-time clock), HW solution
    * Suffers from drift, typically half a second per day

* **NTP** (network time protocol)
    * Designed to compensate for network delay, not perfect though

<br>

*It is impossible to get two computers to **COMPLETELY** agree on time.*

* (Unless they physically share the same clock maybe...)
