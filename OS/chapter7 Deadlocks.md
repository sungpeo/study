# Chapter7. Deadlocks

Sometimes, a waiting process is never again able to change state, because the resources it has requested are held by other waiting processes. This situation is called a deadlock.

## Chapter Objectives

* To develop a description of deadlocks, which prevent sets of concurrent processes from completing their tasks.
* To present a number of different methods for preventing or avoiding deadlocks in a computer system.

## 7.1 System Model

Under the normal mode of operation, a process may utilize a resource in only the following sequence:

1. Request. The process requests the resource. If the request cannot be granted immediately (for example, if the resource is being used by another process), then the requesting process must wait until it can acquire the resource.
2. Use. The process can operate on the resource (for example, if the resource is a printer, the process can print on the printer).
3. Release. The process releases the resource.

resource에 대한 request, release 요청은 system call 일 것이다. (including semaphores and mutex lock). System table은 resource들의 할당 여부를 기록할 뿐 아니라, 어느 프로세스에 할당되었는 지도 기록하고 있다.
어떤 프로세스 세트(모임)가 다른 이벤트를 기다리고 있는데, 그 기다리는 이벤트들이 해당 세트의 다른 프로세서로부터 발생할 수 있는 상태라면, 그 프로세스 세트는 deadlock 상태에 빠진 것이다. 우리가 주로 고민하고 있는 그 이벤트들은 resouce acquisition과 release 이다. resource라 함은 physical resources or logical resources 이다.

Developers of multithreaded applications must remain aware of the possibility of deadlocks. The locking tools presented in Chapter 5 are designed to avoid race conditions. However, in using these tools, developers must pay careful attention to how locks are acquired and released.

## 7.2 Dealock Characterization

In a deadlock, processes never finish executing, and system resources are tied up, preventing other jobs fromstarting.

### 7.2.1 Necessary Conditions

A deadlock situation can arise if the following four conditions hold simultaneously in a system:

1. **Mutual exclusion**. At least one resource must be held in a nonsharable mode; that is, only one process at a time can use the resource. If another process requests that resource, the requesting process must be delayed until the resource has been released.
2. **Hold and wait**. A process must be holding at least one resource and waiting to acquire additional resources that are currently being held by other processes.
3. **No preemption**. Resources cannot be preempted; that is, a resource can be released only voluntarily by the process holding it, after that process has completed its task.
4. **Circular wait**. A set {P0, P1, ..., Pn} of waiting processes must exist such that P0 is waiting for a resource held by P1, P1 is waiting for a resource held by P2, ..., Pn−1 is waiting for a resource held by Pn, and Pn is waiting for a resource held by P0.

The circular-wait condition implies the hold-and-wait condition, so the four conditions are not completely independent.

### 7.2.2 Resource-Allocation Graph

Deadlocks can be described more precisely in terms of a directed graph called a **system resource-allocation graph**. This graph consists of a set of vertices V and a set of edges E. The set of vertices V is partitioned into two different types of nodes: P = {P1, P2, ..., Pn}, the set consisting of all the active processes in the system, and R = {R1, R2, ..., Rm}, the set consisting of all resource types in the system.
A directed edge Pi → Rj is called a **request edge**; a directed edge Rj → Pi is called an **assignment edge**.

Given the definition of a resource-allocation graph, it can be shown that(Figure 7.1), if the graph contains no cycles, then no process in the system is deadlocked. If the graph does contain a cycle, then a deadlock may exist.

## 7.3 Mehods for Handling Deadlocks

Generally speaking, we can deal with the deadlock problem in one of three ways:

* We can use a protocol to prevent or avoid deadlocks, ensuring that the system will never enter a deadlocked state.
* We can allow the system to enter a deadlocked state, detect it, and recover.
* We can ignore the problem altogether and pretend that deadlocks never occur in the system.

