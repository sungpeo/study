# Chapter4. Threads

## Chapter Objectives
* thread라는 용어 소개 - multithreaded computer system에서 CPU 활용의 기본 단위
* PThreads, Windows, 그리고 Java thread 라이브러리 사용을 위한 APIs에 대한 논의
* 묵시적(implicit) threading의 몇가지 전략들엔 무엇이 있을까?
* multithreaded programming에서의 issue들을 검토해보자
* Windows와 Linux에서는 OS가 어떻게 threads를 지원하고 있을까

## 4.1 Overview
Thread는 CPU 사용의 기본 단위로서, thread ID, program counter, register set과 stack으로 이루어져 있다. 같은 프로세스에 포함된 thread들은 code section, data section뿐 아니라 OS의 resource들(such as open files and signals)까지 함께 공유한다. 

### 4.1.1 Motivation
 최근 컴퓨터들의 대부분 software application은 멀티스레드 기반이다. 일반적으로 하나의 프로세스가 여러 스레드를 동작하는 식이다. 예를 들면 워드프로세서의 경우 그래픽 출력을 위한 스레드가 있을 것이고, 다른 스레드는 keystroke을 입력 받으며, 세번째 스레드는 스펠링과 문법 체크를 하고 있을 것이다. 또는 성능 향상을 위해 CPU 계산 집약적인 작업을 병렬처리하도록 설계된 어플리케이션들도 있다.
 만약 웹서버과 전통적인 싱글스레드 프로세스로 구동된다면, 한번에 하나의 클라이언트 처리밖에 못할 것이다. 이런 경우 해결책은 서버가 request를 받을 때마다 개별 프로세스들을 생성하는 것이다. 사실 이 방법은 스레드가 보편적이기 전에는 흔한 방식이었다. 하지만 만약 같은 task를 처리한다고 하면 굳이 프로세스 생성과 같은 부하를 감당할 필요가 있을까? 물론 멀티 스레드를 사용하는 것이 훨씬 효과적일 것이다. 3장에서 배웠던 RPC 시스템에서도 스레드는 중요하다. RPC 서버가 멀티스레드로 돌면서 동시 작업을 수행하기 때문이다.
 요새는 대부분의 operating-system kernel들도 마찬가지로 멀티스레드로 구성된다.

### 4.1.2 Benefits
 4개의 주요 카테고리로 이점을 생각해 볼 수 있다.
 1. Responsiveness. time-consuming 작업은 별개의 스레드로 수행시키면, application은 작업 중에서 사용자에게 즉각 반응할 수 있다. 
 2. Resource sharing. 한 프로세스의 포함된 스레드는 해당 리소스들을 기본적으로 공유하기 때문에, 같은 주소값 안에서 자유롭게 통신할 수 있다.
 3. Economy. 프로세스 생성은 메모리 및 리소스 할당으로 인해 비용이 비싼 편인데 반해, 스레드는 리소스를 프로세스 내에서 공유하기 때문에,생성과 context-switch 하는 것에도 경제적이다.
 4. Scalability. 멀티프로세서 아키텍처에서 멀티스레딩은 더욱 빛을 발한다. 생성된 스레드들이 각기 다른 코어에서 돌면서 병렬처리가 될 것이기 때문이다.


## 4.2 Multicore Programming
컴퓨터 설계의 역사 초반에 더 좋은 computer performance를 내기 위해 single-CPU 시스템들은 multi-CPU 시스템들로 진화했다. 조금 더 최근엔 비슷한 트렌드로써, 하나의 chip에 multiple computing core들이 들어가기 시작했다. 이런 컴퓨터들을 multicore 또는 multiprocessor 시스템이라 하는데, Multithread programming 메카니즘의 동시성이 효율적으로 수행되게 되었다. 싱글 컴퓨팅 코어에서는 스레드가 여럿으로 동시성(concurrency)이 있다고 하더라도, 실질적으로는 그 중 하나의 작업만 수행되고 있는 반면, 멀티 프로세서의 경우 concurrency가 즉 작업이 실제로 병럴(parallel)로 처리되는 것을 뜻하기 때문이다.
요새 시스템들은 수천개의 스레드가 돌기도 하기 때문에 CPU 설계자들은 더욱 thread performance를 향상시키는 데에 집중하고 있다. Modern Intel CPU의 경우 코어당 2개의 스레드를 지원하기도 하며, Oracle T4 CPU 같은 경우엔 하나의 코어가 8개의 스레드를 지원하기도 한다. 여기서 지원한다는 의미는 여러 스레드가 하나의 코어에 로딩되어 빠르게 switching 될 수 있다는 뜻이다.

