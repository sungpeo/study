# Chapter9. Virtual Memory

Virtual memory는 process가 온전히 memory에 있는 것이 아니더라도, 수행시킬 수 있도록 해준다. 주요 이점 중 하나가 물리 메모리보다 큰 프로그램을 허용할 수 있다는 것이다. 이 테크닉을 통해 개발자들의 메모리 제한에 대한 걱정을 덜어준다. Virtual memory는 또한 프로세스들이 파일 공유하는 것 및 shared memory 구현을 쉽게 할 수 있도록 한다
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

* 일반적으로 일어나지 않는 에러 상태를 처리하는 코드를 포함하는 프로그램을 생각해보자. 일어나지 않는 에러일수록 해당 처리 코드가 실행될 일이 없다.
* array, list들이 실제 사용하는 메모리보다 크게 할당된 경우를 들 수 있다.
* 거의 사용되지 않는 특정 옵션이나 기능들이 있을 수 있다.

 일부만을 메모리에 올려 프로그램을 실행시킬 수 있다면 많은 이점을 가져다줄 것이다.

* 프로그램이 더이상 물리 메모리 크기로 한정되지 않는다. 사용자는 매우 큰 가상 주소 공간을 사용함으로써 프로그래밍을 간단히 할 수 있다.
* 각각의 사용자 프로그램이 물리 메모리를 적게 사용하므로, 더 많은 프로그램들이 동시에 처리될 수 있다. CPU의 사용률과 throughput은 증가하지만 response time이나 trunaround time은 변하지 않는다.
* 사용자 프로그램을 메모리에 올리거나 swap하는 I/O가 적어지므로, 각각의 프로그램을 더 빠르게 수행될 것이다.

**Virtual memory**는 사용자가 물리 메모리를 인지하는 방식인 logical memory의 분리(separation)를 수반한다. 이 분리를 통해, 사용가능한 물리 메모리가 적은 상황에서도 프로그래머는 매우 큰 가상 메모리를 사용할 수 있다. 가상 메모리는 프로그래밍을 훨씬 쉽게 만든다. 개발자가 사용 가능한 물리 메모리에 대한 고민 없이, 프로그램의 주요 로직에만 집중할 수 있도록 해주기 때문이다.

프로세스의 **virtual address space**은 프로세스가 메모리에 저장되는 방법에 대한 논리적 (또는 가상의) 관점을 나타낸다. 프로세스는 특정 logical address (0)에서 시작하며 Figure 9.2에서 보이는 것처럼 인접한 메모리에 존재한다.
8장에서 봤듯이, 물리 메모리는 page frame들로 구성되고, 해당 physical page frame들은 근접하여 할당되지는 않을 수도 있다. memory에서 logic page와 physical page를 연결시켜주는 건 memory management unit (MMU)에 달려있다.

Figure 9.2에서 보듯이, 우리는 heap 위의 공간을 활용해서 dynamic memory allocation을 사용할 수 있다. 비슷하게 stack을 아래 공간으로 늘려서 function call들을 수행할 수도 있다. heap과 stack 사이의 큰 빈 공간이 virtual address space의 일부분이고 실제로 사용할 경우에 physical pages를 요청해서 사용할 것이다. **sparse** address spaces(holes)들도 stack이나 heap segments도 사용하거나 dynamically link libraries로 사용되기도 한다.
 또한, logical memory를 physical memory로 분리함으로써, virtual memory는 파일이나 메모리를 여러 프로세스들이 공유해서 사용하도록 할 수 있다. (section 8.5.4). 이것으로 인한 이점은 아래와 같다.

* shared object를 virtual address space에 매핑함으로써, 시스템 라이브러리들은 여러 프로세스가 공유할 수 있다. 프로세스들은 각자의 virtual address space로 라이브러리에서 접근한다고 생각하겠지만, 라이브러리를 포함하는 physical memory의 page는 모든 프로세스들로 공유되고 있는 것이다. 일반적으로 라이브러리 매핑은 read-only로 매핑된다.

* 비슷하게 프로세스들은 메모리를 공유할 수 있다. 3장을 떠올려보면 공유 메모리가 2개 이상의 프로세스들이 커뮤니케이션할 수 있는 통로가 되었다. virtual memory는 하나의 프로세스가 메모리 region을 만들고 그것을 다른 프로세스들과 공유할 수 있게 한다. 이 region을 공유하는 프로세스들은 자신의 virtual address space의 일부로 간주하고 사용한다. 그러나 실제 physical pages들은 공유되고 있는 것이다. (Figure 9.3)

* pages는 프로세스가 fork system call을 통해 생성된 경우에도 공유해서 사용된다.

virtual memory에 대해 더 알아볼 것인데, 먼저 implementin부터 살펴보자.

## 9.2 Demand Paging

디스크로부터 메모리에 execuetable program을 로딩하는 방법을 생각해보자. 첫번째로는 한번에 모든 프로그램을 physical memory에 올리는 것이다. 처음부터 모든 프로그램이 필요하지 않을 수 있다. 사용자가 선택 가능한 옵션이 있을 수 있는데, 이 방법은 모든 옵션을 로딩하게 된다.
다른 대안으로 필요한 만큼 로딩하는 방법이 있다. **demand paging**이라고 알려져 있고, virtual memory system에서 가장 흔히 사용되는 방법이다. access 하지 않는 pages는 physical memory에 로딩될 일이 없는 것이다.

demand-paging system은 process가 secondary memory(usally a disk)에 있을 때의 paging system with swapping과 유사하다. 프로세스를 실행코자 할때 메모리에 swap을 하게 되는데, 전체 프로세스를 swap하기보다는 **lazy swapper**를 사용한다. lazy swapper는 page가 필요해지기 전까지는 절대 page를 swap하지 않는다. demand-paging system의 관점에서는 "swapper"라는 용어 사용이 기술적으로 맞지 않다. 보통 swapper는 프로세스 전체를 조작하는 반면에, **page**는 프로세스의 개별 page에 연관되어 있으므로, depand paging과 연관시켜 볼 때는 "pager"라 하는 것이 올바른 표현이다.
  
### 9.2.1 Basic Concepts

