---
layout: page
title: "OS Chapter 4: The Process Abstraction — Exam Prep"
---

# OS Chapter 4: The Abstraction — The Process
## Complete Exam Preparation Guide

---

# PART 1: CORE ANALYSIS

## 1.1 What is a Process?

**Simple:** A process is a running program. A program sitting on disk is dead code. The OS brings it to life by loading it into memory and executing it — that's a process.

**Academic wording:** A process is the OS's abstraction of a running program. It encompasses the program's code, current activity (represented by the program counter and registers), and associated resources (memory, open files, etc.).

**Key distinction — PROGRAM vs PROCESS:**

| Aspect | Program | Process |
|--------|---------|---------|
| Nature | Passive entity | Active entity |
| Location | Stored on disk | Lives in memory |
| Lifetime | Permanent (until deleted) | Temporary (created &rarr; terminated) |
| State | No state | Has state (running, ready, blocked) |
| Resources | Just bytes (code + static data) | Has memory, registers, open files, stack, heap |
| Multiplicity | One program can spawn many processes | Each process is a unique execution instance |

**Example:** Chrome.exe on your disk = program. Open 3 Chrome tabs = 3 processes (or more, Chrome uses multi-process architecture).

**TRAP ALERT:** A program is NOT a process. They ask this in MCQs. A program becomes a process when loaded into memory and executed. Multiple processes can run the same program simultaneously (each with its own state).

---

## 1.2 CPU Virtualization — The Illusion of Many CPUs

**The Problem:** You have 1 CPU (or a few), but you want to run 50+ programs simultaneously. How?

**The Solution:** **Time sharing** — the OS runs Process A for a bit, stops it, runs Process B, stops it, runs Process C... so fast that each process *thinks* it has its own CPU.

**Cost:** Performance. If 10 processes share 1 CPU, each effectively gets roughly 1/10th of the CPU speed.

**Two types of resource sharing:**

| Time Sharing | Space Sharing |
|-------------|---------------|
| Resource used by one entity at a time, then another | Resource physically divided among entities |
| CPU, network link | Disk space, RAM (in some schemes) |
| Entities take turns | Entities get their own portion |
| Example: CPU runs process A, then B, then C | Example: Disk block assigned to file X stays with X until deleted |

---

## 1.3 Mechanisms vs. Policies

This is a **design principle** the textbook and slides both emphasize heavily.

| &nbsp; | Mechanism | Policy |
|---|-----------|--------|
| Answers | **HOW** to do something | **WHICH** thing to do |
| Level | Low-level machinery/protocol | High-level intelligence/algorithm |
| Example | Context switch (how to stop A, start B) | Scheduling policy (which process runs next) |
| Nature | Implementation detail | Decision-making logic |

**Why separate them?** Modularity. You can change the scheduling policy (e.g., from Round Robin to Priority) without rewriting the context switch code. This is a general software design principle.

**EXAM TIP:** Your lecturer's slide says "Design Tip: Modular separation allows policies to change without rewriting low-level mechanisms." This is likely a short-answer or fill-in-the-blank candidate.

---

## 1.4 Machine State of a Process

To pause and resume a process, the OS must save and restore its **machine state**. The machine state is everything the process can read or update during execution.

**Three components:**

### (a) Address Space (Memory)
- **Code/Text segment:** The program instructions
- **Data segment:** Static/global variables (initialized data)
- **Stack:** Local variables, function parameters, return addresses
- **Heap:** Dynamically allocated memory (malloc/free in C)

### (b) Registers
- **Program Counter (PC)** / Instruction Pointer (IP): Points to the *next* instruction to execute
- **Stack Pointer (SP):** Points to the top of the stack
- **Frame Pointer (FP/BP):** Points to the base of the current stack frame
- General purpose registers (eax, ebx, ecx, etc.)

### (c) I/O Information
- List of open files (file descriptors)
- Default: fd 0 = stdin, fd 1 = stdout, fd 2 = stderr

**TRAP ALERT:** The slides specifically highlight this as an "Inventory" — if they ask "what constitutes the machine state of a process?" you must list all three categories (memory/address space, registers, I/O info). Missing one = lost marks.

---

## 1.5 Process API

Every OS must provide these operations:

| API | What it does | Real-world analogy |
|-----|-------------|-------------------|
| **Create** | Launch a new process (shell command, double-click icon) | Hiring a new employee |
| **Destroy** | Forcefully kill a process | Firing an employee |
| **Wait** | Pause until a process finishes | Waiting for a report to be done before proceeding |
| **Miscellaneous Control** | Suspend (pause) and Resume a process | Putting an employee on leave, then bringing them back |
| **Status** | Get info about a process (state, runtime, etc.) | Checking an employee's performance report |