### 4.2.1 Programming Challenges
 멀티코어를 잘 활용하는 개발을 하려고 하면 일반적으로 5가지의 challenge가 있다.
 1. Identifying tasks. 독립적이어서 개별 코어에서 병렬처리되도록 나눠지는 것이 가능한 task를 찾아야 한다.
 2. Balance.
 3. Data splitting. 개별 task로 나눠지는 것처럼 데이터에 대한 접근 및 조작도 나눠져서 실행될 수 있는 형태여야 한다.
 4. Data dependency. 데이터 디펜던시가 있는 task인지 확인해야 한다. 그런 경우엔 작업간의 synchronize 전략이 필요하게 될 것이다. 이건 5장에서 알아보기로 하자.
 5. Testing and debugging. 병럴처리하기 시작하면 테스팅과 디버깅이 싱글 스레드보다 어려워진다.

### 4.2.2 Types of Parallelism
 * Data parallelism. 데이터를 분산 subset으로 나누어 같은 형태의 작업을 병렬처리하도록 하는 것을 뜻한다.
 * Task parallelism. 같은 데이터를 보더라도 unique한 작업이 병렬로 실행되는 것을 의미한다.


## 4.3 Multithreading Models
지금까지 스레드를 일반적인 관점에 봤다면, 이번엔 스레드를 지원하는 것을 user level에서의 user threads와 kernel에서의 kernel thread로 보도록 하자.
user thread는 kernel 위에서 수행되며 kernel의 support가 없는 반면, kernel thread는 operating system이 직접 관리 지원한다.
user thread와 kernel thread의 관계가 정의되는 방법 중 일반적인 3가지로 다룰 것이다.

### 4.3.1 Many-to-One Model
 thread management를 user space에서 thread library를 통해서만 하고, 실제로는 여러 스레드를 생성하더라도 하나의 kernel thread로 연결된다. 스레드 관리는 효율적이나 코어에서의 병렬처리는 불가능하다. 거의 쓰이지 않는다.

### 4.3.2 One-to-One Model
 각각의 user thread가 하나의 kernel thraed로 매핑된다. 멀티프로세서의 병렬처리를 잘 사용할 수 있고, 더 높은 동시성을 제공한다. 하나의 단점은 kernel thread를 매번 하나씩 생성하는 것은 performance에 부담을 줄 수 있다는 것이다. 그래서 이 모델을 구현한 대부분은 시스템이 지원하는 스레드 수에 제한을 둔다.

### 4.3.3 Many-to-Many Model
 user-level thread의 수와 같거나 더 적은 수의 kernel thread가 연결된다. 높은 concurrency를 지원하면서도 one-to-one처럼 kernel thread가 너무 많아지지 않도록 application에서 지정한 수 만큼의 kernel thread를 활용한다.
 variation으로 two-level model이라고 해서 many-to-many model과 one-to-one model을 동시에 사용하는 것도 있다. 솔라리스 9 이전버전에선 two-level model을 사용했지만, 이후 one-to-one model로 바뀌었다.

## 4.4 Thread Libraries
 thread libary는 개발자가 스레드 생성/관리를 하기 위한 제공하는 API를 제공한다. 스레드 라이브러리를 구현하는 방법은 크게 보면 2가지로 나눌 수 있다.
 첫 째는 kernel support 없이 user space만으로 라이브러리를 제공하는 것이다. 모든 코드와 자료구조 등이 user space에 있기 때문에 모든 function call이 user space에 있고, system call은 없다.
 두 번째 접근 방법은 kernel-level libary가 있다. kernel space와 system call을 포함한다.
 요즘 사용되는 주요 스레드 라이브러리 3개를 소개해보려고 한다: POSIX Pthreads, Windows, and Java.
 
 그 전에 asynchronous threading vs synchronous threading 비교.
 ##### asynchronous threading
  자식 thread를 생성하는 데, 부모 thread와 independent하게 수행되는 경우.

 ##### synchronous threading
  자식 thread가 생성되면, 부모 thread는 자식 thread의 완료를 기다린다. 여러 자식을 생성하고, 그 자식(?)들이 끝나기를 기다리는 전략을 fork-join이라 부른다.
  뒤의 예시들은 synchronous threading 만이 나올 것이다.

### 4.4.1 Pthreads
 Pthreads라고만 하면 POSIX standard (IEEE 1003.1c)를 따르는 specification이다. 여러 OS designer들은 이 specification을 따라 implementation되어 있다.

A separate thread is created with the pthread create() function call.
At this point, the program has two threads: the initial (or parent) thread in main() and the summation (or child) thread performing the summation operation in the runner() function. This program follows the fork-join strategy described earlier.

/* get the default attributes */
pthread attr init(&attr);
/* create the thread */
pthread create(&tid,&attr,runner,argv[1]);
/* wait for the thread to exit */
pthread join(tid,NULL);

### 4.4.2 Windows Threads
The technique for creating threads using theWindows thread library is similar to the Pthreads technique in several ways.
진짜 거의 다를게 없다.
여러 스레드를 실행시키고, fork-join 전략으로 기다리는 경우라면 아래와 같다.
In situations that require waiting for multiple threads to complete, the WaitForMultipleObjects() function is used. This function is passed four parameters:
1. The number of objects to wait for
2. A pointer to the array of objects
3. A flag indicating whether all objects have been signaled
4. A timeout duration (or INFINITE)