process가 swap in되었을 때, the pager는 어떤 pages들이 사용될 것인지 추측한다. 그래서 모든 프로세스를 swapping하지 않고, 그 pages만 memory로 불러온다. 사용하지 않을 page를 메모리로 읽는 작업을 피함으로써, swap time과 physical memory 사용량을 줄인다.
이 방식을 사용하면 메모리에있는 페이지와 디스크에있는 페이지를 구별 할 수있는 하드웨어 지원이 필요하다. Section 8.5.3에 나왔던 valid-invalid bit를 여기서 사용할 수 있다. 그러나 이번엔 이 bit가 "valid"로 세팅되었을 때 의미는 연관 page가 legal이면서 in memory라는 뜻이다. 그리고 "invalid"는 해당 page가 valid하지 않거나 (즉, 프로세스의 logical address space에 있지 않다.), valid하긴 하나 디스크에 있다는 뜻이다. = 메모리에 있는 page에 대한 page-table entry는 평소처럼 세팅되지만, 현재 메모리에 있지 않은 page에 대한 entry는 invalid로 마킹되거나 disk상의 page 주소를 포함한다.
page를 invalid로 마킹하는 건 page에 access 시도만 없다면 상관 없다. (no effect). 필요한 page들만 제대로 예측해서 프로세스가 수행된다면 모든 페이지를 올려놓고 하는 것처럼 정확히 수행될 것이다. 그 프로세스는 **memory resident** page에 접근하는 것은 물론 프로세스 수행이 정상적으로 진행된다.
그런데 만약 메모리로 가져오지 않은 page에 대한 access를 시도하면 어떤 일이 벌어질까? invalid 마크된 page로의 access는 **page fault**를 유발한다. Paging hardware는 page table을 통해 address를 번역하는 과정에서 invalid bit가 세팅되어 있다는 것을 알아차릴 것이고, operating system에 trap을 발생시킬 것이다. 이 trap은 필요한 page를 memory로 가져오는 것에 대한 실패의 결과이다. 이 page fault를 다루는 과정은 아래와 같다. (Figure 9.6)

1. 이 프로세스가 reference가 valid/invalid memory access인지 결정하기 위해, internal table(process control block과 함께 가지고 있는)을 먼저 체크한다.
2. reference가 invalid라면, 프로세스를 종료한다. valid이긴 하나, 아직 page로 가져오지 않았다면, we now page it in.
3. We find a free frame (by taking one from the free-frame list, for example).
4. 필요한 page를 새롭게 할당된 frame에 읽어들이도록 disk operation을 스케줄한다.
5. disk read가 완료되면, 프로세스와 가지고 있는 internal table을 수정해서 이제 메모리에 들어온 page를 나타내도록 한다.
6. trap으로 interrupted 걸린 instrcution을 재시작한다. 프로세스는 문제의 page에 원래부터 메모리에 있던 것처럼 접근할 수 있다.

극단적인 경우 메모리에 page가 전혀 없는 process를 시작시킬 수도 있다. operating system이 process의 첫번째 instruction에 대해 instruction pointer로 set을 했는데, which in on a non-memory-resident page라면, 그 프로세스는 즉시 faults for the page가 발생한다. 이 page를 memory로 옮기고 나면, 프로세스는 continues to execute, faulting as necessary until every page that it needs is in memory. 다 가져오고 나면, 더이상 fault 없이 수행될 수 있다. 이 방법이 **pure demand paging**이며, 필요할 때까지는 page를 메모리에 절대 올리지 않는다.
이론적으로 some program은 instruction execution마다 새로운 메모리 페이지에 대한 접근이 필요하고, 매번 여러개의 page를 요구할 수 있다. 이러면 아마 성능이 구릴 것이다. 다행히도 실행 중인 프로로세스를 분석해보면 이런 상황은 여간해선 일어나지 않는 것을 알 수 있다. 프로그램들은 **locality of reference**를 가지려는 경향이 있다. Section9.6.1에 묘사될 내용인데, 어쨋든 그 결과 demand pagin으로부터 쓸만한 성능이 나오게 된다.

demand paging을 지원하는 hardware는 paging과 swapping을 지원하는 것과 동일하다.

* **Page table.** This table has the ability to mark an entry invalid through a valid–invalid bit or a special value of protection bits.

* **Secondary memory.** This memory holds those pages that are not present in main memory. The secondary memory is usually a high-speed disk. It is known as the swap device, and the section of disk used for this purpose is known as swap space. Swap-space allocation is discussed in Chapter 10.

demand paging을 위한 중요한 requirement는 어떤 instruction이든 page fault 이후에 재시작할 수 있어야 한다는 것이다. page fault가 발생했을 때 인터럽트된 프로세스의 상태(registers, condition code, instruction counter)를 저장하기 때문에, 필요한 page가 메모리에 새로 올라온 것을 제외하고는 완전히 같은 place와 상태로 프로세스를 재시작시킬 수 있어야 한다. 대부분의 경우 이 요구사항은 만족시키기 쉽다. page fault는 어떤 memory reference든 발생할 수 있다. page fault가 instruction fetch에서 발생했다면, 우리는 해당 instruction을 다시 fetching함으로써 재시작할 수 있다. 만약 page fault가 operand를 fetching하는 중에 발생했다면, instruction 다시 fetch하고 decode하고 난 후, operand를 fetch해야 한다.

(ADD 예시, 마지막 step에서의 page fault로 인해 process를 처음부터 재수행)

The major difficulty arises when one instruction may modify several different locations. For example, consider the IBM System 360/370 MVC (move character) instruction, which can move up to 256 bytes from one location to another (possibly overlapping) location. If either block (source or destination) straddles a page boundary, a page fault might occur after the move is partially done. In addition, if the source and destination blocks overlap, the source block may have been modified, in which case we cannot simply restart the instruction.

This problem can be solved in two different ways. In one solution, the microcode computes and attempts to access both ends of both blocks. If a page fault is going to occur, it will happen at this step, before anything is modified. The move can then take place; we know that no page fault can occur, since all the relevant pages are in memory.

The other solution uses temporary registers to hold the values of overwritten locations. If there is a page fault, all the old values are written back into memory before the trap occurs. This action restores memory to its state before the instruction was started, so that the instruction can be repeated.

이것이 demand paging을 위해 paging을 추가하여 생기는 유일한 아키텍처 문제는 아니지만, 그것(?)은 수반된 몇가지 어려움을 설명한다. Paging은 CPU와 memory 사이에 추가되고, user process에 투명한 상태여야만 한다. 그래서 사람들은 종종 pagin이 어느 시스템에든 넣을 수 있겠다고 생각한다. 이 추측이 page fault가 곧 fatal error임을 의미하는 non-demand-paging 환경에서는 사실이지만, page fault가 단지 추가적으로 메모리로 옮기는 과정인 환경에서는 사실이 아니다.

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
    * a. Wait in a queue for this device until the read request is serviced.
    * b. Wait for the device seek and/or latency time.
    * c. Begin the transfer of the page to a free frame.
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

``` calculation
effective access time
= (1 − p) × (200) + p (8 milliseconds)
= (1 − p) × 200 + p × 8,000,000
= 200 + 7,999,800 × p.
```

