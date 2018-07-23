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

### 6.3.2 Shortest-Job-First Scheduling

### 6.3.3 Priority Scheduling

### 6.3.4 Round-Robin Scheduling

### 6.3.5 Multilevel Queue Scheduling

### 6.3.6 Multilevel Feedback Queue Scheduling

## Thread Scheduling

### 6.4.1 Contention Scope

### 6.4.2 Pthread Scheduling

## 6.5 Multiple-Processor Scheduling

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
