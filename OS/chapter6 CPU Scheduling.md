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

One approach to CPU scheduling in a multiprocessor system has all scheduling decisions, I/O processing, and other system activities handled by a single processor—the master server. The other processors execute only user code. This asymmetric multiprocessing is simple because only one processor accesses the system data structures, reducing the need for data sharing.

A second approach uses symmetric multiprocessing (SMP), where each processor is self-scheduling. All processes may be in a common ready queue, or each processor may have its own private queue of ready processes.

### 6.5.2 Processor Affinity

Because of the high cost of invalidating and repopulating caches, most SMP systems try to avoid migration of processes from one processor to another and instead attempt to keep a process running on the same processor. This is known as processor affinity—that is, a process has an affinity for the processor on which it is currently running.

* soft affinity. When an operating system has a policy of attempting to keep a process running on the same processor—but not guaranteeing that it will do so—
* hard affinity. some systems provide system calls

Rather, the “solid lines” between sections of an operating system are frequently only “dotted lines,” with algorithms creating connections in ways aimed at optimizing performance and reliability.

### 6.5.3 Load Balancing

On SMP systems, it is important to keep the workload balanced among all processors to fully utilize the benefits of having more than one processor.

Load balancing attempts to keep the workload evenly distributed across all processors in an SMP system. It is important to note that load balancing is typically necessary only on systems where each processor has its own private queue of eligible processes to execute. (SMP를 지원하는 대부분의 contemporary operating systems은 private queue를 갖는다.)

There are two general approaches to load balancing: push migration and pull migration. With push migration, a specific task periodically checks the load on each processor and—if it finds an imbalance—evenly distributes the load by moving (or pushing) processes from overloaded to idle or less-busy processors. Pull migration occurs when an idle processor pulls a waiting task from a busy processor.

유심히 읽은 사람은 load banlancing이 processor affinity의 장점을 방해한다는 걸 알 수 있을 것이다. System engineering 입장에서 절대적인 rule은 없다. 어떤 시스템에선 imblance threshold를 가지고 pull process를 해올지 결정하기도 한다.

### 6.5.4 Multicore Processors

Traditionally, SMP systems have allowed several threads to run concurrently by providingmultiple physical processors. However, a recent practice in computer hardware has been to place multiple processor cores on the same physical chip, resulting in a multicore processor.

Researchers have discovered that when a processor accesses memory, it spends a significant amount of time waiting for the data to become available. This situation, known as a memory stall, may occur for various reasons, such as a cache miss (accessing data that are not in cache memory).

To remedy this situation, many recent hardware designs have implemented multithreaded processor cores in which two (or more) hardware threads are assigned to each core. That way, if one thread stalls while waiting for memory, the core can switch to another thread. Figure 6.11 illustrates a dual-threaded processor core on which the execution of thread 0 and the execution of thread 1 are interleaved. From an operating-system perspective, each hardware thread appears as a logical processor that is available to run a software thread. Thus, on a dual-threaded, dual-core system, four logical processors are presented to the operating system.

In general, there are two ways to multithread a processing core: coarse-grained and fine-grained multithreading.
대단위 멀티스레딩은 memory stall이 일어나면, 실행 thread를 변경한다. instruction pipeline이 flush되고 새로운 thread에 채워줘야하기 때문에 느리다. 세분화된 멀티스레딩은 더 세분화된 레벨의 granularity로 switching되고, thread switching을 위한 logic을 포함하고 있어, thread switching cost가 작다.

Notice that a multithreaded multicore processor actually requires two different levels of scheduling. On one level are the scheduling decisions that must be  ade by the operating system as it chooses which software thread to run on each hardware thread (logical processor). For this level of scheduling, the operating system may choose any scheduling algorithm, such as those described in Section 6.3. A second level of scheduling specifies how each core decides which hardware thread to run. There are several strategies to adopt in this situation.

## 6.6 Real-Time CPU Scheduling

