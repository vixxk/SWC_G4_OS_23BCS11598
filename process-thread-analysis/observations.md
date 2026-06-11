# Class Notes: Summary Observations on Process & Thread Execution
**Course:** CS-301 Operating Systems Lab  
**Module 2:** Process vs. Thread Execution in Modern Systems  
**Topic:** Concluding Lab Observations, Thread Safety, Concurrency vs. Parallelism  
**Date:** June 11, 2026  

---

## 1. Summary of Experimental Observations
During the process and thread execution lab, several key behaviors were verified:

1.  **Memory Space Isolation:** In process-based execution (using `fork()`), modifying a global variable in a child process did *not* affect the variable in the parent process. The operating system successfully isolated each process's address space.
2.  **Memory Space Sharing:** In thread-based execution (using POSIX threads), modifying a global variable in one thread immediately modified the value read by the main thread. This proved that threads execute in a shared data space.
3.  **Concurrency Hazards:** Running multiple threads that incremented a shared variable without locks caused data corruption (lost updates), resulting in a final count lower than expected. This confirmed the presence of a race condition.
4.  **Synchronization Success:** Applying a POSIX mutex lock around the critical section successfully resolved the race condition, yielding the correct mathematical result at the cost of a minor execution slowdown due to locking overhead.

---

## 2. Theoretical Discussion: Concurrency vs. Parallelism
A common point of confusion in operating systems is the difference between concurrency and parallelism:

*   **Concurrency:** The composition of independently executing processes/threads. It is about **structure**. An application is concurrent if it can manage multiple tasks at the same time (e.g., interleaving execution of thread A and thread B on a single-core CPU via time-slicing).
*   **Parallelism:** The simultaneous execution of multiple tasks. It is about **execution**. An application is parallel if it executes multiple tasks at the exact same physical instant on multi-core CPUs.

| Feature | Concurrency | Parallelism |
| :--- | :--- | :--- |
| **Focus** | Dealing with a lot of things at once (structure). | Doing a lot of things at once (execution). |
| **Hardware Requirement** | Can run on a single-core CPU (via context switching). | Requires multi-core or multi-processor systems. |
| **Goal** | Improve responsiveness and reduce blocking. | Maximize CPU throughput and speed up computation. |

---

## 3. Thread Safety & Reentrancy
Writing multi-threaded applications requires ensuring that code behaves correctly when called by multiple threads.

*   **Thread Safety:** A piece of code is thread-safe if it functions correctly during simultaneous execution by multiple threads. To achieve thread safety, developers use synchronization primitives, thread-local storage, or immutable data.
*   **Reentrancy:** A function is reentrant if it can be interrupted in the middle of its execution and safely called again ("re-entered") before its previous invocation finishes. 
    *   *Rule of Thumb:* All reentrant functions are thread-safe, but not all thread-safe functions are reentrant (e.g., a function that uses a mutex lock is thread-safe, but if it is interrupted by a signal handler that calls the same function, it will deadlock, making it non-reentrant).

### Best Practices for Writing Thread-Safe Code:
1.  **Minimize Shared Mutable State:** Prefer local stack variables or pass data by value rather than using globals.
2.  **Use Thread-Local Storage (TLS):** Allocate variables that are global to a thread but private from other threads.
3.  **Use Fine-Grained Locking:** Lock only the exact lines of code that modify shared variables (Critical Section) to reduce thread blocking.
4.  **Avoid Nested Locks:** To prevent deadlocks, always acquire locks in a predefined, consistent order across all threads.
