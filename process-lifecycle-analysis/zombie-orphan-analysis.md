# Class Notes: Zombie vs. Orphan Processes - Analysis & Mechanics
**Course:** CS-301 Operating Systems Lab  
**Module 1:** Process Management & Execution Models  
**Topic:** Zombie & Orphan Process Behavior and Code Implementation  
**Date:** June 11, 2026  

---

## 1. Introduction: The Parent-Child Lifecycle
In Unix-like operating systems, processes are created in a hierarchical structure using the `fork()` system call.
*   **Parent Process:** The process that calls `fork()` to spawn a new process.
*   **Child Process:** The newly created process, which gets a copy of the parent's memory, file descriptors, and execution state, but has a unique Process ID (PID).
*   **Process Termination & Reaping:** 
    1. When a child process terminates, it calls `exit()`.
    2. The operating system frees its memory and resources, but keeps its **Process Control Block (PCB)** and PID in the **Process Table**. This is to allow the parent process to read the child's exit status.
    3. The parent process reads this status using the `wait()` or `waitpid()` system call.
    4. Reading the status is called **reaping** the process, which fully deletes the process entry from the process table.

Abnormal behaviors occur when the parent process fails to coordinate its termination or reaping with its child process. This gives rise to **Zombie** and **Orphan** processes.

---

## 2. Zombie Processes (Defunct Processes)

### A. Definition and Cause
A **Zombie process** (or defunct process) is a process that has completed its execution (called `exit()`) but still retains an entry in the system process table. 

**Cause:** The parent process is still running but has failed to call `wait()` or `waitpid()` to read the exit code of its terminated child process.

### B. Code Implementation (C Example)
The C code below demonstrates how to intentionally create a zombie process. The child process exits immediately, while the parent sleeps for a long duration without calling `wait()`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t child_pid = fork();

    if (child_pid < 0) {
        perror("Fork failed");
        exit(1);
    }

    if (child_pid == 0) {
        // Child Process
        printf("[CHILD] Child PID: %d. Exiting now...\n", getpid());
        exit(0); // Child terminates immediately
    } else {
        // Parent Process
        printf("[PARENT] Parent PID: %d. Created Child PID: %d.\n", getpid(), child_pid);
        printf("[PARENT] Sleeping for 60 seconds without calling wait()...\n");
        printf("[PARENT] Run 'ps aux | grep Z' in another terminal to observe the Zombie.\n");
        
        sleep(60); // Parent sleeps, keeping child in zombie state
        
        printf("[PARENT] Parent waking up and exiting.\n");
    }
    return 0;
}
```

### C. Observation and Resource Impact
*   **Observation:** In the terminal, a zombie process is identified by a status code of `Z` or `Z+` in the `STAT` column of `ps aux`, and is marked as `<defunct>`.
*   **Resource Impact:** A zombie process does not consume CPU cycles, RAM, or swap memory since its memory space has been deallocated. However, it **consumes a Process Table Slot (PID)**. Since the maximum number of PIDs is finite (`/proc/sys/kernel/pid_max`), a massive leak of zombie processes can prevent the system from spawning any new processes, leading to system crash.

### D. Resolution
1.  **Reaping by Parent:** Modify the parent code to handle the `SIGCHLD` signal asynchronously, calling `wait()` inside a signal handler.
2.  **Killing the Parent:** If the parent is a rogue process, killing the parent process will cause its zombie children to become orphans. The kernel will then re-parent them to `init` (PID 1), which automatically reaps them.

---

## 3. Orphan Processes

### A. Definition and Cause
An **Orphan process** is a process that is actively running, but whose parent process has terminated (died) before the child finished.

**Cause:** The parent process exits or gets killed while the child process is still executing.

### B. Code Implementation (C Example)
The C code below demonstrates how to create an orphan process. The parent exits immediately, while the child continues executing and gets adopted by the init daemon.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t child_pid = fork();

    if (child_pid < 0) {
        perror("Fork failed");
        exit(1);
    }

    if (child_pid == 0) {
        // Child Process
        printf("[CHILD] Child started. PID: %d. Parent PID: %d\n", getpid(), getppid());
        printf("[CHILD] Sleeping for 20 seconds to allow parent to exit first...\n");
        
        sleep(20); // Child sleeps while parent exits
        
        // After waking up, the parent should have changed (re-parented)
        printf("[CHILD] Child woke up. PID: %d. New Parent PID: %d (init/systemd)\n", getpid(), getppid());
    } else {
        // Parent Process
        printf("[PARENT] Parent PID: %d. Exiting now to orphan child...\n", getpid());
        exit(0); // Parent exits immediately
    }
    return 0;
}
```

### C. Observation and Linux Handling (Adoption Mechanism)
*   When a parent process exits, the operating system kernel inspects its children and automatically re-parents them.
*   Traditionally, orphans are adopted by `init` (PID 1). In modern Linux systems running systemd, they are often adopted by a user-level sub-reaper or systemd manager (e.g., systemd-init or `systemd --user`).
*   The adopting process (PID 1) continuously monitors its adopted children and calls `wait()` when they terminate, ensuring they never become permanent zombies.

### D. Resource Impact
Orphan processes continue to run normally and consume CPU and RAM resources to complete their tasks. They do not cause a resource leak after termination because their adopted parent (`init` or user sub-reaper) will guarantee they are reaped.

---

## 4. Key Summary Comparison: Zombie vs. Orphan

| Metric | Zombie Process | Orphan Process |
| :--- | :--- | :--- |
| **Current Execution Status** | Finished/Terminated (`exit()` called). | Still running and executing instructions. |
| **Parent Process Status** | Active/Running but ignoring child termination. | Terminated/Dead. |
| **Parent PID (PPID)** | Points to original parent. | Points to `init` (PID 1) or systemd sub-reaper. |
| **Resource Consumption** | Memory freed; consumes 1 PID slot in Process Table. | Normal CPU and Memory consumption of an active program. |
| **Danger Level** | High (can exhaust PIDs if accumulated). | Low (normal background operation, cleaned by `init`). |
| **How to Resolve** | Parent calls `wait()`, or kill the parent process. | No manual action required; OS handles via adoption. |