To ensure that deadlocks never occur, the system can use either a deadlock prevention or a deadlock-avoidance scheme. **Deadlock prevention** provides a set of methods to ensure that at least one of the necessary conditions (Section 7.2.1) cannot hold. These methods prevent deadlocks by constraining how requests for resources can be made. We discuss these methods in Section 7.4. **Deadlock avoidance** requires that the operating system be given additional information in advance concerning which resources a process will request and use during its lifetime.

deadlock-prevention이나 deadlock-avoidance algorithm을 사용하지 않는다면, deadlock은 발생할 것이다. 이런 환경에서 deadlock이 발생했는 파악해서 recover 하려는 방법이 있다. (Section 7.6 and 7.7)

detect나 recover하는 알고리즘도 없다면, deadlock 발 시, 시스템은 결국 performance 문제가 발생하고, 궁극적으로는 기능이 멈추어 restart가 필요하게 될 것이다. 뭐 이런게 다 있나 싶겠지만, 이것이 가장 많은 OS들이 채택하고 있는 방법이다. 잘 일어나는 것도 아니고, 비용(Expnse) 때문에 어쩔 수 없다.

## 7.4 Dealock Prevention

As we noted in Section 7.2.1, for a deadlock to occur, each of the four necessary conditions must hold. By ensuring that at least one of these conditions cannot hold, we can prevent the occurrence of a deadlock.

### 7.4.1 Mutual Exclusion

The mutual exclusion condition must hold. That is, at least one resource must be nonsharable.

In general, however, we cannot prevent deadlocks by denying the mutual-exclusion condition, because some resources are intrinsically nonsharable.

### 7.4.2 Hold and Wait

To ensure that the hold-and-wait condition never occurs in the system, we must guarantee that, whenever a process requests a resource, it does not hold any other resources.

One protocol that we can use requires each process to request and be allocated all its resources before it begins execution. We can implement this provision by requiring that system calls requesting resources for a process precede all other system calls.

An alternative protocol allows a process to request resources only when it has none. A process may request some resources and use them. Before it can request any additional resources, it must release all the resources that it is currently allocated.

### 7.4.3 No Preemption

The third necessary condition for deadlocks is that there be no preemption of resources that have already been allocated. To ensure that this condition does not hold, we can use the following protocol.

한 프로세스가 어떤 resources를 쓰고 있고(is holding), 바로 할당 받을 수 없는 다른 resource를 요청한 상태라면, 해당 프로세스가 쓰고 있던(is currently holding) 모든 resources가 preempted된다. 암묵적인 release이고, 이 resource들은 프로세스가 기다리고있는 resouce list에 추가된다. 여기 프로세스는 처음에 가지고 있던 resource와 요청했던 새로운 resource들을 모두 얻었을 때 재시작된다.

Alternatively, process가 some resources를 요청하면, 그것이 사용 가능한지 확인한다. 사용 불가능하다면, 해당 resource가 또 다른 resource를 기다리는 중인 waiting process에게 allocated 된 것인지 확인한다. 그렇다면, waiting process로부터 resouce들을 preempt하여 requeting process에 할당시킨다. waiting process에게 할당된 것이 아니었다면, requestin process must wait. While it is
waiting, some of its resources may be preempted, but only if another process requests them. A process can be restarted only when it is allocated the new resources it is requesting and recovers any resources that were preempted while it was waiting.

This protocol is often applied to resources whose state can be easily saved and restored later, such as CPU registers and memory space. It cannot generally be applied to such resources as mutex locks and semaphores.

### 7.4.4 Circular Wait

The fourth and final condition for deadlocks is the circular-wait condition. One way to ensure that this condition never holds is to impose a total ordering of all resource types and to require that each process requests resources in an increasing order of enumeration. 

That is, a process can initially request any number of instances of a resource type say, Ri . After that, the process can request instances of resource type Rj if and only if F(Rj ) > F(Ri ).

We can accomplish this scheme in an application program by developing an ordering among all synchronization objects in the system. All requests for synchronization objects must be made in increasing order.

