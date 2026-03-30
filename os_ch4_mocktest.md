---
layout: page
title: "OS Ch4 Mock Test"
---

# Mock Test: The Process Abstraction

**Instructions:** Answer without looking at notes. Time yourself — aim for 45 minutes. Check answers AFTER completing everything.

---

## Section A: MCQs (1 mark each)

**1.** A process is best described as:

- (a) A file stored on disk
- (b) A program in execution
- (c) A CPU register
- (d) An operating system kernel

**2.** Which transition is INVALID in the 3-state process model?

- (a) Running &rarr; Blocked
- (b) Blocked &rarr; Ready
- (c) Ready &rarr; Blocked
- (d) Ready &rarr; Running

**3.** The counterpart of time sharing is:

- (a) Load balancing
- (b) Multithreading
- (c) Space sharing
- (d) Pipelining

**4.** Which is NOT part of a process's machine state?

- (a) Address space
- (b) Registers
- (c) Disk firmware
- (d) I/O information

**5.** In UNIX, the default file descriptor for standard error is:

- (a) 0
- (b) 1
- (c) 2
- (d) 3

**6.** A zombie process is one that has:

- (a) Been killed by the OS
- (b) Finished execution but not been reaped by its parent
- (c) Lost its parent process
- (d) Been suspended indefinitely

**7.** Modern operating systems use which loading strategy?

- (a) Eager loading
- (b) Lazy loading
- (c) Random loading
- (d) Sequential loading

**8.** The data structure used to store per-process information is called:

- (a) Page Table
- (b) File Allocation Table
- (c) Process Control Block
- (d) Interrupt Vector Table

**9.** A mechanism answers the question:

- (a) WHICH process to run
- (b) WHEN to schedule
- (c) HOW to perform an operation
- (d) WHY a process was blocked

**10.** After I/O completion, a blocked process moves to:

- (a) Running
- (b) Terminated
- (c) Ready
- (d) New

---

## Section B: Short Answer (2-3 marks each)

**11.** Differentiate between a program and a process. (3 marks)

**12.** What is a context switch? What gets saved and restored? (3 marks)

**13.** Why are mechanisms and policies separated in OS design? Give one example of each. (3 marks)

**14.** What is the difference between a zombie process and an orphan process? (3 marks)

**15.** Explain eager loading vs lazy loading. Which do modern OSes use and why? (2 marks)

---

## Section C: Long Answer (5+ marks each)

**16.** Describe the complete process of how a program on disk becomes a running process. List all steps in order. (5 marks)

**17.** Draw and explain the 5-state process lifecycle diagram. Label all states and transitions with their proper names. (5 marks)

**18.** Consider two processes sharing a single CPU:
- Process A: runs 3 CPU cycles, then initiates I/O (takes 3 cycles), then runs 1 more CPU cycle.
- Process B: runs 4 CPU cycles (no I/O).
- OS switches to B when A blocks.

Fill in the trace table and calculate CPU utilization. (5 marks)

---

## Section D: Tricky/Application (3 marks each)

**19.** A system has 4 processes and 1 CPU. What is the maximum number of processes that can be in each state simultaneously? Can all 4 be Blocked? What does that imply?

**20.** A student claims: "When a process finishes its I/O, it immediately starts Running again." Is this correct? Explain why or why not using the concept of mechanism vs policy separation.

---

---

# Answer Key

<details>
<summary>Click to reveal all answers</summary>

<h3>Section A</h3>
<ol>
<li><b>(b)</b> A program in execution</li>
<li><b>(c)</b> Ready &rarr; Blocked — a process must be Running to initiate I/O</li>
<li><b>(c)</b> Space sharing</li>
<li><b>(c)</b> Disk firmware — machine state = address space + registers + I/O info</li>
<li><b>(c)</b> 2 — (0=stdin, 1=stdout, 2=stderr)</li>
<li><b>(b)</b> Finished but parent hasn't called wait()</li>
<li><b>(b)</b> Lazy loading — loads code/data on demand using paging</li>
<li><b>(c)</b> Process Control Block</li>
<li><b>(c)</b> HOW — mechanisms are low-level implementation protocols</li>
<li><b>(c)</b> Ready — NOT Running; the scheduler decides when it actually runs</li>
</ol>

