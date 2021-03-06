# Chapter8. Main Memory

The memory-management algorithms vary from a primitive bare-machine approach to paging and segmentation strategies. Selection of a memory-management method for a specific system depends on many factors, especially on the hardware design of the system.

## Chapter Objectives

메모리 관리 알고리즘은 primitive bare-machine 접근방법에서부터 paging and segmentation 전략까지 다양하다. 시스템을 위한 메모리 관리 선택은 여러 factor의 영향을 받는데, 특별히 하드웨어 시스템 설계에 따른 영향을 받는다.

* To provide a detailed description of various ways of organizing memory hardware
* To explore various techniques of allocating memory to processes.
* To discuss in detail how paging works in contemporary computer systems.

## 8.1 Background

chapter1에서 살펴본 것처럼 메모리는 현대 컴퓨터 시스템의 동작에 주요한 역할을 한다. 메모리는 큰 바이트들의 배열을 가지며, 각각에 대한 주소값을 가지고 있다. CPU는 program counter의 값에 따라 메모리에서 명령(instructions)을 가져온다. 이 명령어들은 특정 메모리 주소에 대한 추가적인 loading과 storing을 발생시킬 수 있다.
예를 들어, 전형적인 instruction-execution 사이클이 메모리로부터 처음 an instruction을 가져왔다. 그 instruction은 디코딩된 후에 또 다른 operands를 메모리로부터 가져오도록 만들 수 있다. 그 instruction이 그 operands까지 수행된 후에 결과는 memory에 다시 저장될 것이다. The memory unit sees only a stream of memory addresses; it does not know how they are generated (by the instruction counter, indexing, indirection,literal addresses, and so on) or what they are for (instructions or data). 따라서 우리는 program이 어떻게 메모리 주소를 만들어내는지는 무시해도 된다. 우리는 오직 수행 중인 프로그램이 메모리 주소를 어떤 순서로 만들어내는 지 관심 있을 뿐이다.
우리는 메모리 관리와 관련된 몇 가지 문제를 다뤄보려고 한다: basic hardware, the binding of symbolic memory addresses to actual physical addresses, and the distinction between logical and physical addresses. 그리고 dynamic linking과 shared libararies에 대해 알아보며 섹션을 마무리할 것이다.

### 8.1.1 Basic Hardware

프로세서에 설치된(built into) 메인 메모리와 레지스터는 CPU가 직접 접근하는 일반 목적의 스토리지일 뿐이다. arguments로써 메모리 주소를 가져오는 기계 명령(machine instructions)은 있으나, disk 주소를 가져오는 것은 없다. 그러므로 수행 중인 명령이든 data건, 반드시 이런 direct-access storage devices들 중에 하나 안에 존재해야 한다.
CPU 내부에 있는 레지스터는 일반적으로 하나의 CPU clock cycle 내에서 접근 가능하다. 대부분의 CPU들은 instruction을 decode할 수 있고, clock tick마다 하나 이상의 operation을 레지스터 위(on register contents)에서 수행할 수 있다. The same cannot be said of main memory, which is accessed via a transaction on the memory bus. memory access를 완료하는 것은 많은 cycle of the CPU clock이 걸릴 수 있다. 이런 경우에 프로세서는 stall (멈춤)이 필요하다. 왜냐하면 프로세서가 수행중인 instruction을 완료시키위에 필요한 data가 없기 때문이다. 이 상황은 메모리 접근의 frequency 때문에 intolerable하다. 해결법은 CPU와 main memory 사이에 fast memory를 추가하는 것이다. 전형적으로 fast access를 위한 CPU chip 위에 위치한다. (cache) CPU에 내장 된 캐시를 관리하기 위해 하드웨어는 운영 체제 제어없이 메모리 액세스 속도를 자동으로 높인다.

