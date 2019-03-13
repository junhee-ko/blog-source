---
layout: post
title:  "CPU Scheduling"
date:   2018-10-06
categories: OS
---

##### CPU and I/O Bursts in program execution 이란 ?

![](/image/os100601.png)

##### CPU-burst time 의 분포는 ?

![](/image/os100602.png)

##### 프로세를 분류하면 ?

1. I/O-bound process
CPU를 잡고 계산하는 시간보다 I/O에 많은 시간이 필요한 job 입니다.
2. CPU-bound process
계산 위주의 job 입니다.

##### CPU 스케쥴링이 왜 필요한가요?

1. IO 대기, Memory stall과 같은 CPU idle time 을 최소화하여 CPU 자원의 활용을 극대화하기 위해 필요합니다.
2. 여러 종료의 job(process)이 섞여 있기 때문에 CPU 스케쥴링이 필요하다.

##### CPU 스케쥴링을 위해 Ready Queue 구현은 어떻게 하나요?

스케쥴링 알고리즘에 따라 FIFO, Queue, tree, Linked List 등을 사용할 수 있습니다.

##### CPU Scheduler & Dispatcher 이란 ?

1. CPU Scheduler 
운영체제 안에서 CPU 스케쥴링 하는 코드입니다. Ready 상태의 프로세스 중에서 이번에 CPU를 줄 프로세스를 고릅니다.

2. Dispatcher
CPU의 제어권을 CPU scheduler 에해 선택된 프로세스에게 넘깁니다. 이 과정을 context switch 라고 합니다.

##### CPU 스케쥴링이 필요한 경우는 ?

1. running -> blocked (I/O 요청하는 systemcall) 
2. running -> ready (할당 시간 만료로 time interrupt)
3. blocked -> ready (I/O 완료 후 interrupt)
4. terminate

1,4 는 nonpreemtive (강제로 빼앗지 않고 자진 반납) 입니다.

##### Scheduling Criteria (성능 척도)

1. system 입장
  - CPU utilization
    - keep the CPU as busy as possible
  - Throughput
    - number of processes that complete their execution per time unit
2. program 입장
  - Turnaround time
    - amount of time to execute a particular process
  - Waiting time
    - amount of time a process has been waiting in the ready
  - Response time
    - amount of time it takes from when a request was submitted unitl the first response is produced, not output (for time-sharing environment)
    - 최초의 CPU 얻기까지 기다린 시간

##### CPU 스케쥴링 알고리즘은 어떤게 있나요?

First Come First Served, Shorted Job First, Priority Scheduling, Round Robin이 있습니다.

##### FCFS (First-Come First-Served) 이란 ?

먼저 온 순서대로 처리하는 알고리즘입니다. 비선점형입니다.

- 도착 순서
  - p1, p2, p3

- burst time 
  - 24 / 3 / 3 

- wating time 
  - 0 / 24/ 27

- Convoy effet
  - short process behind long process

##### SJF (Shorted-Job-First) 이란 ?

CPU burst time이 가장 짧은 프로세스를 제일 먼저 스케쥴링합니다. 비선점형과 선점형이 있습니다. 비선점형의 경우, 일단 CPU를 잡으면 이번 CPU burst가 완료될 때까지 CPU를 선점 당하지 않습니다. 선졈형의 경우, 현재 수행중인 프로세스의 남은 burst time 보다 더 짧은 CPU burst time 을 가지는 새로운 프로세스가 도착하면 CPU를 빼앗깁니다. 이 방법을 SRTF (Shorted-Remaing-Time-First) 라고도 부릅니다.

##### SJF는 어떤 문제가 있나요?

starvation 문제가 있습니다. CPU 사용 시간이 긴 프로세스는 거의 영원히 CPU 를 할당받을 수 없습니다.

##### SJF에서 다음 CPU burst time을 어떻게 알 수 있나요?

과거의 CPU burst time을 이용해서 추정 (exponential averaging) 합니다.

##### Proirity Scheduling 이란 ?

우선순위가 제일 높은 process에게 CPU를 먼저 주는 알고리즘입니다.  선점형 스케줄링은, 더 높은 우선순위의 프로세스가 도착하면 실행중인 프로세스를 멈추고 CPU 를 선점합니다. 비선점형 스케줄링 방식은, 더 높은 우선순위의 프로세스가 도착하면 Ready Queue 의 Head 에 넣습니다.

#####Prirority Sceduling의 문제는 뭔가요?

