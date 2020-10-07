# OS(운영체제)

## 프로세스와 쓰레드

**프로세스:** 메모리 위에 올라가서 실제로 실행되는 프로그램을 의미한다. 
(사용되는 파일, 데이터, 메모리 영역, 주소 공간 포함)

**쓰레드:** 프로세스 내에서 실행되는 작업의 단위를 나타낸다.



## 프로세스의 구조

![img](https://media.vlpt.us/post-images/pa324/06db2c70-1242-11ea-9735-834e8b45e32e/image.png)

- Data Segment

    전역 변수와 static 변수

    BSS영역: 초기화 되지 않은 데이터

    Data영역: 초기화 된 변수 

- Text Segment(Code)

    읽기 전용, heap과 stack의 오버플로우를 대비해서 생성

- Heap

    동적으로 할당한 데이터 (malloc)

    메모리 주소가 낮은 영역부터 채워진다.

- Stack

    프로그램의 지역변수, return address, 함수

    메모리 주소가 높은 영역에서 채워진다. 



## 프로세스의 생성 과정

1. PCB가 생성되며 OS가 실행한 프로그램의 코드를 읽어들여 프로세스에 할당된 메모리의 Text Segment에 저장한다. 
2. 초기화 된 전역변수와 static변수는 Data영역에 저장한다. 
3. Heap과 Stack은 초기 메모리 주소만 초기화 된다. 
4. PCB에 여러 정보가 기록되면 Ready Queue에서 CPU를 할당받기까지 대기한다.



## PCB(Process Control Block)

PCB는 각 프로세스 생성과 동시에 생성되며 프로세스의 정보를 가지고 있는 자료구조이다.

Context Switching이 일어날 때 프로세스의 정보를 PCB에 저장한다. 

**PCB에 저장되는 정보**

- PID
- Process 상태 (ex: new, ready, waiting, running, terminated)
- Program Counter
- CPU 스케줄링 정보
- CPU 레지스터
- 메모리 관리 정보: 페이지 테이블, 세그먼트 테이블 같은 정보
- 어카운팅 정보



## 멀티 쓰레드

**장점:** 멀티 프로세스에 비해서 메모리 공간이 줄어든다. 그 이유는 쓰레드는 static영역 및 heap영역을 공유하기 때문이다. 그래서 context switching의 오버헤드가 줄어든다. 

**단점:** 데이터를 공유하기 때문에 동기화 문제가 발생할 수 있다. 과도한 락으로 인한 성능 저하 문제가 발생할 수 있다. 

### 멀티 프로세싱 vs 멀티 쓰레드

멀티 쓰레드가 context switching이 더 빠르고 메모리가 더 적지만, 오류로 인한 하나의 쓰레드가 종료되면 모든 쓰레드가 죽을 수 있다. 그리고 동기화 문제 역시 존재한다. 

멀티 프로세스는 프로세스간 공유하는 데이터가 없기 때문에 하나의 프로세스가 다른 프로세스에게 영향을 주지 않지만 쓰레드보다 context switching의 오버헤드가 크고 메모리 공간을 더 많이 차지한다. 



## 스케줄러

프로세스를 스케줄링 하기 위한 Queue에는 세가지 종류가 존재한다. 

**Job Queue :** 현재 시스템 내에 있는 모든 프로세스의 집합

**Ready Queue:** 현재 메모리 내에 있으면서 CPU가 잡아서 실행되기를 기다리는 집합

**Device Queue:** Device I/O 작업을 기다리고 있는 프로세스의 집합

프로세스를 넣고 빼주는 스케줄러에도 3가지가 존재한다. 

**Long-term Scheduler(Job Scheduler)**

한정된 메모리에 프로세스가 여러개 올라갈 경우, 대용량 메모리(디스크)에 임시로 저장된다. 

이 pool에 저장되어 있는 프로세스 중에 어떤 것을 Ready Queue로 올릴지 결정하는 역할을 한다. 

- 메모리와 디스크 사이의 스케줄링 담당
- 프로세스에 메모리(및 각종 리소스)를 할당
- degree of Multiprogramming 제어 (실행중인 프로세스의 수 제어)
- 프로세스 상태

    new → ready(in memory)

**Short-term Scheduler(CPU Scheduler)**

- CPU와 메모리 사이의 스케줄링 담당
- Ready Queue에 있는 프로세스 중에 어떤 것을 running 시킬 지 결정
- 프로세스에 CPU를 할당(Scheduler Dispatcher)
- 프로세스의 상태

    ready → running → waiting → ready

**Medium-term Scheduler(Swapper)**

- 여유 공간을 위해서 프로세스를 통째로 디스크로 쫓아냄 (swapping)
- 프로세스에게서 메모리를 deallocate
- degree of Multiprogramming 제어
- 너무 많은 프로그램이 동시에 메모리에 올라가는 것을 제어
- 프로세스의 상태

    ready → suspended

## CPU 스케줄러

*스케줄링 알고리즘은 여러가지 있는데 전부 Ready Queue에 있는 것이 대상이 된다.*

### 1. FCFS (First Come First Served)

처음 들어온 프로세스가 다 처리될 때까지 나머지 프로세스는 Ready Queue에서 대기하고 순차적으로 처리된다. 

- 비선점형이다.
- (Convoy 효과) CPU Burst가 긴 프로세스가 오면 효율성이 떨어진다.

### 2. SJF (Shortest Job First)

CPU burst가 짧은 프로세스에 선할당 된다.

- 비선점형이다.
- (startvation) CPU burst가 긴 프로세스는 자꾸 순위가 밀릴 수 있다.

### 3. SRTF (Shortest Remaining Time First)

새로운 프로세스가 들어왔을 때 남은 CPU Burst가 짧은 것부터 먼저 처리된다.

프로세스가 들어왔을 때마다 스케줄링이 된다.

- 선점형이다.
- Starvation
- 새로운 프로세스가 도착할 때마다 스케줄링을 다시하기 때문에 CPU Burst Time을 측정할 수 없다.

### 4. Priority Scheduling

미리 주어진 우선순위에 따라 스케줄링을 한다. 숫자가 작을수록 우선순위가 높다.

- 선점형 - 우선순위가 높은 프로세스가 도착하면 Running중인 프로세스를 멈추고 우선순위가 높은 프로세스를 실행한다.
- 비선점 - 더 높은 우선순위를 가진 프로세스가 도착하면 Ready Queue에 Head에 넣는다.
- Starvation
- 무기한 봉쇄(Infinite Blocking) 실행 준비는 되었으나 CPU를 사용못하는 프로세스를 무기한 대기시키는 것
- Aging을 통해서 해결 가능 - 기다리는 시간에 따라 우선순위를 높여주게 된다.

### 5. Round Robin

일정한 time quantum 동안 실행시키고 time quantum이 지나면 Ready Queue 가장 뒤로 들어가게 된다. 

- 선점형이다.
- 사용자에게 빠른 응답을 제공할 수 있다. (Response time이 짧다.)
- 아주 공평한 스케줄링이다. n개의 잡이 존재하고 타임 퀀텀이 q일때, 어떤 잡도 (n-1)*q 이상 기다리지 않는다.
- time quantum이 길어지면 FCFS와 같아지고, 너무 짧으면 context switching이 자주 일어나 overhead가 커진다.

Reference:

[https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/OS#멀티-스레드](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/OS#%EB%A9%80%ED%8B%B0-%EC%8A%A4%EB%A0%88%EB%93%9C)

[https://oaksong.github.io/2018/02/12/cpu-scheduling/](https://oaksong.github.io/2018/02/12/cpu-scheduling/)

https://velog.io/@pa324/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B5%AC%EC%A1%B0-18k3jfidll