각각의 프로세스는  독립된 메모리 공간을 가지며 특정 프로세스만 접근할 수 있는 합법적인(Legal) 메모리 주소 영역을 설정하고, 프로세스가 합법적인 영역만을 접근하도록 하는 것이 필요하다. 이를 기준(Base)과 상한(Limit)이라고 불리는 두 개의 레지스터들을 사용하여 보호 기법을 제공한다.
레지스터로 user mode 에서의 생성된 주소값을 모두 비교하여, 다른 사용자의 memory나 OS memory로의 접근을 fatal error로 처리하여 막는다. 이 구성표(scheme)는 사용자 프로그램이 (실수로 또는 의도적으로) 운영 체제 또는 다른 사용자의 코드 또는 데이터 구조를 수정하는 것을 방지한다.

### 8.1.2 Adress Binding

Usually, a program resides on a disk as a binary executable file. To be executed, the program must be brought into memory and placed within a process. Depending on the memory management in use, the process may be moved between disk and memory during its execution. The processes on the disk that are waiting to be brought into memory for execution form the **input queue**.

In most cases, a user program goes through several steps—some of which may be optional—before being executed (Figure 8.3). Addresses may be represented in different ways during these steps. Addresses in the source program are generally symbolic (such as the variable count). A compiler typically binds these symbolic addresses to relocatable addresses (such as “14 bytes from the beginning of this module”). The linkage editor or loader in turn binds the relocatable addresses to absolute addresses (such as 74014). Each binding is a mapping from one address space to another.

* **Compile time.** 컴파일 타임에 프로세스가 위치할 메모리 주소를 알고 있다면, absolute code가 생성될 것이다. 나중에 starting location이 변하면, 이 코드는 recompile이 필요하다.

* **Load time.** 컴파일 타임에 모른다면 컴파일러는 relocatable code를 생성할 것이다. 이 경우 binding은 load time까지 미뤄진다.

* **Execution time.** 프로세스가 실행 중에 옮겨질 수 있다면, binding은 런타임까지 연기되어야만 한다. 특별한 하드웨어가 이 작업을 위해 필요한데, 8.1.3에서 다룰 것이다. Most general-purpose operating systems use this method.

### 8.1.3 Logical versus Physical Address Space

An address generated by the CPU is commonly referred to as a logical address, whereas an address seen by the memory unit—that is, the one loaded into the memory-address register of the memory—is commonly referred to as a physical address.

#### 논리 주소와 물리 주소

* Logical address – generated by the CPU; also referred to as virtual address
* Physical address – address seen by the memory unit

#### Memory-Management Unit (MMU)

* CPU가 메모리에 접근하는 것을 관리하는 컴퓨터 하드웨어 부품
* 가상 메모리 주소를 실제 메모리 주소로 변환

#### Relocation Register

* In MMU scheme, the value in the relocation register is added to every address generated by a user process at the time it is sent to memory

The user program generates only logical addresses and thinks that the process runs in locations 0 to max. However, these logical addresses must be mapped to physical addresses before they are used.

### 8.1.4 Dynamic Loading

To obtain better memory-space utilization, we can use **dynamic loading**. With dynamic loading, a routine is not loaded until it is called.

The advantage of dynamic loading is that a routine is loaded only when it is needed. This method is particularly useful when large amounts of code are needed to handle infrequently occurring cases, such as error routines. In this case, although the total program size may be large, the portion that is used may be much smaller.

Dynamic loading does not require special support from the operating system. It is the responsibility of the users to design their programs to take advantage of such a method. Operating systems may help the programmer, however, by providing library routines to implement dynamic loading.

### 8.1.5 Dynamic Linking and Shared Libraries

**Dynamically linked libraries** are system libraries that are linked to user programs when the programs are run (refer back to Figure 8.3). Some operating systems support only **static linking**, in which system libraries are treated like any other object module and are combined by the loader into the binary program image.

Here, though, linking, rather than loading, is postponed until execution time. This feature is usually used with system libraries, such as language subroutine libraries.

With dynamic linking, a **stub** is included in the image for each library routine reference. The stub is a small piece of code that indicates how to locate the appropriate memory-resident library routine or how to load the library if the routine is not already present.