We see, then, that the effective access time is directly proportional to the **page-fault rate**. If one access out of 1,000 causes a page fault, the effective access time is 8.2 microseconds. The computer will be slowed down by a factor of 40 because of demand paging! If we want performance degradation to be less than 10 percent, we need to keep the probability of page faults at the following level:

``` calculation
 220 > 200 + 7,999,800 × p,
 20 > 7,999,800 × p,
 p < 0.0000025.
```

That is, to keep the slowdown due to paging at a reasonable level, we can allow fewer than one memory access out of 399,990 to page-fault. In sum, it is important to keep the page-fault rate low in a demand-paging system. Otherwise, the effective access time increases, slowing process execution dramatically.

demand paging 의 다른 측면으로 swap space를 핸들링하고 사용한다는 것이다. swap space에 대한 disk I/O는 file system보다는 일반적으로 빠르다. 훨씬 큰 block 단위로 allocate을 하고, file lookups과 indirect allocation method를 사용하지 않기 때문이다. 그래서 전체 file image를 process startup할 때 swap space에 넣어두고 swap space로부터 demand pagin을 수행한다. 다른 방법(option)으로는 초기 demand pages는 file system에서 수행하고, 이후에 page를 쓸때는 swap을 사용한다. 이 접근 방법은 정말 필요한 page들만 file system에서 읽어들여 사용하고 이후로의 paging은 swap space에서 수행하도록 한다.
어떤 시스템들은 binary files을 대상으로 하는 demand paging에 사용되는 swap space의 크기를 제한하려고 시도한다. 이런 파일에 대한 demand pages는 파일시스템에서 바로 가져와서 사용된다. 그러나 page relacement가 호출되면 이 frames은 간단히 overwrite될 수 있다. 그리고 그 page가 다시 필요하면 다시 file system에서 불러올 수 있다. 이 접근방법을 사용함으로써, file system은 backing store로써의 역할을 한다. swap space는 여전히 파일이 아닌 page를 위해 사용된다. 이 page들은 process를 위한 stack과 heap을 포함하고 있다. 이 방법은 Solaris와 BSD UNIX와 같은 여러 시스템들에서 좋은 compromise가 되고 있다.
모바일 시스템은 기본적으로 swapping을 지원하지 않는다. 대신, file system으로부터의 demand-page를 지원하며, 만약 메모리가 제약이 되면, application의 읽기 전용 page(such as code)를 회수한다. 이런 data는 나중에 필요하면 파일 시스템으로부터 demand-paged될 수 있다. iOS에서는 알수없는 memory pages들도 절대 application으로부터 회수되지 않는다.

## 9.3 Copy-on-Write

In Section 9.2, we illustrated how a process can start quickly by demand-paging in the page containing the first instruction. However, process creation using the fork() system call may initially bypass the need for demand paging by using a technique similar to page sharing (covered in Section 8.5.4). 이 테크닉은 빠른 프로세스 생성과 새로 만들어진 프로세스에 할당되어야할 신규 page들의 수를 최소화할 수 있게 한다.

fork() system call은 부모를 복제해서 자식 프로세스스를 만든다. 전통적으로 fork()는 부모 주소 공간이 복사본을 만들어 page들이 중복되도록 동작했다. 그러나 많은 자식 프로세스들이 생성 직후 exec() system call을 수행하는 것을 생각해보면, 부모의 주소 공간을 복사하는 것은 불필요할 수도 있다. 대신 우리는 **copy-on-write**라고 부모와 자식 프로세스가 초기에 같은 page를 공유하도록 동작하는 방법을 사용할 수 있다. 이 공유된 page들은 copy-on-write pages로 마킹된다. 그리고 프로세스 중 하나가 공유된 page에 write를 하려고 하면, 복사본이 생성된다. Copy-on-write is illustrated in Figures 9.7 and 9.8, which show the contents of the physical memory before and after process 1 modifies page C. Copy-on-write is a common technique used by several operating systems, including Windows XP, Linux, and Solaris.

When it is determined that a page is going to be duplicated using copy- on-write, it is important to note the location from which the free page will be allocated. Many operating systems provide a **pool** of free pages for such requests. These free pages are typically allocated when the stack or heap for a process must expand or when there are copy-on-write pages to be managed.
Operating systems typically allocate these pages using a technique known as **zero-fill-on-demand**. Zero-fill-on-demand pages have been zeroed-out before being allocated, thus erasing the previous contents.

Several versions of UNIX (including Solaris and Linux) provide a variation of the fork() system call — vfork() (for **virtual memory fork**) — that operates differently from fork() with copy-on-write. With vfork(), the parent process is suspended, and the child process uses the address space of the parent. Because vfork() does not use copy-on-write, if the child process changes any pages of the parent’s address space, the altered pages will be visible to the parent once it resumes. Therefore, vfork() must be used with caution to ensure that the child process does not modify the address space of the parent. vfork() is intended to be used when the child process calls exec() immediately after creation. Because no copying of pages takes place, vfork() is an extremely efficient method of process creation and is sometimes used to implement UNIX command-line shell interfaces.

## 9.4 Page Relacement

page-fault rate에 대한 앞선 논의를 보면, 우리는 개별 page faults는 최대 한번, 처음 referenced 될 때 발생한다고 짐작했다. 그러나 이 표현은 정확하지 않다. 열개의 페이지로 이뤄진 하나의 프로세스가 정확히 그 중 절반을 쓴다고 하면, 절대 사용되지 않을 5개의 페이지에 대한 I/O load를 절약할 수 있다. 우리는 또한 멀티프로그래밍 정도(degree)를 증가시킬 수 있다. 만약 40개의 frame이 있다면, 우리는 8개의 프로세스를 수행할 수 있다. page 10개 중 5개는 절대 쓰이지 않기 때문에.

이런 방법을 통해 우리는 메모리를 **over-allocating**한다. 만약 같은 종류의 프로세스를 6개를 돌렸다고 치자. 그러나 어떤 특정 data set으로 인해 page 10개가 전부 필요하게 될 수 있다. 그러면 사용가능한 frame은 40개 뿐이지만, 60개를 필요로 하게 된다.

더 나아가 시스템 메모리는 프로그램 pages를 갖고 있기만 한 것은 아니다. I/O를 위한 buffer도 상당한 메모리를 소모한다. 이런 사용은 memory-placement algorithm의 부담을 증가시킬 수 있다. 얼마나 많은 메모리를 I/O에 할당해야할지, program page에 할당해야 할지에 대한 문제는 significant challenge이다. Some systems allocate a fixed percentage of memory for I/O buffers, whereas others allow both user processes and the I/O subsystem to compete for all system memory.

