# Class Notes: Multitasking Paradigms - Advantages, Limits & Use Cases
**Course:** CS-301 Operating Systems Lab  
**Module 2:** Process vs. Thread Execution in Modern Systems  
**Topic:** Multitasking analysis, trade-offs, and industrial case studies  
**Date:** June 11, 2026  

---

## 1. Objective
To document the structural advantages and limitations of process-based and thread-based multitasking models, and analyze how modern production software leverages these models to build robust, scalable applications.

---

## 2. Process-Based vs. Thread-Based Multitasking

*   **Process-Based Multitasking:** Running two or more distinct programs concurrently. The operating system manages CPU scheduling between separate applications (e.g., editing a document in VS Code while listening to music on Spotify).
*   **Thread-Based Multitasking:** Running two or more concurrent execution paths *within the same program* (e.g., a web browser rendering web page content on one thread while loading images over the network on a background thread).

---

## 3. Comparative Trade-offs (Advantages & Limitations)

### A. Process-Based Model

#### Advantages:
1.  **Robust Security & Isolation:** Since processes are completely isolated in memory, a vulnerability or crash in one process does not compromise or terminate other running processes.
2.  **Ease of Development (No Shared State):** Developers do not have to worry about synchronization bugs like race conditions or deadlocks, as memory is private by default.
3.  **Simple Crash Boundaries:** If a process runs out of memory or hits a segmentation fault, only that process is terminated by the OS.

#### Limitations:
1.  **High Resource Footprint:** Spawning thousands of processes will quickly exhaust system RAM and crash the OS.
2.  **Slow Context Switching:** The CPU must flush caches (TLBs) and switch MMU page tables, hurting throughput.
3.  **Complex Inter-Process Communication (IPC):** Sharing data requires operating system system calls (e.g., pipes, sockets, shared memory segments, or message queues), which are slow and difficult to coordinate.

---

### B. Thread-Based Model

#### Advantages:
1.  **High Throughput & Speed:** Creating, context-switching, and terminating threads is up to 100x faster than processes.
2.  **Efficient Communication:** Since threads share the process heap and data segment, they can share large objects and data structures directly without using IPC system calls.
3.  **Scalability on Multi-core CPUs:** Threads easily map to multiple physical CPU cores, allowing parallel execution of a single application's workload.

#### Limitations:
1.  **No Memory Protection:** If a single thread suffers a segmentation fault or an unhandled exception, it crashes the entire process, killing all sibling threads.
2.  **Synchronization Complexity:** Shared memory introduces concurrency hazards:
    *   **Race Conditions:** Multiple threads write to the same memory concurrently, corrupting data.
    *   **Deadlocks:** Two or more threads are blocked forever, waiting for locks held by each other.
    *   **Thread Starvation:** A thread is perpetually denied resources.
3.  **Debugging & Testing Difficulty:** Concurrency issues are non-deterministic (occur randomly depending on thread scheduling), making them notoriously difficult to debug.

---

## 4. Modern Industrial Case Studies

Modern, high-performance systems rarely rely on a single model. Instead, they combine both models to maximize performance and reliability:

### Case Study 1: Google Chrome (Multi-Process Browser)
*   **Architecture:** Historically, web browsers ran all tabs on a single process. A crash in one tab crashed the whole browser. Chrome pioneered the multi-process architecture:
    *   **Browser Process:** Manages UI, address bar, bookmarks, and file system access.
    *   **Render Process:** Each tab runs in an isolated render process. It converts HTML/JS to pixels.
    *   **GPU Process:** Renders 3D graphics.
*   **Rationale:** Sandbox security (a compromise in a tab's JS runtime cannot access the OS file system) and fault tolerance (if a tab hangs, other tabs remain unaffected).

### Case Study 2: Database Management Systems (PostgreSQL vs. MySQL)
*   **PostgreSQL (Process-Based):** Spawns a dedicated OS process for each client connection.
    *   *Advantage:* If a client query runs a bugged procedure that crashes, it only kills that client's connection process. The core database engine continues running.
*   **MySQL (Thread-Based):** Spawns a dedicated thread per connection within a single server process.
    *   *Advantage:* mysql can handle a much higher volume of simultaneous client connections with lower RAM consumption, though a memory corruption bug in one thread could crash the entire MySQL instance.

### Case Study 3: Real-Time Game Engines (e.g., Unreal Engine)
*   **Architecture:** Highly multi-threaded.
    *   **Main Thread:** Coordinates game logic, user input, and frame pacing.
    *   **Render Thread:** Prepares draw calls for the GPU.
    *   **Physics Thread:** Computes rigid body collisions and gravity.
    *   **Audio Thread:** Handles spatial sound mixing and filters.
*   **Rationale:** Microsecond-level responsiveness. Using separate processes would be too slow due to the immense volume of shared game state data (vertices, positions, velocities) that must be accessed at 60+ FPS.