Without dynamic linking, all such programs would need to be relinked to gain access to the new library. So that programs will not accidentally execute new, incompatible versions of libraries, version information is included in both the program and the library. Thus, only programs that are compiled with the new library version are affected by any incompatible changes incorporated in it. Other programs linked before the new library was installed will continue using the older library. This system is also known as **shared libraries**.

Unlike dynamic loading, dynamic linking and shared libraries generally require help from the operating system. If the processes in memory are protected from one another, then the operating system is the only entity that can check to see whether the needed routine is in another process’s memory space or that can allow multiple processes to access the same memory addresses. We elaborate on this concept when we discuss paging in Section 8.5.4.

## 8.2 Swapping

process가 수행되려면 반드시 메모리 내에 있어야한다. 그러나 임시로 backing store로 swapp 될 수 있고, 다시 수행되기 위해 memory에 다시 올려서 사용된다. total physical address space보다 큰 크기를 사용해서 multiprogramming 정도를 높일 수 있다.

### 8.2.1 Standard Swapping

표준 swapping은 메인 메모리와 backing store 사이에서 프로세스를 이동시킴으로 동작한다. backing store는 일반적으로 fast disk가 사용된다. all memory images의 복사본을 담을 정도로 크기도 충분히 커야하며, 저장된 memory images로의 직접 접근을 제공해야한다. 시스템을 **ready queue**를 통해 backing store나 in memory에 있는 프로세스면서 ready to run인 모든 프로세스를 관리한다.

Notice that the major part of the swap time is transfer time. The total transfer time is directly proportional to the amount of memory swapped.

Swapping is constrained by other factors as well. If we want to swap a process, we must be sure that it is completely idle. 만약 I/O가 비동기로 I/O buffer를 위한 user memory에 접근 중이라면, 해당 프로세스는 swapped 될 수 없다.
If we were to swap out process P1 and swap in process P2, the I/O operation might then attempt to use memory that now belongs to process P2. There are two main solutions to this problem: never swap a process with pending I/O, or execute I/O operations only into operating-system buffers.
**double buffering** ??

Standard swappig은 현대 OS에서는 사용되지 않는다. 그건 swapping time도 많이 들고, execution time은 너무 작아서 reasonable한 memory-management solution이 될 수 없다. 대신 modifed version으로 사용된다.

* In one common variation, swapping is normally disabled but will start if the amount of free memory (unused memory available for the operating system or processes to use) falls below a threshold amount.

* Another variation involves swapping portions of processes—rather than entire processes—to decrease swap time.

Typically, these modified forms of swapping work in conjunction with virtual memory, which we cover in Chapter 9.

### 8.2.2 Swapping on Mobile Systems

모바일 환경에서는 typically swapping을 지원하지 않는다. 모바일 기기는 비교적 용량이 큰 hard disk가 아닌 flash memory를 쓰기 때문에, 모바일 OS designer들은 swapping을 피한다. 또한 flash memory는 쓰기 제한 있기 때문이기도 하다.
swapping을 쓰는 대신에, free memory가 일정 threshold 아래로 떨어지면, Apple의 iOS 같은 경우엔 application들이 자발적으로 메모리를 양도(reliquish)하도록 요청한다. Read-only data의 경우 메모리에서 삭제되고 수정된 data들은 남는다. 그러나 충분한 메모리를 만들어내지 못하면 어떤 application이든 Operating system에 의해 꺼질 수도 있다. Android도 iOS와 마찬가지로 swapping은 지원하지 않고, free memory가 부족하면 process를 죽일 수도 있다. 다만 **application state**를 flash memory가 써놓고 빠른 재시작이 가능하도록 한다.

## 8.3 Contiguous Memory Allocation

메인 메모리는 operating system과 user process들이 사용한 가능해야 한다. 그러므로 우리는 가능한한 효율적으로 main memory를 사용할 필요가 있다. 이번 장에선 예전 방법 중 하나인 contiguous(인접한) memory allocation을 설명하겠다.

