# Class Notes: Memory Sharing, Isolation & Concurrency Hazards
**Course:** CS-301 Operating Systems Lab  
**Module 2:** Process vs. Thread Execution in Modern Systems  
**Topic:** Memory Architecture, Synchronization (Mutex/Semaphores), and IPC  
**Date:** June 11, 2026  

---

## 1. Objective
To analyze how memory isolation is maintained between processes, how memory sharing occurs among threads, and to explore the concurrency hazards (race conditions) that arise from shared memory, alongside their mitigations (mutexes, semaphores).

---

## 2. Memory Isolation (Processes) vs. Memory Sharing (Threads)

*   **Process Isolation:** The OS enforces strict memory boundaries using virtual memory page tables. A process cannot read or write to another process's memory space. Attempting to access unauthorized memory addresses triggers a CPU hardware interrupt, and the OS terminates the offending process with a **Segmentation Fault (`SIGSEGV`)**.
*   **Thread Sharing:** Sibling threads share the same virtual address space, including the global variables (Data segment) and dynamically allocated memory (Heap). A thread can read and write to any memory location in the shared process heap.

---

## 3. Concurrency Hazards: The Race Condition
When multiple threads read and write to a shared variable concurrently without synchronization, the final value depends on the exact scheduling order of the threads. This is called a **Race Condition**.

### Code Example: Race Condition in Action (C)
Consider two threads running the following function that increments a shared counter:

```c
#include <stdio.h>
#include <pthread.h>

long long counter = 0; // Shared Global Variable

void* increment_counter(void* arg) {
    for (int i = 0; i < 100000; i++) {
        counter++; // CRITICAL SECTION
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment_counter, NULL);
    pthread_create(&t2, NULL, increment_counter, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("Final Counter Value: %lld (Expected: 200000)\n", counter);
    return 0;
}
```
**Observation:** Running this program yields values like `148392` or `182902`, but rarely `200000`.
**Why?** The assembly instruction for `counter++` consists of three operations:
1.  **Read:** Load the counter value from memory into a CPU register.
2.  **Modify:** Increment the register value.
3.  **Write:** Copy the register value back to memory.
If a context switch occurs between Read and Write, one thread overwrites the increments made by the other.

---

## 4. Synchronization Primitives (Mitigations)
To prevent race conditions, access to the shared resource must be restricted so only one thread enters the **Critical Section** at a time.

### A. Mutex (Mutual Exclusion Lock)
A binary flag used as a locking mechanism.
*   **Acquire Lock:** A thread locks the mutex. If the mutex is already locked, the calling thread blocks (sleeps) until it is released.
*   **Release Lock:** The thread releases the lock, waking up one waiting thread.

#### Resolving the Race Condition with Mutex:
```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void* safe_increment(void* arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&lock);   // Enter Critical Section
        counter++;
        pthread_mutex_unlock(&lock); // Exit Critical Section
    }
    return NULL;
}
```
*Result:* Running with mutex guarantees the final counter is exactly `200000`.

### B. Semaphores
A synchronization tool that uses an integer value to control access by multiple processes/threads.
1.  **Binary Semaphore (Value = 1):** Functions identically to a Mutex, but can be unlocked by a different thread than the one that locked it.
2.  **Counting Semaphore (Value > 1):** Regulates access to a pool of $N$ identical resources (e.g., connection pools).

---

## 5. Process Communication (IPC) vs. Thread Communication
Since processes cannot share memory directly, they must rely on **Inter-Process Communication (IPC)** managed by the kernel:

| Mechanism | Communication Mode | Speed | Description |
| :--- | :--- | :--- | :--- |
| **Pipes / Named Pipes** | Unidirectional Stream | Moderate | A buffer managed by the kernel. Data written by one process is read by another (FIFO). |
| **Sockets** | Bidirectional Stream | Slowest | Used for communication between processes on different machines (network) or locally (Unix domain sockets). |
| **Shared Memory (SHM)** | Direct Shared Buffer | Fastest | The OS maps the same physical memory frame into the virtual address spaces of two processes. *Note:* Requires manual synchronization (semaphores) to prevent race conditions. |
| **Message Queues** | Message Passing | Moderate | System-wide queues where processes read/write structured packets of data. |
| **Thread Memory** | Direct Variable Access | Instant | Sibling threads read/write global/heap memory directly without kernel involvement. |
