# Multithreading

## RxJava
### 스케줄러(Scheduler)
+ Rxjava에서의 스케줄러는 Rxjava 비동기 프로그래밍을 위한 쓰레드(Thread) 관리자이다. 스케줄러를 이용해서 어떤 스레드에서 무엇을 처리할 지에 대해서 제어할 수 있다.
+ 스케줄러를 이용해서 데이터를 통지하는 쪽과 데이터를 처리하는 쪽 스레드를 별도로 지정해서 분리할 수 있다.
+ Rxjava의 스케줄러를 통해 쓰레드를 위한 코드의 간결성 및 스레드 관리의 복잡함을 줄일 수 있다.
+ RxJava에서 스케줄러를 지정하기 위해서 subscribeOn(), observeOn() 유틸리티 연산자를 사용한다.
    + 생산자 쪽의 데이터 흐름을 제어하기 위해서는 subscribeOn() 연산자
    + 소비자 쪽에서 전달받은 데이터 처리를 제어하기 위해서는 observeOn() 연산자
    + 각각 파라미터로 Scheduler를 지정해야 한다.

#### 스케줄러 종류
|스케줄러|설명|
|---|--------|
|Schedulers.io()| - I/O 처리 작업을 할 때 사용하는 스케줄러 <br> - 네트워크 요청 처리, 각종 입/출력 작업, 데이터베이스 쿼리 등에 사용 <br> - 쓰레드 풀에서 쓰레드를 가져오거나 가져올 쓰레드가 없으면 새로운 쓰레드를 생성한다.|
|Schedulers.computation()| - 논리적인 연산 처리 시, 사용하는 스케줄러 <br> - CPU 코어의 물리적 스레드 수를 넘지 않는 범위에서 쓰레드를 생성한다. <br> - 대기 시간 없이 빠르게 계산 작업을 수행하기 위해 사용한다.|
|Schedulers.newThread()| - 요청시마다 매번 새로운 쓰레드를 생성한다. <br> - 매번 생성되면 쓰레드 비용도 많이 들고, 재사용도 되지 않는다.|

**새로운 쓰레드를 생성해도 되는데 왜 Scheduler.io()를 사용하는가?**

+ 화면 단에서 사용되는 request가 하나가 아니라 적게는 한 자리수 많게는 두 자리수까지 갈 수 있는데 이 부분에서 Thread를 계속해서 생성, 수거하게 된다면 사용할 때마다 드는 비용을 무시할 수 없다. 그렇기 때문에 Thread Pool 이 사용된다. 몇 개의 스레드를 생성한 뒤 큐에 Task를 넣고 작업하고 있지 않는 Thread에 Task를 할당하는 방식이다. 작업이 끝난 Thread는 다시 어플리케이션에 결과값을 리턴한다. Thread를 재사용하기 때문에 성능 저하를 방지할 수 있다. 하지만 Thread를 너무 많이 만들어 놓게 되면 메모리만 낭비하게 되므로 주의해서 사용해야 한다.

---
### 출처

https://kimyounghoons.github.io/android/interview/android-interview/

https://yunzai.dev/posts/RxJava_%EC%8A%A4%EC%BC%80%EC%A5%B4%EB%9F%AC%EB%9E%80_%EC%8A%A4%EC%BC%80%EC%A5%B4%EB%9F%AC%EC%9D%98_%EC%A2%85%EB%A5%98(1)/