The memory is usually divided into two partitions: one for the resident operating system and one for the user processes.

We usually want several user processes to reside in memory at the same time. We therefore need to consider how to allocate available memory to the processes that are in the input queue waiting to be brought into memory. In contiguous memory allocation, each process is contained in a single section of memory that is contiguous to the section containing the next process.

각 프로세스는 다음 프로세스를 포함하는 섹션에 인접한 메모리의 단일 섹션에 포함됩니다 ?

### 8.3.1 Memory Protection

메모리 할당에 대해 더 이야기하기 전에 memory protection에 대해 먼저 짚어보자. 앞서 이야기한 2가지 idea를 복합사용함으로써, 프로세스가 소유하지 않은 메모리로의 접근을 막을 수 있다. relocation register와 limit register를 사용함으로써 목적을 달성할 수 있다. The relocation register contains the value of the smallest physical address; the limit register contains the range of logical addresses (for example, relocation = 100040 and limit = 74600).

Such code is sometimes called **transient** operating-system code; it comes and goes as needed. Thus, using this code changes the size of the operating system during program
execution.

### 8.3.2 Memory Allocation

할당하기 위해 분할하는 가장 쉬운 방법은 fixed-size로 partition을 만드는 것이다. 각각의 partition은 정확히 하나의 프로세스만을 가질 수 있다. 그러므로, multiprogramming degree는 partition의 개수에 의해 정해진다. **multiple partition method**이라 불리는 이 방법에서 partition이 비어 있으면 input queue로부터 선택된 process가 비어 있는 partition으로 로딩된다. 프로세스가 끝나면, 파티션은 다시 사용 가능해진다. IBM OS/360 operating system (called MFT)에서 사용되던 방법인데, 지금은 안 쓰인다. 

In the variable-partition scheme, the operating system keeps a table indicating which parts of memory are available and which are occupied. Initially, all memory is available for user processes and is considered one large block of available memory, a hole. Eventually, as you will see, memory contains a set of holes of various sizes.

Memory is allocated to processes until, finally, the memory requirements of the next process cannot be satisfied—that is, no available block of memory (or hole) is large enough to hold that process. The operating system can then wait until a large enough block is available, or it can skip down the input queue to see whether the smaller memory requirements of some other process can be met.
If the new hole is adjacent to other holes, these adjacent holes are merged to form one larger hole. At this point, the system may need to check whether there are processes waiting for memory and whether this newly freed and recombined memory could satisfy the demands of any of these waiting processes.
This procedure is a particular instance of the general **dynamic storage allocation problem**, which concerns how to satisfy a request of size n from a list of free holes. There are many solutions to this problem. The first-fit, best-fit, and worst-fit strategies are the ones most commonly used to select a free hole from the set of available holes.

* **First fit.** Allocate the first hole that is big enough. Searching can start either at the beginning of the set of holes or at the location where the previous first-fit search ended.We can stop searching as soon as we find a free hole that is large enough.

* **Best fit.** Allocate the smallest hole that is big enough. We must search the entire list, unless the list is ordered by size. This strategy produces the smallest leftover hole.

* **Worst fit.** Allocate the largest hole. Again, we must search the entire list, unless it is sorted by size. This strategy produces the largest leftover hole, which may be more useful than the smaller leftover hole from a best-fit approach.

### 8.3.3 Fragmentation

Both the first-fit and best-fit strategies for memory allocation suffer from **external fragmentation**. process들이 작업을 끝내더라도 free memory space들이 조각들로 쪼개져 있어서, (not contiguous) 가용할 수 없게 된다.

Statistical analysis of first fit, for instance, reveals that, even with some optimization, given N allocated blocks, another 0.5 N blocks will be lost to fragmentation. That is, one-third of memory may be unusable! This property is known as the **50-percent rule**.

