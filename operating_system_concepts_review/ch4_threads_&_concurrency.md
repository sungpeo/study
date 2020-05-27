# Chapter 4. 스레드와 병행성

# 이 장의 목표

1. 스레드의 기본 구성요소를 식별하고 스레드와 프로세스를 대조한다.
2. 다중 스레드 프로세스를 설계할 때의 주요 이점과 중대한 과제를 설명한다.
3. 스레드 풀, 포크 조인 및 그랜드 센트럴 디스패치를 포함하여 암시적 스레딩에 대한 다양한 접근 방식을 설명한다.
4. Windows 및 Linux 운영체제가 스레드를 어떻게 나타내는지 설명한다.
5. Pthread, Java 및 Windows 스레딩 API를 사용하여 다중 스레드 응용 프로그램을 설계한다.

## 1. 스레드의 기본 구성요소를 식별하고 스레드와 프로세스를 대조한다.

(같은 프로세스 내) 스레드 공유 속성:

- 코드
- 데이터 섹션
- 열린 파일이나 신호와 같은 운영체제 자원

스레드 개별 속성:

- 스레드 ID
- 프로그램 카운터(PC)
- 레지스터 집합
- 스택

## 2. 다중 스레드 프로세스를 설계할 때의 주요 이점과 중대한 과제를 설명한다.

### 장점 _Benefits

크게 4가지로 나눌 수 있다.

  1. __응답성(responsiveness)__ : 대화형 프로그램에서 사용자 인터페이스와 긴 시간이 걸리는 연산을 비동기 스레드로 분리하여 높은 응답성을 얻을 수 있다.
  2. __자원 공유(resource sharing)__ : 프로세스 간 자원 공유는 IPC로 부하가 발생하지만, 스레드는 같은 주소 공간 내에서 코드와 데이터를 공유하는 이점을 살릴 수 있다.
  3. __경제성(economy)__ : 프로세스 생성을 위해 메모리와 자원을 할당하는 것은 비용이 많이 든다. 또한 context switching도 일반적으로 프로세스보다 스레드 간에서 더 빠르다.
  4. __규모 적응성(scalability)__ : 다중 스레드를 통해 multi core인 경우 병렬 수행의 이점을 얻을 수 있다.

### 프로그래밍 도전과제 _Programming Challenges

  1. __identifying tasks__ : 서로 독립되어 병행 처리 가능하도록 태스크를 분리, 정의하는 것이 필요하다.
  2. __balance__ : 분리된 태스크들이 균등한 기여도를 가지도록 나누는 것도 매우 중요하다.
  3. __data spliting__ : 태스크로 나누어지 듯, 태스크가 접근/처리해야하는 데이터 또한 개별 코어에서 사용 가능토록 나누어져야 한다.
  4. __data dependency__ : 태스크들이 접근하는 데이터에 대한 태스크 종속성이 있는 지 검토하여, 종속성이 있는 경우 적절한 동기화가 필요하다.
  5. __testing and debugging__ : 다중 코어에서 병렬 실행되기에 다양한 실행 경로가 존재하여 테스트, 디버깅이 어렵다.

## 3. 스레드 풀, 포크 조인 및 그랜드 센트럴 디스패치를 포함하여 암시적 스레딩에 대한 다양한 접근 방식을 설명한다.

### 암묵적 스레딩 _Implicit Threading

스레드를 쉽고 올바르게 사용하기 위해 스레딩의 생성과 관리 책임을 컴파일러와 런타임 라이브러리에게 넘기는 전략을 암묵적 스레딩이라 칭한다.

#### 3-1. Thread Pool

무한히 스레드를 만들다보면 시스템 자원 고갈로 인한 문제가 일어날 수 밖에 없다. 이런 문제를 막기 위해 일정한 수의 스레드를 미리 풀로 만들어 사용할 수 있겠다.

스레드 풀의 장점은 다음과 같다.

  1. 기존 스레드를 서비스해주는 것이 새로 만드는 것보다 종종 더 빠르다.
  2. 스레드 개수의 제한을 두어 시스템에서 사용하는 스레드 수를 제어할 수 있다.
  3. 태스크 생성을 분리함으로써 실행을 다르게 할 수 있다. (일정 시간 후 실행, 주기적으로 실행)

##### Java Thread Pools

1. Executor.newSingleThreadExecutor() : 크기가 1인 풀을 생성
2. Executor.newFixedThreadPool(int size) : size 크기의 스레드 풀을 생성
3. Executor.newCachedThreadPool() : 스레드를 재사용하는 무제한 스레드 풀을 생성

#### 3-2. Fork Join

부모 스레드가 하나 이상의 자식 스레드를 생성(fork)한 다음 자식의 종료를 기다린 후 join하고 그 시점부터 자식의 결과를 확인하고 결합한다.

라이브러리는 생성되는 스레드 수를 관리하며 작업 배정을 하기에 동기 버전의 스레드 풀이라 볼 수 있다.

