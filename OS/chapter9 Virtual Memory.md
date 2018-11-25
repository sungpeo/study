# Chapter9. Virtual Memory

Virtual memory는 process가 온전히 memory에 있는 것이 아니더라도, 수행시킬 수 있도록 해준다. 주요 이점 중 하나가 물리 메모리보다 큰 프로그램을 허용할 수 있다는 것이다. 이 테크닉을 통해 개발자들은 메모리 제한에 대한 걱정을 덜어준다. Virtual memory는 또한 프로세스들이 파일을 공유 및 shared memory 구현을 쉽게 할 수 있도록 한다
그러나 virtual memory를 구현하는 것이 쉽지 않으며, 무분별하게 사용할 경우 심각한 성능 저하를 일으킬 수 있다. 이번 챕터에서는 가상 메모리에서 paging 사용과 복잡도 및 비용에 대해 알아보도록 하자.

## Chapter Objectives

* To describe the benefits of a virtual memory system.
* To explain the concepts of demand paging, page-replacement algorithms, and allocation of page frames.
* To discuss the principles of the working-set model.
* To examine the relationship between shared memory and memory-mapped files.
* To explore how kernel memory is managed.

## 9.1 Background

one basic requirement: The instructions being executed must be in physical memory.

이 요구사항을 만족시키는 첫번째 접근 방법은 모든 logical address space를 물리 메모리에 위치시키는 것이다. Dynamic loading을 통해 이 방법을 쓸 순 있으나, 일반적으로 특별한 주의사항과 개발자의 추가적인 작업이 필요하다.

instructions를 실행시키기 위해서 물리 메모리에 있어야한다는 요구사항은 필요하고 맞는 말 같아 보인다. 사실 보면, 많은 경우에 프로그램 전체가 물리 메모리에 있을 필요는 없다.
예를 들면,

* 비일상적인 에러 상태를 처리하는 코드를 포함하는 프로그램이 있다. 거의 일어나지 않는 에러일땐, 해당 코드가 거의 실행되는 경우가 없다.
* array, list들이 실제 사용하는 메모리보다 크게 할당된 경우를 들 수 있다.
* 거의 사용되지 않는 특정 옵션이나 기능들이 있을 수 있다. 

 일부만을 메모리에 올려 프로그램을 실행시킬 수 있다면 많은 이점을 가져다줄 것이다.

* 프로그램이 더이상 물리 메모리 크기로 한정되지 않는다. 사용자는 매우 큰 가상 주소 공간을 사용함으로써 프로그래밍을 간단히 할 수 있다.
* 각각의 사용자 프로그램이 물리 메모리를 적게 사용하므로, 더 많은 프로그램들이 동시에 처리될 수 있다. CPU의 사용률과 throughput은 증가하지만 response time이나 trunaround time은 변하지 않는다.
* 사용자 프로그램을 메모리에 올리거나 swap하는 I/O가 적어지므로, 각각의 프로그램을 더 빠르게 수행될 것이다.

**Virtual memory**는 사용자가 물리 메모리를 인지하는 방식인 logical memory의 분리(separation)를 수반한다. 이 분리를 통해, 사용가능한 물리 메모리가 적은 상황에서도 프로그래머에게 매우 큰 가상 메모리를 사용할 수 있다. 가상 메모리는 프로그래밍을 훨씬 쉽게 만든다. 개발자가 사용 가능한 물리 메모리에 대한 고민 없이, 프로그램의 주요 로직에만 집중할 수 있도록 해주기 때문이다.

프로세스의 **virtual address space**은 프로세스가 메모리에 저장되는 방법에 대한 논리적 (또는 가상의) 관점을 나타낸다. 프로세스는 특정 logical address (0)에서 시작하며 Figure 9.2에서 보이는 것처럼 인접한 메모리에 존재한다.
8장에서 봤듯이, 물리 메모리는 page frame들로 구성되고, 해당 physical page frame들은 근접하여 할당되지는 않을 수도 있다. memory에서 logic page와 physical page를 연결시켜주는 건 memory management unit (MMU)에 달려있다.

