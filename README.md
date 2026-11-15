# OS Laboratory Activity 3

## Multilevel Queue and Real-Time CPU Scheduling

### Student Information

- Name: Gazawe G. Casim
- Course and Section: BCA151 - B186.1
- Instructor: Maria Fe P. Bahinting          
- Date Submitted: 11/12/2026

### I. Introduction

This laboratory activity presents three Python programs that simulate CPU scheduling: Multilevel Queue (MLQ), Rate Monotonic Scheduling (RMS), and Earliest Deadline First (EDF). Each program was run, and its Gantt chart, waiting time, turnaround time, CPU utilization, and deadline results were examined. GitHub was used to store the code, the outputs, and this documentation.

### II. Objectives

- Explain how Multilevel Queue, RMS, and EDF scheduling work.
- Implement MLQ using Round Robin and FCFS.
- Implement preemptive RMS and EDF simulations in Python.
- Generate and read Gantt charts of process and task execution.
- Calculate CPU utilization, waiting time, turnaround time, and deadline compliance.
- Create and manage a GitHub repository for the source code and documentation.

### III. Scheduling Algorithms

1. **Multilevel Queue Scheduling.** Processes are split into separate queues, and each queue has its own scheduling policy. Here Queue 1 (interactive) uses Round Robin with a 2 ms quantum, and Queue 2 (batch) uses FCFS. Queue 1 always has priority over Queue 2.
2. **Rate Monotonic Scheduling.** A fixed-priority preemptive algorithm for periodic tasks. The shorter the period, the higher the priority.
3. **Earliest Deadline First.** A dynamic-priority preemptive algorithm. At every time step, the ready task with the earliest absolute deadline gets the CPU.

### IV. Program Implementation

All three programs run in 1 ms steps and ignore context-switching overhead. No external libraries are needed.

**mlq.py.** Every process has a `remaining` time. Each millisecond, newly arrived processes are placed in Queue 1 or Queue 2. If Queue 1 has a process, it runs for up to 2 ms and then goes to the back of Queue 1 if it is not finished. Queue 2 only runs when Queue 1 is empty, so a Queue 1 arrival interrupts it, and it continues later with its remaining burst time. Completion, turnaround (completion - arrival), and waiting (turnaround - burst) times are computed at the end.

**rms.py.** Each periodic task creates a separate job for every release within the 20 ms simulation, and each job has its own release time, deadline, and remaining time. At each millisecond, the program picks the ready job whose task has the shortest period. Unfinished jobs are kept, and a job that finishes after its deadline (or never finishes) is marked as missed.

**edf.py.** It uses the same job generation as RMS, but at each millisecond it picks the ready job with the earliest absolute deadline. Ties are broken by earlier release time, then by task ID.

### V. Test Data

**MLQ**

| Process | Arrival (ms) | Burst (ms) | Queue |
| --- | --- | --- | --- |
| P1 | 0 | 5 | 1 |
| P2 | 1 | 4 | 1 |
| P3 | 0 | 6 | 2 |
| P4 | 2 | 3 | 2 |
| P5 | 3 | 2 | 1 |

**RMS and EDF** (same task set, simulated for 20 ms)

| Task | Execution Time (C) | Period (T) | Deadline (D) |
| --- | --- | --- | --- |
| T1 | 1 ms | 4 ms | 4 ms |
| T2 | 1 ms | 5 ms | 5 ms |
| T3 | 2 ms | 10 ms | 10 ms |

### VI. Results and Discussion

#### MLQ

Gantt chart:

```
| P1 [0-2] | P2 [2-4] | P1 [4-6] | P5 [6-8] | P2 [8-10] | P1 [10-11] | P3 [11-17] | P4 [17-20] |
```

| PID | AT | BT | CT | TAT | WT |
| --- | --- | --- | --- | --- | --- |
| P1 | 0 | 5 | 11 | 11 | 6 |
| P2 | 1 | 4 | 10 | 9 | 5 |
| P3 | 0 | 6 | 17 | 17 | 11 |
| P4 | 2 | 3 | 20 | 18 | 15 |
| P5 | 3 | 2 | 8 | 5 | 3 |

Average turnaround time: 12.0 ms. Average waiting time: 8.0 ms.

All Queue 1 processes (P1, P2, P5) finished by 11 ms before any Queue 2 process ran. P3 arrived at 0 ms but did not get the CPU until 11 ms, and P4 waited the longest (15 ms). This confirms that Queue 1 has strict priority over Queue 2, and that the lower queue can wait a long time.

#### RMS

Priority order: T1 > T2 > T3.

Gantt chart:

```
| T1 [0-1] | T2 [1-2] | T3 [2-4] | T1 [4-5] | T2 [5-6] | IDLE [6-8] | T1 [8-9] | IDLE [9-10] | T2 [10-11] | T3 [11-12] | T1 [12-13] | T3 [13-14] | IDLE [14-15] | T2 [15-16] | T1 [16-17] | IDLE [17-20] |
```

- CPU utilization: 1/4 + 1/5 + 2/10 = 65%
- RMS sufficient bound for 3 tasks: 77.98%, so the task set passes the bound test
- All 11 task instances met their deadlines (0 misses)