Over-allocation of memory manifests itself as follows. While a user process is executing, a page fault occurs. The operating system determines where the desired page is residing on the disk but then finds that there are no free frames on the free-frame list; all memory is in use (Figure 9.9).
The operating system has several options at this point. It could terminate the user process. However, demand paging is the operating system’s attempt to improve the computer system’s utilization and throughput. Users should not be aware that their processes are running on a paged system—paging should be logically transparent to the user. So this option is not the best choice.
The operating system could instead swap out a process, freeing all its frames and reducing the level of multiprogramming. This option is a good one in certain circumstances, and we consider it further in Section 9.6. Here, we discuss the most common solution: **page replacement.**

### 9.4.1 Basic Page Relacement

Page replacement는 다음과 같은 접근방법을 따른다. free인 frame이 없으면, 현재 사용되지 않는 것을 찾아서 해제시킨다. frame의 내용을 swap space에 쓰고, page table을 수정해서 해당 page가 더이상 메모리가 없다고 표시한다. (Figure 9.10). 이제 해제된 frame을 page for which the process faulted를 hold하는 데 사용한다. page-fault service routine을 수정해서 page replacement를 포함하도록 하자.

1. 디스크에서 필요한 page의 위치를 찾는다.
2. free frame을 찾는다.
    * a. free frame이 있으면, 그것을 사용한다.
    * b. free frame이 없으면, page-replacement algorithm이 **victim frame**을 선택한다.
    * c. victim frame을 디스크에 쓰고, page와 frame table을 그에 맞게 수정한다.
3. 필요한 page를 새로 freed frame으로 읽어들인다. page와 frame tables을 수정한다.
4. page fault가 발생한 데의 user process를 이어간다.

Notice that, free frame이 없다면, 2개의 page transfer가 필요하다. (one out and one in). 이 상황은 page-fault service time을 2배로 증가시키며, 그에 따라 access time도 늘어난다.

우리는 이 overhead를 **modify bit (or dirty bit)**을 사용해서 줄일 수 있다. 이 방법이 사용될 때, 각각의 page나 frame은 하드웨어에 modify bit를 하나씩 가지고 있다. page의 modify bit는 page에 어느 byte가 쓰여졌던 지 상관 없이 언제든 하드웨어로부터 세팅되어, page가 수정됐음을 나타낸다. replacement를 위해 page를 선택할 때 modify bit 을 검토한다. bit가 세팅되어 있다면, disk로부터 읽혀진 후에 수정됐음을 알 수 있다. 이런 경우, page를 disk에 반드시 써야한다. 그러나, 만약 modify bit가 세팅되지 않은 경우엔 disk에서 읽은 후 메모리에서 수정이 없었던 것이고, 그러므로 disk에 다시 쓰지 않아도 된다. 이미 disk에 있으니까. 이 방법은 또한 read-only pages에도 적용된다. (for example, pages of binary code). 그런 page들은 수정될 수 없다. 그래서 필요한 순간이 오면 버리면 된다. 이 방법은 현저하게 page fault를 service하는 데 필요한 시간을 줄일 수 있다. page가 수정되지 않았다면, I/O 시간이 절반으로 줄어들기 때문이다.

Page replacement is basic to demand paging. logical physical memory를 분리하고, 이 메커니즘을 통해 많은 양의 virtual memory를 작은 physical memory 위에서도 제공할 수 있게 된다. demand paging 없이 user addresses are mapped into physical addresses, and the two sets of addresses can be different. 모든 프로세스의 page가 physical memory에 있어야만 한다. 그러나 demand paging을 쓰면, logical address space가 더이상 physical memory에 제한되지 않는다. 20개의 page를 가진 user process가 하나 있다면, demand paging과 replacement algorithm을 사용해서 10개의 frame으로 수행할 수 있다. If a page that has been modified is to be replaced, its contents are copied to the disk. A later reference to that page will cause a page fault. At that time, the page will be brought back into memory, perhaps replacing some other page in the process.

demand paging을 구현하려면 해결해야하는 2가지 문제가 있다. 반드시 **frame-allocation algorithm**과 **page-replacement algorithm**을 개발해야만 한다. 메모리에 여러 프로세스가 있을 때 frame을 process마다 얼마나 할당할 것인지, 그리고 page replacement가 필요할 때 어떤 frame을 교체시킬 것인지를 선택해야 하기 때문이다. 이 문제를 풀기 위한 적절한 알고리즘을 설계하는 것은 중요한 업무이다. 왜냐하면 disk I/O가 매우 비싼 작업이기 때문이다. demand-paging methods에서 약간만 향상이 되더라도 시스템 성능에 큰 도움이 될 정도이다.

There are many different page-replacement algorithms. Every operating system probably has its own replacement scheme. How do we select a particular replacement algorithm? In general, we want the one with the lowest page-fault rate.

우리는 알고리즘을 특정 string of memory reference에서 실행하고 page fault 수를 계산하여 알고리즘을 평가한다. The string of memory references is called a **reference string**. 우리는 reference string을 인위적으로 생성할 수 있다, 또는 주어진 시스템을 추적하고 각각의 memory reference의 주소를 기록할 수 있다. 후자의 경우 매우 큰 데이터가 생성된다. (on the order of 1 million addresses per second). 이 숫자를 줄이기 위해 우리는 2개의 fact를 사용한다.

첫째, 주어진 page size에서의 page number를 고려해야 한다.
둘째, page p에 대한 reference가 있다면, 바로 다음에 page p에 대한 어떤 reference가 오더라도 page fault는 일어나지 않는다.

(예시. page당 100bytes인 경우에서의 reference string sequence.)

Obviously, as the number of frames available increases, the number of page faults decreases.
예시로 보면 3개 이상 available frame이 있었으면, 초기 세팅할 때만 해서 fault는 3번만 일어났을 것이다.

### 9.4.2 FIFO Page Replacement

The simplest page-replacement algorithm is a first-in, first-out (FIFO) algorithm. (Figure 9.12) The FIFO page-replacement algorithm is easy to understand and program. However, its performance is not always good.

followin reference string:

1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

Notice that the number of faults for four frames (ten) is greater than the number of faults for three frames (nine)! This most unexpected result is known as **Belady’s anomaly**: for some page-replacement algorithms, the page-fault rate may increase as the number of allocated frames increases.

### 9.4.3 Optimal Page Relacement

Belady's anomaly의 발견으로 인한 하나의 결과가 **optimal page-relacement algorithm**의 발견이다. page-fault rate가 가장 낮은 algorithm들은 모두 Belady' anomaly를 겪지 않는다. 이런 알고리즘이 있는데, OPT 또는 MIN으로 불린다. It is simply this:

Replace the page that will not be used for the longest period of time.

Use of this page-replacement algorithm guarantees the lowest possible page fault rate for a fixed number of frames.