너무 작은 hole의 경우, hole의 크기보다 tracking하는 드는 overhead가 더 크다. 이런 문제가 생기지 않도록 physical memory를 fixed-sized block으로 쪼개서 unit별로 메모리를 할당해서 쓰기도 한다. 이렇게 되면 할당한 메모리에 비해 필요한 메모리는 작아서, **internal fragmentation**이 발생한다.

external fragmentation에 대한 솔루션 중에 하나로 **compaction**이 있다. The goal is to shuffle the memory contents so as to place all free memory together in one large block. Compaction is not always possible, however. If relocation is static and is done at assembly or load time, compaction cannot be done. It is possible only if relocation is dynamic and is done at execution time. If addresses are relocated dynamically, relocation requires only moving the program and data and then changing the base register to reflect the new base address. When compaction is possible, we must determine its cost. The simplest compaction algorithm is to move all processes toward one end of memory; all holes move in the other direction, producing one large hole of available memory. This scheme can be expensive.

Another possible solution to the external-fragmentation problem is to permit the logical address space of the processes to be noncontiguous, thus allowing a process to be allocated physical memory wherever such memory is available. Two complementary techniques achieve this solution: segmentation (Section 8.4) and paging (Section 8.5). These techniques can also be combined.

Fragmentation is a general problem in computing that can occur wherever we must manage blocks of data. We discuss the topic further in the storage management chapters (Chapters 10 through and 12).

## 8.4 Segmentation

What if the hardware could provide a memory mechanism that mapped the programmer’s view to the actual physical memory? The system would have more freedom to manage memory, while the programmer would have a more natural programming environment. Segmentation provides such a mechanism.

### 8.4.1 Basic Method

프로그래머는 메모리 내부의 모듈이나 data element를 이름으로 구분하지 주소로 생각하지 않는다.
Segmentation is a memory-management scheme that supports this programmer view of memory. A logical address space is a collection of segments. Each segment has a name and a length.

Libraries that are linked in during compile time might be assigned separate segments. The loader would take all these segments and assign them segment numbers.

### 8.4.2 Segmentation Hardware

This mapping is effected by a **segment table**. Each entry in the segment table has a **segment base** and a **segment limit**. The segment table is thus essentially an array of base–limit register pairs.

## 8.5 Paging

Segmentation permits the physical address space of a process to be noncontiguous. Paging is another memory-management scheme that offers this advantage. paging은 외부 파편화를 피하고 compaction이 필요 없다. 
반면에 segmentation은 그렇지 않다. It also solves the considerable problem of fitting memory chunks of varying sizes onto the backing store. Most memory-management schemes used before the introduction of paging suffered from this problem. The problem arises because, when code fragments or data residing in main memory need to be swapped out, space must be found on the backing store. The backing store has the same fragmentation problems discussed in connection with main memory, but access is much slower, so compaction is impossible. Because of its advantages over earlier methods, paging in its various forms is used in most operating systems, from those for mainframes through those for smartphones. Paging is implemented through cooperation between the operating system and the computer hardware.

### 8.5.1 Basic Method

The basic method for implementing paging involves breaking physical memory into fixed-sized blocks called **frames** and breaking logical memory into blocks of the same size called **pages**.

The page size (like the frame size) is defined by the hardware. The size of a page is a power of 2, varying between 512 bytes and 1 GB per page, depending on the computer architecture.
If the size of the logical address space is 2m, and a page size is 2n bytes, then the high-order m− n bits of a logical address designate the page number, and the n low-order bits designate the page offset

page table : pages -> frames

If process size is independent of page size,weexpect internal fragmentation to average one-half page per process. This consideration suggests that small page sizes are desirable. However, overhead is involved in each page-table entry, and this overhead is reduced as the size of the pages increases. Also, disk I/O is more efficient when the amount data being transferred is larger (Chapter 10). Generally, page sizes have grown over time as processes, data sets, and main memory have become larger. Today, pages typically are between 4 KB and 8 KB in size, and some systems support even larger page sizes.

An important aspect of paging is the clear separation between the programmer’s view ofmemory and the actual physical memory. The programmer views memory as one single space, containing only this one program. In fact, the user program is scattered throughout physical memory, which also holds other programs.

