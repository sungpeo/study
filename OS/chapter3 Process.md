# Chapter3. Process



## Chapter Objectives
* 프로세스라는 용어에 대해 알아보자. - 수행 중인 프로그램으로, 모든 계산(computation)의 기초
* 프로세스의 여러 특성들을 살펴보자. (스케줄링, 생성, 삭제)
* 프로세스간 통신을 위한 방법인 shared memory 와 meesage passing을 알아보자.
* Client-server 시스템의 통신 방법을 묘사한다.


## 3.1 Process Concept
jobs, user program, task 등의 activity를 모두 process라 할 수 있다.
job과 process라는 용어를 여기서는 같은 의미로도 쓰이고 있다.

### 3.1.1 The Process
프로세스는 실행 중인 프로그램이다. text section으로 알려진 프로그램 코드 이상으로 현재 상태(program counter)와 프로세스의 레지스터 내용까지도 포함하는 개념이다.
프로세스는 일반적으로 process stack과 data section, heap을 포함한다.
다시 한번 말하지만, 프로그램 자체는 프로세스가 아니다. 
프로그램은 passive entity로서, executable file 같은 형태로 저장되며, 프로세스는 active entity이다.
프로세스는 스스로 다른 코드를 위한 수행 환경 자체가 되기도 한다. JVM이 대표적인 예이다.

### 3.1.2 Process State
* New. The process is being created.
* Running. Instructions are being executed.
* Waiting. The process is waiting for some event to occur (such as an I/O completion or reception of a signal).
* Ready. The process is waiting to be assigned to a processor.
* Terminated. The process has finished execution.

### 3.1.3 Process Control Block
각각의 프로세는 OS 상에서 process control blok으로 나타내어진다. PCB는 Figure 3.3으로 볼 수 있다.
특정 프로세스와 관련해서 다양한 정보를 저장하고 있다. 
* Process state. 상태는 new, ready, running, waiting, halted 등이 될 수 있다.
* Program counter. counter는 이 프로세스가 실행시킬 다음 명령어 주소를 나타낸다.
* CPU registers. Computer 아키텍처에 따라 다양하다. 인터럽트 발생 시, 이 상태 정보가 있어야 올바른 방향으로 프로세스가 진행된다.
* CPU-scheduling information. 프로세스 우선순위, 스케줄링 큐 포인터 및 스케줄링 파라미터들
* Memory-management information. base and limit registers and page tables, or the segment tables.
* Accounting information. CPU 수량, 실사용 시간, 시간 제한, 계정 번호, 프로세스 번호 등
* I/O status information. 프로세스가 할당된 I/O device 목록, open files

### 3.1.4 Threads
지금까지 설명에선 싱글 스레드 기반을 가정하고 있었다. 현대 OS들은 프로세스 안에 여러 스레드 실행해서 성능 향상을 꾀할 수 있도록 한다.
그런 시스템에서는 PCB에 각각의 스레드들에 대한 정보까지도 추가로 포함한다. 4장에서 자세히 보자.


## 3.2 Process Scheduling
### 3.2.1 Scheduling Queues
프로세스가 시스템에 들어오면, 'job queue'에 쌓인다. 이중에서 작업 수행을 기다리는 프로세스들은 'ready queue'에 저장된다.
I/O device에 할당됭 기다리는 프로세스는 목록은 'device queue'라 한다.
* 프로세스는 I/O 요청을 만들고, I/O queue에 위치할 수 있다.
* 프로세스는 자식(child) 프로세스를 생성하고, child의 종료를 기다린다.
* 프로세스는 인터럽트를 통해, 강제로 CPU로부터 제거될 수 있다. 해당 프로세스는 ready queue로 돌아간다.

### 3.2.2 Schedulers
##### long-term scheduler (or job scheduler)
어떤 프로세스를 메모리에 올려 실행시킬 지 결정한다. 'degree of multiprogramming'을 조절한다. 프로세스가 시스템에서 종료될 때마다 스케줄링 하면 되므로,
상대적으로 매운 긴 시간에 한번씩 수행되므로, 빠른 선택보다는 주의 깊은 선택이 중요하다.
I/O-bound process, CPU-bound process
##### short-term scheduler (or CPU sheduler)
CPU에 할당할 프로세스를 선택하는 스케줄러이다. 적어도 100 milliseconds에 한번은 수행될 정도로 interval이 짧다. 물론 빨라야 한다.
##### medium-term scheduler
프로세스를 잠시 뺐다가, 나중에 다시 넣어주는 역할을 한다. 8장에서 자세히 보자.

### 3.2.3 Context Switch
Switching the CPU to another process requires performing a state save of
the current process and a state restore of a different process. This task is known
as a context switch.

## 3.3 Operations on Processes
### 3.3.1 Process Creation
트리 구조로 프로세스는 구성된다. 프로세스를 만든 프로세스가 부모 프로세스가 되고, 새로운 프로세스가 자식 프로세스가 된다.
대부분의 Operating system에서 프로세스는 유니크한 integer number의 process identifier (or pid)를 갖는다.
자식 프로세스가 시작되면 operating system으로부터 직접 리소스를 받기도 하고, 또는 부모 프로세서의 subset으로 리소스를 할당 받기도 한다.

