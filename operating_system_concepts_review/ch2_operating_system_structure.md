# Chapter 2. 운영체제 구조

`이 장의 목표`에 대한 답변을 제시하는 형태로 내용을 요약 정리해본다.

# 이 장의 목표

1. 운영체제에서 제공하는 서비스를 식별한다.

2. 운영체제 서비스를 제공하기 위해 시스템 콜을 사용하는 방법을 설명한다.

3. 운영체제 설계를 위한 모놀리식, 계층화, 마이크로 커널, 모듈 및 하이브리드 전략을 비교 및 대조한다.

4. 운영체제 부팅 프로세스를 설명한다.

5. 운영체제 성능을 모니터링하기 위한 도구를 적용한다.

6. Linux 커널과 상호 작용하기 위한 커널 모듈을 설계하고 구현한다.



## 1. 운영체제에서 제공하는 서비스를 식별한다.

운영체제마다 제공하는 서비스는 다르지만, 이해를 돕기 위해 역할별로 구조적인 분류를 해보고자 한다.

먼저 크게 2 분류로 다음과 같이 나눠볼 수 있다.
1. 프로그래밍이 원활하게 진행되도록 지원하는 서비스
2. 시스템 자체의 효율적인 동작을 보장하기 위해 제공하는 서비스

 그럼, 이제 조금 더 구체적으로 나타내 보자.

### 1. 프로그래밍이 원활하게 진행되도록 지원하는 서비스

- __사용자 인터페이스__: 대부분 `사용자 인터페이스(UI)`가 제공되며, 일반적인 PC에 사용되는 마우스를 활용하는 `그래픽 사용자 인터페이스(GUI)`가 있다. 휴대 전화 및 태블릿과 같은 모바일 시스템은 `터치 스크린 인터페이스`를 제공한다. 또 다른 옵션으로 `명령어 라인 인터페이스(CLI)`가 있다.

- __프로그램 수행__: 시스템은 프로그램을 메모리에 적재해서 실행할 수 있어야 한다. 프로그램은 정상적이든 비정상적(오류를 표시하면서)이든 실행을 끝낼 수 있어야 한다.

- __입출력 연산__: 시스템을 사용자가 쓰려면 당연히 입출력 작업이 동반될 수 밖에 없다. 마우스, 키보드, 모니터와 같은 입출력 장치를 통한 동작 외에도 파일 시스템에 대한 읽기/쓰기, 네트워크 인터페이스를 통한 읽기/쓰기 등이 모두 포함되는 연산을 의미한다. 이 작업이 효율적이면서도 안정적으로 수행되기 위해 통상적으로 사용자들은 입출력 장치를 직접 제어할 수 없고 운영체제를 통하도록 한다.

- __파일 시스템 조작__: 파일 시스템과의 입출력은 당연히 필요하고, 그 것 외에도 관련된 기능이 많다. 파일 이름 지정, 생성/삭제, 검색, 파일에 대한 메타 정보와 소유권에 기반한 권한 관리 등의 역할을 수행하고 있다. 많은 운영체제들은 개인의 선택에 따라 다양한 특성을 가진 파일 시스템을 제공한다.

- __통신(communication)__: 프로세스와 프로세스 간의 정보를 교환하는 일련의 과정을 통신이라 한다. 동일한 컴퓨터 내에서의 통신과 네트워크 상의 다른 컴퓨터의 프로세스와도 통신이 일어난다. `공유 메모리`를 통해서 구현될 수도 있고, `메시지 전달` 기법을 사용하여 구현될 수도 있다.

- __오류 탐지__: 오류를 거의 모든 곳에서 발생할 수 있다. CPU, 메모리 하드웨어, 입출력 장치, 사용자 프로그램 등. 운영체제는 올바르고 일관성 있는 계산을 보장하기 위해 최대한 모든 오류를 의식하기 위해 노력해야 하며 그에 대한 적당한 조치를 해야 한다.

### 2. 시스템 자체의 효율적인 동작을 보장하기 위해 제공하는 서비스

- __자원 할당(resource allocation)__: 운영체제가 관리해야 하는 자원으로 CPU 사이클, 메인 메모리, 파일 저장 장치와 같이 다양한 스케줄링 알고리즘을 활용하는 것과 훨씬 일반적인 형태로 사용하는 입출력 장치 등이 있다. 다수의 프로세스나 다수의 작업이 동시에 실행된다면 운영체제는 그들 각각에 지정된 어떤 규칙에 의해 자원을 할당하게 되는 것이다.

- __기록 작성(logging)__: 어떤 프로그램이 어떤 자원을 어떻게 사용했는지와 같은 정보를 추적하고자 기록을 작성한다. (로그를 남긴다.) 이는 단순한 사용 통계를 내거나, 수행된 작업에 대한 감시/감사(audit)가 같은 형태로도 활용될 수 있다.

