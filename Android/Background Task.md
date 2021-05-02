# Background Task

백그라운드 작업은 다음 세가지 작업으로 분류된다.
+ Immediate
+ Exact
+ Deferred
![image](https://user-images.githubusercontent.com/33089715/116806776-2f194380-ab6a-11eb-818c-9ff5a4725daf.png)

    + 사용자가 애플리케이션과 상호작용하는 동안 작업을 완료해야 하는가?
        + yes → Immediate
        + No → 두 번째 질문
    + 작업이 정확한 시간에 실행되어야 하는가?
        + yes → Exact
        + No → Deferred
    
### Immediate
+ 앱이 Foreground 상태에 있을 때 완료될 수 있는 작업(간단한 API 호출, 짧은 시간의 타이머, 금방 끝나는 계산집약적 알고리즘)등은 `쓰레드`를 사용해서 손쉽게 해결될 수 있다. 
    + 쓰레드를 쓰는 것인 Java의 Thread를 Executor와 함께 사용하거나, Thread를 손쉽게 쓰게 만들어주는 라이브러리인 RxJava나 코틀린의 Coroutine을 사용한다.

+ 만약에 작업이 즉각적으로 사용자에게 응답해주어야 하지만, 앱이 백그라운드 상태에 있거나 기기가 재시작되어도 지속적으로 진행이 되어야하는 작업이라면 WorkManager를 사용한다.

+ 음악을 재생하거나, 달리기 앱을 만들어 현재 상황을 유저에게 지속적으로 보여줘야 하는 경우엔 `Foreground Service`를 사용할 수 있다.

### Deferred(연기)
+ `WorkManager`를 사용하면 앱이 종료되거나 기기가 다시 시작되더라도 지연될 수 있는 비동기 작업을 쉽게 예약할 수 있다.

### Exact
+ 정확한 시점에 실행해야 하는 작업에는 `AlarmManager`를 사용할 수 있다.

## WorkManager
+ 앱이 종료되거나 기기가 다시 시작되어도 실행 예정인 지연 가능한 비동기 작업을 쉽게 예약
+ 안드로이드 백그라운드 작업을 처리하는 방법 중 하나, Android Jetpack 아키텍처의 구성 요소 중 하나

### 주요 기능
WorkerManager는 앱이 종료되거나 기기가 다시 시작되는 경우에도 지연 가능(deferrable)하고 안정적으로 실행 되어야 하는(run reliably) 작업

**WorkManager only**

실행 보장된 지연가능한 작업에 사용한다.
+ 백앤드 서비스에 로그 또는 분석을 전송하는 작업
+ 업로드/다운로드할 컨텐츠 암호화/복호화

**FCM + WorkManager**

외부 이벤트로 시작되는 작업에 사용한다.
+ 새로운 컨텐츠 동기화 (예. 이메일)
> FCM : Firebase Cloud Messaging

### 주요 클래스
![image](https://user-images.githubusercontent.com/33089715/116807724-a8676500-ab6f-11eb-954d-05f486d02e19.png)

**Worker**
+ 추상 클래스, 처리해야하는 백그라운드 작업으 처리 코드를 이 클래스를 상속받아 doWork() 메서드를 오버라이드 하여 작성
+ doWork()
    - 작업을 완료하고 결과에 따라 Worker 클래스 내에 정의된 enum인 Result의 값 중 하나를 리턴해야 한다.
    - Success, Failure, Retry의 3개 값이 있으며 리턴되는 이 값에 따라 WorkManager는 해당 작업을 마무리 할 것인지, 재시도 할 것인지 실패로 정의하고 중단할 것인지 결정

**WorkRequest**
- WorkManager를 통해 실제 요청하게 될 개별 작업
- 처리해야 할 작업인 Work와 작업 반복 여부 및 작업 실행 조건, 제약 사항 등 이 작업을 어떻게 처리할 것인지에 대한 정보가 담겨있다.
- 반복여부에 따라 onTimeWorkRequest(반복하지 않을 작업, 즉 한번만), PeriodicWorkRequest(여러번 실행할 작업의 요청을 나타내는 클래스)로 나뉜다.

**WorkerManager**
+ 처리해야하는 작업을 자신의 큐에 넣고 관리
    + WorkManager에 enqueue하면 작업이 요청된다. 
+ 싱글톤으로 구현되어있기 때문에 getInstance()로 WorkManager의 인스턴스를 받아 사용
+ beginWith()라는 API를 이용하면 WorkContinuation으로 변환되며, WorkContinuation는 여러 WorkRequest들을 묶을 수 있는 API를 제공한다.


**WorkState**
- WorkRequest의 id와 해당 WorkRequest의 현재 상태를 담는 클래스
- 이 WorkState의 상태정보를 이용해서 자신이 요청한 작업의 현재 상태를 파악할 수 있다.
- ENQUEUED, RUNNING, SUCCEEDED, FAILED, BLOCKED, CANCELLED의 6개의 상태
---
### 출처
https://developer.android.com/guide/background?hl=ko
https://fornewid.medium.com/workmanager-%EC%A0%95%EB%A6%AC-62e9f6f53767