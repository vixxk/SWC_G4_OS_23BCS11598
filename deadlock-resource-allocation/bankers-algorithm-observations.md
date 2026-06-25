# Class Notes: Safe Resource Allocation Strategy (Banker's Algorithm)
**Course:** CS-301 Operating Systems Lab  
**Module 5:** Deadlock Management & Resource Allocation  
**Topic:** Banker's Algorithm, Safe States, Need Matrix, and Resource-Request Handling  
**Date:** June 25, 2026  

---

## 1. Objective
To understand Banker's Algorithm for deadlock avoidance, calculate Need Matrices, validate safe system states, and simulate resource-request evaluations using a structured numerical example and code implementation.

---

## 2. Safe vs. Unsafe States
A state is **safe** if the system can allocate resources to each process (up to its maximum claims) in some order and still avoid a deadlock.
*   **Safe State:** There exists a **Safe Sequence** $\langle P_1, P_2, \dots, P_n \rangle$ of processes such that, for each $P_i$, the resources that $P_i$ can still request can be satisfied by the currently available resources plus the resources held by all prior processes in the sequence.
*   **Unsafe State:** Not all processes can finish execution without entering a circular wait. *Note: An unsafe state is not a deadlock, but it may lead to one.*

---

## 3. Banker's Algorithm Data Structures
Let $n$ be the number of processes and $m$ be the number of resource types.

1.  `Available[m]`: Available instances of each resource type.
2.  `Max[n][m]`: Maximum demand of each process.
3.  `Allocation[n][m]`: Number of resource instances currently allocated to each process.
4.  `Need[n][m]`: Remaining resource instances needed to complete execution.
    $$\text{Need}[i][j] = \text{Max}[i][j] - \text{Allocation}[i][j]$$

---

## 4. Practice Problem: Numerical Analysis
**Problem Statement:** Consider 5 processes $P_0$ through $P_4$ and 3 resource types $A, B, C$.
*   Total instances in system: $A = 10, B = 5, C = 7$

At $t_0$, the allocation state is:
*   **Allocation Matrix:**
    *   $P_0$: $0\ 1\ 0$ | $P_1$: $2\ 0\ 0$ | $P_2$: $3\ 0\ 2$ | $P_3$: $2\ 1\ 1$ | $P_4$: $0\ 0\ 2$
*   **Max Matrix:**
    *   $P_0$: $7\ 5\ 3$ | $P_1$: $3\ 2\ 2$ | $P_2$: $9\ 0\ 2$ | $P_3$: $2\ 2\ 2$ | $P_4$: $4\ 3\ 3$
*   **Available Vector:**
    *   $A = 3, B = 3, C = 2$

---

### Step 1: Compute the Need Matrix
$$\text{Need} = \text{Max} - \text{Allocation}$$

*   $\text{Need}[P_0] = [7, 5, 3] - [0, 1, 0] = [7, 4, 3]$
*   $\text{Need}[P_1] = [3, 2, 2] - [2, 0, 0] = [1, 2, 2]$
*   $\text{Need}[P_2] = [9, 0, 2] - [3, 0, 2] = [6, 0, 0]$
*   $\text{Need}[P_3] = [2, 2, 2] - [2, 1, 1] = [0, 1, 1]$
*   $\text{Need}[P_4] = [4, 3, 3] - [0, 0, 2] = [4, 3, 1]$

#### Need Matrix Table:
| Process | Need (A B C) |
| :---: | :---: |
| **$P_0$** | 7 4 3 |
| **$P_1$** | 1 2 2 |
| **$P_2$** | 6 0 0 |
| **$P_3$** | 0 1 1 |
| **$P_4$** | 4 3 1 |

---