- __보호(protection)과 보안(security)__: 다중 사용자 컴퓨터의 경우 사용자간의 정보가 통제되길 원할 것이다. 또한 서로 다른 여러 프로세스가 병행 수행되면서 서로에게 영향을 미치거나 운영체제 자체를 방해해서는 안된다. 이러한 시스템 자원에 대한 모든 접근을 통제 보장하는 `보호`가 필요하며, 외부로부터의 `시스템 보안` 역시 중요하다.


## 2. 운영체제 서비스를 제공하기 위해 시스템 콜을 사용하는 방법을 설명한다.

`시스템 콜`이란 위에서 설명한 서비스들에 대해 사용 가능한 인터페이스들을 말하며, 사용 자체를 의미하기도 한다. (ex. 시스템 콜을 한다.)

먼저, Unix 명령(cp)를 사용하는 예시를 통해 시스템 콜이 어떻게 사용되는 지 알아보자.

``` sh
cp in.txt out.txt
```
이 명령은 입력 파일 in.txt를 출력 파일 out.txt로 복사한다.
이 작업을 시스템콜로 풀어보면 다음과 같다.

1. 입력 파일 in.txt 열기
 1-1. 해당 파일이 존재하지 않을 경우, 비정상적으로 종료
2. 출력 파일 생성
 2-1. 해당 파일이 존재할 경우, 덮어 쓰기
3. 루프
 3-1. 입력 파일로부터 읽어 들임
  3-1-1. 읽기에 실패하면 루프 탈출
 3-2. 출력 파일에 씀
4. 출력 파일 닫기
5. 화면에 완료 메시지 출력
6. 정상적으로 종료

위에서 살펴본 것처럼 하나의 cp 하나의 명령을 위해 동반되는 시스템 콜이 아주 많다. 그래서 사용하기 편리하도록 Unix 명령처럼 응용프로그램에서 사용하기 좋은 기능들을 모아서 인터페이스를 미리 구축해두어 활용한다. 이를 응용 프로그램 인터페이스(Application Program Inteface) `API`라 한다.
- Windows API, POSIX API, Java API 등


## 3. 운영체제 설계를 위한 모놀리식, 계층화, 마이크로 커널, 모듈 및 하이브리드 전략을 비교 및 대조한다.

### 모놀리식 구조 _Monolithic Structure

커널의 모든 기능을 단일 주소 공간에서 실행되는 단일 정적 이진 파일에 넣는 구조이다.
그렇기 때문에 한 부분의 변경이 광범위한 영향을 줄 수 있는 밀접하게 결한된 시스템 구조이다.

시스템 콜 인터페이스에 오버헤드가 거의 없고 커널 안에서의 통신 속도가 빠르다.


### 계층적 접근 _Layered Approach

모놀리식 구조의 결합 구조로 인한 단점을 극복하기 위해 느슨하게 결합된 모듈화 방법 중 하나이다.
사용자 인터페이스를 최상위 층으로 하여 여러가지 계층을 가지며 하위 층을 인터페이스를 통해서만 사용하게 된다.
그러므로, 구현과 디버깅에 있어 문제가 되는 계층만을 대상으로 진행하므로 단순화되는 장점이 있다.

대신 계층별 오버헤드가 있어서 전반적인 성능 문제를 안고 있다.

### 마이크로 커널 _Microkernels

가장 초기엔 모놀리식 구조를 가졌고, 커널이 커지면서 관리하기가 힘들어지자 `마이크로커널` 방식이 등장하게 되었다. 말 그래도 주요 커널만으로 이뤄진 더 작은 커널을 구성하고, 나머지 구성요소들을 사용자 수준 프로그램으로 구현하는 것이다. 통상 마이크로커널엔 최소한의 프로세스와 메모리 관리를 제공한다. 
결과적으로 작은 커널은 변경할 대상이 작으므로 하드웨어 이식이 쉽고, 사용자 프로세스 분리된 서비스를 사용함에 따라 높은 보안과 신뢰성을 제공한다.

구조상 가중된 시스템 기능 오버헤드 때문에 성능이 나빠진다.

### 모듈 _Modules

운영체제를 설계하는 데 이용되는 최근 기술 중 최선책은 `적재가능 커널 모듈(loadable kernel modules, LKM)` 기법의 사용일 것이다. 이 접근법에서 커널은 핵심적인 구성요소의 집합을 가지고 있고 부팅 때 또는 실행 중에 부가적인 서비스들을 모듈을 통하여 링크할 수 있다.

전체적인 결과는 커널의 각 부분이 정의되고 보호된 인터페이스를 가진다는 점에서 계층 구조를 닮았다. 그러나 모듈에서 임의의 다른 모듈을 호출할 수 있다는 점에서 계층 구조보다 유연하다. 중심 모듈은 단지 핵심 기능만을 가지고 있고 다른 모듈의 적재 방법과 모듈들과 어떻게 통신하는지 안다는 점에서는 마이크로 커널과 유사하다. 그러나 통신하기 위해 메시지 전달을 호출할 필요가 없으므로 더 효율적이다.

