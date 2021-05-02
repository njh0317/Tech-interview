# 액티비티 생명주기(LifeCycle)

    액티비티(Activity)는 메모리에 상주되면서 소멸되기까지의 생명주기를 가지고 있다. 
    Activity 클래스는 활동이 상태 변화(시스템이 활동을 생성, 중단 또는 다시 시작하거나, 활동이 있는 프로세스를 종료하는 등)를 알아차릴 수 있는 여러 콜백을 제공한다.

![image](https://user-images.githubusercontent.com/33089715/116804191-d2149200-ab57-11eb-8a1c-570db1f58b92.png)

## Lifecycle callback

### onCreate()
액티비티가 생성될 때 호출되며 사용자 인터페이스 초기화에 사용
+ 다음 메소드 : onStart()
### onRestart()
액티비티가 멈췄다가 다시 시작되기 바로 전에 호출
+ 다음 메소드 : onStart()

### onStart()
액티비티가 사용자에게 보여지기 바로 직전에 호출
+ 다음 메소드 : onResume(), onStop()

### onResume()
액티비티가 사용자와 상호작용 하기 바로 전에 호출
+ 다음 메소드 : onPause()
### onPause()
다른 액티비티가 보여질 때 호출됨. 데이터 저장, 스레드 중지 등의 처리를 하기에 적당한 메소드
+ 다음 메소드 : onResume() or onStop()
### onStop()
액티비티가 더 이상 사용자에게 보여지지 않을 때 호출됨. 메모리가 부족할 경우에는 onStop() 메소드가 호출되지 않을 수도 있음
+ 다음 메소드 : onRestart() or onDestroy()
### onDestroy()
액티비티가 소멸될 때 호출됨. finish() 메소두가 호출되거나 시스템이 메모리 확보를 위해 액티비티를 제거할 때 호출
+ 다음 메소드 : X

## 관련 문제
### 메인 액티비티가 다른 하위 액티비티(detail activity)를 호출했을 때 생명주기 호출 순서
    [MainActivity] onPause()
    [DetailActivity] onCreate()
    [DetailActivity] onStart()
    [DetailActivity] onResume()
    [MainActivity] onStop()

### 반대로 하위 액티비티(detail activity)에서 메인 액티비티로 돌아갈 때 생명주기 호출 순서
    [DetailActivity] onPause()
    [MainActivity] onRestart()
    [MainActivity] onStart()
    [MainActivity] onResume()
    [DetailActivity] onStop()
    [DetailActivity] onDestroy()
---

### 출처
https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko