# Class Notes: Scheduling Algorithm Trade-offs & Follow-up Exercises
**Course:** CS-301 Operating Systems Lab  
**Module 3:** CPU Scheduling Algorithms  
**Topic:** Follow-up Practice Problems, Short Notes, and Comparative Analysis  
**Date:** June 11, 2026  

---

## 1. Follow-up Practice Task: Additional Scheduling Numerical
**Problem Statement:** Four processes arrive at different times with the following characteristics. Solve for FCFS, Non-Preemptive SJF, and Preemptive SJF (SRTF):
*   $P_1$: $AT = 0\text{ ms}$, $BT = 5\text{ ms}$
*   $P_2$: $AT = 1\text{ ms}$, $BT = 3\text{ ms}$
*   $P_3$: $AT = 2\text{ ms}$, $BT = 8\text{ ms}$
*   $P_4$: $AT = 4\text{ ms}$, $BT = 2\text{ ms}$

---

### A. FCFS Scheduling Solution
**Order of Execution:** $P_1 \rightarrow P_2 \rightarrow P_3 \rightarrow P_4$ (by Arrival Time)

#### Calculation Table:
| Process | Arrival Time ($AT$) | Burst Time ($BT$) | Completion Time ($CT$) | Turnaround Time ($TAT$) | Waiting Time ($WT$) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | 0 | 5 | 5 | 5 | 0 |
| **$P_2$** | 1 | 3 | 8 | 7 | 4 |
| **$P_3$** | 2 | 8 | 16 | 14 | 6 |
| **$P_4$** | 4 | 2 | 18 | 14 | 12 |
| **Total** | - | 18 | - | **40** | **22** |

*   **Average Turnaround Time (FCFS):** $\frac{40}{4} = 10.0\text{ ms}$
*   **Average Waiting Time (FCFS):** $\frac{22}{4} = 5.5\text{ ms}$

---

### B. Non-Preemptive SJF Solution
*   **Time 0:** Only $P_1$ has arrived. $P_1$ runs to completion ($t = 5$).
*   **Time 5:** $P_2, P_3, P_4$ have arrived. Sorted by burst time: $P_4\ (2) < P_2\ (3) < P_3\ (8)$.  
    $P_4$ executes next ($t = 5 \rightarrow 7$).
*   **Time 7:** $P_2$ executes next ($t = 7 \rightarrow 10$).
*   **Time 10:** $P_3$ executes last ($t = 10 \rightarrow 18$).

#### Calculation Table:
| Process | Arrival Time ($AT$) | Burst Time ($BT$) | Completion Time ($CT$) | Turnaround Time ($TAT$) | Waiting Time ($WT$) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | 0 | 5 | 5 | 5 | 0 |
| **$P_4$** | 4 | 2 | 7 | 3 | 1 |
| **$P_2$** | 1 | 3 | 10 | 9 | 6 |
| **$P_3$** | 2 | 8 | 18 | 16 | 8 |
| **Total** | - | 18 | - | **33** | **15** |

*   **Average Turnaround Time (NP-SJF):** $\frac{33}{4} = 8.25\text{ ms}$
*   **Average Waiting Time (NP-SJF):** $\frac{15}{4} = 3.75\text{ ms}$

---

### C. Preemptive SJF (SRTF) Solution
*   **Time 0:** $P_1$ runs (Remaining $P_1 = 5$).
*   **Time 1:** $P_2$ (BT=3) arrives. Remaining $P_1 = 4$. Since $BT(P_2) < Rem(P_1)$, $P_1$ is preempted. $P_2$ runs.
*   **Time 2:** $P_3$ (BT=8) arrives. Remaining $P_2 = 2$. $P_2$ continues.
*   **Time 4:** $P_4$ (BT=2) arrives. $P_2$ has completed (ran $t = 1 \rightarrow 4$).  
    Queue has: $P_1$ (rem=4), $P_3$ (rem=8), $P_4$ (rem=2). Shortest is $P_4$. $P_4$ runs ($t = 4 \rightarrow 6$).
