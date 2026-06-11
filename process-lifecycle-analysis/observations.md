# Class Notes: Foreground vs. Background Process Execution & Job Control
**Course:** CS-301 Operating Systems Lab  
**Module 1:** Process Management & Execution Models  
**Topic:** Execution Modes, Keyboard Signals, and Session Hangup (SIGHUP)  
**Date:** June 11, 2026  

---

## 1. Objective
To compare and contrast foreground and background execution behaviors in Linux shells, analyze how terminal control signals affect running processes, and understand techniques for session persistence.

---

## 2. Foreground vs. Background Execution: Comparative Analysis
Every process spawned in a terminal session (e.g., bash or zsh) runs in one of two execution modes:

### A. Foreground Execution
*   **Definition:** The process is given exclusive control of the terminal window's standard input (stdin) and standard output (stdout/stderr).
*   **Behavior:** The shell prompt is blocked. The user cannot input other command lines until the foreground process completes, halts, or is moved to the background.
*   **Spawn Syntax:** Running a command normally (e.g., `sleep 100`).

### B. Background Execution
*   **Definition:** The process runs concurrently with the shell, relinquishing control of stdin, but sharing stdout/stderr (unless redirected).
*   **Behavior:** The shell prompt remains active immediately, allowing the user to enter new commands while the task runs.
*   **Spawn Syntax:** Appending an ampersand (`&`) to the end of the command (e.g., `sleep 100 &`).

---

## 3. Keyboard Control Signals
Signals are asynchronous notifications sent by the kernel or processes to notify a target process of an event. In foreground execution, keyboard combinations map directly to signals:

*   **`Ctrl + C` (Sends `SIGINT` - Signal Interrupt):**
    *   **Behavior:** Requests the foreground process to terminate immediately.
    *   **Action:** The default action is process termination. Processes can catch or ignore this signal to perform cleanup operations.
*   **`Ctrl + Z` (Sends `SIGTSTP` - Signal Terminal Stop):**
    *   **Behavior:** Requests the foreground process to suspend/pause execution.
    *   **Action:** The process is suspended, its state is frozen in memory, and control is returned to the shell prompt. The process is not dead; it is in the `T` (Stopped) state.

---

## 4. Shell Job Control Commands
A shell maintains a table of active background and suspended processes, called **jobs**. The following commands manage these jobs:

1.  **`jobs`:**
    *   Lists all background and suspended jobs spawned in the current shell session, displaying their job ID, state (Running, Stopped), and the spawning command.
2.  **`&` (Ampersand operator):**
    *   Spawns a process directly into the background (e.g., `python3 script.py &`).
3.  **`bg %[job_id]` (Background command):**
    *   Resumes a stopped/suspended job (usually paused via `Ctrl + Z`) in the background, allowing it to execute concurrently.
4.  **`fg %[job_id]` (Foreground command):**
    *   Brings a background or stopped job into the foreground, giving it terminal control.

### Walkthrough Example of Job Control:
```bash
$ sleep 200                  # Spawn process in foreground
^Z                           # User presses Ctrl + Z
[1]+  Stopped                 sleep 200

$ jobs                       # Verify job status
[1]+  Stopped                 sleep 200

$ bg %1                      # Resume job in background
[1]+ sleep 200 &

$ jobs                       # Check job status again
[1]+  Running                 sleep 200 &

$ fg %1                      # Bring job back to foreground
sleep 200
^C                           # Kill process with SIGINT
```

---

## 5. Session Termination and Hangup (`SIGHUP`)
A terminal window or SSH session acts as a parent session. What happens to child jobs when this session is closed?

*   **`SIGHUP` (Signal Hang Up - Signal 1):** When a terminal window or SSH session is closed, the shell sends a `SIGHUP` signal to all processes (jobs) associated with its controlling terminal.
*   **Default Action:** By default, processes receiving `SIGHUP` terminate immediately.

### Strategies for Session Persistence (Backgrounding Beyond Session Lifetime):
If a long-running process needs to survive the closure of the terminal, the following methods are used:

1.  **`nohup` (No Hang Up):**
    *   Prefixing a command with `nohup` prevents the process from receiving the `SIGHUP` signal.
    *   It redirects standard output and standard error to a file called `nohup.out` by default.
    *   *Usage:* `nohup python3 long_running_script.py &`
2.  **`disown`:**
    *   A shell-builtin command that removes a job from the shell's active jobs list.
    *   When the terminal exits, the shell does not send a `SIGHUP` to disowned processes.
    *   *Usage:* Run command -> suspend it (`Ctrl + Z`) -> run `bg` -> run `disown %[job_id]`.
3.  **Terminal Multiplexers (`tmux` / `screen`):**
    *   Creates a persistent server session that manages terminals independently of the window manager or SSH connection.
    *   Users can disconnect (detach) from the session and reconnect (attach) later from a different terminal without interrupting running processes.