### 4.4.3 Java Threads
모든 자바 프로그램은 최소 스레드 하나를 main() method로 JVM에서 사용한다. 그 외에 스레드 생성하는 방법이 두가지가 있다.
1. Thread 클래스를 상속 받아서 run() method를 override하거나
2. (일반적으로) Runnable interface를 구현한 클래스를 사용하거나
Creating a Thread object does not specifically create the new thread; rather, the start() method creates the new thread. Calling the start() method for the new object does two things:
1. It allocates memory and initializes a new thread in the JVM.
2. It calls the run() method, making the thread eligible to be run by the JVM.
(Note again that we never call the run() method directly. Rather, we call the start() method, and it calls the run() method on our behalf.)

## 4.5 Implicit Threading
멀티코어 프로세싱의 성장에 따라 application이 쓰는 스레드는 수백개에서 수천개까지 늘어났다. 그러나 제대로된 병렬 프로그래밍을 하는 것은 매우 어렵다. 그래서 application 개발들이 multi threading을 관리하는 것을 컴파일러와 run-time 라이브러리들에 맡기는 설계의 변화가 일어나고 있다. 우린 이것은 implicit threading이라고 일컫는다.

### 4.5.1 Thread Pools
 4.1 section에서 우리는 multithreaded web server 구현을 위해 매 request마다 thread를 생성했다. 여기엔 두가지 문제가 있다. 매번 thread를 생성하는 걸리는 시간문제가 있고, 두번째는 조금 더 문제가 되는 것으로 무한정 스레드를 만들어주다보면, system resource에 문제가 생길 것이다. 이 문제애 대한 solution으로 우리는 thread pool을 사용한다.
 Thread pools은 아래와 같은 이점을 제공한다.
1. Servicing a request with an existing thread is faster than waiting to create a thread.
2. A thread pool limits the number of threads that exist at any one point. This is particularly important on systems that cannot support a large number of concurrent threads.
3. task 생성에 대한 업무를 기존 다른 작업들과 분리해냄으로써, task 생성에 대한 다른 strategy들을 적용할 수도 있다. 예를 들면, task들이 일정 시간 지연 후에 수행되도록 한다던가, 일정 기간을 두고 스케줄링 되게 하거나처럼 말이다.

### 4.5.2 OpenMP
OpenMP는 C, C++ 또는 FORTRAN로 작성한 API이며, 컴파일러 지시어 집합이다. OpenMP는 share-momory 환경을 사용하여 병렬 프로그래밍을 지원한다. OpenMP에 코드 블럭을 병렬 지역(parellel regions)로 지정하면, 해당 코드는 병렬로 수행된다.

### 4.5.3 Grand Central Dispatch
Grand Central Dispatch (GCD)—a technology for Apple’s Mac OS X and iOS operating systems—is a combination of extensions to the C language, an API, and a run-time library that allows application developers to identify sections of code to run in parallel. Like OpenMP, GCD manages most of the details of threading.
GCD identifies extensions to the C and C++ languages known as blocks. A block is simply a self-contained unit of work.

### 4.5.4 Other Approaches
Other commercial approaches include parallel and concurrent libraries, such as Intel’s Threading Building Blocks (TBB) and several products fromMicrosoft.
Java 언어와 API는 동시성 프로그래밍에 대한 지원을 대폭 늘려나가고 있다. 예시로 java.util.concurrent
를 들 수 있는데, 이것은 암묵적 스레드(implicit thread) 생성 및 관리를 지원하는 package이다.


## 4.6 Threading Issues
### 4.6.1 The fork() and exec() System Calls
### 4.6.2 Signal Handling
### 4.6.3 Thread Cancellation
### 4.6.4 Thread-Local Storage
### 4.6.5 Scheduler Activations


## 4.7 Ooperating-System Examples


## 4.8 Summary
Athread is a flow of control within a process.Amultithreaded process contains several different flows of controlwithin the same address space.
User-level thread는 kernel의 관여가 없으므로 생성/변경이 훨씬 빠르다. 하지만 실제 병렬 처리를 위해서는 kernel thread와 연결되어야 하며, 그 관계를 3가지로 설명한다. (many-to-one, one-to-one, many-to-many)
모던 OS들은 스레드 지원을 kernel에서 다 한다. 
그리고 application 개발자들이 thread를 사용하기 위한 thread libary는 크게 POSIX Pthreads, Windows threads, Java threads가 일반적이다.
명시적으로 스레드 생성을 위해 thread library를 쓰는 것 외에도, 암묵적인 방법을 사용할 수 있다. thread management를 컴파일러 런타임 라이브러리에게 맡기는 방법으로 thread pools, OpenMP, Grand Central Dispatch를 사용한 방법을 알아봤다.
그 외에 멀티스레드 프로그램 개발자들은 알아야할 내용으로 fork(), exec()의 의미와 signal handling, thread cancellation, thread-local storage, schedul activations과 같은 이슈들이 있다.