### 하이브리드 _Hybrid Systems

결국 위에서 나타내는 설계 전략 중 하나만을 채택한 운영체제는 거의 존재하지 않는다. 

예를 들어, Linux는 운영체제 전부가 하나의 주소 공간에 존재하여 효율적인 성능을 제공하기 때문에 모놀리식 구조이다. 그러나 이 운영체제들은 모듈을 사용하기 때문에 새로운 기능을 동적으로 커널에 추가할 수 있다. 

## 4. 운영체제 부팅 프로세스를 설명한다.

커널을 적재하여 컴퓨터를 시작하는 과정을 시스템 `부팅`이라고 한다. 시스템 대부분에서 부팅 과정은 다음과 같다.

1. `부트스트랩 프로그램` 또는 `부트 로더`라고 불리는 작은 코드가 커널의 위치를 찾는다.
2. 커널의 메모리에 적재되고 시작된다.
3. 커널은 하드웨어를 초기화 한다.
4. 루트 파일 시스템이 마운트 된다.

1번 작업을 하는 데에는 기존 방식인 `BIOS`와 `UEFI`가 있고, 현재는 대부분 UEFI로 대체되었다.

공간을 절약하고 부팅 시간을 줄이기 위해 Linux 커널 이미지는 압축 파일이며 메ㅇ모리에 적재된 후 압축이 풀어진다. 부팅 과정에서 부트 로더는 일반적으로 `initramfs`로 알려진 임시 RAM 파일 시스템을 생성한다. 이 파일 시스템에 실제 루프 파일 시스템을 지원하기 위해 설치해야 하는 드라이버와 커널 모듈이 저장되어 있다. 커널이 시작되고 필요한 드라이버가 설치되면 커널은 루트 파일 시스템을 임시 RAM 위치에서 적절한 루트 파일 시스템으로 전환한다. 마지막으로 Linux는 시스템의 초기 프로세스인 systemd 프로세스를 생성한 다음 다른 서비스를 시작한다.

참고:
 
### UEFI _Unified Extensible Firmware Interface

부팅 관련 코드를 ROM이 아닌 하드 디스크에 .efi 파일로 저장한다. 그 파일은 EFI System Partition (ESP)라 불리는 특수 파티션 내에 위치하며 이 위치에 부트 로더 프로그램을 저장하고 있다. (BIOS 방식은 BIOS라는 '소형 부트 로더'가 디스크의 '초기 부트 로더'를 부트 블록에서 읽어내고 해당 부트 로더가 부트 스트랩 프로그램을 주소를 가리키는 방식으로 동작한다.) UEFI가 하나의 완전한 부팅 관리자로서 동작하기 때문에 더 빠르다. 그리고 64-bit GUID partition table (GPT)를 사용하기 때문에 BIOS의 Master Boot Record(MBR)보다 더 큰 사이즈의 파티션을 지원할 수 있다.

### GRUB _Grand Unified Bootloader

Linux 및 UNIX 시스템을 위한 공개 소스 부트스트랩 프로그램이다. 시스템의 부트 매개변수는 GRUB 구성 파일에 설정되며 GRUB의 실행 시작 시점에 적재된다. GRUB는 부팅 시 커널 매개변수를 수정하거나 부팅 가능한 다른 커널 중 하나를 선택하는 것이 가능하다.

## 5. 운영체제 성능을 모니터링하기 위한 도구를 적용한다.

### Counter

- ps: 프로세스에 대한 정보
- top: 현재 프로세스의 실시간 통계

- vmstat: 메모리 사용량 통계
- netstat: 네트워크 인터페이스에 대한 통계
- iostat: 디스크의 I/O 사용량

Linux 시스템의 카운터 기반 도구는 대부분 `/proc` 파일 시스템에 통계를 읽는다. (/proc는 커널 메모리에 존재하는 의사 파일 시스템)

### Tracing

추적 도구는 시스템 콜과 관련된 단계와 같은 특정 이벤트에 대한 데이터를 수집

- strace: 프로세스에 의해 호출된 시스템 콜 추적
- gdb: 소스 레벨 디버거

- perf: 리눅스 성능 도구 모음
- tcpdump: 네트워크 패킷 수집

## 6. Linux 커널과 상호 작용하기 위한 커널 모듈을 설계하고 구현한다.

### BCC _BPF Compiler Colllection

BCC is a toolkit for creating efficient kernel tracing and manipulation programs, and includes several useful tools and examples. It makes use of extended BPF (Berkeley Packet Filters), formally known as eBPF, a new feature that was first added to Linux 3.15. Much of what BCC uses requires Linux 4.1 and above

#### Tools:
<center><a href="images/bcc_tracing_tools_2019.png"><img src="images/bcc_tracing_tools_2019.png" border=0 width=700></a></center>







