---
layout: page
title: "OS Ch4 Flashcards — Active Recall"
---

# Flashcards: The Process Abstraction

**How to use:** Read the question. CLOSE YOUR EYES or cover the answer. Say/write your answer FIRST. Then check.

---

## Definitions

**Q1: What is a process?**

<details><summary>Answer</summary>
<p>A running program. The OS abstraction of a program in execution — includes code, registers (PC, SP), address space (code, data, stack, heap), and I/O state (open file descriptors).</p>
</details>

**Q2: What is the difference between a program and a process?**

<details><summary>Answer</summary>
<p>Program = passive, on disk, just bytes. Process = active, in memory, executing, has state and resources. One program can spawn multiple processes.</p>
</details>

**Q3: What is time sharing?**

<details><summary>Answer</summary>
<p>OS technique where the CPU is shared by running processes in rapid turns, creating the illusion of simultaneous execution. Cost = performance. Counterpart = space sharing (dividing resource physically, e.g., disk).</p>
</details>

**Q4: What is a mechanism? What is a policy?**

<details><summary>Answer</summary>
<p>Mechanism = HOW (low-level protocol, e.g., context switch). Policy = WHICH (high-level decision algorithm, e.g., scheduling policy). Separated for modularity — change one without rewriting the other.</p>
</details>

**Q5: What is a PCB?**

<details><summary>Answer</summary>
<p>Process Control Block — kernel data structure storing all info about a process: state, PID, registers/context, memory info, parent pointer, open files, current directory.</p>
</details>

**Q6: What is a context switch?**

<details><summary>Answer</summary>
<p>Save current process's registers into its PCB, load next process's registers from its PCB, resume execution. This is the mechanism that enables time sharing.</p>
</details>

**Q7: What is a zombie process?**

<details><summary>Answer</summary>
<p>A process that finished execution but still has an entry in the process table because parent hasn't called wait() to read its exit status. Too many zombies = process table full = no new processes.</p>
</details>

**Q8: What is an orphan process?**

<details><summary>Answer</summary>
<p>A child process still running whose parent has terminated. Adopted by init (PID 1) which calls wait() on it. Different from zombie — orphan is ALIVE, zombie is DEAD but unreported.</p>
</details>

---

## Machine State &amp; Process Creation

**Q9: What are the 3 components of a process's machine state?**

<details><summary>Answer</summary>
<p>1. Address Space (code, data, stack, heap)<br>
2. Registers (PC, SP, FP, general purpose)<br>
3. I/O Info (open file descriptors)</p>
</details>

**Q10: What are the 3 default UNIX file descriptors?**

<details><summary>Answer</summary>
<p>fd 0 = stdin, fd 1 = stdout, fd 2 = stderr. Set up during process creation (I/O setup step).</p>
</details>

**Q11: List the 5 steps of process creation in order.**

<details><summary>Answer</summary>
<p>1. Load code &amp; static data into memory<br>
2. Allocate stack (initialize argc/argv)<br>
3. Allocate heap<br>
4. I/O setup (open fd 0, 1, 2)<br>
5. Jump to main()</p>
</details>

**Q12: Eager vs Lazy loading?**

<details><summary>Answer</summary>
<p>Eager = load ALL code/data before execution (old/simple OSes). Lazy = load only what's needed during execution (modern, uses paging). Lazy is more efficient.</p>
</details>

---

## Process States

**Q13: Name the 3 basic process states.**

<details><summary>Answer</summary>
<p>Running (executing on CPU), Ready (can run, waiting for CPU), Blocked (waiting for I/O or event).</p>
</details>

**Q14: Name all 4 VALID transitions in the 3-state model.**

<details><summary>Answer</summary>
<p>1. Ready &rarr; Running (Scheduled/Dispatched)<br>
2. Running &rarr; Ready (Descheduled/Preempted)<br>
3. Running &rarr; Blocked (I/O initiate)<br>
4. Blocked &rarr; Ready (I/O done)</p>
</details>

**Q15: Which 2 transitions are INVALID and why?**

<details><summary>Answer</summary>
<p>1. Ready &rarr; Blocked: Can't initiate I/O without running first.<br>
2. Blocked &rarr; Running: Must pass through Ready — scheduler decides who runs.</p>
</details>

**Q16: What does the 5-state model add?**

<details><summary>Answer</summary>
<p>Adds New/Start (process being created, transition: Admit &rarr; Ready) and Terminated (process finished, transition: Running &rarr; Release). Also uses names: Dispatch, Pause, Event Wait, Event Completion.</p>
</details>

**Q17: After I/O completes, does the process go to Running?**

<details><summary>Answer</summary>
<p>NO. It goes to Ready. The scheduler then decides when to move it to Running. This preserves separation of mechanism and policy.</p>
</details>

---

## Process API &amp; Tracing

**Q18: Name the 5 Process API operations.**

<details><summary>Answer</summary>
<p>Create, Destroy, Wait, Miscellaneous Control (suspend/resume), Status.</p>
</details>

**Q19: In a system with 1 CPU and 3 processes, can all 3 be Blocked?**

<details><summary>Answer</summary>
<p>Yes. All 3 waiting for I/O = CPU is idle = 0% utilization. Bad for performance but valid state.</p>
</details>

**Q20: How do you calculate CPU utilization in a trace?**

<details><summary>Answer</summary>
<p>(Number of time units where ANY process was Running) / (Total time units) &times; 100%.</p>
</details>

---

## Quick Fire (Yes/No)

**Q21:** Can one program have multiple processes?
<details><summary>Answer</summary><p>Yes</p></details>

**Q22:** Can one process run multiple programs?
<details><summary>Answer</summary><p>No (in basic model)</p></details>

**Q23:** Is disk space time-shared or space-shared?
<details><summary>Answer</summary><p>Space-shared</p></details>

**Q24:** Is the CPU time-shared or space-shared?
<details><summary>Answer</summary><p>Time-shared</p></details>

**Q25:** Does a zombie process use CPU?
<details><summary>Answer</summary><p>No — it's dead, only occupies process table entry</p></details>

**Q26:** Who adopts orphan processes?
<details><summary>Answer</summary><p>init (PID 1)</p></details>

**Q27:** What register tells us the next instruction?
<details><summary>Answer</summary><p>Program Counter (PC) / Instruction Pointer (IP)</p></details>

**Q28:** Where are local variables stored?
<details><summary>Answer</summary><p>Stack</p></details>

**Q29:** Where does malloc() allocate memory?
<details><summary>Answer</summary><p>Heap</p></details>

**Q30:** What's another name for the process list entry?
<details><summary>Answer</summary><p>Process Control Block (PCB)</p></details>
