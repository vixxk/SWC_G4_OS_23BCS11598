# Class Notes: Dynamic Memory Allocation Strategies & Fragmentation
**Course:** CS-301 Operating Systems Lab  
**Module 6:** Memory Management & Allocation Strategies  
**Topic:** Memory Fit Algorithms (First, Best, Worst), Internal & External Fragmentation  
**Date:** June 25, 2026  

---

## 1. Objective
To examine dynamic memory allocation methods for contiguous memory management, analyze First Fit, Best Fit, and Worst Fit algorithms through a structured numerical problem, and compare internal and external fragmentation behaviors.

---

## 2. Contiguous Memory Allocation Fit Algorithms
When a process requests memory, the OS must choose a block (hole) from the pool of available free memory partition spaces. The three standard strategies are:

1.  **First Fit:**
    *   *Algorithm:* Allocate the first hole that is big enough.
    *   *Properties:* Fast search time since it stops at the first match. Tends to clutter the beginning of memory with small, unusable holes.
2.  **Best Fit:**
    *   *Algorithm:* Allocate the smallest hole that is big enough.
    *   *Properties:* Requires searching the entire list (unless sorted). Leaves behind the smallest leftover holes (fragments), which can be too small to be useful.
3.  **Worst Fit:**
    *   *Algorithm:* Allocate the largest hole available.
    *   *Properties:* Requires searching the entire list. Leaves behind the largest leftover holes, which have a higher chance of hosting another incoming process.

---

## 3. Fragmentation: Types & Definitions
*   **Internal Fragmentation:**
    *   Memory block allocated to a process is slightly larger than the requested size. The unused portion inside the allocated block is wasted and cannot be used by other processes.
*   **External Fragmentation:**
    *   Total free memory space is sufficient to satisfy a request, but the space is not contiguous; it is split into multiple small holes scattered throughout memory.
*   **Compaction:**
    *   A solution to external fragmentation where the OS moves all memory contents to one side to merge all free spaces into a single, contiguous block. *Drawback:* Expensive CPU overhead.

---

## 4. Practice Problem: Numerical Analysis
**Problem Statement:** Five memory partitions are available in physical order:
$$P_1 = 100\text{ KB},\ P_2 = 500\text{ KB},\ P_3 = 200\text{ KB},\ P_4 = 300\text{ KB},\ P_5 = 600\text{ KB}$$

Four processes request memory in order:
*   $J_1 = 212\text{ KB}$
*   $J_2 = 417\text{ KB}$
*   $J_3 = 112\text{ KB}$
*   $J_4 = 426\text{ KB}$

Evaluate the placement of jobs under First Fit, Best Fit, and Worst Fit.

---

### A. First Fit Allocation Tracing
1.  **$J_1\ (212\text{ KB})$:** Checks $P_1$ (100K, too small), checks $P_2$ (500K, fits!).  
    *   $J_1 \rightarrow P_2$. Leftover $P_2 = 500 - 212 = 288\text{ KB}$.
2.  **$J_2\ (417\text{ KB})$:** Checks $P_1$ (100K, too small), checks remaining $P_2$ (288K, too small), checks $P_3$ (200K, too small), checks $P_4$ (300K, too small), checks $P_5$ (600K, fits!).  
    *   $J_2 \rightarrow P_5$. Leftover $P_5 = 600 - 417 = 183\text{ KB}$.
3.  **$J_3\ (112\text{ KB})$:** Checks $P_1$ (100K, too small), checks remaining $P_2$ (288K, fits!).  
    *   $J_3 \rightarrow P_2$ (in its remaining part). Leftover $P_2 = 288 - 112 = 176\text{ KB}$.
4.  **$J_4\ (426\text{ KB})$:** Checks partitions. Only $P_1$ (100K), $P_3$ (200K), $P_4$ (300K), and remaining parts are free. None are $\ge 426\text{ KB}$.  
    *   $J_4$ must **wait** (External fragmentation occurs).

---

### B. Best Fit Allocation Tracing
1.  **$J_1\ (212\text{ KB})$:** Smallest block $\ge 212\text{ KB}$ is $P_4\ (300\text{ KB})$.  
    *   $J_1 \rightarrow P_4$. Leftover $P_4 = 300 - 212 = 88\text{ KB}$.
2.  **$J_2\ (417\text{ KB})$:** Smallest block $\ge 417\text{ KB}$ is $P_2\ (500\text{ KB})$.  
    *   $J_2 \rightarrow P_2$. Leftover $P_2 = 500 - 417 = 83\text{ KB}$.
3.  **$J_3\ (112\text{ KB})$:** Smallest block $\ge 112\text{ KB}$ is $P_3\ (200\text{ KB})$.  
    *   $J_3 \rightarrow P_3$. Leftover $P_3 = 200 - 112 = 88\text{ KB}$.
4.  **$J_4\ (426\text{ KB})$:** Smallest block $\ge 426\text{ KB}$ is $P_5\ (600\text{ KB})$.  
    *   $J_4 \rightarrow P_5$. Leftover $P_5 = 600 - 426 = 174\text{ KB}$.

*Result:* All jobs are successfully allocated!

---

### C. Worst Fit Allocation Tracing
1.  **$J_1\ (212\text{ KB})$:** Largest block is $P_5\ (600\text{ KB})$.  
    *   $J_1 \rightarrow P_5$. Leftover $P_5 = 600 - 212 = 388\text{ KB}$.
2.  **$J_2\ (417\text{ KB})$:** Largest block is $P_2\ (500\text{ KB})$.  
    *   $J_2 \rightarrow P_2$. Leftover $P_2 = 500 - 417 = 83\text{ KB}$.
3.  **$J_3\ (112\text{ KB})$:** Largest block is remaining $P_5\ (388\text{ KB})$.  
    *   $J_3 \rightarrow P_5$. Leftover $P_5 = 388 - 112 = 276\text{ KB}$.
4.  **$J_4\ (426\text{ KB})$:** Partition checks: none are $\ge 426\text{ KB}$.  
    *   $J_4$ must **wait**.

---

## 5. Python Code: Memory Fit Algorithms Simulation
```python
def first_fit(blocks, files):
    allocation = [-1] * len(files)
    block_sizes = list(blocks)
    
    for i, file_size in enumerate(files):
        for j, block_size in enumerate(block_sizes):
            if block_size >= file_size:
                allocation[i] = j
                block_sizes[j] -= file_size
                break
    return allocation

def best_fit(blocks, files):
    allocation = [-1] * len(files)
    block_sizes = list(blocks)
    
    for i, file_size in enumerate(files):
        best_idx = -1
        for j, block_size in enumerate(block_sizes):
            if block_size >= file_size:
                if best_idx == -1 or block_size < block_sizes[best_idx]:
                    best_idx = j
        if best_idx != -1:
            allocation[i] = best_idx
            block_sizes[best_idx] -= file_size
    return allocation

if __name__ == "__main__":
    blocks = [100, 500, 200, 300, 600]
    jobs = [212, 417, 112, 426]
    
    print("First Fit Placements:", first_fit(blocks, jobs))
    print("Best Fit Placements :", best_fit(blocks, jobs))
```
