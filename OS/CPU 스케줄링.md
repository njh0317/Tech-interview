# CPU 스케줄링
CPU가 하나의 프로세스 작업이 끝났을 때 다음 프로세스가 어떤 프로세스인지 선택하는 알고리즘을 CPU Scheduling 알고리즘이라 한다.

## Preemptive VS Non-preemptive

### Preemptive : SRTF, RR, Priority Scheduling, MultiLevel Scheduling

프로세스가 정상적으로 수행 중인 가운데 다른 프로세스가 CPU를 강제로 점유하여 실행할 수 있는 것

+ 장점
    + 높은 우선순위를 가진 작업을 빠르게 처리할 수 있다.

+ 단점
    + 프로세스 간의 context switching 이 자주 일어난다.
    + 우선순위가 낮은 작업은 starvation 현상을 겪게 된다.

### Non-preemptive : FCFS , SJF

한 프로세스가 한번 CPU를 점유했다면, I/O나 인터럽트 발생 또는 프로세스가 종료될 때 까지 다른 프로세스가 CPU를 점유하지 못하는 것

+ 장점
    + 모든 프로세스에 대해 공정한 처리가 가능하다.
    + 프로세스 간의 오버헤드가 적다.

+ 단점 : 긴 작업이 짧은 작업을 기다리는 경우가 발생한다.

## CPU scheduling Algorithm

### 1. First-Come, First-Served(FCFS)

먼저 온 프로세스가 먼저 CPU를 점유하는 방식이다.

단순하지만 모든 부분에서 효율적인 것은 아니다.

<img src="https://user-images.githubusercontent.com/33089715/117655857-979f9a80-b1d2-11eb-8237-fedadc275600.png" width = "300">

![image](https://user-images.githubusercontent.com/33089715/117655915-a5552000-b1d2-11eb-9c62-626167bf36ed.png)
+ 평균 대기 시간 = (0+24+27)/3 = 17msec

![image](https://user-images.githubusercontent.com/33089715/117655935-aab26a80-b1d2-11eb-94ef-ccb0573d360b.png)
+ 평균 대기 시간 = (6+3+0)/3 = 3msec
+ 들어온 순서대로 수행한다고 해서 반드시 효율적이지 않다.

**Convoy Effect**

위의 예제 중 p1, p2, p3 순서대로 들어온 것을 Convoy Effect 라고 한다. **CPU 시간을 오래 사용하는 프로세스가 먼저 수행하는 동안 나머지 프로세스들은 그 만큼 오래 기다리는 것**을 의미한다.

### 2. Shortest-Job-First(SJF)

가장 짧게 수행되는 프로세스가 가장 먼저 수행되는 것
<img src="https://user-images.githubusercontent.com/33089715/117656568-78edd380-b1d3-11eb-9b0e-92e9fed743b7.png" width = "300">
![image](https://user-images.githubusercontent.com/33089715/117656628-8d31d080-b1d3-11eb-9b57-c92c5e05390f.png)
+ 평균 대기 시간 = (3+16+9+0)/4 = 7msec
+ FCFS의 경우 = (0+6+14+21)/4 = 10.25msec

**특징**
+ 효율적으로 보이지만 비 현실적이다. CPU Burst time(점유시간)을 미리 알 수 없기 때문
+ preemptive와 non-preemptive 둘 다 사용 가능하다.

### 2-2. Shortest remaining time first(SRTF)

SJF 을 선점형으로 사용하게 된 경우이다.

**단점**

바로 처리 시간이 긴 프로세스는 CPU를 점유하지 못하는 **기아상태(Starvation)** 가 될 수 있다

### Priorty
우선순위가 높은 프로세스가 먼저 선택되는 스케줄링 알고리즘이다.

+ 우선순위를 정하는 방법에는 time limit, memory requirement, I/O to CPU burst(I/O 작업은 길고, CPU 작업은 짧은 프로세스 우선 등)등이 있다.

+ **문제점 - Starvation**
    + CPU 점유를 오랫동안 하지 못하는 현상(우선순위가 낮은 경우) ready queue에 계속해서 높은 우선순위의 프로세스가 들어온다면 우선순위가 낮은 프로세서의 경우 계속해서 기다리는 상황 발생

+ **해결 방법 - Aging**

    - ready queue에서 기다리는 동안 일정 시간이 지나면 우선순위를 일정량 높여준다. 그러면 우선순위가 매우 낮은 프로세스라 하더라도, 기다리는 시간이 길어질 수록 우선순위도 계속 높아지므로 수행될 가능성이 커진다.

    - 즉, **Aging이란, 대기시간에 비례하는 우선순위를 부여해 가까운 시간 안에(유한 시간 안에)자원을 할당하는 방법이다.**

### 4. Round-Robin(RR)

원 모양으로 모든 프로세스가 돌아가며 스케줄링 하는 것

- 일정 시간을 정해 하나의 프로세스가 이 시간 동안 수행하고 다시 대기 상태로 돌아간다. 다음 프로세스도 똑같이 수행 후 대기
- **장점** : 응답시간(Response time)이 짧다.
- **Time Quantum(Time Slice)** 일정 시간 : time quantum에 매우 의존적이다.(time quantum 에 따라 달라진다.)

    + 무한대로 설정시 : FCFS와 동일하게 동작

    + 0에 가깝게 설정 : switching overhead 가 매우 증가해서 비효율적이다.

### 5. Multilevel Queue
백그라운드에서 돌아가는 프로세스보다 사용자와 상호작용하는 앞단의 프로세스가 더 중요하게 여겨진다. 따라서 각 경우에 다른 알고리즘을 적용해준다.
1. background : FCFS 
2. foreground : RR

### 6. Multilevel Feedback Queue

큐 별로 다른 우선순위, 다른 스케줄링을 갖는다.
- 처음에 모든 프로세스는 가장 위의 큐에서 CPU의 점유를 대기한다. 이 상태로 진행하다가 이 큐에서 기다리는 시간이 오래 걸리면 아래의 큐로 프로세스를 옮긴다.
- 만약 우선순위로 큐를 사용하는 상황에서 우선순위가 낮은 아래의 큐에 있는 프로세스에서 starvation 상태가 발생하면 이를 우선순위가 높은 위의 큐로 옮길 수도 있다.

---
### 출처

https://velog.io/@codemcd/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9COS-6.-CPU-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81
