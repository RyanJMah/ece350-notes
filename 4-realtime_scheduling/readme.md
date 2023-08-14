# ECE 350 - Real-Time Scheduling

"A real-time system is a system that is supposed to respond to events within a certain amount of real (wall-clock) time."

From Prof. Murray Dunne in ECE 455:

* "A **hard real-time** system is a system in which tardiness must always be zero"
  * Or, probability of tardiness is very low ($10^{-7}$) 
* A **soft real-time** system only requires a "best effort" for tardiness

### RTOS vs. Non-Real-Time OS

Most general-purpose operating systems are designed to maximize throughput. They make few guarantees about WCET of syscalls, concurrency control, etc.

An RTOS is designed with the following in mind:

1. **Determinism**
   * Operations are predictable
   * "No matter how unlucky the timing or sequence of events, we can still start the task on time to successfully complete it"
   * "How long it takes before the OS acknowledges the request/interrupt"
2. **Responsiveness**
   * "How long it takes after the acknowledgement to handle it"
3. **User (administrator) control**
   * Either more control than a general-purpose OS, or no control
   * More control:
     * OS has no way of knowing which tasks are real-time, or which ones are soft/firm/hard
     * Admin must specify which tasks are real-time and which are not, etc.
   * No Control:
     * OS takes no instructions from users or admin
     * Just runs as it was programmed to at compile time (example: FreeRTOS)
4. **Reliability**
   * Zarnett's notes gloss over this, so I will too
5. **Fail-soft operation**
   * "If it is not possible to succeed in completing all tasks, the system will try its best to complete as many tasks as it can, with priority given to hard real-time tasks"

## Vocabulary

"Any non-preemptive scheduling algorithms will not be suitable for real-time systems, because the whole idea is based around prioritization of hard real-time tasks."

* I know this is not true, since cyclic executives exist (from ECE 455), but for what we learn in ECE 350 I guess this is true

### Types of Tasks

From Prof. Murray Dunne, ECE 455:

* "A **task** is a distinct unit of work the system must do"
* "A task is composed of **jobs**"

1. **Fixed-Instance Tasks**
   * Something that executes a fixed number of times (e.g., system initialization, etc.)
2. **Periodic Tasks**
   * Repeat jobs at regular intervals
3. **Sporadic Tasks**
   * Repeat jobs at some minimum inter-arrival time
4. **Aperiodic Tasks**
   * Jobs occur randomly (or not at all)

### Worst-Case Execution Time (WCET)

Two methods of estimating WCET:

1. **Code Analysis**
   * Look at source code (or disassembly of compiled source-code) and determine the longest-path
   * Can then determine WCET in cycles, given the host MCU's architecture
2. **Empirical Observation**
   * Measure execution time with an oscilloscope or something

### Utilization Bound Test (UB Test)

CPU utilization of a set of tasks is:


$$
U = \sum_{i=1}^{N}{\frac{C_i}{\tau_i}}
$$


Max CPU utilization is 1, otherwise the tasks aren't schedulable.

## Scheduling Algorithms

### Earliest Deadline First (EDF)

Priority-based scheduling, where tasks which have earlier deadlines have higher priority.

EDF is **optimal** (i.e., it will always find a feasible schedule if one exists) if:

* Preemption is enabled and
* There is only 1 processor

To implement EDF, a priority queue is reasonable. Need to make sure that soft real-time tasks with earlier deadlines don't get priority over hard real-time tasks though.

**UB Test for EDF:**


$$
U = \sum_{k=1}^{N}{\frac{C_k}{\text{min}(D_k, \tau_k)}} \le 1
$$


where $D_k$ is the deadline for task $k$, and $\tau_k$ is the period of task $k$.

#### Deadline Interchange

Deadline-based approaches are subject to a problem similar to priority inversion.

Suppose task $A$ has locked a mutex, then task $B$ is released which also need that mutex and has a sooner deadline than $A$ - then $A$ will be preempted by $B$, until $B$ gets blocked waiting on the mutex. If other tasks $C$, $D$, $E$, ... that also want the resource, $A$ could be waiting a long time to proceed. It could wait so long that $B$ misses the deadline.

Solution is very similar to priority inheritance, $A$ needs to be assigned a temporary new deadline such that it can finish the critical section and release the mutex.

### Least Slack First (LST)

Similar to EDF. **Slack time** is the amount of time remaining before a tasks must be scheduled to meet its deadline.

**SLACK TIME IS:** `deadline - remaining_execution_time`

From ECE 455:

* "Slack time is the amount of time remaining before the deadline of a task, minus the remaining execution time of that task"

If a task will take 10ms to execute and its deadline is 50ms away, then there is $50 - 10 = 40ms$ slack time remaining.

LST is **optimal** if:

* Preemption is enabled and
* There is only 1 processor
* (NOTE: these are the same conditions as EDF)
  * LST requires knowing the WCET of all tasks though, EDF doesn't

### Rate-Monotonic Scheduling (RMS)

Tasks with a lower periodicity get a higher priority.

RMS is **optimal** if:

* Preemption is enabled and
* There is only 1 processor and
* Tasks are *harmonic* (their periodic are integer multiples of each other)

**UB Test for RMS**


$$
U = \sum_{k=1}^{N}{\frac{C_k}{\tau_k}} \le n(2^{1/n} - 1)
$$



### Deadline-Monotonic Scheduling (DMS)

Tasks with a lower deadline get higher priority.

Same as RMS if $D_k = \tau_k$ (if deadlines of tasks are equal to their period), i.e., tasks are *simply periodic*.

Can sometimes find a feasible schedule where RMS cannot, if tasks are non-simply periodic.

## Aperiodic Servers

Basically, design pattern - implement a periodic task that runs aperiodic/sporadic tasks from a queue. This task is called a "***polling server***".

The server gets to run for a fixed interval of time, then gets preempted to run the periodic tasks (kinda like slack time in a cyclic executive). If the non-periodic task gets preempted in this fashion, it will just finish running the next time the server runs.

Can have multiple servers as well. Could make one server a higher priority than another server to give priority to non-periodic tasks.

**Deadline-Deferrable Servers**
* If the polling server completes all tasks from the queue before it has finished running, it will immediately yield
* Deadline-deferrable servers will keep running and polling the queue until its interval expires

**Deadline Sporadic Servers**
* Instead of wasting the execution time polling an empty queue, "save" it for later
* Can run it later (before the server is "technically" scheduled to run again)
  * Schedule as slack time between periodic tasks
* Kinda like Duolingo heart system, but instead of hearts, its execution time for the server

## Multicore Real-Time Scheduling

I doubt he will ask about this tbh (copium).

Basically, one option is to come up with an offline schedule that says "Task A will run on CPU0, Task B will run on CPU1, etc." This is basically the exact same as uniprocessor real-time scheduling (this is what people normally do).

He thinks that this in "un-optimal," and brings up something proposed in a research paper - **P-Fairness**.

* P stands for "proportional"
* Allocate CPU time such that tasks are forced to proceed at proportionate rates
  * "Time is divided up into little piece and each process gets a proportional share of CPU time" (proportional to WHAT though?????)
* Scheduling algorithm consists of 3 parts:
  1. Schedule all urgent tasks
  2. Do not schedule urgent tasks
  3. Schedule other tasks in order of highest lag to lowest until capacity is filled

Just pray he doesn't ask this on the final tbh.