(reference string으로 해보면 이 말이 맞는 걸 보여줌)
그러나 불행히도 이 optimal page-replacment algorithm은 구현하기 어렵다. refence string의 future knowledge를 알아야 하기 때문이다. (We encountered a similar situation with the SJF CPU-scheduling algorithm in Section 6.3.2.)

### 9.4.4 LRU Page Replacement

If we use the recent past as an approximation of the near future, then we can replace the page that has not been used for the longest period of time. This approach is the **least recently used (LRU) algorithm**.

When a page must be replaced, LRU chooses the page that has not been used for the longest period of time. 우리는 이 전략이 과거로 돌이켜보는 optimal page-replacement algorithm이라고 생각할 수 있다.

LRU 정책은 종종 page-replacement algorithm으로 사용되고, 좋은 것으로 간주된다. 제일 문제는 어떻게 LRU를 구현하는 가이다. 이 알고리즘은 hardware의 상당한 assistance이 필요하다. 마지막 사용으로부터의 지난 시간 순서를 찾는 것이 문제인데, 가능한 구현이 2가지가 있다.

* Counter. 가장 간단한 방법으로 page-table entry에 time-of-use field를 만들어서 사용할 때마다 counter를 증가시킨다. 그러면 smallest time value를 갖는 것이 교체 대상이 된다.

* Stack. double linked list로 구성해서 최근에 사용한 page를 stack에서 꺼내서 맨 위에 위치시킨다. 그러면 stack 가장 아래에 있는 page가 제일 오래전에 쓴 page가 된다.

Like optimal replacement, LRU replacement does not suffer from Belady’s anomaly. Both belong to a class of page-replacement algorithms, called **stack algorithms**, that can never exhibit Belady’s anomaly.

LRU의 구현은 표준 TLB 레지스터 이외의 하드웨어 지원이 없다면 생각할 수 없다. 그리고 clock fields 나 satck을 memory reference마다 update해야만 하기 떄문에 10배 이상 느려질 수도 있다. 메모리 관리를 위한 부하를 이정도까지 감내할 수 있는 시스템은 거의 없다.

### 9.4.5 LRU-Approximation Page Replacement

LRU page replacement 적용 가능한 하드웨어를 제공하는 컴퓨터 시스템들이 어느 정도 있다. 어떤 시스템들은 하드웨어 지원이 전혀 없어서 다른 page-replace algorithms(FIFO 같은)을 써야만 하는 경우도 있다. 그러나 많은 시스템들은 **reference bit**의 형태로 약간의 도움은 주고 있다. page가 read든 write든 지간에 어떤 byte에라도 referenced라면, reference bit은 해당 page에 세팅이 된다. Reference bits는 page tabld에서의 각각의 entry와 관련이 있다.

처음엔, operating system이 모든 bit들을 0으로 세팅한다. 사용자 프로세스가 수행됨으로써, 각 각의 page referenced의 bit는 hardware에 의해 1로 세팅된다. 시간이 조금 지나면, reference bits를 검사해서 사용 순서는 모르더라도 어떤 page가 쓰였는지 안 쓰였는지 여부는 알 수 있다. 이 정보가 LRU replacement와 비슷한 page-replacement algorithm들의 기본이 된다.

#### 9.4.5.1 Additional-Reference-Bits Algorithm

일정 주기마다 reference bits를 기록해서 추가적으로 순서 정보를 얻을 수도 있다. 메모리에 있는 테이블에 page마다 8-bit byte를 보관할 수 있다. 일정 주기마다 (say, every 100 millisencds), a timer interrupt transfers control to the operating system. operating system은 매 페이지마다 reference bit를 하나씩 옮기는데, 8-bit byte에서 high-oerder로 옮기고, 나머지 bit들도 오른쪽으로 하나씩 밀려서 low-order bit가 버려지는 식이다. 이 8-bit shift register는 최근 8번의 periods 간의 page history를 가지고 있다. period마다 한번 이상씩 사용됐다면 11111111이 될테고, 11000100은 01110111보다 더 최근에 사용됐다고 알 수 있다.이 8-bit byte 값을 usigned interger로 변환하면, 가장 낮은 숫자의 page가 LRU page를 의미하므로 교체 대상이 된다. 물론 값이 unique하지는 않을 것이므로, 가장 작은 값 전부를 swap out 해버리거나, 그 중에서 FIFO를 쓰거나 하는 식으로 선택할 수 있다.

shift register에 포함된 the number of bits of history가 다양하고, 가능한 빠르게 업데이트하기 위해 값을 선택할 수도 있다. 극단적인 예로 숫자를 0으로 줄이면 reference bit만 남아 있는 셈이다. 이 알고리즘을 **second-chance page-replacement algorithm**이라 한다.

#### 9.4.5.2 Second-Chance Algorithm

second-chance replacement의 기본 알고리즘은 FIFO replacement이다. page가 선택될 때, 우리는 refence bit를 조사한다. 만약 value가 0이면, 이 page에 대한 교체를 진행하고, 1이라면 second chance를 주고 다음 FIFO page를 선택한다. page가 두번째 찬스를 얻을 때 reference bit이 정리되고 arrival time 또한 현재 시간으로 재설정된다. 그러므로 이 페이지는 다른 모든 페이지가 교체될 때(또는 두번째 찬스)까지 교체되지 않는다. 게다가 page의 reference bit가 충분히 자주 세팅이 반복되면, 교체대상에 빼버린다.

second-chance algorithm(**clock** 알고리즘으로도 불리는)을 구현하는 방법 중 하나는 circular queue를 쓰는 것이다. pointer가 다음에 어떤 page가 교체되어야할지 나타내고 있다. a frame이 필요할 때, 그 포인터가 reference bit로 0을 가진 page을 찾을 때까지 진행한다. 진행하는 동안 지나온 페이지의 reference bit을 정리(0)하면서 넘어간다. victim page를 찾으면, page는 교체되고 그 자리에 새로운 페이지를 삽입한다. 최악의 경우는 모든 비트가 세팅된 상태로 전체 큐를 돌면서 모든 페이지에 2번째 찬스를 주게 된다. 이 경우엔 FIFO replacement가 되는 것이다.

#### 9.4.5.3 Enhanced Second-Chance Algorithm

우리는 reference bit과 modify bit를 고려해서 second-chance algorithm을 발전시킬 수 있다. 2개의 비트를 보면 아래아 같은 4가지 클래스(상황)으로 구분할 수 있다.
1. (0,0) 최근에 사용되지도 않았고, 수정되지 않음 - best page to replace
2. (0,1) 최근에 쓰이지는 않았지만 변경된 경우 - 교체되려면 수정된 내용으로 쓰여져야 됨.
3. (1,0) 최근에 쓰였지만 깨끗한 경우(수정되지 않음) - 아마 곧 다시 쓰일 것.
4. (1,1) 최근에 쓰이고 변경된 경우