Figure 9.2에서 보듯이, 우리는 heap 위의 공간을 활용해서 dynamic memory allocation을 사용할 수 있다. 비슷하게 stack을 아래 공간으로 늘려서 function call들을 수행할 수도 있다. heap과 stack 사이의 큰 빈 공간이 virtual address space의 일부분이고 실제로 사용할 경우에 physical pages를 요청해서 사용할 것이다. **sparse** address spaces(holes)들도 stack이나 heap segments도 사용하거나 dynamically link libraries로 사용되기도 한다.
 또한, logical memory를 physical memory로 분리함으로써, virtual memory는 파일이나 메모리를 여러 프로세스들이 공유해서 사용하도록 할 수 있다. (section 8.5.4). 이것으로 인한 이점은 아래와 같다.

  * shared object를 virtual address space에 매핑함으로써, 시스템 라이브러리들은 여러 프로세스가 공유할 수 있다. 프로세스들은 각자의 virtual address space로 라이브러리에서 접근한다고 생각하겠지만, 라이브러리를 포함하는 physical memory의 page는 모든 프로세스들로 공유되고 있는 것이다. 일반적으로 라이브러리 매핑은 read-only로 매핑된다.

  * 비슷하게 프로세스들은 메모리를 공유할 수 있다. 3장을 떠올려보면 공유 메모리가 2개 이상의 프로세스들이 커뮤니케이션할 수 있는 통로가 되었다. virtual memory는 하나의 프로세서그ㅏ 메모리 region을 만들고 그것을 다른 프로세스들과 공유할 수 있게 한다. 이 region을 공유하는 프로세스들은 자신의 virtual address space의 일부로 간주하고 사용한다. 그러나 실제 physical pages들은 공유되고 있는 것이다. (Figure 9.3)

  * pages는 프로세스가 fork system call을 통해 생성된 경우에도 공유해서 사용된다. 

  virtual memory에 대해 더 알아볼 것인데, 먼저 implementin부터 살펴보자.

## 9.2 Demand Paging

디스크로부터 메모리에 execuetable program을 로딩하는 방법을 생각해보자. 첫번째로는 한번에 모든 프로그램을 physical memory에 올리는 것이다. 처음부터 모든 프로그램이 필요하지 않을 수 있다. 사용자가 선택 가능한 옵션이 있을 수 있는데, 이 방법은 모든 옵션을 로딩하게 된다. 
다른 대안으로 필요한 만큼 로딩하는 방법이 있다. **demand paging**이라고 알려져 있고, virtual memory system에서 가장 흔히 사용되는 방법이다. access 하지 않는 pages는 physical memory에 로딩될 일이 없는 것이다. 

demand-paging system은 process가 secondary memory(usally a disk)에 있을 때의 paging system with swapping과 유사하다. 프로세스를 실행시코자가 할때 메모리에 swap을 하게 되는데, 전체 프로세스를 swap하기보다는 **lazy swapper**를 사용한다. lazy swapper는 page가 필요해지기 전까지는 절대 page를 swap하지 않는다. demand-paging system의 관점에서는 "swapper"라는 용어 사용이 기술적으로 맞지 않다. 보통 swapper는 프로세스 전체를 조작하는 반면에, **page**는 프로세스의 개별 page에 연관되어 있으므로, depand paging과 연관시켜 볼 때는 "pager"라 하는 것이 올바른 표현이다.
  
### 9.2.1 Basic Concepts

