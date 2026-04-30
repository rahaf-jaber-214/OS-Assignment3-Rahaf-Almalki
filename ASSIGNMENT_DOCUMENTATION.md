# Assignment 3 - Complete Documentation

**Student Name**: [Rahaf Jaber Almalki]  
**Student ID**: [445052147]  
**Date Submitted**: [30 April 2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [29 Ap, 2:30]
**What I implemented**: Added ReentrantLock to protect shared counters (contextSwitchCount, completedProcessCount, totalWaitingTime)

**Challenges encountered**: Identifying all critical sections that needed locking.

**How I solved it**: Reviewed the code carefully and placed locks around shared variable updates using try-finally.

**Testing approach**: Ran the program multiple times to check if values remain consistent.

**Time spent**: 1 hours

---

### Entry 2 - [29, 3:00]
**What I implemented**: Added synchronization for execution log (ArrayList) and improved output display.

**Challenges encountered**: reventing concurrent modification issues

**How I solved it**: Used ReentrantLock when adding elements to the execution log

**Testing approach**: Tested with multiple processes and checked log size and correctness

**Time spent**: 2 hours

---

### Entry 3 - [29, 6:00]
**What I implemented**: Implemented Semaphore to control CPU access and added statistics output.

**Challenges encountered**: Understanding where to place acquire() and release() correctly.

**How I solved it**: Wrapped CPU execution code inside semaphore control with try-finally.

**Testing approach**: Tested with different scenarios and verified consistent results

**Time spent**: 1 hour

---

### Entry 4 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
1-`executionLog.add(message)`.  
Shared resource: `ArrayList<String>`.  
Problem: `ArrayList` is not thread-safe; concurrent `add()` calls can corrupt internal 
structure, throw `ConcurrentModificationException`, or lose entries.  
Incorrect behaviour: Program may crash or log entries may disappear.

2-– `contextSwitchCount++` (and the other counters).  
Shared resource: the integer counters.  
Problem: `++` is not atomic; two threads can read the same value, increment, and write 
back, causing a lost update.  
Incorrect behaviour: The final counter value is less than the actual number of incremen
ts
[Race conditions occur when multiple threads access shared variables like contextSwitchCount and completedProcessCount at the same time, causing incorrect values. Another issue happens in executionLog (ArrayList) where concurrent writes can lead to data corruption or ConcurrentModificationException. This happens because threads are not synchronized when accessing shared resources]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
ReentrantLock-is a mutual exclusion lock (binary). It guarantees that only one thre
ad holds the lock at a time. I used it for the counters and the log because those resourc
es require exclusive access.

Semaphore- maintains a set of permits. A binary semaphore (permits = 1) acts like a 
lock, but semaphores can also allow N concurrent accesses (e.g., a connection pool). I us
ed a `Semaphore(1)` to limit CPU execution – only one process can run at any moment, exac
tly matching a single-core CPU

ReentrantLock is used to protect shared data and ensure only one thread modifies it at a time. Semaphore is used to control access to a limited resource. In my code, I used ReentrantLock for counters and execution log, and a binary Semaphore (1 permit) to control CPU access. Locks protect data, while semaphores control resource access.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
Deadlock occurs when two or more threads wait forever for each other’s locked resou
rces.- Prevention techniques I used:
1. Lock ordering – I never acquire more than one lock at a time, so cyclic wait can
not happen.
2. try-finally blocks – Every `lock()` or `acquire()` is followed by a `finally` block that releases the resource. This guarantees release even if an exception occurs, prev
enting resource leaks.
3- Additionally, the semaphore is acquired at the very beginning of the critical section a
nd released immediately after, so there is no nested locking

[Deadlock happens when threads wait forever for each other’s resources. I prevented it by using try-finally to always release locks. This ensures that locks are not held permanently even if an error occurs.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**: Lock Granularity Design Decision**- I chose **fine-grained locking** – three separate `ReentrantLock`s, one per counter (`contextSwitchLock`, `completedProcessLock`, `waitingTimeLock`).- **Why:** The three counters are completely independent (updating one does not depend on the others). With a single coarse-grained lock, threads updating different counters would 
still block each other, creating unnecessary contention. Fine-grained locking allows true 
parallelism: while one thread increments `contextSwitchCount`, another can simultaneously 
increment `completedProcessCount`.- **Trade-offs:** Fine-grained requires more code and careful reasoning, but for independent resources the concurrency gain is worth it. Coarse-grained is simpler but reduces throughput.- Because the counters are independent, fine-grained locking provides **better concurrency** – it exactly follows the principle: protect each shared resource with its own lock

I used one shared lock (coarse-grained) for all three counters. This was simpler and ensured consistency across related variables. The trade-off is reduced concurrency because only one thread can update any counter at a time. Separate locks (fine-grained) would improve concurrency but increase complexity and risk of errors. Since the counters are related, a single lock was the safer choice

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount,completedProcessCount, totalWaitingTim

**Why they need protection**:  The read-modify-write operations (increment, addition) are not atomic
**Synchronization mechanism used**:  ReentrantLock

**Code snippet**:
```java
 public static void incrementContextSwitch() {
        contextSwitchLock.lock();
        try {
            contextSwitchCount++;
        } finally {
            contextSwitchLock.unlock();
        }}
```

**Justification**: Each counter is independent, so separate locks maximise concurrency

---

### Critical Section #2: Execution Log

**What resource**: executionLog

**Why it needs protection**:ArrayList is not thread‑safe; concurrent add() calls cause
corruption or exceptions

**Synchronization mechanism used**: ReentrantLock logLock

**Code snippet**:
```java
 public static void logExecution(String message) {
        logLock.lock();
        try {
            executionLog.add(message);
        } finally {
            logLock.unlock();
        }
    }
```

**Justification**:Exclusive access is required to preserve the logʼs integrity.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: Simulate a single‑core CPU – only one process can execute at a time.

**Number of permits and why**: 1 there are only one thread could access cpu at the same time  

**Where implemented**:  Process.run() and Process.runToCompletion()

**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try {
// ... execution code ...
} finally {
SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: Guarantees that even though many threads are ready, only one proceeds intothe CPU at any moment – exactly like a real uniprocessor system

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)
Context switches:32
Completed processes: 18
Total waiting time: always 1055834ms
Average waiting time: identical each run 

**Why synchronization is necessary**: 
 Without locks, the counters could lose increments
and the log could throw exceptions. The fact that results are now deterministic proves
race conditions are eliminated
**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**:  Added a loop that ran the program 20 times.

**Results**: : No ConcurrentModificationException or any other exception occurred.

**What this proves**:  The logLock successfully serialises access to the ArrayList ,
making it thread‑safe.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: completedProcessCount should equal number of processes created; total
waiting time should be consistent with per‑process waiting times

**Actual values**:  All values matched the manual calculation (checked with print statements).

**Analysis**: The synchronisation does not change the logical behaviour – it only ensures
correctness under concurrency

---

### Test 4: Different Scenarios
**Scenario tested**: [Changed Semaphore(1) to Semaphore(2) temporarily]

**Purpose**: : Observe effect of allowing two concurrent processes

**Results**:  With 2 permits, execution overlapped (interleaving in output). Still no race
conditions because counters remained protected.

**What I learned**: 
Semaphores are extremely flexible – they control the degree of
concurrency without changing the core logic.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

A binary semaphore ( Semaphore(1) ) is functionally similar to a mutex, but a mutex
(ReentrantLock) is usually preferred for mutual exclusion because it provides
ownership and reentrancy.

Synchronisation adds overhead, but the safety it buys is essential for any
multithreaded program

Fine‑grained locking is powerful: protecting independent resources with separate
locks unlocks real parallelism.

The try-finally pattern is non‑negotiable – forgetting to unlock in a finally block
leads to deadlocks that are very hard to debug.

Race conditions are subtle: the code can run correctly many times and then suddenly
fail. Synchronisation makes concurrent programs predictable.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**:  Banking systems – When multiple tellers update the same account balance, locks
prevent lost deposits or withdrawals.

**Example 2**: Print spooler – A semaphore with a limit equal to the number of printers controls
access to physical printers.

---

### How I would explain synchronization to others:
Think of a common whiteboard that multiple students need to use. When more than one person writes on it at the same time, the result becomes messy and unclear. A mutex works like allowing only one student to hold the marker at any given moment, ensuring their writing stays organized. A semaphore is similar to having a few markers, so a small group can write simultaneously but still in a controlled way. Without such coordination, the board would quickly become disorganized—just like shared data in a program without proper synchronization.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/rahaf-jaber-214/OS-Assignment3-Rahaf-Almalki/edit/main/ASSIGNMENT_DOCUMENTATION.md

**Number of commits**: 9

**Commit messages**: 
1. set my student id 445052147
2. add reentrantlok
3. add semaphore 
4. switch countr

---

## Summary

**Total time spent on assignment**: 20 hours

**Key takeaways**: 
1.  Fine‑grained locking improves performance for independent resources
2. try-finally is the only safe way to release locks
3. A semaphore can control both mutual exclusion and resource limits.

**Most challenging aspect**:  Deciding on lock granularity and proving that separate locks
are safe

**What I'm most proud of**:  The final program runs deterministically

---

**End of Documentation**