Since the operating system is managing physical memory, it must be aware of the allocation details of physical memory—which frames are allocated, which frames are available, how many total frames there are, and so on. This information is generally kept in a data structure called a **frame table**.

The operating system maintains a copy of the page table for each process, just as it maintains a copy of the instruction counter and register contents. This copy is used to translate logical addresses to physical addresses whenever the operating system must map a logical address to a physical address manually. It is also used by the CPU dispatcher to define the hardware page table when a process is to be allocated the CPU. Paging therefore increases the context-switch time.

### 8.5.2 Hardware Support

Each operating system has it own methods for storing page tables. 어떤 건 프로세스마다 하나의 page table을 할당하기도 한다. 페이지 테이블에 대한 포인터는 프로세스 제어 블록의 다른 레지스터 값 (명령 카운터와 같은)과 함께 저장된다. dispatcher가 process를 시작시킬 때, user register를 reload하고, define the correct hardware page-table values fromthe stored user page table. 다른 operating system은 process의 context-switched overhead를 줄이기 위해, one or at most a few page tables만을 제공하기도 한다.
page table의 하드웨어 구현은 several ways가 있다. 가장 간단한 방법으로는 dedicated registers의 set으로 page table을 구현하는 방법이다. 이 registers들은 paging-adress translation의 efficient하게 하기 위해 매우 빠른 속도로 처리되는 로직으로 만들어져야 한다. 모든 메모리 access는 paging map을 통해야 하므로, efficiency가 major consideration이다. CPU dispatcher는 these registers를 다른 register들과 마찬가지 방법으로 reload한다. page-table의 register를 load하거나 modify하는 명령은 당연히 privileged이고, operating system만이 memory map을 변경할 수 있다.

The use of registers for the page table is satisfactory if the page table is reasonably small (for example, 256 entries). 그러나 가장 최근 컴퓨터들은 매우 큰(ex. 1 million entries) page table을 허용한다. 이런 machine들은 fast register로 page table을 구현하는 건 불가능하다. Rather, the page table is kept in main memory, and a page-table base register (PTBR) points to the page table. Changing page tables requires changing only this one register, substantially reducing context-switch time.
이 접근법의 문제는 user memory location에 접근하는 데 걸리는 시간이다. memory location i에 접근하려면 먼저 PTBR에서 page table을 살펴보고, 거기서 frame number와 page offset을 통해 actual address에 접근해야 한다. With this scheme, two memory accesses are needed to access a byte (one for the page-table entry, one for the byte). We might as well resort to swapping!

**translation look-aside buffer(TLB)**
The TLB is used with page tables in the following way. The TLB contains only a few of the page-table entries. When a logical address is generated by the CPU, its page number is presented to the TLB. If the page number is found, its frame number is immediately available and is used to access memory. As just mentioned, these steps are executed as part of the instruction pipeline within the CPU, adding no performance penalty compared with a system that does not implement paging.

TLB miss라고 해서 page number가 없으면, memory reference page table에서 값을 가져오고, 그 값으로 TLB에 추가로 저장한다. 이때 TLB에 대한 replacement 전략은 LRU, round robin 등이 있다. 더 나아가 some TLB들은 주요 kernel code 같은 것들 TLB entry에서 지워지지 않도록 박아 놓고 사용할 수 있도록 지원한다. (**wired down**)

Some TLBs store **address-space identifiers (ASIDs)** in each TLB entry.
When the TLB attempts to resolve virtual page numbers, it ensures that the ASID for the currently running process matches the ASID associated with the virtual page. If the ASIDs do not match, the attempt is treated as a TLB miss. In addition to providing address-space protection, an ASID allows the TLB to contain entries for several different processes simultaneously. If the TLB does not support separate ASIDs, then every time a new page table is selected (for instance, with each context switch), the TLB must be flushed (or erased) to ensure that the next executing process does not use the wrong translation information.

