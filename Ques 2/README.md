# Question 2: Preventing Zombie Processes & Terminating Unresponsive Children

## Design Overview

The `process_manager.c` program creates three child processes using `fork()`. Two of the children complete their work quickly to simulate normal tasks, while the third intentionally sleeps for a much longer time to represent an unresponsive process. The parent continuously monitors all child processes using `waitpid()` with the `WNOHANG` option, allowing it to immediately reap finished children and prevent zombie processes. If a child continues running beyond the specified timeout, the parent first sends `SIGTERM` to request graceful termination and uses `SIGKILL` only if the child still fails to exit.

---

## Commands Executed and Their Purpose

### `gcc process_manager.c -o process_manager`

Compiled the C program into an executable named `process_manager` using the GCC compiler. The program compiled successfully without any compilation errors.

### `./process_manager`

Executed the program.

During execution, the following behavior was observed:

* The parent process created three child processes and displayed the PID of each one.
* Child processes **0** and **2** completed their tasks within a few seconds and were immediately collected by the parent's `waitpid(WNOHANG)` loop, preventing them from becoming zombie processes.
* Child process **1** intentionally remained active beyond the timeout period. After waiting for five seconds, the parent sent a `SIGTERM` signal to terminate it gracefully.
* The program ended with the message **"All children handled. No zombies remain."**, confirming that every child process had been successfully managed.

### `ps aux | grep defunct`

Executed this command in another terminal while the program was running to check for zombie (`<defunct>`) processes.

No zombie processes were displayed, confirming that every child process was properly reaped using `waitpid()` and removed from the process table after termination.

---

## How `fork()`, `waitpid()`, and Signals Work Together

### `fork()`

The `fork()` system call creates a new child process by duplicating the parent process. Both processes continue execution independently, allowing each child to perform its assigned task while the parent manages all running children.

### `waitpid(pid, &status, WNOHANG)`

The parent uses `waitpid()` with the `WNOHANG` option to monitor child processes without blocking. This enables the parent to repeatedly check whether any child has finished while continuing to supervise the remaining processes. Whenever a child exits, it is immediately reaped, preventing zombie processes from accumulating.

### Signal Handling (`SIGTERM` and `SIGKILL`)

If a child process exceeds the allowed execution time, the parent first sends **SIGTERM**. Since this signal can be handled by the child, it allows the process to terminate cleanly by releasing resources such as open files or allocated memory.

If the child does not terminate after receiving **SIGTERM**, the parent sends **SIGKILL**, which cannot be ignored or caught. This guarantees that the operating system immediately terminates the process.

Together, these mechanisms implement a complete process management cycle:

**Process Creation (`fork`) → Non-blocking Monitoring (`waitpid` + `WNOHANG`) → Graceful Termination (`SIGTERM`) → Forced Termination (`SIGKILL`, if required) → Final Cleanup (`waitpid`)**

This approach is commonly used in operating systems and server applications to efficiently manage multiple child processes while preventing zombie processes and freeing system resources.

---

## Screenshots

* **screenshots/01_compile.png** – Compiling the source code using GCC and verifying successful compilation.
* **screenshots/02_run_process_manager.png** – Executing the program and observing process creation, child termination, and reaping.
* **screenshots/03_verify_no_zombies.png** – Running `ps aux | grep defunct` to verify that no zombie processes remained.

---

## Buffering Issue Identified During Testing

During testing, an unexpected issue appeared when the program's output was redirected to a file. Some lines, such as **"Parent created child..."**, appeared multiple times even though they were printed only once in the source code.

This occurred because standard output becomes fully buffered when redirected to a file. If the parent prints output before calling `fork()`, the buffered data is copied into each child process. When a child later exits, it flushes its copy of the buffer, causing the same text to be written multiple times.

To eliminate this issue, the program initializes line-buffered output at the beginning of `main()` using:

`setvbuf(stdout, NULL, _IOLBF, 0);`

With line buffering enabled, every line is flushed immediately after it is printed. As a result, child processes no longer inherit unflushed output, eliminating duplicate lines in both `output.txt` and the captured screenshots.

---

## Files Included

* `process_manager.c` – Source code of the program.
* `process_manager` – Compiled executable.
* `output.txt` – Sample output generated during program execution.
* `screenshots/` – Screenshots showing compilation, execution, and zombie process verification.