Java 1.7에서 새로운 기본 풀인 ForkJoinPool을 도입하여 abstract class ForkJoinTask를 상속하는 작업을 배정 받을 수 있다. invoke를 통해 초기 작업 제출.

``` java
ForkJoinPool pool = new ForkJoinPool();

// 합산될 정수를 포함하는 배열
int[] array = new int[SIZE];

// SumTask는 ForkJoinTask를 상속 구현
SumTask task = new SumTask(0, SIZE -1, array);
int sum = pool.invoke(task);
```

Java fork-join 전략은 추상 기본 클래스 ForkJoinTask를 중심으로 구성되며 RecursiveTask 및 RecursiveAction 클래스는 이 클래스를 확장한다.
이 두 클래스의 근본적인 차이점은 RecursiveTask가 compute()에 지정된 반환 값을 통해 결과를 반환하고 RecursiveAction은 결과를 반환하지 않는다는 것이다.

아래는 fork-join을 사용하여 배열의 내용을 합산하는 분할-정복 알고리즘 구현 예시이다.

``` java
import java.util.concurrent.*;

public class SumTask extends RecursiveTask<Integer> {
  // "충분히" 작은 크기여서 더 이상 태스크를 만들지 않도록 하는 기준
  static final int THRESHOLD = 1000;

  private int begin;
  private int end;
  private int[] array;

  public SumTask(int begin, int end, int[] array) {
    this.begin = begin;
    this.end = end;
    this.array = array;
  }

  protected Integer compute() {
    if (end - begin < THRESHOLD) {
      int sum = 0;
      for (int i = begin; i <= end; i++)
        sum += array[i];

      return sum;
    } else {
      int mid = (begin + end) / 2;

      SumTask leftTask = new SumTask(begin, mid, array);
      SumTask rightTask = new SumTask(mid + 1, end, array);

      leftTask.fork();
      rightTask.fork();

      return rightTask.join() + leftTask.join();
    }
  }
}
```

java의 fork-join 모델의 흥미로운 점은 라이브러리가 작업자 스레드 풀을 생성하고 사용 가능한 작업자 간 부하의 균형을 조정하는 작업 관리에 있다.

ForkJoinPool의 각 스레드는 fork된 작업의 큐를 유지 관리하며,
스레드의 큐가 비어 있으면 work stealing 알고리즘을 사용하여 다른 스레드 큐에서 작업을 가져올 수 있으므로 모든 스레드 간에 부하를 분산시킬 수 있다.

#### 3-3. Grand Central Dispatch (GCD)

macOS 및 iOS를 위해 Apple에서 개발한 기술이다.

GCD는 런타임 실행을 위해 __디스패치 큐__에 넣어서 스케줄하는데, 디스패치 큐는 2가지로 나눠진다.

  1. 개인 디스패치 큐 (메인 큐): 각 프로세스별 고유한 직렬 큐를 통해 serial 작업을 수행한다.
  2. 전역 디스패치 큐 (병행 큐): FIFO이지만 한번에 여러 태스크를 제거하여 병렬 실행 가능하며 4가지 주요 서비스 품질 클래스로 나뉜다.
    - QOS_CLASS_USER_INTERACTIVE: 사용자 대화형: 아주 빨리 적은 양의 작업만 해야한다.
    - QOS_CLASS_USER_INITIATED: 사용자 시작: 인터페이스와 관련 있으나 파일 또는 URL을 여는 것과 같은 작업으로 사용자 대화형 큐만큼 빨리 서비스될 필요는 없다.
    - QOS_CLASS_UTILITY: 유틸리티: 시간이 오래 걸리지만 즉각적인 결과를 요구하지 않는 태스크를 나타낸다.
    - QOS_CLASS_BACKGROUND: 백그라운드: 사용자에게 보이지 않으며 시간에 민감하지 않다. ex) 메일박스 시스템 색인 만들기, 백업 수행

## 4. Windows 및 Linux 운영체제가 스레드를 어떻게 나타내는지 설명한다.

Linux는 Windows 운영체제 제품군과 함꼐 일대일 모델을 구현한다.

### 일대일 모델 _One-to-One Model

각 사용자 스레드를 각각 하나의 커널 스레드로 매핑한다.

#### Windows Threads

스레드 구성 요소
- 스레드 ID
- 처리기의 상태를 나타내는 레지스터 집합
- 프로그램 카운터
- 사용자 모드에서 실행될 때 필요한 사용자 스택, 커널 모드에서 실행될 때 필요한 커널 스택
- 런타임 라이브러리와 동적 링크 라이브러리(DLL) 등이 사용하는 개별 데이터 저장 영역

스레드 주요 자료구조
- ETHREAD: executive thread block
- KTHREAD: kernel thread block
- TEB: thread environment block

#### Linux Threads

Linux 는 프로세스나 스레드보다 태스크라는 용어를 사용한다.