Real-time OS 환경에서 CPU Scheduling하는 데에는 또 다른 이슈들이 있다. 일반적으로 우리는 soft real-time systems과 hard real-time systems를 구분해야 한다.

* Soft real-time systems provide no guarantee as to when a critical real-time process will be scheduled. They guarantee only that the process will be given preference over noncritical processes.
* Hard real-time systems have stricter requirements. A task must be serviced by its deadline; service after the deadline has expired is the same as no service at all.

### 6.6.1 Minimizing Latency

event-driven 환경의 real-time system이라면..
When an event occurs, the system must respond to and service it as quickly as possible.We refer to event latency as the amount of time that elapses from when an event occurs to when it is serviced (Figure 6.12).

Two types of latencies affect the performance of real-time systems:

1. Interrupt latency refers to the period of time fromthe arrival of an interrupt at the CPU to the start of the routine that services the interrupt.
2. Dispatch latency

Obviously, it is crucial for real-time operating systems to minimize interrupt latency to ensure that real-time tasks receive immediate attention.
Indeed, for hard real-time systems, interrupt latency must not simply be minimized, it must be bounded to meet the strict requirements of these systems
One important factor contributing to interrupt latency is the amount of time interrupts may be disabled while kernel data structures are being updated. Real-time operating systems require that interrupts be disabled for only very short periods of time.

The amount of time required for the scheduling dispatcher to stop one process and start another is known as dispatch latency. Providing real-time tasks with immediate access to the CPU mandates that real-time operating systems minimize this latency as well. The most effective technique for keeping dispatch latency low is to provide preemptive kernels.

### 6.6.2 Priority-Based Scheduling

As a result, the scheduler for a real-time operating system must support a priority-based algorithm with preemption.
Note that providing a preemptive, priority-based scheduler only guarantees soft real-time functionality. Hard real-time systems must further guarantee that real-time tasks will be serviced in accord with their deadline requirements, and making such guarantees requires additional scheduling features.

Before we proceed with the details of the individual schedulers, however, we must define certain characteristics of the processes that are to be scheduled. First, the processes are considered periodic. (processing time t, deadline d, and a period p)

Then, using a technique known as an admission-control algorithm, the scheduler does one of two things. It either admits the process, guaranteeing that the process will complete on time, or rejects the request as impossible if it cannot guarantee that the task will be serviced by its deadline.

### 6.6.3 Rate-Monotonic Scheduling

The rate-monotonic scheduling algorithm schedules periodic tasks using a static priority policy with preemption.

period가 짧을 수록 높은 priority를 갖는데, 이 정책의 근거는 CPU가 더 자주 필요한 작업에 더 높은 우선 순위를 할당하는 것이다. Furthermore, rate-monotonic scheduling assumes that the processing time of a periodic process is the same for each CPU burst.

Despite being optimal, then, rate-monotonic scheduling has a limitation: CPU utilization is bounded, and it is not always possible fully to maximize CPU resources. The worst-case CPU utilization for scheduling N processes is 

N(2^1/N − 1).

With one process in the system, CPU utilization is 100 percent, but it falls to approximately 69 percent as the number of processes approaches infinity.

### 6.6.4 Earliest-Deadline-First Scheduling

Earliest-deadline-first (EDF) scheduling dynamically assigns priorities according to deadline.
Unlike the rate-monotonic algorithm, EDF scheduling does not require that processes be periodic, nor must a process require a constant amount of CPU time per burst. The only requirement is that a process announce its deadline to the scheduler when it becomes runnable. 이론적으로도 optimal인데, context switching 비용이 있기 때문에 CPU utilization이 100%이 되는 작업은 스케줄링할 수는 없다.

### 6.6.5 Proportional Share Scheduling