process가 swap in되었을 때, the pager는 어떤 pages들이 사용될 것인지 추측한다. 그래서 모든 프로세스를 swapping하지 않고, 그 pages만 memory로 불러온다. 사용하지 않을 page를 메모리로 읽는 작업을 피함으로써, swap time과 physical memory 사용량을 줄인다.
이 방식을 사용하면 메모리에있는 페이지와 디스크에있는 페이지를 구별 할 수있는 하드웨어 지원이 필요하다. Section 8.5.3에 나왔던 valid-invalid bit를 여기서 사용할 수 있다. 그러나 이번엔 이 bit가 "valid"로 세팅되었을 때 의미는 연관 page가 legal이면서 in memory라는 뜻이다. 그리고 "invalid"는 해당 page가 valid하지 않거나 (즉, 프로세서의 logical address space에 있지 않다.), valid하긴 하나 디스크에 있다는 뜻이다. = 메모리에 있는 page에 대한 page-table entry는 평소처럼 세팅되지만, 현재 메모리에 있지 않은 page에 대한 entry는 invalid로 마킹되거나 disk상의 page 주소를 포함한다.
page를 invalid로 마킹하는 건 page에 access 시도만 없다면 상관 없다. (no effect). 필요한 page들만 제대로 예측해서 프로세스가 수행된다면 모든 페이지를 올려놓고 하는 것처럼 정확히 수행될 것이다. **memory resident** page에 접근하는 것은 물론 프로세스 수행이 정상적으로 진행된다.
그런데 만약 메모리로 가져오지 않은 page에 대한 access를 시도하면 어떤 일이 벌어질까? invalid 마크된 page로의 access는 **page fault**를 유발한다. Paging hardware는 page table을 통해 address를 번역하는 과정에서 invalid bit가 세팅되어 있따는 것을 알아차릴 것이고, operating system에 trap을 발생시킬 것이다. 이 trap은 필요한 page를 memory로 가져오는 것에 대한 실패의 결과이다. 이 page fault를 다루는 과정은 아래와 같다. (Figure 9.6)

1. 이 프로세스가 reference가 valid/invalid memory access인지 결정하기 위해,internal table(process control block과 함께 가지고 있는)을 먼저 체크한다.
2. reference가 invalid라면, 프로세스를 종료한다. valid이긴 하나, 아직 page로 가져오지 않았다면, we not page it in.
3. We find a free frame (by taking one from the free-frame list, for example).
4. 필요한 page를 새롭게 할당된 frame에 읽어들이도록 disk operation을 스케줄한다.
5. disk read가 완료되면, 프로세스와 가지고 있는 internal table을 수정해서 이제 메모리에 들어온 page를 나타내도록 한다.
6. trap으로 interrupted 걸린 intrcution을 재시작한다. 프로세스는 문제의 page에 원래부터 메모리에 있던 것처럼 접근할 수 있다.

극단적인 경우 메모리에 page가 전혀 없는 process를 시작시킬 수도 있다. operating system이 process의 첫번째 instruction에 대해 instruction pointer로 set을 했는데, which in on a non-memory-resident page라면, 그 프로세스는 즉시 faults for the page가 발생한다. 이 page를 memory로 옮기고 나면, 프로세스는 continues to execute, faulting as necessary until every page that it needs is in memory. 다 가져오고 나면, 더이상 fault 없이 수행될 수 있다. 이 방법이 **pure demand paging**이며, 필요할 때까지는 page를 메모리에 절대 올리지 않는다.
이이론적으로 some program은 instruction execution마다 새로운 메모리 페이지에 대한 접근이 필요하고, 매번 여러개의 page를 요구할 수 있다. 이러면 아마 성능이 구릴 것이다. 다행히도 실행 중인 프로로세스를 분석해보면 이런 상황은 여간해선 일어나지 않는 것을 알 수 있다. 프로그램들은 **locality of reference**를 가지려는 경향이 있다. Section9.6.1에 묘사될 내용인데, 어쨋든 그 결과 demand pagin으로부터 쓸만한 성능이 나오게 된다. 

demand paging을 지원하는 hardware는 paging과 swapping을 지원하는 것과 동일하다.
 * **Page table.** This table has the ability to mark an entry invalid through a valid–invalid bit or a special value of protection bits.

 * **Secondary memory.** This memory holds those pages that are not present in main memory. The secondary memory is usually a high-speed disk. It is known as the swap device, and the section of disk used for this purpose is known as swap space. Swap-space allocation is discussed in Chapter 10.