Alternatively, we can require that a process requesting an instance of resource type Rj must have released any resources Ri such that F(Ri ) ≥ F(Rj ). Note also that if several instances of the same resource type are needed, a single request for all of them must be issued.

이러면, circurlar-wait condition이 지속될수 없고, 우린 이걸 증명할 수 있다. (proof by contradiction) ...

Keep in mind that developing an ordering, or hierarchy, does not in itself prevent deadlock. It is up to application developers to write programs that follow the ordering

Although ensuring that resources are acquired in the proper order is the responsibility of application developers, certain software can be used to verify that locks are acquired in the proper order and to give appropriate warnings when locks are acquired out of order and deadlock is possible.
One lock-order verifier, which works on BSD versions of UNIX such as FreeBSD, is known as **witness**. Witness uses mutual-exclusion locks to protect critical sections, as described in Chapter 5.

It is also important to note that imposing a lock ordering does not guarantee deadlock prevention if locks can be acquired dynamically.

(Figure 7.5) Deadlock is possible if two threads simultaneously invoke the transaction() function, transposing different accounts.

## 7.5 Deadlock Avoidance

Possible side effects of preventing deadlocks by this method, however, are low device utilization and reduced system throughput.
An alternative method for avoiding deadlocks is to require additional information about how resources are to be requested.

The various algorithms that use this approach differ in the amount and type of information required. The simplest and most useful model requires that each process declare the maximum number of resources of each type that it may need.

A deadlock-avoidance algorithm dynamically examines the resource-allocation state to ensure that a circular-wait condition can never exist. The resource allocation *state* is defined by the number of available and allocated resources and the maximum demands of the processes. In the following sections, we explore two deadlock-avoidance algorithms.

### 7.5.1 Safe State

A state is safe if the system can allocate resources to each process (up to its maximum) in some order and still avoid a deadlock. More formally, a system is in a safe state only if there exists a **safe sequence**. A sequence of processes <P1, P2, ..., Pn> is a safe sequence for the current allocation state if, for each Pi , the resource requests that Pi can still make can be satisfied by the currently available resources plus the resources held by all Pj, with j < i. In this situation, if the resources that Pi needs are not immediately available, then Pi can wait until all Pj have finished.

Not all unsafe states are deadlocks, however (Figure 7.6). An unsafe state may lead to a deadlock.

Given the concept of a safe state, we can define avoidance algorithms that ensure that the system will never deadlock. The idea is simply to ensure that the system will always remain in a safe state. Initially, the system is in a safe state. Whenever a process requests a resource that is currently available, the system must decide whether the resource can be allocated immediately or whether the process must wait. The request is granted only if the allocation leaves the system in a safe state.

### 7.5.2 Resource-Allocation Graph Alogrithm

If we have a resource-allocation system with only one instance of each resource type, we can use a variant of the resource-allocation graph defined in Section 7.2.2 for deadlock avoidance. In addition to the request and assignment edges already described, we introduce a new type of edge, called a **claim edge**. A claim edge Pi → Rj indicates that process Pi may request resource Rj at some time in the future. This edge resembles a request edge in direction but is represented in the graph by a dashed line.

If no cycle exists, then the allocation of the resource will leave the system in a safe state. If a cycle is found, then the allocation will put the system in an unsafe state. In that case, process Pi will have to wait for its requests to be satisfied.

### 7.5.3 Banker’s Algorithm

The resource-allocation-graph algorithm is not applicable to a resource allocation system with multiple instances of each resource type. The deadlock avoidance algorithm that we describe next is applicable to such a system but is less efficient than the resource-allocation graph scheme. This algorithm is commonly known as the banker’s algorithm.

When a new process enters the system, it must declare the maximum number of instances of each resource type that it may need. This number may not exceed the total number of resources in the system. When a user requests a set of resources, the system must determine whether the allocation of these resources will leave the system in a safe state. If it will, the resources are allocated; otherwise, the process must wait until some other process releases enough resources.

Let n = number of processes, and m = number of resources types