---

## 1.6 Process Creation: From Disk to Memory

This is a **step-by-step process** — very likely to appear as a "describe the steps" question.

### The 5 Steps:

**Step 1: Load Code & Static Data into Memory**
- Read the executable from disk
- Place code and initialized data into the process's address space
- **Eager loading** (old OSes): Load everything at once before running
- **Lazy loading** (modern OSes): Load pieces on demand during execution (uses paging/swapping)

**Step 2: Allocate Stack**
- Create runtime stack for local variables, function parameters, return addresses
- Initialize with `argc` and `argv` (command-line arguments to main())

**Step 3: Allocate Heap**
- Set up heap space for dynamic memory allocation
- Initially small, grows as program calls `malloc()`

**Step 4: I/O Setup**
- Open three default file descriptors: stdin (0), stdout (1), stderr (2)
- Connect to terminal for input/output

**Step 5: Jump to main()**
- Transfer CPU control to the process's entry point (`main()`)
- Process begins execution

**EXAM TIP:** The slides show a nice vertical diagram of this flow. If asked to "describe process creation," list these 5 steps in order. The book and slides both emphasize this sequence.

**TRAP ALERT:** Eager vs. Lazy loading is a common MCQ/short question.
- Eager = load ALL code/data BEFORE execution starts (simple, old)
- Lazy = load ONLY what's needed WHEN it's needed (efficient, modern, requires paging)

---

## 1.7 Process States

### The Basic 3-State Model (from textbook)

| State | Meaning | CPU usage |
|-------|---------|-----------|
| **Running** | Currently executing instructions on the CPU | YES — using CPU |
| **Ready** | Able to run, but OS hasn't scheduled it yet | NO — waiting for CPU |
| **Blocked** (Waiting) | Cannot run until some event completes (e.g., I/O) | NO — waiting for event |

### Transitions:

| From | To | Trigger | Name |
|------|-----|---------|------|
| Ready | Running | OS picks this process to run | **Scheduled** (Dispatched) |
| Running | Ready | OS decides to run something else | **Descheduled** (Preempted / Paused) |
| Running | Blocked | Process initiates I/O or waits for event | **I/O Initiate** (Event Wait) |
| Blocked | Ready | I/O completes or event occurs | **I/O Done** (Event Completion) |

