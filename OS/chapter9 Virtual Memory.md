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

• A program would no longer be constrained by the amount of physical
memory that is available. Users would be able to write programs for an
extremely large virtual address space, simplifying the programming task.
• Because each user program could take less physical memory, more
programs could be run at the same time, with a corresponding increase in
CPU utilization and throughput but with no increase in response time or
turnaround time.
• Less I/O would be needed to load or swap user programs into memory, so
each user program would run faster.