demand paging을 위한 중요한 requirement는 어떤 instruction이든 page fault 이후에 재시작할 수 있어야 한다는 것이다. page fault가 발생했을 때 인터럽트된 프로세스의 상태(registers, condition code, instruction counter)를 저장하기 때문에, 필요한 page가 메모리에 새로 올라온 것을 제외하고는 완전히 같은 place와 상태로 프로세스를 재시작시킬 수 있어야 한다. 대부분의 경우 이 요구사항은 만족시키기 쉽다. page fault는 어떤 memory reference든 발생할 수 있다. page fault가 instruction fetch에서 발생했다면, 우리는 해당 instruction을 다시 fetching함으로써 재시작할 수 있다. 만약 page fault가 operand를 fetching하는 중에 발생했다면, instruction 다시 fetch하고 decode하고 난 후, operand를 fetch해야 한다.

(ADD 예시, 마지막 step에서의 page fault로 인해 process를 처음부터 재수행)

The major difficulty arises when one instruction may modify several different locations. For example, consider the IBM System 360/370 MVC (move character) instruction, which can move up to 256 bytes from one location to another (possibly overlapping) location. If either block (source or destination) straddles a page boundary, a page fault might occur after the move is partially done. In addition, if the source and destination blocks overlap, the source block may have been modified, in which case we cannot simply restart the instruction.
This problem can be solved in two different ways. In one solution, the microcode computes and attempts to access both ends of both blocks. If a page fault is going to occur, it will happen at this step, before anything is modified. The move can then take place; we know that no page fault can occur, since all the relevant pages are in memory. The other solution uses temporary registers to hold the values of overwritten locations. If there is a page fault, all the old values are written back into memory before the trap occurs. This action restores memory to its state before the instruction was started, so that the instruction can be repeated.
이것이 demand paging을 위해 paging을 추가하여 생기는 유일한 아키텍처 문제는 아니지만, 그것(?)은 수반된 몇가지 어려움을 나타낸다. 

9.2.2 Performance of Demand Paging

Demand paging은 컴퓨터 시스템의 성능에 큰 영향을 끼칠 수 있다. 왜인지 보기 위해, demand-paged memory를 위한 **effective access time**을 계산해보자. 대부분의 컴퓨터 시스템에서 memory-access time은 10~200 nanoseconds 사이로 관리된다. page faults가 없다면 효과적인 access time은 memory access time과 동일하다. 그러나 만약 page fault가 발생하면, 먼저 관련 page를 disk로부터 읽고, 필요한 word로 access해야 한다.

Let p be the probability of a page fault (0 ≤ p ≤ 1). We would expect p to be close to zero—that is, we would expect to have only a few page faults. The effective access time is then
effective access time = (1 − p) × ma + p × page fault time.

To compute the effective access time, we must know how much time is needed to service a page fault. A page fault causes the following sequence to occur:

1. Trap to the operating system.
2. Save the user registers and process state.
3. Determine that the interrupt was a page fault.
4. Check that the page reference was legal and determine the location of the page on the disk.
5. Issue a read from the disk to a free frame:
a. Wait in a queue for this device until the read request is serviced.
b. Wait for the device seek and/or latency time.
c. Begin the transfer of the page to a free frame.
6. While waiting, allocate the CPU to some other user (CPU scheduling, optional).
7. Receive an interrupt from the disk I/O subsystem (I/O completed).
8. Save the registers and process state for the other user (if step 6 is executed).
9. Determine that the interrupt was from the disk.
10. Correct the page table and other tables to show that the desired page is now in memory.
11. Wait for the CPU to be allocated to this process again.
12. Restore the user registers, process state, and new page table, and then resume the interrupted instruction.

In any case, we are faced with three major components of the page-fault service time:
1. Service the page-fault interrupt.
2. Read in the page.
3. Restart the process.

