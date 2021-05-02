# UI와 BackgroundThread
![image](https://user-images.githubusercontent.com/33089715/116810868-18321b80-ab81-11eb-8e24-9bcb79cf1c87.png)

UI는 메인쓰레드만이 접근할 수 있다. 만약 외부 쓰레드 없이 메인쓰레드 만으로 모든 로직을 구현하게 되면 오래걸리는 작업을 수행했을 때 사용자는 멈춰있는 화면만 계속해서 보게되고, ANR(Android Not Responding) 에러가 발생할 수 있다. 따라서 다른 방법을 사용해 로직을 수행한 후 메인쓰레드로 전달해줘야 한다.

**안드로이드에서 제공하는 Message 나 Runnable 객체를 UI 스레드 쪽에서 동작시키기 원할 경우 사용하는 방법**

1. Handler.post()
2. Activity.runOnUiThread()
3. View.post()
4. AsyncTask 클래스

## Thread, Handler, Looper
핸들러를 생성하는 쓰레드만 다른 쓰레드가 전송하는 Message와 Runnable 객체를 받을 수 있다.
Handler와 Looper를 이용해 Sub Thread에서 Main Thread로 UI 작업을 전달하던지, Main Thread 내에서 자체적으로 처리하던지 해야한다.

### Handler
+ Message와 Runnable 객체를 처리
+ Runnable 메시지는 run() 메소드를 호출해 처리하고, Message는 handleMessage() 메소드를 이용해 처리

### Looper
+ Looper는 하나의 쓰레드만을 담당할 수 있고 하나의 쓰레드도 오직 하나의 Looper 만을 가질 수 있다.
+ Looper는 MessageQueue가 비어있는 동안은 아무 행동도 안하고 메시지가 들어오면 해당 메시지를 꺼내 적절한 Handler로 전달한다.

**생성법**
+ Handler는 무조건 Looper와 MessageQueue가 필요하므로 어떤 스레드와 연동되기 위해서는 Looper가 꼭 필요하다.

    ``` java
    Thread t = new Thread(new Runnable(){
        @Override
        public void run() {
            Looper.prepare();
            handler = new Handler();
            Looper.loop();
        }
    }); 
    t.start();
    ```
    + Looper.prepare() : messageQueue 준비
    + handler = new Handler() : 원하는 Handler 생성
    + Looper.loop() : run() 메소드의 마지막에 호출됨으로써 Message 전달을 기다리는 작업 시작
    + Activity 에서 onDestroy()때 handler.getLooper().quit() 호출해야 한다.(종료)

+ 또 다른 방법

    ```java
    HandlerThread t = new HandlerThread("My Handler Thread"); 
    t.start(); 
    handler = new Handler(t.getLooper());
    ```
    + 안드로이드에서 제공하는 HandlerThread 클래스 활용
    + 기본적으로 Looper를 가지고 있고, Thread를 start 시키면 자동으로 Loop 도 생성

### 동작 과정
![image](https://user-images.githubusercontent.com/33089715/116811342-c8a11f00-ab83-11eb-8890-5719120bfd99.png)

1. 메시지는 다른 스레드에 속한 MessageQueue 에서 전달된다.
2. MessageQueue에 메시지를 넣을 땐 Handler의 sendMessage()를 사용한다.
3. Looper는 MessageQueue에서 Loop()를 통해 반복적으로 처리할 메시지를 Handler에 전달한다.
4. Handler는 handleMessage()를 통해 메시지를 처리한다.

## RunOnUiThread
현재 스레드가 UI 스레드이면 UI 자원을 사용하는 작업에 대해 즉시 실행한다.
만약 현재 스레드가 UI 스레드가 아니라면 작업이 UI 스레드의 자원 사용 이벤트 큐에 들어가게 된다.

**runOnUiThread와 Handler를 사용하는 방법의 차이**

- **Handler**는 post 방식을 통해 매번 발생시키지만, **runOnUiThread**는 현재 시점이 UI 스레드이면 바로 실행시킨다.

### 구조
```java
public final void runOnUiThread(Runnable action) {
	if (Thread.currentThread() != mUiThread) {
		mHandler.post(action); 
	} else { 
		action.run();
	} 
}
```
+ Thread.currentThread()로 현재 쓰레드가 UI 쓰레드인지 확인
```java
new Thread(new Runnable() {
	@Override
	public void run() {
		for(i = 0; i<=100; i++) {
			// 현재 UI 스레드가 아니기 때문에 메시지 큐에 Runnable을 등록 함
			runOnUiThread(new Runnable() {
				public void run() {
					// 메시지 큐에 저장될 메시지의 내용
					textView.setText("runOnUiThread 님을 통해 텍스트 설정");
				}
			});
		}
	}
}).start();
```
+ subThread의 경우에도 Handler의 사용에 비해 간단하게 Main Thread(UI Thread)가 수행하도록 메시지 큐에 넣는다. 

## View.post()
Handler와 사용하는 인터페이스가 비슷하지만 가장 중요한 차이는 attach 가 됐는지 여부를 확인한 후, attach된 경우에 실행된다.
```java
public boolean post(Runnable action){
    final AttachInfo attatchInfo = mAttachInfo;
    if(attachInfo != null) {
        return attachInfo.mHandler.post(action)
    }
    getRunQueue().post(action);
    return true;
}
```
+ View가 attach, 즉 화면에 보여지는 시기에 맞춰 실행하는 경우 View.post()를 사용한다.
+ View.post()는 attach 되지 않으면 post 된 모든 runnable들의 실행이 attach 될 때 까지 연기(postpone)된다.
    + getRunQueue는 View에만 존재하는 pending된 Runnable 들의 HandlerActionQueue이다.

## AsyncTask 클래스
AsyncTask는 메인스레드에서 생성 후 실행되며, 메인 스레드에서 처리시간이 오래 걸리는 작업을 백그라운드 스레드로 넘기고 계속 메인 스레드 작업을 진행하기 위해 사용된다. AsyncTask는 비동기 태스크로써 백그라운드 스레드라는 별도의 스레드에서 작업을 수행하기 때문에  AsyncTask를 실행시켜 놓고 메인 스레드는 다음 작업을 바로 할 수 있다.

### 동작 과정
![image](https://user-images.githubusercontent.com/33089715/116811650-529db780-ab85-11eb-9ce6-03cbd2e10d8b.png)

1. execute() 명령어를 통해 AsyncTask 실행
2. AsyncTask로 백그라운드 작업을 실행하기 전에 onPreExcuted() 실행(이미지 로딩이 필요하면 로딩중 이미지를 띄우기 등 **스레드 작업 이전에 수행할 동작**)
3. 새로 만든 스레드에서 백그라운드 작업 수행. execute() 메소드를 호출할 때 사용된 파라미터 전달 받음
4. doInBackground()에서 중간 중간 진행 상태를 UI에 업데이트 하도록 하려면 publishProgress() 메소드를 호출함
5. onProgressUpdate() 메소드는 publishProgress()가 호출될 때마다 자동으로 호출된다.
6. doInBackground() 메소드에서 작업이 끝나면 onPostExcuted()로 결과 파라미터를 리턴하면서 그 리턴값을 통해 스레드 작업이 끝났을 때의 동작 구현

> **onPreExecute(), onProgressUpdate(), onPostExecute()** 메소드는 메인 스레드에서 실행되므로 **UI 객체에 자유롭게 접근**

### 제약조건 및 단점

- AsyncTask는 비교적 오래 걸리지 않을 작업에 유용
- Task 캔슬이 용이
- 로직과 UI 조작이 동시에 일어나야 할 때 유용하게 사용
- 제약조건과 단점에 주의해야한다.

**제약조건**

1. API16 미만 버전에서는 AsyncTask 선언을 UI Thread에서 해주지 않으면 오류 발생
2. executes(Params)는 UI 스레드에서 직접 호출해야 한다.
3. 수동으로 onPreExecute(), onPostExecute(Result), doInBackground(Params...), onProgressUpdate(Progress...) 호출하면 안된다.
4. Task는 오직 한 번만 실행할 수 있다.

**단점**

1. 하나의 객체이므로 재사용 불가능
2. 구현한 액티비티 종료 시 별도의 지시가 없다면 종료되지 않는다.
3. Activity 종료 후 재시작 시 AsyncTask의 Reference는 invalid 해지며 onPostExecute() 메소드는 새로운 Activity에 어떠한 영향도 미치지 않는다.
4. AsyncTask의 기본 처리 작업 개수 1개
---
### 출처
https://itmining.tistory.com/5

https://itmining.tistory.com/6?category=640759

https://duehd88.tistory.com/entry/Background-Thread에서-UI-접근하기 

https://itmining.tistory.com/7?category=640759