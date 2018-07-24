# 5. Process Synchronization

정리를 못해서.. 일단 메모해둔 것들만 써놓음.

## 5.4 Synchronization Hardware

test_and_set

Bounded-waiting mutual exclusion with test_and_set()

### 5.5 Mutex Locks

application 개발자가 쓰라고, OS designer가 만들어준 것.
acquire()
release()

busy waiting 문제가 있음. spin lock 이라고도 함.
context switching을 일으키는 면에서는 효율적이라고 볼 수 있음.

## 5.6 Semaphores

Busy waiting이 필요 없는 sychronization tool.

wait : 세모포어 변수를 검사해서 lock
signal : 세마포어를 놔줌.

- counting semaphore
- binary semaphore

busy waiting 문제가 생길 수 있음.
block이랑 wakeup?

### 5.6.3. Deadlocks and Starvation

### 5.6.4. Priority Inversion

L->H로 돌고 있는데, M이 들어오면, H가 더 기다려야되nitor
priority-inheritance protocol

## 5.7 Classic Problems of Sychronization

### 5.7.1. The bouded-Buffer Problem

Figure 5.9. The structure of the producer process.
Figure 5.10.

### 5.7.2 The Readers-Writers Problem

writer를 위한 rw_mutex

Figure 5.12.
read_count 때문에 mutex를 쓰고,
read만 할 떄는 parellel이어도 상관 없다.

### 5.7.3.

hungry eating thinking
deadlock은 제거할 수 있는데, starvation은 일어날 수 있다.

## 5.8 Monitors

세마포어를 쓰기는 쉽지만 문제가 발생할 수 있다.

- 순서가 바뀌는거
- 호출을 계속 하는거
- 동시에 호출하는거

### 5.8.1 Monitor Usage

monitor type