page replacement 요청이 오면, clock algorithm과 같은 방식을 쓴다. 그러나 어떤 페이지의 reference bit가 1인지 찾는 것 대신에, 해당 페이지가 어떤 클래스에 속하는지 판단한다.
Notice that we may have to scan the circular queue several times before we find a page to be replaced. The major difference between this algorithm and the simpler clock algorithm is that here we give preference to those pages that have been modified in order to reduce the number of I/Os required.

### 9.4.6 Counting-Based Page Replacement

page replacement에 사용할 수 있는 다른 알고리즘들도 많다. 예를 들어 우리는 reference의 수의 counter를 페이지별로 가지면서 아래 두가지 방법을 개발할 수 있다.

1. **Least Freqently Used**(**LFU**)는 가장 카운트가 적은 page가 교체 대상이 될 것이다. 현재 사용되고 있는 page는 reference count가 클 수밖에 없기 때문이다. 그러나 알려진 문제로 초반에만 많이 쓰이고 뒤에서 쓰이지 않는 경우에도 메모리에 남게 되는 점이 있다. 하나의 해결책으로 regular interval마다 카운트 bit를 오른쪽으로 하나씩 이동시키는 방법이 있다. 평균 사용 횟수가 지수적으로 줄게 될 것이다.

2. **Most Frequenntly Used**(**MFU**) page-replacement algorithm is based on the argument that the page with the smallest count was probably just brought in and has yet to be used. //걍 반대

어쨋든 둘다 구현하기도 어렵고 OPT replacement에 가깝지도 않아서 일상적으로 사용되진 않는다.

### 9.4.7 Page-Buffering Algorithms

특정 page-replacement algorithm을 위해서 other procedures도 종종 쓰인다. 예를 들면, 시스템들은 일반적으로 free frame pool을 가지고 있다. page fault가 일어났을 때 그 중에 victim  frame을 선택한다. 그러나 victim이 written out되기 전에 필요한 page가 free frame으로 읽어진다. 이 프로시져는 victim page가 쓰여지길 기다리지 않고, 가능한 빨리 그 프로세스를 재시작하도록 한다. 그리고 victim이 나중에 다 쓰여질 때, 그 frame은 free-frame pool에 추가된다. (= 필요한 page 읽는 걸 먼저하고, victim page write out을 비동기로)

이 아이디어의 확장이 modified pages 목록을 유지하는 것이다. paging device가 idle이면 언제나, 수정된 페이지가 선택되어 disk에 쓰여진다. 그리고 modify bit는 reset된다. 이 방식은 replacement를 위해 page를 선택했을 때 clear해서 disk에 쓸 필요가 없을 확률을 높여준다.

또 다른 modification은 free frame pool은 유지하되 어떤 page가 프레임에 들어있는지 기억하고 있는 것이다. frame이 disk에 쓰일 때 frame 내용은 변하지 않으므로, 그 frame이 다른 용도로 쓰이기 전에 필요해지면 old page는 free-frame pool에다가 바로 가져가다 쓸 수 있다. 이 경우엔 I/O가 일어나지 않는다. page fault가 일어나면 먼저 필요한 page가 free-frame pool에 있는지부터 확인한다. 그렇지 않으면 free frame을 하나를 선택해서 거기에 내용을 읽어 들어야 한다.

이 테크닉은 VAX/VMS systm에서 FIFO replacement algorithm과 같이 쓰인다. FIFO replacement algorithm이 실수로 아직 사용 중인 page를 교체할 때, 해당 page는 free-frame에서 빠르게 검색되어 사용되므로, I/O가 필요하지 않다. free-frame buffer는 상대적으로 구리지만 간단한 FIFO replacement algorithm에 대한 protection을 제공한다. 이 방법은 VAX 초기 버전이 reference bit을 제대로 구현하지 않았기 때문에 필요하다. 

Unix system의 몇 버전들은 이 메서드와 second-chance algorithm을 함께 사용했다. 이 방법은 잘못된 victim이 선택되었을 때에 발생하는 페널티를 줄이기 위한 용도로 어느 page-replacement algorithm이든지 간에 확장하여 사용할 수 있다.

### 9.4.8 Applications and Page Replacement

In certain cases, OS가 buffering을 전혀 없다면 OS의 virtual memory를 통한 application의 data 접근은 성능이 안 좋다. 전형적인 예제로 database가 있는데, 그것은 직접 memory management와 I/O buffering을 제공한다. 이런 application들은 일반 목적의 OS 알고리즘보다 스스로 메모리와 디스크 사용을 하는 것이 더 낫다는 것을 이해하고 있다. 그러나 만약 OS가 I/O buffering을 하고 application도 하고 있다면, I/O 사용을 위해 드는 메모리가 두배가 된다.

다른 예시로, data warehouses 빈번하게 massive sequential disk read, 후에 computations and writes가 일어난다. LRU 알고리즘은 old page를 지우고 새로운 것들을 보존하고 있을 것이다. 그러나 application을 새로운 것보다는 older page을 읽으려고 할 것이다.(다시 sequential reads가 발생). 여기서는 MFU가 LRU보다 효과적일 것이다.

이런 문제들로 인해 몇 OS는 어떤 파일시스템의 구조도 갖지 않은 채 a large sequential array of logical blocks으로 disk partition을 쓰는 기능을 사용한 special program을 제공한다. This array is sometimes called the **raw disk**, and I/O to this array is termed raw I/O. 특정 application인 그들 특정 목적으로 storage service를 구현함으로써 더 효과적일 수 있겠지만, 대부분의 application을 일반적인 file system에서의 성능이 더 낫다는 걸 명심(?)해라.

## 9.5 Allocation of Frames

allocation issue로 넘어가자. 다양한 프로세스들 중에서 fixed amount의 free memory를 어떻게 할당할 수 있을까? 만약 93 free frames과 two processes를 가졌다면 각 프로세스들은 frame을 얼마나 가질까?

가장 단순한 케이스는 single-user system이다. 메모리 128KB에 1KB pages들로 이루어진 single-user system을 떠올려보자. 이 시스템은 128 frames를 가지고 있다. OS가 35KB를 쓰고, 93 frames를 유저 프로세스를 위해 남겨놨다. 순수하게 demand paging할 때 93 frames은 free-frame list에 올라와 있을 것이다. user process가 수행하기 시작할때 순차적인 page faults가 발생할 것이다. 첫 93 page faults를 free-frame list에 가져오면 된다. free frame list를 다 쓰면, page-replacement algorithm이 93개의 in-memory page 중에서 하나를 선택해서 94번째와 교체할 것이다. 그리고 프로세스가 죽으면 93 frames은 다시 free-frame list로 돌아간다.

이 simple strategy에 여러 variations이 존재한다.

