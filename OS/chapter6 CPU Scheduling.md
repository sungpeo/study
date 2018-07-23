# Chapter6. CPU Scheduling

## Chapter Objectives

* CPU scheduling에 대한 소개, which is the basis for multiprogrammed operating systems.
* To describe various CPU-scheduling algorithms.
* To discuss evaluation criteria for selecting a CPU-scheduling algorithm for a particular system.
* To examine the scheduling algorithms of several operating systems.

## 6.1 Basic Concepts

The objective of multiprogramming is to have some process running at all times, to maximize CPU utilization.
Scheduling of this kind is a fundamental operating-system function. Almost all computer resources are scheduled before use. The CPU is, of course, one of the primary computer resources. Thus, its scheduling is central to operating-system design.

### 6.1.1 CPU–I/O Burst Cycle

Process execution begins with a CPU burst. That is followed by an I/O burst, which is followed by another CPU burst, then another I/O burst, and so on. Eventually, the final CPU burst ends with a system request to terminate execution (Figure 6.1).
(Figure 6.2) The curve is generally characterized as exponential or hyperexponential, with a large number of short CPU bursts and a small number of long CPU bursts.

### 6.1.2 CPU Scheduler

Conceptually, however, all the processes in the ready queue are lined up waiting for a chance to run on the CPU. The records in the queues are generally process control blocks (PCBs) of the processes

### 6.1.3 Preemptive Scheduling

CPU-scheduling decisions may take place under the following four circumstances:

1. When a process switches from the running state to the waiting state (for example, as the result of an I/O request or an invocation of wait() for the termination of a child process)
2. When a process switches from the running state to the ready state (for example, when an interrupt occurs)
3. When a process switches from the waiting state to the ready state (for example, at completion of I/O)
4. When a process terminates

When scheduling takes place only under circumstances 1 and 4, we say that the scheduling scheme is nonpreemptive or cooperative. Otherwise, it is preemptive.

Under nonpreemptive scheduling, once the CPU has been allocated to a process, the process keeps the CPU until it releases the CPU either by terminating or by switching to the waiting state. This scheduling method was used by Microsoft Windows 3.x.

Unfortunately, preemptive scheduling can result in race conditions when
data are shared among several processes.

Certain operating systems, including most versions of UNIX, deal with this problem by waiting either for a system call to complete or for an I/O block to take place before doing a context switch. Unfortunately, this kernel-execution model is a poor one for supporting real-time computing where tasks must complete execution within a given time frame. In Section 6.6, we explore scheduling demands of real-time systems.

So that these sections of code are not accessed concurrently by several processes, they disable interrupts at entry and reenable interrupts at exit. It is important to note that sections of code that disable nterrupts do not occur very often and typically contain few instructions.

Another component involved in the CPU-scheduling function is the dispatcher.
The dispatcher is the module that gives control of the CPUto the process selected by the short-term scheduler. This function involves the following:

* Switching context
* Switching to user mode
* Jumping to the proper location in the user program to restart that program

The dispatcher should be as fast as possible, since it is invoked during every process switch. The time it takes for the dispatcher to stop one process and start another running is known as the dispatch latency.

## 6.2 Scheduling Criteria

In choosing which algorithm to use in a particular situation, we must consider the properties of the various algorithms.
Many criteria have been suggested for comparing CPU-scheduling algorithms.

* CPU utilization. We want to keep the CPU as busy as possible. Conceptually, CPU utilization can range from 0 to 100 percent. (In a real system 40 ~ 90)
* Throughtput. One measure of work is the number of processes that are completed per time unit, called throughput. For long processes, this rate may be one process per hour; for short transactions, it may be ten processes per second.
* Turnaround time. The interval from the time of submission of a process to the time of completion is the turnaround time. Turnaround time is the sum of the periods spent waiting to get into memory, waiting in the ready queue, executing on the CPU, and doing I/O.
* Waiting time. Waiting time is the sum of the periods spent waiting in the ready queue.
* Response time. Thus, another measure is the time from the submission of a request until the first response is produced. This measure, called response time, is the time it takes to start responding, not the time it takes to output the response.