At 10 ms, T3's second job was interrupted by T2's new job and by T1 at 12 ms because they have shorter periods, and it finished at 14 ms (deadline 20 ms).

#### EDF

- CPU utilization: 65%, which passes the classical EDF test (U <= 100%)
- All 11 task instances met their deadlines (0 misses)

The Gantt chart is exactly the same as RMS:

```
| T1 [0-1] | T2 [1-2] | T3 [2-4] | T1 [4-5] | T2 [5-6] | IDLE [6-8] | T1 [8-9] | IDLE [9-10] | T2 [10-11] | T3 [11-12] | T1 [12-13] | T3 [13-14] | IDLE [14-15] | T2 [15-16] | T1 [16-17] | IDLE [17-20] |
```

For this task set, the job with the earliest absolute deadline is always also the job with the shortest period (for example, at 10 ms T2 has deadline 15 and T3 has deadline 20, and at 12 ms T1 has deadline 16 and T3 has deadline 20). Both algorithms therefore pick the same job every time. They can still behave differently on heavier task sets.

Screenshots of all three runs are in the `screenshots/` folder.

### VII. Algorithm Comparison

| Criteria | MLQ | RMS | EDF |
| --- | --- | --- | --- |
| Scheduling basis | Queue priority | Task period | Absolute deadline |
| Priority assignment | Fixed queue priority | Fixed task priority | Dynamic task priority |
| Scheduling type | Depends on queue policy | Preemptive | Preemptive |
| Main application | Process classification | Periodic real-time tasks | Deadline-based real-time tasks |
| Deadline monitoring | Not inherent | Yes | Yes |

### VIII. Guide Questions

**1. What is the main difference between Multilevel Queue and Multilevel Feedback Queue scheduling?**
In MLQ, a process stays in the queue it was first assigned to. In Multilevel Feedback Queue, a process can move between queues depending on its behavior, for example moving down if it uses too much CPU time or moving up if it has waited too long.

**2. Why does MLQ use different scheduling algorithms for different process queues?**
Different kinds of processes have different needs. Interactive processes need quick responses, so Round Robin fits them. Batch processes only need to finish and do not need fast responses, so FCFS is simple and has low overhead.

**3. How does RMS determine which task receives the CPU first?**
It gives the CPU to the ready task with the shortest period. In this simulation, T1 (4 ms) goes first, then T2 (5 ms), then T3 (10 ms).

**4. Why is RMS considered a fixed-priority scheduling algorithm?**
Priorities are set once from the periods and never change while the system runs. Every job of T1 always has a higher priority than every job of T2 or T3.

**5. How does EDF determine which task should execute next?**
It looks at all ready jobs and runs the one with the earliest absolute deadline (release time + relative deadline).

**6. Why is EDF considered a dynamic-priority scheduling algorithm?**
A task's priority is not fixed. It depends on the absolute deadline of its current job, which changes every time a new job is released. The same task can therefore have a high priority at one moment and a low priority at another.

**7. What happens when a newly released task has an earlier deadline than the currently running task under EDF?**
The running task is preempted and the new task takes the CPU. The preempted task keeps its remaining execution time and continues later.

**8. Why is CPU utilization important in real-time scheduling?**
It shows whether the task set can be scheduled at all. If total utilization is too high, the CPU does not have enough time to finish every job before its deadline. Utilization tests such as the RMS bound and the EDF 100% limit make it possible to check this before the system runs.

**9. Can a task set with CPU utilization below 100% still miss a deadline? Explain the conditions under which this may happen.**
Yes. With RMS, a task set can have utilization below 100% but above the RMS bound and still miss a deadline. For example, T1 (C = 2, T = 5) and T2 (C = 4, T = 7) have U = 0.4 + 0.571 = 97.1%, but under RMS T2 misses its deadline at 7 ms because T1 keeps preempting it. It can also happen under EDF if the classical assumptions are broken, such as when deadlines are shorter than periods, when there is context-switching overhead, when tasks cannot be preempted, when tasks share resources and block each other, or when more than one processor is involved.

**10. Which algorithm would you recommend for periodic real-time tasks, RMS or EDF? Justify your answer.**
EDF is recommended when the goal is to use the CPU as fully as possible, because it can schedule any independent periodic task set up to 100% utilization, while RMS is only guaranteed up to about 69% to 78% (77.98% for three tasks). RMS is easier to implement and its behavior is more predictable because priorities are fixed, which is why it is used in many real systems. For this lab's task set (65% utilization), both worked equally well, but for a heavily loaded system EDF is the better choice.

### IX. Conclusion

This laboratory activity showed how MLQ, RMS, and EDF decide which process or task gets the CPU, and it gave practice in turning those rules into working Python simulations. Reading the Gantt charts made preemption visible, such as Queue 1 interrupting Queue 2 and T3 being interrupted by T1 and T2. The activity also covered computing waiting time, turnaround time, and CPU utilization, and checking deadline compliance. Finally, GitHub was used to organize and submit the code, outputs, and documentation.