### 9.5.1 Minimum Number of Frames

Our strategies for the allocation of frames are constrained in various ways. 우리는 page sharing이 있지 않은 이상엔 가용가능한 frame 수의 총합보다 많은 양을 할당할 수 없다. 우리는 또한 적어도 최소 frame number만큼은 할당해야 한다. 후자의 요구사항에 맞춰 자세히 들여다보자.

적어도 최소 number of frames를 할당하는 첫번째 이유는 성능이다. 명백하게도 각각의 프로세스에 할당된 frame의 수가 감소하면, page-fault rate는 증가하고 process execution을 느려진다. 그리고 instruction을 수행하기 전에 page fault가 발생하면, 그 instruction는 재시작해야만 한다. 결과적으로 어떤 single instruction이라도 관련된 page들을 담고 있으려면 충분한 frame을 갖고 있어야 한다.

For example, consider a machine in which all memory-reference instructions may reference only one memory address. In this case, we need at least one frame for the instruction and one frame for the memory reference. 게다가 one-level indirect addressing이 허락된다면, paging 하는 데에 프로세스당 적어도 3개의 frame이 필요하다. 2 frame만 있으면 어떨지 생각해봐라.

frame 최소수는 컴퓨터 아키텍처에 의해 결정된다. 예를 들어, PDP-11에 대한 이동 명령은 일부 어드레싱 모드에 대해 하나 이상의 단어를 포함하므로, 명령 자체가 두 페이지에 걸칠 수 있다. 두 개의 피연산자(operand)가 indirect references일 수도 있다, for a total of six frames.

Theoretically, a simple load instruction could reference an indirect address that could reference an indirect address (on another page) , ...
To overcome this difficulty, we must place a limit on the levels of indirection (for example, limit an instruction to at most 16 levels of indirection).

Whereas the minimum number of frames per process is defined by the architecture, the maximum number is defined by the amount of available physical memory. In between, we are still left with significant choice in frame allocation.

### 9.5.2 Allocation Algorithms

m 개의 frame을 n개의 프로세스로 나누는 가장 간단한 방법은 모두에게 같은 양을 주는 것이다. (m/n frames) This scheme is called **equal allocation**.

An alternative is to recognize that various processes will need differing amounts of memory. we can use **proportional allocation**, in which we allocate available memory to each process according to its size.

그런데 알아야 될 것이이 euqal이나 proportional allocataion이나 process의 high-priority에 대한 차이가 없다. 하나의 솔루션은 size 뿐 아니라 priority까지 고려하여 combination으로 ratio of frames를 정하는 것이다.

### 9.5.3 Global versus Local Allocation

With multiple processes competing for frames, we can calssify page-replacement algorithms into two broad catergories: **global replacement** and **local replacement**. global은 frame을 다른 프로세스에서도 가져올 수 있고, Local은 its own set of allocated frames에서 가져온다. 그래서 global은 high-priority process의 frame allocation을 높이는 데에는 좋다. 그런데 문제가 프로세스가 자기 자신의 page-fault rate를 제어할 수 없다는 점이 있다. 그래서 같은 프로세스라도 외부 상황에 의해 처리 시간이 많이 달라질 수 있다.

Local replacement might hinder a process, however, by not making available to it other, less used pages of memory. Thus, global replacement generally results in greater system throughput and is therefore the more commonly used method.

### 9.5.4 Non-Uniform Memory Access

Thus far in our coverage of virtual memory, we have assumed that all main memory is created equal—or at least that it is accessed equally. On many computer systems, that is not the case.

As you might expect, the CPUs on a particular board can access the memoryon that board with
less delay than they can access memory on other boards in the system. Systems in which memory access times vary significantly are known collectively as non-uniform memory access (NUMA) systems, and without exception, they are slower than systems in which memory and CPUs are located on the same motherboard.

그래서 page frame을 어디에 저장시켜서 관리하느냐가 performance에 큰 영향을 끼친다. scheduling system에도 변화가 비슷하게 필요하다. 그래서 frame와 CPU가 최대한 가깝게 하는 것이 목표다.
process를 스케줄할때 이전 CPU를 그대로 사용하도록 하고 memory-management system은 frame을 process와 가깝게 할당하려는 거로로 cache hit을 올리고 memory access time을 줄일 수 있다.

In fact, there is a hierarchy of **lgroups** based on the amount of latency between the groups. Solaris tries to schedule all threads of a process and allocate all memory of a process within an lgroup.

## 9.6 Thrashing

low-priority process에 할당된 frame 수가 컴퓨터 아키텍처의 요구 minimum number보다 낮아지면, 해당 프로세스의 수행을 미뤄야한다. 그리고 page out 및 할당된 frame을 전부 해제시킨다. This provision introduces a swap-in, swap-out level of intermediate CPU scheduling.

충반한 frame을 갖지 못한 프로세스를 보자. active use의 page를 지원할만큼의 frame을 갖지 못한 프로세스가 있다면, 빠르게 pgae-fault가 반복되어 일어날 것이다. This high paging activity is called **thrashing**. A process is thrashing if it is spending more time paging than executing.

### 9.6.1 Cause of Thrashing

Thrashing은 심각한 performance problems을 일으킨다.
operating system이 CPU utilization을 모니터한다. CPU utilization이 너무 낮아지면 우린 새로운 프로세스를 통해서 degree of multiprogramming을 증가시킬 것이다. global page-replacement algorithm이 사용된다. 프로세스가 새로운 상태에 들어오면서 더 많은 frame이 필요하다고 하면 faulting이 시작되면서 다른 프로세스들로부터 frame을 가져온다. 그러면서 그 프로세들은 또 fault가 될 것이다. 이런 faulting proceses는 반드시 swap pages in and out을 위한 pagin device를 사용해야 한다. 프로세스들은 paging device들을 기다리게 되면서 CPU utilization이 감소한다.

No work is getting done, because the processes are spending all their time paging.

We can limit the effects of thrashing by using a **local replacement algorithm** (or **priority replacement algorithm**). local replacement에서는 하나의 프로세스가 thrashing을 시작하더라도 다른 프로세스의 frame을 가져올 수 없으므로 이후의 thrash를 발생시키지 않는다. 그러나 이건 해결책이 아니다.

To prevent thrashing, we must provide a process with as many frames as it needs. But how do we know how many frames it “needs”? There are several techniques. The working-set strategy (Section 9.6.2) starts by looking at how many frames a process is actually using. This approach defines the **locality model** of process execution.

.

### 9.6.2 Working-Set Model

As mentioned, the **working-set model** is based on the assumption of locality. This model uses a parameter, ㅅ, to define **the working-set window**. The idea is to examine the most recent ㅅ page references. The set of pages in the most recent ㅅ page references is the **working set** (Figure 9.20).

