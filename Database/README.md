# Database

1. [데이터베이스 풀](#데이터베이스-풀)
   * [Connection Pool](#connection-pool)
   * [DB에 접근하는 단계](#db에-접근하는-단계)
   * [Connection이 부족하면?](#connection이-부족하면)
   * [왜 사용할까?](#왜-사용할까)
   * [Thread Pool](#thread-pool)
   * [Thread Pool과 Connection Pool](#thread-pool과-connection-pool)
2. [정규화](#정규화)
   * 



## 데이터베이스 풀

### Connection Pool

클라이언트의 요청에 따라 각 애플리케이션의 스레드에서 데이터베이스에 접근하기 위해서는 Connection이 필요하다. 이런 Connection을 여러 개 생성해 두어 저장해 놓은 공간(캐시), 또는 이 <u>공간</u>의 Connection을 필요할 때 꺼내 쓰고 반환하는 <u>기법</u>을 Connection Pool이라고 한다.

<img src="https://user-images.githubusercontent.com/33208360/117453378-d7227880-af7f-11eb-9dc9-f3fa76fce7a8.png" alt="image" style="zoom: 67%;" />

### DB에 접근하는 단계

1. 웹 컨테이너가 실행되면서 DB와 연결된 Connection 객체들을 미리 생성하여 Pool에 저장한다.
2. DB에 요청 시, Pool에서 Connection 객체를 가져와 DB에 접근한다.
3. 처리가 끝나면 다시 Pool에 반환한다.

### Connection이 부족하면?

모든 요청이 DB에 접근하고 있고 남은 Connection이 없다면, 해당 클라이언트는 대기 상태로 전환시키고 Pool에 Connection이 반환되면 대기 상태에 있는 클라이언트에게 순차적으로 제공된다.

### 왜 사용할까?

* 매 연결마다 Connection 객체를 생성하고 소멸시키는 비용을 줄일 수 있다.
* 미리 생성된 Connection 객체를 사용하기 때문에, DB 접근 시간이 단축된다.
* DB에 접근하는 Connection의 수를 제한하여, 메모리와 DB에 걸리는 부하를 조정할 수 있다.

### Thread Pool

* 비슷한 맥락
* 매 요청마다 요청을 처리할 Thread를 만드는 것이 아닌, 미리 생성한 Pool 내의 Thread를 소멸시키지 않고 재사용하여 효율적으로 자원을 활용하는 기법

### Thread Pool과 Connection Pool

* WAS(Web Application Server)에서 Thread Pool과 Connection Pool내의 Thread와 Connection의 수는 직접적으로 메모리와 관련있기 때문에, 많이 사용하면 할수록 메모리를 많이 점유하게 된다. 반대로 메모리를 위해 적게 지정한다면, 서버에서는 많은 요청을 처리하지 못하고 대기할 수 밖에 없다.
* 보통 WAS의 Thread의 수가 Connection의 수보다 많은 것이 좋다. 모든 요청이 DB에 접근하는 작업이 아니기 때문이다.

</br>

## 정규화

### 정규화가 생겨난 배경

한 릴레이션에 여러 엔티티의 속성들을 혼합하게 되면 정보가 중복 저장되며, 저장 공간을 낭비하게 된다. 또 중복된 정보로 인해 **이상 현상(Anomaly)**이 발생하게 된다. 이러한 문제를 해결하기 위해 정규화