- fork(): 프로세스를 복제하는 기능(프로세스 레벨에서 태스크를 복사)
- clone(): 부모와 자식 태스크가 자료 구조를 얼마나 공유할지 지정하여 태스크를 복사(공유되는 정보가 많은 스레드 레벨로 태스크가 복사된다고 볼 수 있음)
  - CLONE_FS: 파일 시스템 정보 공유
  - CLONE_VM: 같은 메모리 공간 공유
  - CLONE_SIGHAND: 신호 처리기 공유
  - CLONE_FILES: 열린 파일의 집합 공유

## 5. Pthread, Java 및 Windows 스레딩 API를 사용하여 다중 스레드 응용 프로그램을 설계한다.

### Pthreads

Pthreads는 POSIX 가 스레드 생성과 동기화를 위해 제정한 표준 API이다.

Linux와 macOS를 포함한 많은 시스템이 Pthreads 명세를 구현하고 있으며 Windows는 자체적으로는 지원하지 않지만 타사가 구현한 버전을 구할 수 있다.

다음 C언어로 작성된 음이 아닌 정수의 합을 구하는 다중 스레드 프로그램(Pthreads API를 사용)을 보자.

이 예제는 단순하게 계산만을 수행하는 하나의 스레드를 생성하고 종료되면 결과를 받아 출력한다.

``` c
#include <pthread.h>
#include <stdio.h>

#include <stdlib.h>

int sum; /* this data is shared by the threads */
void *runner(void *param); /* threads call this function */

int main(int argc, char *argv[])
{
  pthread_t tid; /* the thread identifier */
  pthread_attr attr; /* set of thread attributes */

  /* set the default attributes of the thread */
  pthread_attr_init(&attr);

  /* create the thread */
  /* 새로운 스레드가 실행할 함수 runner(), 명령어 라인 상에 제공된 정수 매개변수 argv[1] */
  pthread_create(&tid, &attr, runner, argv[1]);

  /* wait for the thread to exit */
  pthread_join(tid, NULL);

  printf("sum = %d\n", sum);
}

/* The thread will execute in this function */
void *runner(void *param)
{
  int i, upper = atoi(param);
  sum = 0;

  for (i = 1; i <= upper; i++)
    sum += i

  pthread_exit(0);
}
```

10개의 스레드를 조인하는 Pthread 코드

``` c
#define NUM_THREADS 10

/* an array of threads to be joined upon */
pthread_t workers[NUM_THREADS];

for (int i = 0; i < NUM_THREADS; i++)
  pthread_join(workers[i], NULL);
```

### ~~Windows Threads~~

### Java Thread

1. Thread 클래스를 상속하여 새 클래스를 만들고 run() 메서드를 재정의
2. Runnable 인터페이스를 구현하는 클래스를 정의

2번째 방법이 일반적으로 사용되며 예시로 살펴보자.

``` java
class Task implements Runnable
{
  public void run() {
    System.out.println("I am a thread.");
  }
}
```

Runnable을 구현한 클래스를 Thread 생성자의 파라미터로 전달하여 만든 객체를 start하여 스레드를 생성한다.

``` java
Thread worker = new Thread(new Task());
worker.start();
```

start() 메서드 호출하면 두가지 작업을 수행한다.

  1. 메모리가 할당되고, JVM 내에 새로운 스레드가 초기화된다.
  2. run() 메서드를 호출하면 스레드가 JVM에 의해 수행될 자격을 갖게된다. ( start()를 호출하여 이 메서드가 우리 대신 run()을 호출하도록 하라)

#### Java Executor Framework

Java 1.5부터 새로운 병행 처리 기능을 도입하였다. java.util.concurrent 패키지를 통해 제공되는 Executor 프레임워크이다.

그리고 java.util.concurrent.Callable 인터페이스를 추가로 정의하여 Runnable과 유사하게 작동하나 결과를 반환할 수 있도록 하였다. Callableㅇ 인터페이스의 call() 메서드의 코드가 별도의 스레드에서 실행된다.
Callable 작업에서 반환된 결과를 Future 객체라 한다.

``` java
import java.util.concurrent.*;

class Summation implements Callable<Integer> {
  private int upper;
  public Summation(int upper) {
    this.upper = upper;
  }

  /* The thread will execute in this method */
  public Integer call() {
    int sum = 0;
    for (int i=1; i <= upper; i++)
      sum += i;

    return new Integer(sum);
  }
}

public class Driver {
  public static void main(String[] args) {
    int upper = Integer.parseInt(args[0]);

    ExecutorService pool = Execcutors.newSingleThreadExecutor();
    Future<Integer> result = pool.submit(new Summation(upper));

    try {
      System.out.println("sum = " + result.get());
    } catch (InterruptedException | ExecutionException ie) { }
  }
}
```

이 방법은 Callable 및 Future를 사용해서 세르드의 결과를 반환하여 사용할 수 있었다.

또한 스레드 생성과 스레드가 만든 결과를 분리함으로 다른 기능과 결합하게 좋은 구조를 가진다.