The first and third tasks can be reduced, with careful coding, to several
hundred instructions. These tasks may take from 1 to 100 microseconds each. The page-switch time, however, will probably be close to 8 milliseconds. (A typical hard disk has an average latency of 3 milliseconds, a seek of 5 milliseconds, and a transfer time of 0.05 milliseconds. Thus, the total paging time is about 8 milliseconds, including hardware and software time.) Remember also that we are looking at only the device-service time. If a queue of processes is waiting for the device, we have to add device-queueing time as we wait for the paging device to be free to service our request, increasing even more the time to swap.

With an average page-fault service time of 8 milliseconds and a memory- access time of 200 nanoseconds, the effective access time in nanoseconds is

 effective access time = (1 − p) × (200) + p (8 milliseconds)
 = (1 − p) × 200 + p × 8,000,000
 = 200 + 7,999,800 × p.

We see, then, that the effective access time is directly proportional to the **page-fault rate**. If one access out of 1,000 causes a page fault, the effective access time is 8.2 microseconds. The computer will be slowed down by a factor of 40 because of demand paging! If we want performance degradation to be less than 10 percent, we need to keep the probability of page faults at the following level:

 220 > 200 + 7,999,800 × p, 
 20 > 7,999,800 × p,
 p < 0.0000025.

That is, to keep the slowdown due to paging at a reasonable level, we can allow fewer than one memory access out of 399,990 to page-fault. In sum, it is important to keep the page-fault rate low in a demand-paging system. Otherwise, the effective access time increases, slowing process execution dramatically.

demand paging 의 다른 측면으로 swap space를 핸들링하고 사용한다는 것이다. swap space에 대한 disk I/O는 file system보다는 일반적으로 빠르다. 훨씬 큰 block 단위로 allocate을 하고, file lookups과 indirect allocation method를 사용하지 않기 때문이다. 그래서 전체 file image를 process startup할 때 swap space에 넣어두고 swap space로부터 demand pagin을 수행한다. 다른 방법(option)으로는 초기 demand pages는 file system에서 수행하고, 이후에 page를 쓸때는 swap을 사용한다. 이 접근 방법은 정말 필요한 page들만 file system에서 읽어들여 사용하고 이후로의 paging은 swap space에서 수행하도록 한다.
어떤 시스템들은 binary files을 대상으로 하는 demand paging에 사용되는 swap space의 크기를 제한하려고 시도한다. 이런 파일에 대한 demand pages는 파일시스템에서 바로 가져와서 사용된다. 그러나 page relacement가 호출되며느 이 frames은 간단히 overwrite될 수 있다. 그리고 그 page가 다시 필요하면 다시 file system에서 불러올 수 있다. 이 접근방법을 사용함으로써, file system은 backing store로써의 역할을 한다. swap space는 여전히 파일이 아닌 page를 위해 사용된다. 이 page들은 process를 위한 stack과 heap을 포함하고 있다. 이 방법은 Solaris와 BSD UNIX와 같은 여러 시스템들에서 좋은 compromise가 되고 있다.
모바일 시스템은 기본적으로 swapping을 지원하지 않는다. 대신, file system으로부터의 demand-page를 지원하며, 만약 메모리가 제약이 되면, application의 읽기 전용 page(such as code)를 회수한다. 이런 data는 나중에 필요하면 파일 시스템으로부터 demand-paged될 수 있다. iOS에서는 알수없는 memory pages들도 절대 application으로부터 회수되지 않는다.

## 9.3 Copy-on-Write

In Section 9.2, we illustrated how a process can start quickly by demand-paging in the page containing the first instruction. However, process creation using the fork() system call may initially bypass the need for demand paging by using a technique similar to page sharing (covered in Section 8.5.4). 이 테크닉은 빠른 프로세스 생성과 새로 만들어진 프로세스에 할당되어야할 신규 page들의 수를 최소화할 수 있게 한다.