The most important property of the working set, then, is its size. If we compute the working-set size, WSSi , for each process in the system, we can then consider that
D = (Sigma) WSSi ,
where Dis the total demand for frames

This working-set strategy prevents thrashing while keeping the degree of multiprogramming as high as possible.

### 9.6.3 Page-Fault Frequency

The working-set model is successful, and knowledge of the working set can be useful for prepaging (Section 9.9.1), but it seems a clumsy way to control thrashing. A strategy that uses the **page-fault frequency** (PFF) takes a more direct approach.

The specific problem is how to prevent thrashing. Thrashing has a high page-fault rate. Thus, we want to control the page-fault rate. When it is too high, we know that the process needs more frames. Conversely, if the page-fault rate is too low, then the process may have too many frames

### 9.6.4 Concluding Remarks

//결국은 메모리를 충분히 줘야 해결됩니다.

Practically speaking, thrashing and the resulting swapping have a disagreeably large impact on performance. The current best practice in implementing a computer facility is to include enough physical memory, whenever possible, to avoid thrashing and swapping.
From smartphones through mainframes, providing enough memory to keep all working sets in memory concurrently, except under extreme conditions, gives the best user experience.

## 9.7 Memory-Mapped Files

Consider a sequential read of a file on disk using the standard system calls

### 9.7.1 Basic Mechanism

//메모리에서의 파일에 대한 쓰기가 디스크에 바로 동기화되는 것은 아니다.

Memory mapping a file is accomplished by mapping a disk block to a page (or pages) in memory. 
Subsequent reads and writes to the file are handled as routine memory accesses.

Note that writes to the file mapped in memory are not necessarily immediate (synchronous) writes to the file on disk. Some systems may choose to update the physical file when the operating system periodically checks whether the page in memory has been modified.When the file is closed, all the memory-mapped data are written back to disk and removed from the virtual memory of the process.

Solaris still memory-maps the file; however, the file is mapped to the kernel address space. Regardless of how the file is opened, then, Solaris treats all file I/O as memory-mapped, allowing file access to take place via the efficient memory subsystem.

### 9.7.2 Shared Memory In the Windows API

...

### 9.7.3 Memory-Mapped I/O

To allow more convenient access to I/O devices, many computer architectures provide **memory-mapped I/O**.

## 9.8 Allocating Kernel Memory

유저모드로 수행 중인 프로세스가 추가 메모리를 요구할 때, kernel에 의해 관리되던 free page frames 목록 중에서 pages가 할당된다.

이 목록은 일반적으로 9.4 절에서 설명한 것과 같은 페이지 교체 알고리즘을 사용하여 채워지며 앞에서 설명한 것처럼 physical memory에 흩어져있는 free page가 포함되어있을 가능성이 높다. 만약 유저 프로세스가 single byte of memory를 요청했다면, page frame을 하나 통째로 할당받을 것이고, 그 결과 internal fragmentation이 발생한다.

Kernel memory is often allocated from a free-memory pool different from the list used to satisfy ordinary user-mode processes. There are two primary reasons for this:

1. 커널이 요청하는 메모리는 사이즈가 다양하다. 어떤 것들은 페이지하나 사이즈보다 작다. 따라서 커널은 메모리를 보수적으로 사용해야하며 조각화로 인한 낭비를 최소화해야 한다. This is especially important because many operating systems do not subject kernel code or data to the paging system

2. Pages allocated to user-mode processes do not necessarily have to be in contiguous physical memory. However, certain hardware devices interact directly with physical memory—without the benefit of a virtual memory interface—and consequently may require memory residing in physically contiguous pages.

뒤 섹션에서 kernel process에 할당되는 free memory managing strategies 2개를 하나씩 살펴볼 것이다. : "buddy system" and slab allocation.

### 9.8.1 Buddy System

//물리적으로 인접한 segment에 할당함. 2의 제곱으로 크기로 잘라서 할당함. 인접한 버디를 빨리 결합해서 더 큰 세그먼트를 형성할 수 있는 장점이 있다. 2의 제곱으로 생성하기 때문에, 내부에 단편화의 가능성이 높아진다. (ex. 33을 요청하면 64를 할당하게됨)

### 9.8.2 Slab Allocation

//캐시를 사용해서 커널 객체를 저장하는 방식. (Figure 9.27) 

The slab allocator provides two main benefits:
1. No memory is wasted due to fragmentation. Fragmentation is not an issue because each unique kernel data structure has an associated cache, and each cache is made up of one or more slabs that are divided into chunks the size of the objects being represented. Thus, when the kernel requests memory for an object, the slab allocator returns the exact amount of memory required to represent the object.

2. Memory requests can be satisfied quickly. The slab allocation scheme is thus particularly effective for managing memory when objects are frequently allocated and deallocated, as is often the case with requests from the kernel. The act of allocating—and releasing—memory can be a time-consuming process. However, objects are created in advance and thus can be quickly allocated from the cache. Furthermore, when the kernel has finished with an object and releases it, it is marked as free and returned to its cache, thus making it immediately available for subsequent requests from the kernel.

SLOB : 이건 3가지로 오브젝트를 분류해서 사용 : small, medium, large

SLUB : Slab보다 메타 크기를 줄여서 효율적으로 사용함.


## 9.9 Other Considerations

### 9.9.1 Prepaging

### 9.9.2 Page Size

### 9.9.3 TLB Reach

### 9.9.4 Inverted Page Tables

### 9.9.5 Program Structure

### 9.9.6 I/O Interlock and Page Locking

## 9.10 Operating-System Examples

...

## 9.11 Summary

...

In addition to a page-replacement algorithm, a frame-allocation policy is needed. Allocation can be fixed, suggesting local page replacement, or dynamic, suggesting global replacement. The working-set model assumes that processes execute in localities. The working set is the set of pages in the current locality. Accordingly, each process should be allocated enough frames for its current working set. If a process does not have enough memory for its working
set, it will thrash. Providing enough frames to each process to avoid thrashing may require process swapping and scheduling.

Most operating systems provide features for memory mapping files, thus allowing file I/O to be treated as routine memory access. The Win32 API implements shared memory through memory mapping of files.

Kernel processes typically require memory to be allocated using pages that are physically contiguous. The buddy system allocates memory to kernel processes in units sized according to a power of 2, which often results in fragmentation. Slab allocators assign kernel data structures to caches associated with slabs, which are made up of one or more physically contiguous pages. With slab allocation, no memory is wasted due to fragmentation, and memory requests can be satisfied quickly.

In addition to requiring us to solve the major problems of page replacement and frame allocation, the proper design of a paging system requires that we consider prepaging, page size, TLB reach, inverted page tables, program structure, I/O interlock and page locking, and other issues.