수행되는 방식은 2가지가 있다.
1. 부모 프로세스와 자식 프로세스가 동시에(concurrently) 수행되는 경우
2. 일부 혹은 모든 자식 프로세스가 종료되기를 부모 프로세스가 기다리는(wait) 경우

address-spcae에 대해서도 2가지가 있다.
1. 부모 프로세스의 duplicate로서, 자식 프로세스 생성
2. 새로운 프로그램으로 구성된 자식 프로세스

...

### 3.3.2 Process Termination

어떤 시스템에서는 child가 혼자 실행되는 것을 허용하지 않는데, 그런 시스템에선 cascading termination이 일어난다.
A process that has terminated, but whose parent has not yet called wait(), is known as a zombie process.
Now consider what would happen if a parent did not invoke wait() and instead terminated, thereby leaving its child processes as orphans.

## 3.4 Interprocess Communication
Several reasons
* Information sharing.
* Computation speedup.
* Modularity.
* Convenience.

There are two fundamental models of interprocess communication: shared memoryand message passing.

### 3.4.1 Shared-Memory Systems
A producer can produce one item while
the consumer is consuming another item. The producer and consumer must
be synchronized, so that the consumer does not try to consume an item that
has not yet been produced.
여기에 사용되는 버퍼에도 2가지가 있다 : unbounded buffer, bounded buffer

### 3.4.2 Message-Passing Systems
Message passing provides a mechanism to allow processes to communicate
and to synchronize their actions without sharing the same address space. It is
particularly useful in a distributed environment, where the communicating
processes may reside on different computers connected by a network.

#### 3.4.2.1 Naming
* direct communication
* indirect communication

#### 3.4.2.2 Synchonization
* Blocking send. The sending process is blocked until the message is received by the receiving process or by the mailbox.
* Nonblocking send. The sending process sends the message and resumes operation.
* Blocking receive. The receiver blocks until a message is available.
* Nonblocking receive. The receiver retrieves either a valid message or a null.

#### 3.4.2.3 Buffering
* Zero capacity. The queue has a maximum length of zero; thus, the link
cannot have any messageswaiting in it. In this case, the sender must block
until the recipient receives the message.

* Bounded capacity. The queue has finite length n; thus, at most n messages
can reside in it. If the queue is not full when a new message is sent, the
message is placed in the queue (either the message is copied or a pointer
to the message is kept), and the sender can continue execution without
waiting. The link’s capacity is finite, however. If the link is full, the sender
must block until space is available in the queue.

* Unbounded capacity. The queue’s length is potentially infinite; thus, any
number of messages can wait in it. The sender never blocks.


## 3.5 Examples of IPC Systems
### 3.5.1 An Example: POSIX Shared Memory
### 3.5.2 An Example: Mach
### 3.5.3 An Example: Windows


## 3.6 Communicationin Client-Server Systems
### 3.6.1 Sockets
A socket is defined as an endpoint for communication.

Java provides three different types of sockets. Connection-oriented (TCP)
sockets are implemented with the Socket class. Connectionless (UDP) sockets
use the DatagramSocket class. Finally, the MulticastSocket class is a subclass
of the DatagramSocket class. A multicast socket allows data to be sent to
multiple recipients.

### 3.6.2 Remote Procedure Calls
The RPC was designed as a way to abstract the procedure-call mechanism for use between systems with network connections.
In contrast to IPC messages, the messages exchanged in RPC communication are well structured and are thus no longer just packets of data.

The RPC system hides the details that allow communication to take place by providing a stub on the client side.

##### data representation에 관한 머신에 따른 차이
To resolve differences like this, many RPC systems define amachine-independent representation of data. One such representation
is known as external data representation (XDR).

##### "at most once" vs "exactly one" 

##### RPC port 문제
그래서 랑데뷰(rendezvous) 메커니즘을 활용한다. fixed port에서 물어보면, port number를 리턴해줌.

### 3.6.3 Pipes

#### 3.5.3.1 Ordinary Pipes 
Ordinary pipes allow two processes to communicate in standard producer–
consumer fashion: the producer writes to one end of the pipe (the write-end)
and the consumer reads fromthe other end (the read-end)

Ordinary pipes on Windows systems are termed anonymous pipes, and
they behave similarly to their UNIX counterparts

#### 3.5.3.2 Named Pipes
Ordinary pipes provide a simple mechanism for allowing a pair of processes
to communicate. However, ordinary pipes exist only while the processes are
communicating with one another.
Named pipes provide a much more powerful communication tool. Communication
can be bidirectional, and no parent–child relationship is required.
Once a named pipe is established, several processes can use it for communication.

FIFO는 양방향 통신을 허용하지만 반이중 전송만 허용됩니다.
Windows에서의 Named pipe는 커뮤네이션 메커니즘도 풍부하고, 전이중 통신을 허용합니다.

## 3.7 Summary