The percentage of times that the page number of interest is found in the TLB is called the hit ratio. To find the **effective memory-access time**, we weight the case by its probability (hit ratio).

TLBs are a hardware feature and therefore would seem to be of little concern to operating systems and their designers. But the designer needs to understand the function and features of TLBs, which vary by hardware platform.

### 8.5.3 Protection

Memory protection in a paged environment is accomplished by protection bits associated with each frame. Normally, these bits are kept in the page table.
One bit can define a page to be read–write or read-only.
An attempt to write to a read-only page causes a hardware trap to the operating system (or memory-protection violation).
We can easily expand this approach to provide a finer level of protection.
One additional bit is generally attached to each entry in the page table: a valid–invalid bit. When this bit is set to valid, the associated page is in the process’s logical address space and is thus a legal (or valid) page. When the bit is set toinvalid, the page is not in the process’s logical address space.

Some systems provide hardware, in the form of a page-table length register (PTLR), to indicate the size of the page table. This value is checked against every logical address to verify that the address is in the valid range for the process. Failure of this test causes an error trap to the operating
system.

### 8.5.4 Shared Pages

An advantage of paging is the possibility of sharing common code.
If the code is **reentrant code** (or **pure code**), however, it can be shared, as shown in Figure 8.16. Reentrant code is non-self-modifying code: it never changes during execution. Thus, two or more processes can execute the same code at the same time.
Each process has its own copy of registers and data storage to hold the data for the process’s execution. The data for two different processes will, of course, be different.
editor 관련 코드는 복사본 하나만 물리 메모리에서 유지하고, 각자 page table들이 동일 physical copy를 보도록 하고, 각각의 data space만 user별로 저장함으로써 눈에 띄는 절약 효과가 있다.

The sharing of memory among processes on a system is similar to the sharing of the address space of a task by threads, described in Chapter 4. Furthermore, recall that in Chapter 3we described shared memory as a method of interprocess communication. Some operating systems implement shared memory using shared pages.

## 8.6 Structure of the Page Table

In this section, we explore some of the most common techniques for structuring the page table, including hierarchical paging, hashed page tables, and inverted page tables.

### 8.6.1 Hierarchical Paging

Most modern computer systems support a large logical address space (2^32 to 2^64)

One simple solution to this problem is to divide the page table into smaller pieces.We can accomplish this division in several ways.
One way is to use a two-level paging algorithm, in which the page table itself is also paged (Figure 8.17).

The address-translation method for this architecture is shown in Figure 8.18. Because address translation works from the outer page table inward, this scheme is also known as a **forward-mapped page table**.

380p

### 8.6.2 Hashed Page Tables

A common approach for handling address spaces larger than 32 bits is to use a hashed page table, with the hash value being the virtual page number. Each entry in the hash table contains a linked list of elements that hash to the same location (to handle collisions). Each element consists of three fields: 

(1) the virtual page number,
(2) the value of themapped page frame, and
(3) a pointer to the next element in the linked list

Clustered page tables are particularly useful for sparse address spaces, where memory references are noncontiguous and scattered throughout the address space.

### 8.6.3 Inverted Page Tables

보통, 각각의 프로세스는 하나의 연관 page table을 갖는다. page table은 process가 사용하는 page당 하나의 entry를 갖는다. (또는 virtual adress 당 하나의 slot)
이 테이블 표현은 프로세스가 페이지의 가상 주소를 통해 페이지를 참조하므로 자연스러운 것이다. operating system은 이 reference를 물리 메모리 주소로 변환해야 한다. page table은 가상 주소로 정렬되어 있기 때문에, operating system은 연관된 물리 주소 entry가 어디 위치하고 있는지 계산하고 직접 value를 사용할 수 있다. 이 방법의 단점 중 하나는 각각의 pagetable이 수백개의 entries을 가지게 될 것이라는 것이다. 
이 문제를 해결하기 위해 우리는 **inverted page table**을 사용할 수 있다. 
Invertedpage tables often require that an address-space identifier (Section 8.5.2) be stored in each entry of the page table, since the table usually contains several different address spaces mapping physical memory. Storing the address-space identifier ensures that a logical page for a particular process is mapped to the corresponding physical page frame