**CRITICAL — transitions that DO NOT exist:**
- **INVALID:** Ready &rarr; Blocked (can't block without running first)
- **INVALID:** Blocked &rarr; Running (must go through Ready first — the OS decides when to schedule)

### The Extended 5-State Model (from slides)

Your lecturer's slides show a **5-state model** with two additional states:

| State | Meaning |
|-------|---------|
| **Start/New** | Process is being created |
| **Ready** | Waiting to be dispatched to CPU |
| **Running** | Executing on CPU |
| **Waiting/Blocked** | Waiting for I/O or event |
| **Terminated** | Finished execution, being cleaned up |

**Extra transitions in 5-state model:**
- New &rarr; Ready: **Admit** (process created and admitted to ready queue)
- Running &rarr; Terminated: **Release** (process finishes or is killed)

**TRAP ALERT:** Your slides show BOTH the 3-state and 5-state diagrams. Know which one they're asking about. If the question says "basic states" = 3. If it says "process lifecycle" or shows Start/Terminated = 5. The xv6 code in the textbook actually has 6 states: UNUSED, EMBRYO, SLEEPING, RUNNABLE, RUNNING, ZOMBIE.

---

## 1.8 Process State Tracing

The textbook gives two concrete examples. These are **very likely exam questions** (fill in the trace table).

### Example 1: CPU-Only (No I/O)
Two processes, both CPU-bound. Process0 runs first, then Process1.

```
Time | Process0 | Process1 | Notes
  1  | Running  | Ready    |
  2  | Running  | Ready    |
  3  | Running  | Ready    |
  4  | Running  | Ready    | Process0 done
  5  |    –     | Running  |
  6  |    –     | Running  |
  7  |    –     | Running  |
  8  |    –     | Running  | Process1 done
```
Total time: 8 units. CPU utilization: 100% (CPU busy all 8 units).

### Example 2: CPU + I/O
Process0 does CPU work then I/O. Process1 is CPU-only.

```
Time | Process0 | Process1 | Notes
  1  | Running  | Ready    |
  2  | Running  | Ready    |
  3  | Running  | Ready    | Process0 initiates I/O
  4  | Blocked  | Running  | Process0 blocked, Process1 runs
  5  | Blocked  | Running  |
  6  | Blocked  | Running  |
  7  | Ready    | Running  | I/O done
  8  | Ready    | Running  | Process1 done
  9  | Running  |    –     |
 10  | Running  |    –     | Process0 done
```
Total time: 10 units. CPU utilization: 10/10 = 100% (overlapped I/O with CPU work).

**Key insight:** The OS maximizes CPU utilization by switching to another process when one blocks for I/O. Without this, CPU would sit idle during I/O (wasted time).

**TRAP ALERT for tracing questions:**
- When a process finishes I/O, it goes to **Ready**, NOT directly to Running
- The OS **decides** whether to switch immediately or let the current process keep running
- CPU utilization = (time units CPU was busy) / (total time units)

---

## 1.9 Data Structures: The PCB

**Process Control Block (PCB)** — the data structure the OS uses to store all information about a process.

### What's in a PCB (from xv6 example):

| Field | Purpose |
|-------|---------|
| `state` | Current process state (RUNNING, ZOMBIE, etc.) |
| `pid` | Unique Process ID |
| `*parent` | Pointer to parent process |
| `*mem` | Start of process memory |
| `sz` | Size of process memory |
| `*kstack` | Bottom of kernel stack for this process |
| `context` | Saved register values (for context switch) |
| `*ofile[NOFILE]` | Array of open files |
| `*cwd` | Current working directory |
| `*tf` | Trap frame for current interrupt |

**The Process List:** The OS maintains a list of all PCBs — this is how it tracks every process in the system. Also called the **process table**.

**Context Switch:** When the OS switches from Process A to Process B:
1. Save A's registers into A's PCB (`context` field)
2. Load B's registers from B's PCB
3. Resume B's execution

The `context` struct in xv6 stores: eip, esp, ebx, ecx, edx, esi, edi, ebp.

---

## 1.10 Special Process States

### Zombie State
- A process that has **finished executing** but still has an entry in the process table
- Why? The parent needs to read the child's exit status (return code)
- Cleaned up when the parent calls `wait()`
- In UNIX: return 0 = success, non-zero = error

### Embryo/New State
- Process is being created but not yet ready to run
- Transitional state during setup

### Key concept — Zombie vs. Orphan:

| &nbsp; | Zombie | Orphan |
|---|--------|--------|
| Definition | Child has FINISHED but parent hasn't collected its exit status | Child is STILL RUNNING but parent has terminated |
| Who's dead? | The child (finished execution) | The parent |
| Status | Has entry in process table, no resources | Still consuming resources, still executing |
| Resolution | Parent calls `wait()` to reap it | `init` process (PID 1) adopts it |
| Danger | Too many = process table fills up | Less dangerous — `init` will reap them |

**TRAP ALERT:** Zombie &ne; Orphan. This is one of the most commonly confused pairs in OS exams. A zombie is a DEAD child waiting to be reaped. An orphan is a LIVING child whose parent died.

---

# PART 2: SLIDE vs. BOOK COMPARISON

## What Overlaps (Both Cover)

| Topic | Book | Slides |
|-------|------|--------|
| Program vs Process definition | ✅ Detailed | ✅ Visual with analogy (recipe vs cooking) |
| CPU Virtualization via time sharing | ✅ | ✅ |
| Time sharing vs space sharing | ✅ | ✅ |
| Mechanisms vs Policies | ✅ With references | ✅ With visual (gears vs gavel) |
| Machine state (memory, registers, I/O) | ✅ Text | ✅ Diagram with inventory layout |
| Process API (5 operations) | ✅ | ✅ Icons for each |
| Process creation steps | ✅ Figure 4.1 | ✅ Vertical flow diagram |
| Eager vs Lazy loading | ✅ | ✅ |
| 3-state model (Running/Ready/Blocked) | ✅ | ✅ Triangle diagram |
| 5-state model | ❌ Not explicitly | ✅ Full diagram with Admit/Release/Dispatch |
| Process tracing tables | ✅ Figures 4.3, 4.4 | ✅ Gantt-chart style + tables |
| PCB / Data structures | ✅ Full xv6 code | ✅ Simplified struct |
| Zombie state | ✅ | ✅ Exam note callout |
| Context switch concept | ✅ Mentioned | ✅ Summary slide |

## Only in the Slides

| Topic | Exam Risk |
|-------|-----------|
| **5-state process lifecycle** (New &rarr; Ready &rarr; Running &rarr; Waiting &rarr; Terminated) with labeled transitions (Admit, Dispatch, Pause, Event Wait, Event Completion, Release) | **HIGH** — the slide devotes a full page to this, likely a diagram question |
| **Gantt-chart style timeline** for process tracing | MEDIUM — visual variant of the trace table |
| "Exam Note: ZOMBIE STATE" callout box | **HIGH** — lecturer flagged this explicitly |
| Summary slide with 3 pillars: Process, Context Switch, PCB | HIGH — likely structures a short-answer or essay |

## Only in the Book

| Topic | Exam Risk |
|-------|-----------|
| **Homework simulation** (process-run.py) with SWITCH_ON_IO, SWITCH_ON_END, IO_RUN_LATER, IO_RUN_IMMEDIATE flags | LOW for theory exam, but concepts of switching behavior may appear |
| Full xv6 `proc` struct with all fields (mem, sz, kstack, chan, killed, ofile, cwd, tf) | MEDIUM — may ask about specific PCB fields |
| xv6 `context` struct listing specific registers (eip, esp, ebx, ecx, edx, esi, edi, ebp) | MEDIUM |
| 6 states in xv6: UNUSED, EMBRYO, SLEEPING, RUNNABLE, RUNNING, ZOMBIE | LOW-MEDIUM — probably won't test xv6-specific enums but zombie is important |
| Historical references (Multics, Hydra, Brinch Hansen's Nucleus) | LOW |
| Discussion of scheduling decisions in tracing examples | MEDIUM — understanding WHY the OS made a particular switch |
| `argc`/`argv` initialization during stack setup | MEDIUM |
| Three default file descriptors (0=stdin, 1=stdout, 2=stderr) | **HIGH** — classic exam question |

## Especially Test-Worthy (Lecturer Emphasis)

Based on slide layout and emphasis:
1. **Program vs Process** — dedicated full slide, likely a definition question
2. **5-state diagram** — TWO slides devoted to this, one simplified and one detailed
3. **Process creation steps** — full-page visual diagram
4. **PCB structure** — highlighted with "Exam Note" callout
5. **Zombie state** — explicitly called out as exam-relevant on the slide
6. **Context switch** — appears in summary as one of 3 key takeaways
7. **Mechanisms vs Policies** — full dedicated slide with visual metaphor

---

# PART 3: EXTERNAL CLARIFICATION

The following are NOT in your course materials but are commonly tested and closely related:

### Orphan Process (supplementary — not in your chapter but commonly paired with zombie)
- A child process whose parent terminates before the child finishes
- Adopted by `init` (PID 1) in UNIX/Linux
- `init` will call `wait()` on orphans, preventing them from becoming permanent zombies
- **Source:** General OS knowledge, GeeksforGeeks, Baeldung

### Why Zombies Are Dangerous
- Each zombie occupies an entry in the process table
- Process table is finite
- Too many zombies = no new processes can be created = system freeze
- **Source:** General OS knowledge

### Context Switch — Slightly More Detail
The textbook mentions it but defers the full explanation. For completeness:
1. Timer interrupt fires (or process makes a system call/blocks)
2. OS saves current process's registers into its PCB
3. OS selects next process (scheduling policy)
4. OS loads next process's registers from its PCB
5. OS jumps to that process's PC location
- **Cost:** Context switches are NOT free. Saving/loading registers + cache pollution = overhead.

---

# PART 4: TEACH ME PROPERLY

## Concept 1: What is a Process?

**Simple:** Think of a recipe (program) vs actually cooking (process). The recipe sits in a book doing nothing. When you start cooking, you're following the recipe, using ingredients (memory), utensils (registers), and the stove (CPU). You could have 3 people cooking the same recipe simultaneously — 3 processes from 1 program.

**Academic:** A process is an instance of a program in execution. It includes the program code (text section), current activity (program counter, register contents), process stack (temporary data), data section (global variables), and heap (dynamically allocated memory).

**Exam answer format:** "A process is the OS abstraction of a running program. It consists of the program's code loaded into memory, its current execution state captured by registers (including the Program Counter), its address space (code, data, stack, heap), and I/O state (open file descriptors)."

## Concept 2: Time Sharing

**Simple:** Imagine one TV with 4 people wanting to watch different channels. You give each person 5 minutes, then switch. Everyone gets to watch, but nobody gets full-speed viewing.

**Academic:** Time sharing is an OS technique where the CPU is allocated to different processes in rapid succession, creating the illusion of simultaneous execution. The trade-off is performance degradation as each process receives only a fraction of the CPU time.

## Concept 3: Process States & Transitions

**Simple:** Think of a restaurant.
- **Running** = chef is actively cooking your order (executing on CPU)
- **Ready** = your order is in the queue, ingredients prepped, just waiting for an open stove (CPU available)
- **Blocked** = your order needs an ingredient that's being delivered — can't cook until it arrives (waiting for I/O)

**Key rule:** A blocked process CANNOT go directly to Running. It must first become Ready (ingredient arrives), then wait to be Scheduled (chef picks it up).

## Concept 4: PCB

**Simple:** Think of a patient's medical file in a hospital. Every patient has a file with their name, condition, medications, room number, doctor assigned. When a doctor switches between patients, they read the file to know where they left off. PCB = that file, but for processes.

**Academic:** The Process Control Block is a kernel data structure that contains all information needed to manage a process, including its state, program counter, CPU registers, memory management info, I/O status, and accounting information. The OS maintains a process list (process table) of all PCBs.

---

# PART 5: PREDICTED LIKELY QUESTIONS

## Definitions (Short Answer)
1. Define a process.
2. What is the difference between a program and a process?
3. Define time sharing. How does it differ from space sharing?
4. What is a mechanism? What is a policy? Give one example of each.
5. What is a Process Control Block (PCB)?
6. What is a context switch?
7. What is a zombie process?
8. What is the address space of a process?

## Conceptual Questions
9. Why does the OS need to virtualize the CPU?
10. What is the advantage of separating mechanisms from policies?
11. Why does a blocked process go to Ready instead of directly to Running when its I/O completes?
12. What is lazy loading and why is it preferred over eager loading?
13. Why is the zombie state necessary?
14. What happens if the parent never calls wait() on a terminated child?

## "Explain" Questions
15. Explain the steps involved in process creation (from program on disk to running process).
16. Explain the three basic process states and the transitions between them.
17. Explain what machine state the OS must save/restore during a context switch.
18. Explain how the OS provides the illusion of multiple CPUs using a single physical CPU.

## Comparison Questions
19. Compare time sharing vs space sharing with examples.
20. Compare mechanisms vs policies with examples.
21. Compare program vs process.
22. Compare eager loading vs lazy loading.
23. Compare Running vs Ready state.
24. Compare Blocked vs Ready state.

## Diagram Questions
25. Draw the 3-state process transition diagram. Label all transitions.
26. Draw the 5-state process lifecycle diagram with transitions.
27. Draw the memory layout of a process (code, data, heap, stack).
28. Given a trace scenario with I/O, fill in the process state table.

## Problem-Solving / Application
29. Two processes: P0 has 5 CPU instructions, P1 has 5 CPU instructions. No I/O. Draw the state trace table. What is CPU utilization?
30. P0 runs for 3 CPU cycles then does I/O (takes 4 cycles). P1 has 4 CPU instructions. Draw the trace. Calculate CPU utilization.
31. If a system has 3 processes and only 1 CPU, at most how many can be in Running state? At most how many can be in Ready state? At most how many can be in Blocked state?

---

# PART 6: TRAP AREAS & TRICKY QUESTIONS

## Trap 1: Program = Process?
**NO.** A program is passive (on disk). A process is active (in memory, executing). One program can have multiple processes.

## Trap 2: Can a process go from Blocked &rarr; Running?
**NO.** Blocked &rarr; Ready &rarr; Running. The OS scheduler must pick it from the Ready queue. Even if only one process exists, the transition is logically Blocked &rarr; Ready &rarr; Running.

## Trap 3: Can a process go from Ready &rarr; Blocked?
**NO.** A process can only initiate I/O (and become blocked) while it is Running. A Ready process isn't executing anything, so it can't request I/O.

## Trap 4: "Descheduled" vs "Preempted"
Same thing. Moving from Running &rarr; Ready because the OS decided to give the CPU to someone else. Different textbooks use different terms. Your slides use "Pause" for this transition.

## Trap 5: The 3-state vs 5-state model
- 3-state: Running, Ready, Blocked (what the textbook focuses on)
- 5-state: adds New/Start and Terminated
- Your slides show BOTH. Know which model the question is asking about.
- The 5-state model uses different transition names: Admit, Dispatch, Pause, Event Wait, Event Completion, Release

## Trap 6: Stack vs Heap

| Stack | Heap |
|-------|------|
| Automatic allocation | Manual allocation (malloc/free) |
| Local variables, parameters, return addresses | Dynamic data structures (linked lists, trees) |
| Fixed size (usually) | Grows as needed |
| LIFO order | No order |
| Fast | Slower |
| Managed by compiler | Managed by programmer |

## Trap 7: Zombie vs Terminated
- Zombie is a SPECIFIC state where the process has exited but its PCB entry still exists (parent hasn't called wait())
- Terminated/Cleaned-up is when the parent has read the exit status and the PCB entry is removed
- Every process briefly enters zombie state before being fully cleaned up

## Trap 8: "Which of these is NOT part of machine state?"
Expected wrong answers in MCQ: disk contents, network state, other processes' memory. Machine state = address space + registers + I/O info of THAT process only.

## Trap 9: File Descriptors
- fd 0 = stdin (standard input)
- fd 1 = stdout (standard output)
- fd 2 = stderr (standard error)
- These are set up during process creation (Step 4)
- Trick question: "How many file descriptors does a new UNIX process have by default?" Answer: 3

## Trap 10: CPU Utilization Calculation in Tracing
- CPU utilization = (# of time units where SOME process was Running) / (total time units)
- If I/O overlaps with another process's CPU work &rarr; utilization stays high
- If no overlap (system waits during I/O) &rarr; utilization drops

---

# PART 7: MUST-MEMORIZE LIST

## Definitions to Memorize
1. **Process:** A running program; the OS's abstraction of a program in execution
2. **Time sharing:** Technique where the CPU is shared by running processes in turns
3. **Space sharing:** Technique where a resource is physically divided among users
4. **Mechanism:** Low-level method/protocol for HOW to implement functionality (e.g., context switch)
5. **Policy:** High-level algorithm for WHICH decision to make (e.g., scheduling policy)
6. **Context switch:** Saving the state of one process and loading the state of another to switch CPU execution
7. **PCB:** Data structure that stores all information about a process (state, PID, registers, memory, files)
8. **Address space:** The memory a process can access (code, data, stack, heap)
9. **Zombie process:** A process that has completed execution but retains its process table entry until reaped by parent
10. **Program Counter (PC):** Register holding the address of the next instruction to execute

## Key Classifications
- **Machine state components:** Address Space, Registers, I/O Info
- **Process API operations:** Create, Destroy, Wait, Control, Status
- **Basic process states:** Running, Ready, Blocked
- **Extended process states:** New, Ready, Running, Waiting, Terminated
- **Process creation steps:** Load code/data &rarr; Allocate stack &rarr; Allocate heap &rarr; I/O setup &rarr; Jump to main()
- **Default file descriptors:** 0 (stdin), 1 (stdout), 2 (stderr)

## Key Contrasts
- Program vs Process
- Time sharing vs Space sharing
- Mechanism vs Policy
- Eager vs Lazy loading
- Running vs Ready
- Blocked vs Ready
- Zombie vs Orphan
- Scheduled vs Descheduled

## Keywords Teachers Expect
- "Abstraction" when describing what a process is
- "Virtualization" when describing how the OS provides illusion of many CPUs
- "Time sharing" as the technique for CPU virtualization
- "Machine state" when describing what defines a process
- "Context switch" when describing how the OS switches between processes
- "Modularity" when explaining why mechanisms and policies should be separated

---

# PART 8: PRACTICE QUESTIONS WITH ANSWERS

## EASY

### Q1: What is a process?
**Answer:** A process is a running program. It is the OS's abstraction of a program in execution, encompassing the program's code, its current execution state (registers including PC), its address space (code, data, stack, heap), and its I/O state (open file descriptors).

**Why it matters:** This is the most fundamental definition of the chapter. Will almost certainly appear.
**Source:** Textbook Section 4.1, Slide 2.

---

### Q2: Name the three components of a process's machine state.
**Answer:** (1) Address space/Memory — instructions, data the process reads/writes. (2) Registers — PC, stack pointer, frame pointer, general-purpose registers. (3) I/O information — list of open files/file descriptors.

**Why it matters:** Machine state is what the OS must save/restore during context switches.
**Source:** Textbook Section 4.1, Slide 5.

---

### Q3: What is the difference between time sharing and space sharing?
**Answer:** Time sharing allows multiple entities to use a resource by taking turns over time (e.g., CPU). Space sharing divides a resource physically among entities (e.g., disk blocks). Time sharing's cost is performance; space sharing's cost is capacity.

**Source:** Textbook TIP box, Slide 3.

---

### Q4: A process is in the Ready state. Can it transition directly to Blocked? Why or why not?
**Answer:** No. A process can only become Blocked by initiating an I/O operation or waiting for an event, and it can only do that while it is Running (executing instructions). A Ready process is not executing, so it cannot initiate I/O.

**Source:** Textbook Section 4.4, state transition diagram.

---

## MEDIUM

### Q5: Describe the steps the OS performs to create a process from a program on disk.
**Answer:**
1. **Load code and static data** from the executable on disk into the process's address space in memory (eagerly or lazily).
2. **Allocate and initialize the stack** for local variables, function parameters, and return addresses. Initialize argc and argv.
3. **Allocate heap memory** for dynamic data (malloc). Initially small.
4. **Set up I/O** by opening three default file descriptors: stdin (0), stdout (1), stderr (2).
5. **Jump to main()** — transfer CPU control to the entry point, beginning execution.

**Why it matters:** Multi-step "explain" question. Very likely on any OS exam covering this chapter.
**Source:** Textbook Section 4.3, Slide 7.

---

### Q6: Consider two processes. Process A runs for 3 time units, then does I/O (takes 3 units). Process B has 4 CPU instructions. Only 1 CPU. The OS switches to B when A blocks. Fill in the trace table.

**Answer:**
```
Time | Process A | Process B | Notes
  1  | Running   | Ready     |
  2  | Running   | Ready     |
  3  | Running   | Ready     | A initiates I/O
  4  | Blocked   | Running   | A blocked, B scheduled
  5  | Blocked   | Running   |
  6  | Blocked   | Running   | A's I/O completes → A becomes Ready
  7  | Ready     | Running   | B finishes
  8  | Running   |    –      | A runs remaining work
```

CPU utilization = 7/8 = 87.5% (CPU idle at no point, assuming A has 1 more instruction after I/O; if A finishes at time 8, then 8/8 = 100%).

**Note:** The exact answer depends on assumptions about how many instructions A has left after I/O. The key concept is that overlapping I/O with CPU work maximizes utilization.

**Source:** Textbook Figure 4.4, Slide 10-11.

---

### Q7: What is a zombie process? Why does it exist? What would happen if there were too many zombie processes?
**Answer:** A zombie process is one that has completed execution but still has an entry in the process table because its parent has not yet read its exit status via wait(). It exists so the parent can retrieve the child's return code (0 = success, non-zero = error). If too many zombies accumulate, the process table fills up, preventing the OS from creating new processes, potentially halting the system.

**Source:** Textbook Section 4.5, Slide 12 (Exam Note callout).

---

### Q8: Explain why mechanisms and policies are separated in OS design. Give an example.
**Answer:** Separating mechanisms (HOW to do something) from policies (WHICH decision to make) provides modularity. You can change the decision-making algorithm without rewriting the implementation code, and vice versa. Example: The context switch mechanism (save registers, load new registers, switch stack) stays the same regardless of whether the OS uses Round Robin, Priority, or FCFS scheduling policy to decide which process runs next.

**Source:** Textbook TIP box Section 4, Slide 4.

---

## CHALLENGING

### Q9: In a system with 3 processes and 1 CPU: (a) What is the maximum number of processes that can be in Running state? (b) What about Ready? (c) What about Blocked? (d) Can all 3 be Blocked simultaneously? What does that imply?

**Answer:**
(a) Maximum 1 in Running (only 1 CPU).
(b) Maximum 2 in Ready (the others when one is Running).
(c) Maximum 3 in Blocked (all three could be waiting for I/O).
(d) Yes, all 3 can be Blocked simultaneously. This means the CPU is idle — no process is using it. This is bad for CPU utilization. The OS itself may use this time for housekeeping, or the CPU simply idles.

**Source:** Derived from state model logic, Textbook Section 4.4.

---

### Q10: A process is in the Running state. List ALL possible states it can transition to and the event that causes each transition.
**Answer:**
- Running &rarr; **Ready**: The OS descheduled it (timer interrupt/preemption, higher-priority process arrived)
- Running &rarr; **Blocked**: The process initiated an I/O operation or called wait() for an event
- Running &rarr; **Terminated**: The process finished execution (exit()) or was killed (destroy)

It CANNOT go from Running &rarr; New (New is only the initial creation state).

**Source:** State transition diagrams from both textbook and slides.

---

### Q11: Why does a process that finishes I/O move to Ready instead of directly to Running?
**Answer:** Because the OS scheduler must make the decision of which process to run next. When I/O completes, the previously-blocked process is now *eligible* to run (Ready), but another process might currently be Running. The OS cannot preempt the running process without a scheduling decision. Moving to Ready preserves the separation of mechanism (I/O completion &rarr; Ready) and policy (scheduler decides who runs next). If the OS were to directly move a process from Blocked &rarr; Running, it would need to simultaneously preempt whatever is currently running, which is a scheduling policy decision, not a state transition mechanism.

**Source:** Textbook Section 4.4, logical extension of mechanism/policy separation.

---

# PART 9: TEACHER-STYLE GUESSING

## Most Likely Direct Questions
1. "Define a process." (1-2 marks)
2. "Differentiate between a program and a process." (3-5 marks)
3. "Draw and explain the process state diagram." (5-8 marks)
4. "What are the steps involved in process creation?" (5 marks)
5. "What is a PCB? What information does it store?" (3-5 marks)
6. "Explain time sharing vs space sharing." (3 marks)

## Likely Indirect Questions
1. "How does the OS manage multiple processes on a single CPU?" &rarr; They want: time sharing, context switch, scheduling
2. "What happens when a process requests I/O?" &rarr; They want: state transition Running &rarr; Blocked, OS schedules another process
3. "Why is modularity important in OS design?" &rarr; They want: mechanism/policy separation

## Likely Confusion Tactics
1. MCQ with "Ready &rarr; Blocked" as an option (WRONG transition)
2. MCQ asking "Blocked &rarr; Running?" (WRONG — must go through Ready)
3. Asking about zombie state and expecting you to know parent must call wait()
4. Giving a tracing problem with I/O and asking when CPU is idle
5. Asking "what is NOT part of machine state?" with disk contents as a trap option

## Expected Near-Textbook Language
- "A process is simply a running program"
- "Time sharing allows the OS to provide the illusion of many CPUs"
- "Mechanisms answer HOW; policies answer WHICH"
- "The register context will hold, for a stopped process, the contents of its registers"

---

# PART 10: FINAL EXAM PREP

## One-Page Rapid Revision

**PROCESS** = Running program = Code + State + Resources.

**CPU Virtualization** = Time sharing = run one, stop, run another = illusion of many CPUs.

**Time sharing** (CPU, turns) vs **Space sharing** (disk, portions).

**Mechanism** = HOW (context switch) vs **Policy** = WHICH (scheduler).

**Machine State** = Address Space (code/data/stack/heap) + Registers (PC, SP, FP) + I/O (file descriptors: 0=stdin, 1=stdout, 2=stderr).

**Process API** = Create, Destroy, Wait, Control, Status.

**Process Creation** = Load code &rarr; Stack &rarr; Heap &rarr; I/O setup &rarr; Jump to main().

**Eager** = load all before running. **Lazy** = load on demand (modern).

**3 States:** Running (on CPU), Ready (waiting for CPU), Blocked (waiting for I/O).

**Transitions:** Ready&rarr;Running (Scheduled), Running&rarr;Ready (Descheduled), Running&rarr;Blocked (I/O initiate), Blocked&rarr;Ready (I/O done). NO Ready&rarr;Blocked. NO Blocked&rarr;Running.

**5-State Model adds:** New (Admit&rarr;Ready) and Terminated (Running&rarr;Release).

**PCB** = Per-process data structure = state + PID + registers + memory info + open files + parent pointer.

**Context Switch** = Save A's registers to A's PCB, load B's registers from B's PCB, run B.

**Zombie** = finished child, parent hasn't called wait(). **Orphan** = running child, parent is dead &rarr; adopted by init.

---

## "Night Before Exam" Summary

1. Process = running program. Program = passive. Process = active.
2. OS virtualizes CPU via time sharing. Cost = performance.
3. Mechanism = HOW (low-level). Policy = WHICH (high-level). Separate for modularity.
4. Machine state = memory (address space) + registers (PC, SP) + I/O (file descriptors).
5. Process creation: Load &rarr; Stack &rarr; Heap &rarr; I/O &rarr; main().
6. 3 states: Running, Ready, Blocked. Know all 4 valid transitions. Know the 2 INVALID ones.
7. 5-state model adds New and Terminated.
8. PCB stores everything about a process. Process list = collection of PCBs.
9. Context switch = save/restore registers via PCB.
10. Zombie = dead child, unreported. Orphan = alive child, dead parent.
11. Default file descriptors: 0 (stdin), 1 (stdout), 2 (stderr).
12. Lazy loading > Eager loading (modern, efficient, uses paging).

---

## Top 10 Most Likely Questions

1. Define process. Differentiate from program.
2. Draw and label the process state diagram (3-state or 5-state).
3. List and explain the steps of process creation.
4. What is a PCB? What does it contain?
5. Explain time sharing vs space sharing with examples.
6. Explain mechanisms vs policies with examples.
7. What are the components of a process's machine state?
8. Fill in a process state tracing table (given scenario with I/O).
9. What is a zombie process? Why does it exist?
10. What is a context switch? What gets saved/restored?

---

## Top 10 Trap Areas

1. Ready &rarr; Blocked is NOT a valid transition
2. Blocked &rarr; Running is NOT a valid transition (must pass through Ready)
3. Zombie &ne; Orphan (dead child vs alive child with dead parent)
4. Program &ne; Process (passive vs active)
5. Eager vs Lazy loading (which is modern?)
6. File descriptors: 0, 1, 2 (stdin, stdout, stderr) — know the numbers
7. Time sharing = CPU, Space sharing = Disk (don't swap them)
8. Mechanism answers HOW, Policy answers WHICH (not the reverse)
9. In tracing: after I/O completes, process goes to READY, not RUNNING
10. CPU utilization calculation — count ALL time units where CPU was busy, divide by total

---

## Priority Study Order (Limited Time)

1. **Process states & transitions** (3-state AND 5-state diagrams) — highest ROI
2. **Process creation steps** (5 steps in order)
3. **Program vs Process distinction**
4. **Machine state components** (3 categories)
5. **Mechanisms vs Policies**
6. **PCB contents and purpose**
7. **Context switch concept**
8. **Time sharing vs Space sharing**
9. **Zombie state**
10. **Process tracing examples**