*   **Time 6:** $P_4$ completes. Queue has: $P_1$ (rem=4), $P_3$ (rem=8). Shortest is $P_1$. $P_1$ runs ($t = 6 \rightarrow 10$).
*   **Time 10:** $P_1$ completes. $P_3$ runs last ($t = 10 \rightarrow 18$).

#### Calculation Table:
| Process | Arrival Time ($AT$) | Burst Time ($BT$) | Completion Time ($CT$) | Turnaround Time ($TAT$) | Waiting Time ($WT$) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | 0 | 5 | 10 | 10 | 5 |
| **$P_2$** | 1 | 3 | 4 | 3 | 0 |
| **$P_4$** | 4 | 2 | 6 | 2 | 0 |
| **$P_3$** | 2 | 8 | 18 | 16 | 8 |
| **Total** | - | 18 | - | **31** | **13** |

*   **Average Turnaround Time (SRTF):** $\frac{31}{4} = 7.75\text{ ms}$
*   **Average Waiting Time (SRTF):** $\frac{13}{4} = 3.25\text{ ms}$

---

## 2. Scheduling Summary Comparison Table

| Algorithm | Average Waiting Time ($WT$) | Average Turnaround Time ($TAT$) | Primary Characteristic |
| :--- | :---: | :---: | :--- |
| **First-Come, First-Served (FCFS)** | $5.50\text{ ms}$ | $10.00\text{ ms}$ | Simple, non-preemptive, prone to Convoy Effect. |
| **Shortest Job First (NP-SJF)** | $3.75\text{ ms}$ | $8.25\text{ ms}$ | Non-preemptive, optimal scheduling at decision points. |
| **Shortest Remaining Time First (SRTF)** | $3.25\text{ ms}$ | $7.75\text{ ms}$ | Preemptive, optimal minimum average waiting time. |

---

## 3. Short Notes on CPU Scheduling Algorithms

### A. First-Come, First-Served (FCFS) Scheduling
*   **Description:** The simplest CPU scheduling algorithm. The process that requests the CPU first is allocated the CPU first. It is managed using a First-In-First-Out (FIFO) queue.
*   **Preemptive Status:** Strictly **Non-preemptive**. Once a process gets the CPU, it holds it until it terminates or blocks for I/O.
*   **Advantages:**
    1.  Very easy to implement and understand.
    2.  Fairness in terms of arrival order (no process is bypassed).
    3.  **No Starvation:** Every process is guaranteed to execute eventually.
*   **Disadvantages:**
    1.  **Convoy Effect:** Long CPU-bound processes force multiple short I/O-bound processes to wait, degrading system throughput.
    2.  **Not Optimal:** High average waiting time compared to other algorithms.

### B. Shortest Job First (SJF) Scheduling
*   **Description:** Associates each process with the length of its next CPU burst. When the CPU becomes free, the process with the shortest next CPU burst is selected.
*   **Preemptive Status:** Can be either:
    1.  **Non-preemptive SJF:** The running process is allowed to finish its burst even if a shorter process arrives.
    2.  **Preemptive SJF (SRTF):** If a newly arrived process has a shorter remaining burst time than the currently executing process, the current process is preempted.
*   **Advantages:**
    1.  **Optimal:** Mathematically proven to provide the minimum average waiting time for a given set of processes.
*   **Disadvantages:**
    1.  **Starvation (Indefinite Blocking):** Long-running processes can wait indefinitely if a continuous stream of short-duration processes keeps arriving.
    2.  **Unpredictable CPU Bursts:** It is impossible to know the exact length of the next CPU burst in advance. The OS must approximate it using exponential smoothing of past bursts:
        $$\tau_{n+1} = \alpha t_n + (1 - \alpha) \tau_n$$
        where $t_n$ is the actual length of the $n$-th burst, $\tau_n$ is the predicted value, and $\alpha$ is a weighting parameter ($0 \le \alpha \le 1$).