It is desirable to maximize CPU utilization and throughput and to minimize turnaround time, waiting time, and response time. In most cases, we  optimize the average measure. However, under some circumstances, we prefer to optimize the minimum or maximum values rather than the average.

## 6.3 Scheduling Algorithms

CPU scheduling deals with the problem ofdeciding which of the processes in the ready queue is to be allocated the CPU.There aremanydifferent CPU-scheduling algorithms.

### 6.3.1 First-Come, First-Served Scheduling

By far the simplest CPU-scheduling algorithm is the first-come, first-served (FCFS) scheduling algorithm.
굳이 더 설명 안해도 될 정도로 간단하다. 다만 평균 waiting time이 꽤 길어질 수 있다. 게다가 하나의 긴 작업으로도 모든 다른 프로세스들이 기다리게되는 convoy effect가 발생한다.  그 중에 가장 큰 이유는 nonpreemptive하기 때문이다.

### 6.3.2 Shortest-Job-First Scheduling

Adifferent approach toCPU scheduling is the shortest-job-first (SJF) scheduling algorithm. Note that a more appropriate term for this scheduling method would be the shortest-next-CPU-burst algorithm, because scheduling depends on the length of the next CPU burst of a process, rather than its total length.
The SJF scheduling algorithm is provably optimal, in that it gives the minimum average waiting time for a given set of processes.

Although the SJF algorithm is optimal, it cannot be implemented at the level of short-term CPU scheduling. With short-term scheduling, there is no way to know the length of the next CPU burst.
가장 접근 방법으로 추정치를 사용하는 방법이 있다. next CPU burst는 일반적으로 exponential average로 예측된다.

The value of tn contains our most recent information, while n stores the past history. The parameter  controls the relative weight of recent and past history in our prediction.

The SJF algorithm can be either preemptive or nonpreemptive. The choice arises when a new process arrives at the ready queue while a previous process is still executing. The next CPU burst of the newly arrived  rocess may be shorter than what is left of the currently executing process.Apreemptive SJF algorithm will preempt the currently executing process, whereas a nonpreemptive SJF algorithm will allow the currently running process to finish its CPU burst. Preemptive SJF scheduling is sometimes called shortest-remaining-time-first scheduling.

### 6.3.3 Priority Scheduling

The SJF algorithm is a special case of the general priority-scheduling algorithm. An SJF algorithm is simply a priority algorithm where the priority (p) is the inverse of the (predicted) next CPU burst.

A major problem with priority scheduling algorithms is indefinite blocking, or starvation. Asolution to the problem of indefinite blockage of low-priority processes is aging. Aging involves gradually increasing the priority of processes that wait in the system for a long time.

### 6.3.4 Round-Robin Scheduling

The round-robin (RR) scheduling algorithm is designed especially for timesharing systems. It is similar to FCFS scheduling, but preemption is added to enable the system to switch between processes. A small unit of time, called a time quantum or time slice, is defined. A time quantum is generally from 10 to 100 milliseconds in length. The ready queue is treated as a circular queue.

The average waiting time under the RR policy is often long.
Each process must wait no longer than (n − 1) × q time units until its next time quantum.

Turnaround time also depends on the size of the time quantum.

Although the time quantum should be large compared with the contextswitch time, it should not be too large. As we pointed out earlier, if the time quantum is too large, RR scheduling degenerates to an FCFS policy. A rule of thumb is that 80 percent of the CPU bursts should be shorter than the time quantum.

### 6.3.5 Multilevel Queue Scheduling

Another class of scheduling algorithms has been created for situations in which processes are easily classified into different groups. For example, a common division is made between foreground (interactive) processes and background (batch) processes. These two types of processes have different response-time requirements and so may have different scheduling needs. In addition, foreground processes may have priority (externally defined) over background processes.

A multilevel queue scheduling algorithm partitions the ready queue into several separate queues (Figure 6.6). Each queue has its own scheduling algorithm.

In addition, there must be scheduling among the queues, which is commonly implemented as fixed-priority preemptive scheduling.

Let’s look at an example of a multilevel queue scheduling algorithm with five queues, listed below in order of priority:

1. System processes
2. Interactive processes
3. Interactive editing processes
4. Batch processes
5. Student processes

