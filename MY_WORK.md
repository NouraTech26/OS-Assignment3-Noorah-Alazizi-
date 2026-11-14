
# MY_WORK.md - Assignment 3: Synchronization

## Development Log
- **2026-11-10:** Created the GitHub repository fork (`OS-Assignment3-Noorah-Alazizi`) and updated student ID (`444051471`) in `SchedulerSimulationSync.java`[cite: 1, 2].
- **2026-11-11:** Implemented Feature 1 by adding a `ReentrantLock` to protect the ready queue operations (`add`, `poll`, `processOf`, `snapshot`) in `ReadyQueue`[cite: 2].
- **2026-11-12:** Implemented Feature 2 by adding a `ReentrantLock` (`statsLock`) across all shared counters and reader methods in `SharedStats` with proper `try/finally` blocks[cite: 2].
- **2026-11-13:** Implemented Feature 3 by replacing the active busy-waiting loop in `CpuWorker` with a counting `Semaphore` (`acquire` and `release`)[cite: 2].
- **2026-11-14:** Tested the complete simulation multiple times, verified that the verification block shows all check marks (`✔`) with 0 busy-waiting polls, and prepared documentation[cite: 2].

## Reflection
1. **What was the most challenging part of this assignment, and how did you overcome it?**
   The most challenging part was understanding how multiple CPU threads interact simultaneously with the shared ready queue and correctly managing synchronization without causing deadlocks. I overcame this by carefully analyzing the lifecycle of process threads, ensuring every critical section in the ready queue and statistics classes was protected by `ReentrantLock` using `try/finally` blocks to guarantee locks are always released.

2. **Why is busy waiting inefficient, and how did the counting semaphore improve the simulation?**
   Busy waiting wastes valuable CPU cycles because threads continuously poll an empty queue in a loop instead of performing useful work or resting, which artificially inflates CPU usage and requires workaround checks. By implementing a counting `Semaphore`, the CPU worker threads block gracefully using `.acquire()` until a process is actually available in the queue, eliminating unnecessary polling entirely (bringing busy-waiting polls to 0).

3. **How did you decide on lock granularity for the shared statistics?**
   For the shared statistics (`SharedStats`), I used a single dedicated `ReentrantLock` (`statsLock`) rather than multiple locks per variable. This choice ensures thread safety while keeping the implementation clean and avoiding complex multi-lock coordination, since statistical updates are fast, isolated, and executed frequently by concurrent CPU threads.

4. **What did you learn about concurrent programming and debugging race conditions in Java?**
   This assignment provided a deep practical understanding of how unpredictable race conditions can be when multiple threads interleave their reads and writes. I learned the critical importance of protecting shared data structures, the necessity of proper lock scoping, and how helpful simulation tools like `RaceWindow.pause()` and verification blocks are in exposing hidden concurrency bugs reliably.

## Technical Answers
1. **Race Condition in Your Output:**
   A race condition occurs when two or more threads access shared data concurrently and try to modify it simultaneously, leading to incorrect results depending on the exact execution interleaving. In the starter code, updating counters like `contextSwitchCount` without locks allowed multiple threads to read the same old value before writing back, causing lost updates. This was fixed by wrapping the critical section in `statsLock.lock()` and `try/finally`.

2. **Critical Sections and Lock Granularity:**
   A critical section is a block of code that accesses shared resources that must not be concurrently accessed by more than one thread. Lock granularity refers to the scope of data that a lock protects. In `ReadyQueue`, the lock protects the underlying `LinkedList` and `HashMap` structures during `add()`, `poll()`, `processOf()`, and `snapshot()`, ensuring structural integrity during concurrent access.

3. **Busy Waiting vs Semaphore:**
   Busy waiting involves threads actively looping and testing a condition (such as checking if an empty queue has items), which consumes CPU time wastefully. The counting semaphore solves this by maintaining an internal permit counter; when the ready queue is empty, CPUs call `semaphore.acquire()` and sleep until `semaphore.release()` is called upon adding a process, allowing efficient thread blocking and wake-up.

4. **Classical Problems and Deadlock:**
   Deadlock happens when two or more threads are blocked forever, each waiting for a resource held by another. In our scheduler, if the final process finishes and the CPU threads are not properly awakened via a semaphore release (`semaphore.release(numCPUs)`), the CPU threads remain blocked indefinitely waiting for permits. The watchdog timer detects this if threads fail to join before the timeout deadline.

