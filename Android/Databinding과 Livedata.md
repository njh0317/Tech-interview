# Databinding과 Livedata

## Databinding
Databinding(데이터바인딩)은 xml 파일에 Data를 연결(binding)해서 사용할 수 있게 도와주는 Android Jetpack 라이브러리의 하나의 기능이다.
+ 데이터바인딩은 애플리케이션 레이아웃을 binding 하는데 필요한 글루 코드를 최소화 한다.
    > 글루코드 : 프로그램의 요구사항 구현에는 기여하지 않지만 본래 호환성이 없는 부분끼리 결합하기 위해 작동하는 코드
+ findViewById를 사용하지 않아도 되며, 보통 MVVM 패턴을 구현할 때 "LiveData"와 함께 거의 필수적으로 사용한다.

### 장점
데이터 바인딩을 사용하면 데이터를 UI 요소에 연결하기 위해 필요한 코드를 최소화할 수 있다.
+ findViewById()를 호출하지 않아도, 자동으로 xml에 있는 View들을 만들어 준다.
+ RecyclerView에 각각의 item 을 set 해주는 작업도 자동으로 진행
+ data가 바뀌면 자동으로 View 를 변경하게 구현 가능
+ xml 리소스만 보고도 view 에 어떤 데이터가 들어가는지 파악 가능

## LiveData
Data의 변경을 관찰할 수 있는 Data Holder 클래스
+ 일반적인 Observable과 다르게 **LiveData는 안드로이드 생명주기(LifeCycle)**을 알고 있다.(Lifecycle-Aware)
    + LiveData는 활성상태(active)일 때만 update
    + 활성상태 - STARTED, RESUME
- LiveData객체는 Observer 객체와 함께 사용
    - 데이터에 변화가 일어날 경우 **Observer 객체에 변화를 알려주고**, **Observer의 onChanged()메소드가 실행**

### 장점
- **Data와 UI간 동기화**
    - Observer 객체를 사용함으로써 데이터의 변화가 일어나는 곳마다 매번 UI를 업데이트하는 코드를 작성할 필요 없이 통합적이고 확실하게 데이터의 상태와 UI를 일치시킬 수 있다.
- **메모리 누수(Memory Leak)가 없다.**
    - Observer 객체는 안드로이드 생명주기 객체와 결합되어 있기 때문에 컴포넌트가 Destroy 될 경우 메모리상에서 스스로 해제
- **Stop 상태의 액티비티와 Crash가 발생하지 않는다.**
    - 액티비티가 Back Stack 에 있는 것처럼 Observer의 생명주기가 inactive(비활성화) 일 경우, Observer는 LiveData의 어떤 이벤트도 수신하지 않는다.
- **생명주기에 대한 추가적인 handling을 하지 않아도 된다.**
    - 데이터를 관찰 하기만 하면 된다.
- **항상 최신 데이터를 유지**
    - 화면 구성이 변경되어도 데이터를 유지한다.
- **자원(Resource)를 공유할 수 있다.**
    - LiveData를 상속하여 자신만의 LiveData 클래스를 구현할 수 있고 싱글톤 패턴을 이용하여 시스템 서비스를 둘러싸면(Wrap) 앱 어디에서나 자원을 공유할 수 있다.

### 주의 할 점
- Generic을 사용해 관찰하고자 하는 데이터의 타입(Type)을 갖는 LiveData 인스턴스 생성
- LiveData 클래스의 observe() 메소드를 사용해 Observer 객체를 LiveData에 결합

    이 때, observer() 메소드는 **LifecycleOwner 객체**를 필요로 하며 보통은 Activity를 전달

    LiveData에 저장된 데이터에 변화가 있으면 결합된 LifecycleOwner에 의해 상태가 active(활성)인 한 모든 데이터에 대해 Trigger 발생

    - **LiveData에 Observer를 결합하는 코드는 컴포넌트의 onCreate() 메소드 내에 위치하는 것이 바람직하다**.(아래 이유 2가지)
        - onResume()에 할 경우 **pause()**나 **stop()**에 의해 잠시 백그라운드상에서 inactive된 앱이 다시 active(활성화)가 되면서 LiveData에 대한 코드가 **중복호출**이 될 수 있기 때문, 이는 LiveData의 장점 중 {생명주기에 대한 추가적인 handling을 하지 않아도 됨}에 대해 반대되는 방식
        - 액티비티나 프래그먼트가 active(활성화)되자마자 UI에 표시 할 수 있는 데이터를 가질 수 있기 때문에 해당 컴포넌트는 **STARTED** 상태가 되자 마자 LiveData 객체로부터 **가장 최신의 값을 수신**해야한다.
- Observer 객체 생성

    onChanged() 메서드 정의해야함

### 기타 질문
#### MutableLiveData와 LiveData를 구분하는 이유
- MutableLiveData : 값을 get/set 모두를 할 수 있다.
    - 값의 get/set 모두를 허용하기 때문에 값의 변경과 읽기 모두 가능하다.
    - postValue(in Worker Thread), setValue(in Main Thread) 메서드를 이용해 값을 변경할 수 있다.
- LiveData : 값의 get()만 할 수 있다.
    - 읽기 전용의 데이터를 만들기 떄문에 값의 변경은 불가능하다.

- ViewModel과 View의 역할을 분리하기 위해 사용한다.
- ViewModel은 언제나 새로운 값의 변경이 일어나고, 다시 읽을 수 있는 형태로 사용하는 것이도, View는 값의 입력이 아닌 읽기만을 허용한다.

#### LiveData는 인터럽트 방식인가 폴링 방식인가?
+ 폴링 방식은 정해진 시간 또는 순번에 상태를 확인해서 상태 변화가 있는지 없는지 체크를 하는 방식이다.
+ 인터럽트 방식은 main문을 실행하는 도중 외부에서 정해져있는 인터럽트 핀에 신호가 들어오면 MCU는 즉각적으로 하고 있는 동작을 멈추고 인터럽트 서비스 루틴을 실행하는 것이다.
+ LiveData는 데이터가 변경된 시점에 Observer가 실행되기 때문에 인터럽트 방식이라고 볼 수 있다.
​
#### 옵저버 패턴 설명
+ 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many) 의존성을 정의한다.
+ 데이터 전달 방식은 2가지가 있다. 주체 객체에서 옵저버로 데이터를 보내는 방식(푸시 방식), 옵저버에서 주체 객체의 데이터를 가져가는 방식(풀 방식)
+ Subject 와 Observer가 존재하고 Subject 에 Observer를 등록하고 난 뒤 subject의 데이터가 변경되면 등록된 Observer가 호출 되는 방식

---
### 출처
https://velog.io/@jojo_devstory/Android-Databinding%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90
https://velog.io/@jojo_devstory/Android-LiveData...%EB%84%8C-%EB%88%84%EA%B5%AC%EB%83%90
https://kimyounghoons.github.io/android/interview/android-interview/