Although this scheme decreases the amount of memory needed to store each page table, it increases the amount of time needed to search the table when a page reference occurs. Because the inverted page table is sorted by physical address, but lookups occur on virtual addresses, the whole table might need to be searched before a match is found. This search would take far too long.

### 8.6.4 Oracle SPARC Solaris

This TLB walk functionality is found on many modern CPUs. If a match is found in the TSB, the CPU copies the TSB entry into the TLB, and the memory translation completes. If no match is found in the TSB, the kernel is interrupted to search the hash table. The kernel then creates a TTE from the appropriate hash table and stores it in the TSB for automatic loading into the TLB by the CPU memory-management unit. Finally, the interrupt handler returns control to the MMU, which completes the address translation and retrieves the requested byte or word from main memory.

## 8.7 Example: Intel 32 and 64-bit Architectures

### 8.7.2 x86-64

Intel has had an interesting history of developing 64-bit architectures. Its initial entry was the IA-64 (later named Itanium) architecture, but that architecture was not widely adopted. Meanwhile, another chip manufacturer— AMD — began developing a 64-bit architecture known as x86-64 that was based on extending the existing IA-32 instruction set. The x86-64 supported much larger logical and physical address spaces, as well as several other architectural advances. Historically, AMD had often developed chips based on Intel’s architecture, but now the roles were reversed as Intel adopted AMD’s x86-64 architecture. In discussing this architecture, rather than using the commercial names AMD64 and Intel 64,we will use themore general termx86-64.

Support for a 64-bit address space yields an astonishing 264 bytes of addressable memory—a number greater than 16 quintillion (or 16 exabytes). However, even though 64-bit systems can potentially address this much memory, in practice far fewer than 64 bits are used for address representation in current designs. The x86-64 architecture currently provides a 48-bit virtual address with support for page sizes of 4 KB , 2 MB, or 1 GB using four levels of paging hierarchy. The representation of the linear address appears in Figure 8.25. Because this addressing scheme can use PAE, virtual addresses are 48 bits in size but support 52-bit physical addresses (4096 terabytes).

## 8.8 Example: ARM Architecture

Although Intel chips have dominated the personal computer market for over 30 years, chips for mobile devices such as smartphones and tablet computers often instead run on 32-bit ARM processors.

The paging system in use depends on whether a page or a section is being referenced. One-level paging is used for 1-MB and 16-MB sections; two-level paging is used for 4-KB and 16-KB pages. Address translation with the ARM MMU is shown in Figure 8.26.

The ARM architecture also supports two levels of TLBs. At the outer level are two micro TLBs—a separate TLB for data and another for instructions. The micro TLB supports ASIDs as well. At the inner level is a single main TLB. Address translation begins at the micro TLB level. In the case of a miss, the main TLB is then checked. If both TLBs yield misses, a page table walk must be performed in hardware.

## 8.9 Summary

Memory-management algorithms for multiprogrammed operating systems range from the simple single-user system approach to segmentation and paging. The most important determinant of the method used in a particular system is the hardware provided. Every memory address generated by the CPU must be checked for legality and possibly mapped to a physical address. The checking cannot be implemented (efficiently) in software. Hence, we are constrained by the hardware available

* **Hardware support**. A simple base register or a base–limit register pair is sufficient for the single- and multiple-partition schemes, whereas paging and segmentation need mapping tables to define the address map.

* **Performance**.

* **Fragmentation**. Systems with fixed-sized allocation units, such as the single-partition scheme and paging, suffer from internal fragmentation. Systems with variable-sized allocation units, such as the multiple-partition scheme and segmentation, suffer from external fragmentation.

* **Relocation**. One solution to the external-fragmentation problem is compaction. Compaction involves shifting a program in memory in such a way that the program does not notice the change.

* **Swapping**.

* **Sharing**.

* **Protection**.