starvation 문제가 있습니다. 낮은 우선순위의 프로세스는 CPU 를 할당받을 수 없습니다.

##### Prirority Sceduling의 starvation 문제를 어떻게 해결해야할까요?

aging을 사용합니다. 아무리 우선순위가 낮은 프로세스라도 오래 기다리면 우선순위를 높여주면 됩니다.

##### Round Robin 이란 ?

각 프로세스는 동일한 크기의 할당 시간 (time quantum) 을 가집니다. 할당 시간이 지나면 프로세스는 선점 당하고 ready queue의 제일 뒤에 가서 다시 줄을 서는 알고리즘입니다.

Response time이 빠릅니다. time quantum large 하면 FCFS 와 같고, time quantum small 하면 context switch overhead가 커집니다.

##### Multilevel Queue 란 ?

![](/image/os100603.png)

- Ready queue를 여러 개로 분할합니다.
  - foreground (interactive)
  - background (batch - no human interaction)
- 각 큐는 독립적인 스케쥴링 알고리즘을 가집니다.
  - foreground - RB
  - background - FCFS
- 어느 큐에 CPU를 줄지 결정하고, 그 큐 안에서 누구에게 CPU를 줄지 결정해야합니다.
- 큐에 대한 스케쥴링이 필요합니다.
  - fixed priority scheduling	
    - serve all from foreground then from background
    - Possibility of starvation
  - time slice
    - 각 큐에 CPU time을 적절한 비율로 할당합니다.
    - ex) 80 % to foreground in RB , 20% to background in FCFS

##### Multilevel Feedback Queue 란 ?

![](/image/os100604.png)

- 프로세스가 다른 큐로 이동 가능합니다.
- againg을 이와 같은 방식으로 구현 가능합니다.
- Multilevel Feedback Queue Scheduler를 정의하는 파라미터들
  - Queue의 수
  - 각 큐의 스케쥴링 알고리즘
  - Process를 상위 큐로 보내는 기준
  - Process를 하 큐로 내쫓는 기준
  - 프로세스가 CPU 서비스를 받으려 할 때 들어갈 큐를 결정하는 기준
- 할당 시간 끝나면 아래로 강등됩니다. CPU 사용 시간이 짧은 프로세스에게 우선 순위를 많이 줍니다.

##### Multiple-Processor Scheduling 이란 ?

CPU가 여러개인 경우

- Homogeneous process인 경우
  - Queue에 한 줄로 세워서 각 프로세서가 알아서 꺼내가게 할 수 있음
  - 반드시 특정 프로세서에서 수행되어야 하는 프로세스가 있는 경우에는 문제가 더 복잡해짐
- Load sharing
  - 일부 프로세서에 job이 몰리지 않도록 부하를 적절히 공유하는 메커니즘 필요
  - 별개의 큐를 두는 방법 vs 공동 큐를 사용하는 방법
- Sysmmetric Multiprocessing (SMP)
  - 각 프로세서가 각자 알아서 스케쥴링 결정
- Asysmmetric Multiprocessing
  - 하나의 프로세서가 시스템 데이터의 접근과 공유를 책임지고 나머지 프로세서는 거기에 따름

##### Real-Time Scheduling 이란 ?

- Hard real-time systems
  - 정해진 시간 안에 반드시 끝내도록 스케쥴링해야 함
- Soft real-time systems
  - 일반 프로세스에 비해 높은 priority를 갖도록 해야 함

##### Thread Scheduling 이란 ?

- Local Scheduling
  - User level thread의 경우 사용자 수준의 thread library에 의해 어떤 thread를 스케쥴할지 결정
- Global Scheduling
  - Kenel level thread의 경우 일반 프로세스와 마찬 가지로 커널의 단기 스케쥴러가 어떤 thread를 스케쥴할지 결정

##### Algorithm Evaluation

- Queueing models

  - server가 CPU, 확률 분포로 주어지는 arrivate rate와 service rate 등을 통해 각종 performance index 값을 계산

  ![](/image/os100605.png)

- Implemetatation(구현) & Measurement(성능 측정)

  - 실제 시스템에 알고리즘을 구현하여 실제 작업에 대해서 성능을 측정 비교

- Simulation (모의 실험)

  - 알고리즘을 모의 프로그램으로 작성후 trace(input data) 를 입력으로 하여 결과 비교

##### reference

이화여자대학교 반효경 (http://www.kocw.net/home/search/kemView.do?kemId=1046323)