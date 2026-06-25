# Class Notes: Semaphore-Based Synchronization & Concurrency Control
**Course:** CS-301 Operating Systems Lab  
**Module 4:** Process Synchronization & Concurrency  
**Topic:** Semaphores (Binary & Counting), Mutexes, and Classic Synchronization Problems  
**Date:** June 25, 2026  

---

## 1. Objective
To examine the operations and implementation details of semaphores, compare them with mutual exclusion (mutex) locks, and solve classic concurrency problems (Producer-Consumer/Bounded-Buffer, Readers-Writers, and Dining Philosophers).

---

## 2. Semaphores: Definition and Operations
A **Semaphore** is a synchronization tool introduced by Edsger Dijkstra. It is an integer variable $S$ that is accessed only through two standard atomic, indivisible operations: `wait()` (also called `P` or `down`) and `signal()` (also called `V` or `up`).

### A. The `wait()` Operation:
```c
void wait(Semaphore S) {
    while (S <= 0) {
        // Busy wait / Block process in wait queue
    }
    S--;
}
```

### B. The `signal()` Operation:
```c
void signal(Semaphore S) {
    S++;
    // Wake up a blocked process from wait queue
}
```

---

## 3. Types of Semaphores
1.  **Binary Semaphore:**
    *   The integer value can range only between $0$ and $1$.
    *   Behaves similarly to a mutex lock. Used for providing **mutual exclusion**.
2.  **Counting Semaphore:**
    *   The integer value can range over an unrestricted domain.
    *   Used to control access to a resource pool containing a finite number of instances. The semaphore is initialized to the number of resources available.

---

## 4. Classic Problems of Synchronization

### A. The Bounded-Buffer (Producer-Consumer) Problem
A pool of $N$ buffers, each capable of holding one item. 
*   **Producer:** Generates data items and puts them into the buffer.
*   **Consumer:** Takes data items out of the buffer and processes them.

#### Semaphores Used:
*   `mutex` (Binary): Initialized to $1$. Protects buffer pointer updates.
*   `empty` (Counting): Initialized to $N$. Tracks empty slots.
*   `full` (Counting): Initialized to $0$. Tracks filled slots.

#### C Pseudo-Code:
```c
// Producer Process
while (true) {
    // Produce item
    wait(empty);
    wait(mutex);
    // Add item to buffer
    signal(mutex);
    signal(full);
}

// Consumer Process
while (true) {
    wait(full);
    wait(mutex);
    // Remove item from buffer
    signal(mutex);
    signal(empty);
    // Consume item
}
```

---

### B. The Readers-Writers Problem
A database is shared among multiple concurrent processes.
*   **Readers:** Only read the database; multiple readers can access concurrently.
*   **Writers:** Update the database; only one writer can access, and no readers are allowed during updates.

#### Semaphores & Variables:
*   `rw_mutex` (Binary): Initialized to $1$. Protects writing.
*   `mutex` (Binary): Initialized to $1$. Protects updates to `read_count`.
*   `read_count`: Integer tracking the number of active readers.

#### C Pseudo-Code:
```c
// Writer Process
while (true) {
    wait(rw_mutex);
    // Write / Update data
    signal(rw_mutex);
}

// Reader Process
while (true) {
    wait(mutex);
    read_count++;
    if (read_count == 1) {
        wait(rw_mutex); // First reader blocks writers
    }
    signal(mutex);
    
    // Read data
    
    wait(mutex);
    read_count--;
    if (read_count == 0) {
        signal(rw_mutex); // Last reader unblocks writers
    }
    signal(mutex);
}
```

---

### C. The Dining Philosophers Problem
Five philosophers spend their lives thinking and eating. They share a circular table with five chopsticks. A philosopher needs two chopsticks (left and right) to eat.

```
          Philosopher 0
       [Chopstick 4] [Chopstick 0]
   Philosopher 4     Philosopher 1
    [Chopstick 3]   [Chopstick 1]
       Philosopher 3   Philosopher 2
              [Chopstick 2]
```

#### Deadlock Risk:
If all philosophers sit down, pick up their left chopstick simultaneously, they will all block waiting for their right chopstick, resulting in a **deadlock**.

#### C Implementation for a Philosopher $i$:
```c
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <stdio.h>

#define N 5

sem_t chopsticks[N];

void* philosopher(void* num) {
    int id = *(int*)num;

    while (1) {
        printf("Philosopher %d is thinking...\n", id);
        sleep(1);

        // Deadlock avoidance: uneven allocation
        if (id % 2 == 0) {
            sem_wait(&chopsticks[id]);           // Left
            sem_wait(&chopsticks[(id + 1) % N]); // Right
        } else {
            sem_wait(&chopsticks[(id + 1) % N]); // Right
            sem_wait(&chopsticks[id]);           // Left
        }

        printf("Philosopher %d is eating!\n", id);
        sleep(2);

        sem_post(&chopsticks[id]);
        sem_post(&chopsticks[(id + 1) % N]);
    }
}
```
*Note:* Alternating the order of chopsticks acquired by odd/even philosophers breaks the circular wait condition, avoiding deadlocks.