Another possibility is to time-slice among the queues. Here, each queue gets a certain portion of the CPU time,which it can then schedule among its various processes.

### 6.3.6 Multilevel Feedback Queue Scheduling

The multilevel feedback queue scheduling algorithm, in contrast, allows a process to move between queues. The idea is to separate processes according to the characteristics of their CPU bursts. If a process uses too much CPU time, it will be moved to a lower-priority queue. This scheme leaves I/O-bound and interactive processes in the higher-priority queues. In addition, a process that waits too long in a lower-priority queue may be moved to a higher-priority queue. This form of aging prevents starvation.

In general, a multilevel feedback queue scheduler is defined by the following parameters:

* The number of queues
* The scheduling algorithm for each queue
* The method used to determine when to upgrade a process to a higher-priority queue 
* The method used to determine when to demote a process to a lower-priority queue
* The method used to determine which queue a process will enter when that process needs service

The definition of a multilevel feedback queue scheduler makes it the most general CPU-scheduling algorithm. It can be configured to match a specific system under design. Unfortunately, it is also the most complex algorithm, since defining the best scheduler requires some means by which to select values for all the parameters.

## 6.4 Thread Scheduling

To run on a CPU, user-level threads must ultimately be mapped to an associated kernel-level thread, although this mapping may be indirect and may use a lightweight process (LWP). In this section, we explore scheduling issues involving user-level and kernel-level threads and offer specific examples of scheduling for Pthreads.

### 6.4.1 Contention Scope

One distinction between user-level and kernel-level threads lies in how they are scheduled. On systems implementing the many-to-one (Section 4.3.1) and many-to-many (Section 4.3.3) models, the thread library schedules user-level threads to run on an available LWP. This scheme is known as processcontention scope (PCS), since competition for the CPU takes place among threads belonging to the same process.

To decide which kernel-level thread to schedule onto a CPU, the kernel uses system-contention scope (SCS). Competition for the CPU with SCS scheduling takes place among all threads in the system. Systems using the one-to-one model (Section 4.3.2), such as Windows, Linux, and Solaris, schedule threads using only SCS.

Typically, PCS is done according to priority—the scheduler selects the runnable thread with the highest priority to run.

### 6.4.2 Pthread Scheduling

Now, we highlight the POSIX Pthread API that allows specifying PCS or SCS during thread creation. Pthreads identifies the following contention scope values:

* PTHREAD SCOPE PROCESS schedules threads using PCS scheduling.
* PTHREAD SCOPE SYSTEM schedules threads using SCS scheduling.

The Pthread IPC provides two functions for getting—and setting—the contention scope policy:

* pthread attr setscope(pthread attr t *attr, int scope)
* pthread attr getscope(pthread attr t *attr, int *scope)

In Figure 6.8, we illustrate a Pthread scheduling API. The program first determines the existing contention scope and sets it to PTHREAD SCOPE SYSTEM. It then creates five separate threads that will run using the SCS scheduling policy. Note that on some systems, only certain contention scope values are allowed. For example, Linux and Mac OS X systems allow only PTHREAD SCOPE SYSTEM.

## 6.5 Multiple-Processor Scheduling

If multiple CPUs are available, load sharing becomes possible—but scheduling problems become correspondingly more complex.

Here, we discuss several concerns in multiprocessor scheduling. We concentrate on systems in which the processors are identical—homogeneous—in terms of their functionality. We can then use any available processor to run any process in the queue. Note, however, that even with homogeneous multiprocessors, there are sometimes limitations on scheduling. Consider a system with an I/O device attached to a private bus of one processor. Processes that wish to use that device must be scheduled to run on that processor.

### 6.5.1 Approaches to Multiple-Processor Scheduling

### 6.5.2 Processor Affinity

### 6.5.3 Load Balancing

### 6.5.4 Multicore Processors

## 6.6 Real-Time CPU Scheduling

### 6.6.1 Minimizing Latency

### 6.6.2 Priority-Based Scheduling

### 6.6.3 Rate-Monotonic Scheduling

### 6.6.4 Earliest-Deadline-First Scheduling

### 6.6.5 Proportional Share Scheduling

### 6.6.6 POSIX Real-Time Scheduling

## 6.7 Operating-System Examples