### Step 2: Safety Algorithm Verification
*   **Initial Work** = $[3, 3, 2]$. **Finish** = `[False, False, False, False, False]`.
*   **Iteration 1:** Check $P_0$. Is $\text{Need}[P_0] \le \text{Work}$? $[7, 4, 3] \le [3, 3, 2]$ (No).
*   **Iteration 2:** Check $P_1$. Is $\text{Need}[P_1] \le \text{Work}$? $[1, 2, 2] \le [3, 3, 2]$ (Yes!).  
    *   $\text{Work} = \text{Work} + \text{Allocation}[P_1] = [3, 3, 2] + [2, 0, 0] = [5, 3, 2]$
    *   $\text{Finish}[P_1] = \text{True}$.
*   **Iteration 3:** Check $P_2$. Is $\text{Need}[P_2] \le \text{Work}$? $[6, 0, 0] \le [5, 3, 2]$ (No).
*   **Iteration 4:** Check $P_3$. Is $\text{Need}[P_3] \le \text{Work}$? $[0, 1, 1] \le [5, 3, 2]$ (Yes!).  
    *   $\text{Work} = \text{Work} + \text{Allocation}[P_3] = [5, 3, 2] + [2, 1, 1] = [7, 4, 3]$
    *   $\text{Finish}[P_3] = \text{True}$.
*   **Iteration 5:** Check $P_4$. Is $\text{Need}[P_4] \le \text{Work}$? $[4, 3, 1] \le [7, 4, 3]$ (Yes!).  
    *   $\text{Work} = \text{Work} + \text{Allocation}[P_4] = [7, 4, 3] + [0, 0, 2] = [7, 4, 5]$
    *   $\text{Finish}[P_4] = \text{True}$.
*   **Iteration 6:** Check $P_0$. Is $\text{Need}[P_0] \le \text{Work}$? $[7, 4, 3] \le [7, 4, 5]$ (Yes!).  
    *   $\text{Work} = \text{Work} + \text{Allocation}[P_0] = [7, 4, 5] + [0, 1, 0] = [7, 5, 5]$
    *   $\text{Finish}[P_0] = \text{True}$.
*   **Iteration 7:** Check $P_2$. Is $\text{Need}[P_2] \le \text{Work}$? $[6, 0, 0] \le [7, 5, 5]$ (Yes!).  
    *   $\text{Work} = \text{Work} + \text{Allocation}[P_2] = [7, 5, 5] + [3, 0, 2] = [10, 5, 7]$
    *   $\text{Finish}[P_2] = \text{True}$.

**Conclusion:** All processes finished! The state is **safe**, and a Safe Sequence is:
$$\langle P_1, P_3, P_4, P_0, P_2 \rangle$$

---

## 5. Python Code: Safety Check Simulation
```python
def is_safe_state(alloc, max_res, avail):
    n = len(alloc)
    m = len(avail)
    
    need = [[max_res[i][j] - alloc[i][j] for j in range(m)] for i in range(n)]
    work = list(avail)
    finish = [False] * n
    safe_seq = []
    
    while len(safe_seq) < n:
        allocated_in_round = False
        for i in range(n):
            if not finish[i]:
                # Check if need <= work
                if all(need[i][j] <= work[j] for j in range(m)):
                    # Add allocated to work
                    for j in range(m):
                        work[j] += alloc[i][j]
                    finish[i] = True
                    safe_seq.append(i)
                    allocated_in_round = True
                    print(f"P{i} allocated. Work updated to: {work}")
                    break
        if not allocated_in_round:
            print("System is in UNSAFE STATE!")
            return False, []
            
    print(f"System is in SAFE STATE. Safe Sequence: {safe_seq}")
    return True, safe_seq

if __name__ == "__main__":
    alloc = [
        [0, 1, 0],
        [2, 0, 0],
        [3, 0, 2],
        [2, 1, 1],
        [0, 0, 2]
    ]
    max_res = [
        [7, 5, 3],
        [3, 2, 2],
        [9, 0, 2],
        [2, 2, 2],
        [4, 3, 3]
    ]
    avail = [3, 3, 2]
    is_safe_state(alloc, max_res, avail)
```
