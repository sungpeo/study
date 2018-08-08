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

A set of processes is in a deadlocked state when every process in the set is waiting for an event that can be caused only by another process in the set.

## 7.2 Dealock Characterization

### 7.2.1 Necessary Conditions

A deadlock situation can arise if the following four conditions hold simultaneously in a system:

1. **Mutual exclusion**. At least one resource must be held in a nonsharable mode; that is, only one process at a time can use the resource. If another process requests that resource, the requesting process must be delayed until the resource has been released.
2. **Hold and wait**. A process must be holding at least one resource and waiting to acquire additional resources that are currently being held by other processes.
3. **No preemption**. Resources cannot be preempted; that is, a resource can be released only voluntarily by the process holding it, after that process has completed its task.
4. **Circular wait**. A set {P0, P1, ..., Pn} of waiting processes must exist such that P0 is waiting for a resource held by P1, P1 is waiting for a resource held by P2, ..., Pn−1 is waiting for a resource held by Pn, and Pn is waiting for a resource held by P0.

The circular-wait condition implies the hold-and-wait condition, so the four conditions are not completely independent.

### 7.2.2 Resource-Allocation Graph

Deadlocks can be described more precisely in terms of a directed graph called a system resource-allocation graph. A directed edge Pi → Rj is called a request edge; a directed edge Rj → Pi is called an assignment edge.

Given the definition of a resource-allocation graph, it can be shown that, if the graph contains no cycles, then no process in the system is deadlocked. If the graph does contain a cycle, then a deadlock may exist.

## 7.3 Mehods for Handling Deadlocks

* We can use a protocol to prevent or avoid deadlocks, ensuring that the system will never enter a deadlocked state.
* We can allow the system to enter a deadlocked state, detect it, and recover.
* We can ignore the problem altogether and pretend that deadlocks never occur in the system.

To ensure that deadlocks never occur, the system can use either a deadlock prevention or a deadlock-avoidance scheme. **Deadlock prevention** provides a set of methods to ensure that at least one of the necessary conditions (Section 7.2.1) cannot hold. These methods prevent deadlocks by constraining how requests for resources can be made. We discuss these methods in Section 7.4. **Deadlock avoidance** requires that the operating system be given additional information in advance concerning which resources a process will request and use during its lifetime.

## 7.4 Dealock Prevention

As we noted in Section 7.2.1, for a deadlock to occur, each of the four necessary conditions must hold. By ensuring that at least one of these conditions cannot hold, we can prevent the occurrence of a deadlock.

### 7.4.1 Mutual Exclusion

In general, however, we cannot prevent deadlocks by denying the mutual-exclusion condition, because some resources are intrinsically nonsharable.

### 7.4.2 Hold and Wait

One protocol that we can use requires each process to request and be allocated all its resources before it begins execution. We can implement this provision by requiring that system calls requesting resources for a process precede all other system calls.

An alternative protocol allows a process to request resources only when it has none. A process may request some resources and use them. Before it can request any additional resources, it must release all the resources that it is currently allocated.

### 7.4.3 No Preemption

If a process is holding some resources and requests another resource that cannot be immediately allocated to it (that is, the process must wait), then all resources the process is currently holding are preempted. In otherwords, these resources are implicitly released.

This protocol is often applied to resources whose state can be easily saved and restored later, such as CPU registers and memory space. It cannot generally be applied to such resources as mutex locks and semaphores.

### 7.4.4 Circular Wait

That is, a process can initially request any number of instances of a resource type say, Ri . After that, the process can request instances of resource type Rj if and only if F(Rj ) > F(Ri ).

We can accomplish this scheme in an application program by developing an ordering among all synchronization objects in the system. All requests for synchronization objects must be made in increasing order.

One lock-order verifier, which works on BSD versions of UNIX such as FreeBSD, is known as **witness**. Witness uses mutual-exclusion locks to protect critical sections, as described in Chapter 5.

It is also important to note that imposing a lock ordering does not guarantee deadlock prevention if locks can be acquired dynamically.

## 7.5 Deadlock Avoidance

Possible side effects of preventing deadlocks by this method, however, are low device utilization and reduced system throughput.
An alternative method for avoiding deadlocks is to require additional information about how resources are to be requested.

A deadlock-avoidance algorithm dynamically examines the resource-allocation state to ensure that a circular-wait condition can never exist. The resource allocation *state* is defined by the number of available and allocated resources and the maximum demands of the processes.

### 7.5.1 Safe State

A state is safe if the system can allocate resources to each process (up to its maximum) in some order and still avoid a deadlock. More formally, a system is in a safe state only if there exists a safe sequence.

Whenever a process requests a resource that is currently available, the system must decide whether the resource can be allocated immediately or whether the process must wait.

### 7.5.2 Resource-Allocation Graph Alogrithm

If we have a resource-allocation system with only one instance of each resource type, we can use a variant of the resource-allocation graph defined in Section 7.2.2 for deadlock avoidance. In addition to the request and assignment edges already described, we introduce a new type of edge, called a **claim edge**.

A claim edge Pi → Rj indicates that process Pi may request resource Rj at some time in the future. This edge resembles a request edge in direction but is represented in the graph by a dashed line.

If a cycle is found, then the allocation will put the system in an unsafe state. In that case, process Pi will have to wait for its requests to be satisfied.

### 7.5.3 Banker’s Algorithm

The resource-allocation-graph algorithm is not applicable to a resource allocation system with multiple instances of each resource type.

When a new process enters the system, it must declare the maximum number of instances of each resource type that it may need. This number may not exceed the total number of resources in the system. When a user requests a set of resources, the system must determine whether the allocation of these resources will leave the system in a safe state. If it will, the resources are allocated; otherwise, the process must wait until some other process releases enough resources.

Let n = number of processes, and m = number of resources types

* Available: Vector of length m. If Available[ j ] = k, there are k instances of resource type Rj available
* Max: n x m matrix. If Max[ i, j ] = k, then process Pi may request at most k instances of resource type Rj
* Allocation: n x m matrix. If Allocation[ i, j ] = k then Pi is currently allocated k instances of Rj
* Need: n x m matrix. If Need[ i, j ] = k then Pi may need k more instances of Rj to complete its task

Need[ i, j ] = Max[ i, j ] – Allocation[ i, j ]

#### 7.5.3.1 Safety Algorithm

We can now present the algorithm for finding out whether or not a system is in a safe state. This algorithm can be described as follows:

1. Let Work and Finish be vectors of length m and n, respectively. Initialize Work = Available and Finish[i] = false for i = 0, 1, ..., n − 1.

2. Find an index i such that both
    a. Finish[i] == false
    b. Needi ≤Work
    If no such i exists, go to step 4.

3. Work =Work + Allocationi
    Finish[i] = true
    Go to step 2.

4. If Finish[i] == true for all i, then the system is in a safe state.
    This algorithm may require an order ofm × n2 operations to determine whether
    a state is safe.

#### 7.5.3.2 Resource-Request Algorithm

When a request for resources is made by process Pi , the following actions are taken:

1. If Requesti ≤Needi , go to step 2. Otherwise, raise an error condition, since the process has exceeded its maximum claim.

2. If Requesti ≤ Available, go to step 3. Otherwise, Pi must wait, since the resources are not available.

3. Have the system pretend to have allocated the requested resources to process Pi by modifying the state as follows:

#### 7.5.3.3 An Illustrative Example

(생략)

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

### 7.7.1 Process Termination

### 7.7.2 Resource Preemption