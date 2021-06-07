# Service
서비스는 화면 없이도 데이터를 주고받는 기능을 실행하고, 메시지를 받아 처리하는데 사용되는 것이다.

예시

+ 카톡 화면이 보이지 않는 상태에서 메시지가 왔다고 알림이 뜸
+ 카톡 앱이 사용자의 눈에 보이지 않는 상태에서도 무언가 실행되고 있다는 것을 의미

## Service 의 종류
1. Foreground
2. Background
3. Bound
> 1, 2는 Started Service라 불리고 3은 Bound Service라 불린다.
> 세 가지 종류의 서비스가 있지만, 독립된 것이 아니라 구현에 따라 binding(bound)되는 용도로도 쓰이며 동시에 Foreground, Background로도 쓰일 수 있다.

백그라운드 서비스와 포그라운드 서비스의 가장 큰 차이점 : **사용자가 서비스의 존재에 대해 인지할 수 있는지 없는지에 대한 차이**

### Foreground
안드로이드의 Notification(필수)을 사용하여 진행되는 상황을 사용자가 인지할 수 있도록 한다. 사용자가 인지하고 있기 때문에 메모리 부족시 시스템에 의한 종료 대상에서 제외된다.

### Background
백그라운드에서 실행되지만 유저에게 특별히 진행상황이나 결과에 대한 메시지를 전달할 필요가 없는 경우에 쓰인다.

### Bound
서비스가 bind된 상태, Serivce-client 모델에서 서비스가 특별한 클라이언트에 결합된 상태를 의미한다.

## 동작 과정
![image](https://user-images.githubusercontent.com/33089715/116812086-c17c1000-ab87-11eb-8653-c5d0fb3420fc.png)

### onCreate()
서비스 객체가 새롭게 생성이 될 때 호출, `onStartCommand()`, `onBind()` 보다 먼저 호출

### onStartCommand()
시스템이 `startService` 함수를 이용해 서비스를 시작 할 때 호출. 서비스를 시작하면 서비스를 중단하는 것은 우리의 몫. 서비스 내에서 직접 `stopSelf`를 호출하거나 밖에서 `stopService`를 호출해주면 된다. 새롭게 `startCommand` 호출이 일어날 때 여러번 호출 가능

Service는 기본적으로 프로세스에 의해 종료가 되더라도 다시 살아나는 구조를 가지고 있다.(Service의 종료 메소드인 stopService() 메소드를 호출하기 전까지) 프로세스에 의해 종료된 Service는 onCreate() -> onStartCommand() 메소드 순으로 호출된다.

**반환값** : 서비스가 시스템에 의해 소멸되었을 때, 어떤 것을 반환하냐에 따라 재시작될 때 onStartCommand()가 호출되는 방식이 달라진다.
+ START_NOT_STICKY : 서비스를 명시적으로 다시 시작할 때 까지 만들지 않는다. 시스템에 의해 강제 종료되어도 괜찮은 작업을 진행할 때 사용.
+ START_STICKY : 서비스를 다시 만들지만 마지막 Intent를 onStartCommand의 인자로 다시 전달하지 않는다. 즉, intent값을 null로 초기화 시켜서 재시작 한다.
+ START_REDELIVER_INTENT : 서비스를 다시 시작하면서 마지막 Intent를 onStartCommand의 인자로 다시 전달하지 한다.

### onBind()
새로운 binding이 연결될 때 호출

### onDestroy()
서비스 객체가 파괴될 때 호출

## IntentService

서비스는 기본적으로 프로세스의 메인쓰레드에서 동작한다. ANR(Application Not Responding)을 피하고 싶다면 따로 쓰레드를 파서 원하는 작업을 해주어야 한다. 이를 자동으로 한 것이 IntentService. 
> 하지만 API26 부터 background Service limitation에 의해 deprecated 되엇고 내부적으로 Service가 아닌 JobScheduler를 사용하는 JobIntentService를 사용하기 권장
### 특징
- Service를 상속받아서 사용하도록 조금 더 편리하게 나온 것
- 기본적으로 백그라운드 Thread에서 동작, 따라서 따로 처리가 필요
- 작업이 순차적으로 수행되고, 한 번에 하나의 작업만 수행
- onHandleIntent()에서 작업 수행
- API30에서 deprecated되었기 때문에 JobIntentService로 바뀜

### 동작 과정
![Untitled](https://user-images.githubusercontent.com/33089715/116812236-b7a6dc80-ab88-11eb-8967-e7353398f3e5.png)

1. Client에서 startService를 실행해 IntentService를 실행하게 된다. IntentService로 Intent를 전달
2. IntentService는 Intent를 받아서 IntentService에 존재하는 IntentQueue에 Intent를 넣는다.
3. IntentService 내에 Looper는 Queue에 존재하는 Intent를 하나씩 꺼내서 onHandleIntent() 함수를 호출한다. 호출시 해당 Intent를 인자로 전달
4. Queue가 모두 비워 질 때까지 Looper는 onHandleIntent() 함수를 호출

### Service와 IntentService 차이
**이용**

- **Service** : UI 없이 실행될 수 있지만 매우 길지 않는 상황. 만약 오래 걸리는 작업을 Service에서 실행하고자 하면 Service안에서 스레드를 사용해야 한다.
- **IntentService** : 오래걸리지만 메인스레드와 관련이 없는 작업을 할 때 주로 이용. 만약 메인 스레드와 관련된 작업을 해야 한다면 메인스레드 Handler나 Broadcast intent를 이용해야 한다.

**사용 방법**

- **Service** : startService() 메소드에 의해 실행
- **IntentService** : Intent 사용에 의해 실행된다. 새로운 스레드가 생성되며 onHandleIntent()가 불린다.

**호출 위치**

- **Service**와 **IntentService** 모두 아무 스레드에서 생성, 액티비티 뿐 아니라 다른 곳에서도 실행 가능

**실행중인 위치**

- **Service** : 백그라운드에서 동작하지만 메인스레드에 포함
- **IntentService** : 새로운 스레드에서 동작

**중지 방법**

- **Service** : 사용자의 몫. stopSelf()나 stopService()에 의해 동작이 멈춤. Service Binding의 경우 필요 없다.
- **IntentService** : onHandleIntent() 내의 모든 동작이 수행되면 멈춘다. 멈추기 위한 다른 메소드 호출이 불필요

**단점**

- **service** : 메인 스레드에 포함되므로 무거운 작업일 때 메인 스레드에 영향을 주어 느려지거나 할 수 있다.
- **IntentService** : 병렬적으로 수행될 수 없으므로 연속적인 Intent 호출에 관해서 순차적으로 처리된다.
---
### 출처
https://developer.android.com/guide/components/services?hl=ko
https://jhshjs.tistory.com/48