fork() system call은 부모를 복제해서 자식 프로세스스를 만든다. 전통적으로 fork()는 부모 주소 공간이 복사본을 만들어 page들이 중복되도록 동작했다. 그러나 많은 자식 프로세스들이 생성 직후 exec() system call을 수행하는 것을 생각해보면, 부모의 주소 공간을 복사하는 것은 불필요할 수도 있다. 대신 우리는 **copy-on-write**라고 부모와 자식 프로세스가 초기에 같은 page를 공유하도록 동작하는 방법을 사용할 수 있다. 이 공유된 page들은 copy-on-write pages로 마킹된다. 그리고 프로세스 중 하나가 공유된 page에 write를 하려고 하면, 복사본이 생성된다. Copy-on-write is illustrated in Figures 9.7 and 9.8, which show the contents of the physical memory before and after process 1 modifies page C. Copy-on-write is a common technique used by several operating systems, including Windows XP, Linux, and Solaris.

When it is determined that a page is going to be duplicated using copy- on-write, it is important to note the location from which the free page will be allocated. Many operating systems provide a **pool** of free pages for such requests. These free pages are typically allocated when the stack or heap for a process must expand or when there are copy-on-write pages to be managed.
Operating systems typically allocate these pages using a technique known as **zero-fill-on-demand**. Zero-fill-on-demand pages have been zeroed-out before being allocated, thus erasing the previous contents.

Several versions of UNIX (including Solaris and Linux) provide a variation of the fork() system call — vfork() (for **virtual memory fork**) — that operates differently from fork() with copy-on-write. With vfork(), the parent process is suspended, and the child process uses the address space of the parent. Because vfork() does not use copy-on-write, if the child process changes any pages of the parent’s address space, the altered pages will be visible to the parent once it resumes. Therefore, vfork() must be used with caution to ensure that the child process does not modify the address space of the parent. vfork() is intended to be used when the child process calls exec() immediately after creation. Because no copying of pages takes place, vfork() is an extremely efficient method of process creation and is sometimes used to implement UNIX command-line shell interfaces.

## 9.4 Page Relacement

page-fault rate에 대한 앞선 논의를 보면, 우리는 개별 page faults는 최대 한번, 처음 referenced 될 때 발생한다고 짐작했다. 그러나 이 표현은 정확하지 않다. 열개의 페이지로 이뤄진 하나의 프로세스가 정확히 그 중 절반을 쓴다고 하면, 절대 사용되지 않을 5개의 페이지에 대한 I/O load를 절약할 수 있다. 우리는 또한 멀티프로그래밍 정도를 증가시킬 수 있다. 만약 40개의 frame이 있따면, 우리는 8개의 프로세스를 수행할 수 있다. page 10개 중 5개는 절대 쓰이지 않기 때문에.

이런 방법을 통해 우리는 메모리를 **over-allocating**한다. 만약 같은 종류의 프로세스를 6개를 돌렸다고 치자. 그러나 어떤 특정 data set으로 인해 page 10개가 전부 필요하게 될 수 있다. 그러면 사용가능한 frame은 40개 뿐이지만, 60개를 필요로 하게 된다.

더 나아가 시스템 메로리는 프로그램 pages를 갖고 있기만 한 것은 아니다. I/O를 위한 buffer도 상당한 메모리를 소모한다. 이런 사용은 memory-placement algorithm의 부담을 증가시킬 수 있다. Deciding how much memory to allocate to I/O and how much to program pages is a significant challenge. Some systems allocate a fixed percentage of memory for I/O buffers, whereas others allow both user processes and the I/O subsystem to compete for all system memory.

Over-allocation of memory manifests itself as follows. While a user process is executing, a page fault occurs. The operating system determines where the desired page is residing on the disk but then finds that there are no free frames on the free-frame list; all memory is in use (Figure 9.9).
The operating system has several options at this point. It could terminate the user process. However, demand paging is the operating system’s attempt to improve the computer system’s utilization and throughput. Users should not be aware that their processes are running on a paged system—paging should be logically transparent to the user. So this option is not the best choice.
The operating system could instead swap out a process, freeing all its frames and reducing the level of multiprogramming. This option is a good one in certain circumstances, and we consider it further in Section 9.6. Here, we discuss the most common solution: **page replacement.**

### 9.4.1 Basic Page Relacement