* **Available**: Vector of length m. If Available[ j ] = k, there are k instances of resource type Rj available
* **Max**: n x m matrix. If Max[ i, j ] = k, then process Pi may request at most k instances of resource type Rj
* **Allocation**: n x m matrix. If Allocation[ i, j ] = k then Pi is currently allocated k instances of Rj
* **Need**: n x m matrix. If Need[ i, j ] = k then Pi may need k more instances of Rj to complete its task

Need[ i, j ] = Max[ i, j ] – Allocation[ i, j ]

#### 7.5.3.1 Safety Algorithm

We can now present the algorithm for finding out whether or not a system is in a safe state. This algorithm can be described as follows:

1. Let Work and Finish be vectors of length m and n, respectively. Initialize Work = Available and Finish[i] = false for i = 0, 1, ..., n − 1.

2. Find an index i such that both
    a. Finish[i] == false
    b. Need_i ≤ Work
    If no such i exists, go to step 4.

3. Work = Work + Allocation_i
    Finish[i] = true
    Go to step 2.

4. If Finish[i] == true for all i, then the system is in a safe state.
    This algorithm may require an order of (m × n2) operations to determine whether
    a state is safe.

#### 7.5.3.2 Resource-Request Algorithm

When a request for resources is made by process Pi , the following actions are taken:

1. If Requesti ≤ Needi , go to step 2. Otherwise, raise an error condition, since the process has exceeded its maximum claim.

2. If Requesti ≤ Available, go to step 3. Otherwise, Pi must wait, since the resources are not available.

3. Have the system pretend to have allocated the requested resources to process Pi by modifying the state as follows:

#### 7.5.3.3 An Illustrative Example

(생략) 333쪽 357/944

## Deadlock Detection

If a system does not employ either a deadlock-prevention or a deadlock avoidance algorithm, then a deadlock situation may occur. In this environment, the system may provide:

* An algorithm that examines the state of the system to determine whether a deadlock has occurred
* An algorithm to recover from the deadlock

### 7.6.1 Single Instance of Each Resource Type

If all resources have only a single instance, then we can define a deadlock detection algorithm that uses a variant of the resource-allocation graph, called a **wait-for** graph.

As before, a deadlock exists in the system if and only if the wait-for graph contains a cycle. To detect deadlocks, the system needs to maintain the wait-for graph and periodically *invoke an algorithm* that searches for a cycle in the graph. An algorithm to detect a cycle in a graph requires an order of n2 operations, where n is the number of vertices in the graph.

### 7.6.2 Several Instances of a Resource Type

The wait-for graph scheme is not applicable to a resource-allocation system with multiple instances of each resource type.

* **Available**: A vector of length m indicates the number of available resources of each type
* **Allocation**: An n x m matrix defines the number of resources of each type currently allocated to each process
* **Request**: An n x m matrix indicates the current request of each process. If Request[ i, j ] = k, then process Pi is requesting k more instances of resource type Rj

1. Let Work and Finish be vectors of length m and n, respectively. Initialize Work = Available. For i = 0, 1, ..., n–1, if Allocationi = 0, then Finish[i] = false. Otherwise, Finish[i] = true.
2. Find an index i such that both
    a. Finish[i] == false
    b. Requesti ≤Work
    If no such i exists, go to step 4.
3. Work =Work + Allocationi
    Finish[i] = true
    Go to step 2.
4. If Finish[i] == false for some i, 0≤i<n, then the system is in a deadlocked state. Moreover, if Finish[i] == false, then process Pi is deadlocked.

### 7.6.3 Detection-Algorithm Usage

When should we invoke the detection algorithm? The answer depends on two factors:

1. How often is a deadlock likely to occur?
2. How many processes will be affected by deadlock when it happens?

invoking the deadlock-detection algorithm for every resource request will incur considerable overhead in computation time. A less expensive alternative is simply to invoke the algorithm at defined intervals—for example, once per hour or whenever CPU utilization drops below 40 percent. (A deadlock eventually cripples system throughput and causes CPU utilization to drop.) If the detection algorithm is invoked at arbitrary points in time, the resource graph may contain many cycles. In this case, we generally cannot tell which of the many deadlocked processes “caused” the deadlock.