Proportional share schedulers must work in conjunction with an admission-control policy to guarantee that an application receives its allocated shares of time. An admission-control policy will admit a client requesting a particular number of shares only if sufficient shares are available. Total of share가 최대(ex.100)를 넘을 수 있는 새로운 프로세스가 요청되면 deny한다.

### 6.6.6 POSIX Real-Time Scheduling

The POSIX standard also provides extensions for real-time computing—POSIX.1b. Here, we cover some of the POSIX API related to scheduling real-time threads. POSIX defines two scheduling classes for real-time threads:

* SCHED_FIFO
* SCHED_RR

SCHED FIFO schedules threads according to a first-come, first-served policy using a FIFO queue as outlined in Section 6.3.1. However, there is no time  licing among threads of equal priority. Therefore, the highest-priority real-time thread at the front of the FIFO queue will be granted the CPU until it terminates or blocks. SCHED RR uses a round-robin policy. It is similar to SCHED FIFO except that it provides time slicing among threads of equal priority.

## 6.7 Operating-System Examples

### 6.7.1 Example: Linux Scheduling

in release 2.6.23 of the kernel, the Completely Fair Scheduler(CFS) became the default Linux scheduling algorithm.

Scheduling in the Linux system is based on scheduling classes. Each class is assigned a specific priority. By using different scheduling classes, the kernel can accommodate different scheduling algorithms based on the needs of the system and its processes. The scheduling criteria for a Linux server, for example, may be different from those for a mobile device running Linux.

Rather than using strict rules that associate a relative priority value with the length of a time quantum, the CFS scheduler assigns a proportion of CPU processing time to each task. This proportion is calculated based on the nice value(-20 ~ +19) assigned to each task.

CFS doesn’t use discrete values of time slices and instead identifies a targeted latency, which is an interval of time during which every runnable task should run at least once.

The CFS scheduler doesn’t directly assign priorities. Rather, it records how long each task has run by maintaining the virtual run time of each task using the per-task variable vruntime.

Linux uses two separate priority ranges, one for real-time tasks and a second for normal tasks. Real-time tasks are assigned static priorities within the range of 0 to 99, and normal (i.e. non real-time) tasks are assigned priorities from 100 to 139.

### 6.7.2 Example: Windows Scheduling

### 6.7.3 Example: Solaris Scheduling

## 6.8 Algorithm Evaluation

Our criteria may include several measures, such as these:

* Maximizing CPU utilization under the constraint that the maximum response time is 1 second
* Maximizing throughput such that turnaround

### 6.8.1 Deterministic Modeling

One major class of evaluation methods is analytic evaluation. Analytic evaluation uses the given algorithm and the system workload to produce a formula or number to evaluate the performance of the algorithm for that workload.
Deterministic modeling is one type of analytic evaluation. This method takes a particular predetermined workload and defines the performance of each algorithm for that workload.

Deterministic modeling is simple and fast. It gives us exact numbers, allowing us to compare the algorithms. However, it requires exact numbers for input, and its answers apply only to those cases.

### 6.8.2 Queueing Models

The computer system is described as a network of servers. Each server has a queue of waiting processes. The CPU is a server with its ready queue, as is the I/O system with its device queues. Knowing arrival rates and service rates, we can compute utilization,  average queue length, average wait time, and so on. This area of study is called queueing-network analysis.

Thus, n = (lambda) × W.
This equation, known as Little’s formula

### 6.8.3 Simulations

To get a more accurate evaluation of scheduling algorithms, we can use simulations.

we can use trace tapes.We create a trace tape by monitoring the real system and recording the sequence of actual events (Figure 6.25).

Simulations can be expensive, often requiring hours of computer time. A more detailed simulation provides more accurate results, but it also takes more computer time. In addition, trace tapes can require large amounts of storage space. Finally, the design, coding, and debugging of the simulator can be a major task.

### 6.8.4 Implementation

The most flexible scheduling algorithms are those that can be altered by the system managers or by the users so that they can be tuned for a specific application or set of applications.

## 6.9 Summary