<h3>Section B</h3>

<p><b>11.</b> Program = passive entity stored on disk (just code + static data). Process = active entity in memory with state (registers, address space, open files). One program can create multiple processes. Program has no state; process has state (running/ready/blocked).</p>

<p><b>12.</b> Context switch = saving one process's state and loading another's to switch CPU execution. Saved/restored: all CPU registers (PC, SP, FP, general purpose registers like eip, esp, ebx, ecx, edx, esi, edi, ebp in x86). Stored in the process's PCB (context field). Loaded from the next process's PCB.</p>

<p><b>13.</b> Separated for modularity — can change decision logic without rewriting implementation code. Mechanism example: context switch (HOW to stop one process, start another). Policy example: scheduling algorithm like Round Robin or Priority (WHICH process to run next). Changing from FCFS to Round Robin doesn't require rewriting the context switch code.</p>

<p><b>14.</b> Zombie = child process that has FINISHED execution but still has process table entry because parent hasn't called wait(). It's dead but unreported. Orphan = child process still RUNNING whose parent has terminated. It's alive but parentless. Zombies are reaped when parent calls wait(). Orphans are adopted by init (PID 1).</p>

<p><b>15.</b> Eager = load all code and data into memory before execution starts (simple, old OSes). Lazy = load only what's needed during execution on demand (modern OSes). Modern OSes use lazy because it's more efficient — no wasted memory on unused code. Relies on paging/swapping mechanisms.</p>

<h3>Section C</h3>

<p><b>16.</b> Five steps in order:</p>
<ol>
<li>Load code and static data from disk executable into the process's address space in memory</li>
<li>Allocate and initialize the runtime stack (set up argc and argv for main)</li>
<li>Allocate heap memory for dynamic allocation (initially small, grows via malloc)</li>
<li>Set up I/O — open 3 default file descriptors (0=stdin, 1=stdout, 2=stderr)</li>
<li>Jump to entry point main() — OS transfers CPU control to the process</li>
</ol>

<p><b>17.</b> Five states: New/Start, Ready, Running, Waiting/Blocked, Terminated.<br>
Transitions:</p>
<ul>
<li>New &rarr; Ready: <b>Admit</b></li>
<li>Ready &rarr; Running: <b>Dispatch</b></li>
<li>Running &rarr; Ready: <b>Pause</b> (preemption/descheduled)</li>
<li>Running &rarr; Waiting: <b>Event Wait</b> (I/O initiate)</li>
<li>Waiting &rarr; Ready: <b>Event Completion</b> (I/O done)</li>
<li>Running &rarr; Terminated: <b>Release</b></li>
</ul>
<p>Key rule: process executes instructions ONLY in the Running state.</p>

<p><b>18.</b> Trace table:</p>
<pre>
Time  ProcA     ProcB     Notes
 1    Running   Ready
 2    Running   Ready
 3    Running   Ready     A initiates I/O
 4    Blocked   Running   A blocked, B runs
 5    Blocked   Running
 6    Blocked   Running   A's I/O completes
 7    Ready     Running   B finishes
 8    Running   -         A runs last cycle, done
</pre>
<p>CPU utilization = 8/8 = <b>100%</b>. CPU was never idle because B filled the gap while A was blocked.</p>

<h3>Section D</h3>

<p><b>19.</b> Running: max 1 (only 1 CPU). Ready: max 3 (when 1 is Running). Blocked: max 4 (all waiting for I/O). Yes all 4 can be Blocked simultaneously — this means CPU is completely idle (0% utilization). The OS itself may run housekeeping tasks but no user process is executing.</p>

<p><b>20.</b> Incorrect. After I/O completes, the process moves to <b>Ready</b>, not Running. The scheduling <b>policy</b> (not the I/O completion mechanism) decides which process runs next. The currently-running process might continue. Moving directly to Running would violate the separation of mechanism and policy — the I/O completion mechanism shouldn't make scheduling decisions.</p>

</details>