## 7.7 Recovery from Deadlock

When a detection algorithm determines that a deadlock exists, several alternativesare available. One possibility is to inform the operator that a deadlock has occurred and to let the operator deal with the deadlock manually. Another possibility is to let the system recover from the deadlock automatically. There are two options for breaking a deadlock. One is simply to abort one or more processes to break the circular wait. The other is to preempt some resources from one or more of the deadlocked processes.

### 7.7.1 Process Termination

* **Abort all deadlocked processes.** This method clearly will break the deadlock cycle, but at great expense. The deadlocked processes may have computed for a long time, and the results of these partial computations must be discarded and probably will have to be recomputed later.
* **Abort one process at a time until the deadlock** cycle is eliminated. This method incurs considerable overhead, since after each process is aborted, a deadlock-detection algorithm must be invoked to determine whether any processes are still deadlocked.

we should abort those processes whose termination will incur the minimum cost. Unfortunately, the term minimum cost is not a precise one. Many factors may affect which process is chosen, including:

1. What the priority of the process is
2. How long the process has computed and how much longer the process will compute before completing its designated task
3. Howmany andwhat types of resources the process has used (for example, whether the resources are simple to preempt)
4. How many more resources the process needs in order to complete
5. How many processes will need to be terminated
6. Whether the process is interactive or batch

### 7.7.2 Resource Preemption

To eliminate deadlocks using resource preemption, we successively preempt some resources fromprocesses and give these resources to other processes until the deadlock cycle is broken.
If preemption is required to deal with deadlocks, then three issues need to be addressed:

1. **Selecting a victim.** As in process termination, we must determine the order of preemption to minimize cost. Cost factors may include such parameters as the number of resources a deadlocked process is holding and the amount of time the process has thus far consumed.

2. **Rollback.** Since, in general, it is difficult to determine what a safe state is, the simplest solution is a total rollback: abort the process and then restart it. Although it is more effective to roll back the process only as far as necessary to break the deadlock, this method requires the system to keep more information about the state of all running processes.

3. **Starvation.** In a system where victim selection is based primarily on cost factors, it may happen that the same process is always picked as a victim. As a result, this process never completes its designated task, a starvation situation any practical system must address. Clearly, we must ensure that a process can be picked as a victim only a (small) finite number of times. The most common solution is to include the number of rollbacks in the cost factor.

## 7.8 Summary

There are three principal methods for dealing with deadlocks:
1. pevent or avoid deadlocks
2. detect it, and then recover
3. Ignore the problem

A deadlock can occur only if four necessary conditions hold simultaneously in the system: mutual exclusion, hold and wait, no preemption, and circular wait. To prevent deadlocks, we can ensure that at least one of the necessary conditions never holds.

A method for avoiding deadlocks, rather than preventing them, requires that the operating system have a priori information about how each process will utilize system resources. The banker’s algorithm, for example, requires a priori information about the maximum number of each resource class that each process may request. Using this information, we can define a deadlock avoidance algorithm.

If a system does not employ a protocol to ensure that deadlocks will never occur, then a detection-and-recovery scheme may be employed. A deadlock detection algorithm must be invoked to determine whether a deadlock has occurred. If a deadlock is detected, the system must recover either by terminating some of the deadlocked processes or by preempting resources from some of the deadlocked processes.

Where preemption is used to deal with deadlocks, three issues must be addressed: selecting a victim, rollback, and starvation. In a system that selects victims for rollback primarily on the basis of cost factors, starvation may occur, and the selected process can never complete its designated task.

Researchers have argued that none of the basic approaches alone is appropriate for the entire spectrum of resource-allocation problems in operating systems. The basic approaches can be combined, however, allowing us to select an optimal approach for each class of resources in a system.

