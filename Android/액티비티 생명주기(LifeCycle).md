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
액티비티가 소멸될 때 호출됨. finish() 메소가 호출되거나 시스템이 메모리 확보를 위해 액티비티를 제거할 때 호출
+ 다음 메소드 : X

## 액티비티 데이터 유지

### Activity가 종료되는 경우
+ 사용자가 ‘뒤로 가기(Back)’ 버튼을 눌러 Activity를 종료한 경우
+ Activity가 백그라운드에 있을 때 시스템 메모리가 부족해진 경우(OS가 강제 종료시킴)
+ 언어 설정을 변경할 때
+ 화면을 가로/세로 회전할 때
+ 폰트 크기나 폰트를 변경했을 때

### 화면 회전
특히 액티비티 화면 회전시 onDestroy까지 갔다가 새로 생성되기 때문에 유지되어야 하는 데이터를 적절한 처리를 해줘야 한다.

화면을 회전할 때 발생하는 이벤트들은 다음과 같다.

+ onPause() → onSaveInstanceState() → onStop() → onDestory() → onCreate() → onStart() → onResume()

화면 회전을 방지하는 방법은 다음과 같다.
1. onSaveInstanceState 혹은 viewmodel 활용하기
2. 화면 회전 막기

### onSaveInstanceState
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onSaveInstanceState(outState: Bundle?) {
        super.onSaveInstanceState(outState)
        outState?.putString("stringKey", "This is a String value.")
        outState?.putBoolean("booleanKey", true)
        /* 맨 처음에는 저장되어있는 Bundle 데이터가 없으므로, outState가 null일 수 있다.
        따라서 outState 뒤에 ? 를 붙여 safeCall 한다. */
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
}
```

### onRestoreInstanceState
데이터 복구는 onCreate 혹은 onRestoreInstanceState 메소드로 할 수 있다. 둘 다 인스턴스 상태 정보를 포함하는 동일한 Bundle을 수신한다.

+ onCreate 메소드는 시스템이 활동의 새 인스턴스를 생성하든, 이전 인스턴스를 재생성하든 상관없이 호출되므로 읽기를 시도하기 전에 번들 상태가 null인지 반드시 확인해야 한다. null일 경우, 시스템은 이전에 소멸된 활동의 인스턴스를 복원하지 않고 새 인스턴스를 생성한다.
    ```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState) // Always call the superclass first

        // Check whether we're recreating a previously destroyed instance
        if (savedInstanceState != null) {
            with(savedInstanceState) {
                // Restore value of members from saved state
                currentScore = getInt(STATE_SCORE)
                currentLevel = getInt(STATE_LEVEL)
            }
        } else {
            // Probably initialize members with default values for a new instance
        }
        // ...
    }
    ```
+ onRestoreInstanceState()는 onStart() 메서드 다음에 호출된다. 시스템은 복원할 저장 상태가 있을 경우에만 onRestoreInstanceState()를 호출한다. 따라서 Bundle이 null인지 확인할 필요가 없다.
    ```kotlin
    override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
        // Always call the superclass so it can restore the view hierarchy
        super.onRestoreInstanceState(savedInstanceState)

        // Restore state members from saved instance
        savedInstanceState?.run {
            currentScore = getInt(STATE_SCORE)
            currentLevel = getInt(STATE_LEVEL)
        }
    }
    ```


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
    
### 안드로이드 생명 주기에 관해 주의해야할 점이 있다면?
    액티비티가 죽으면 가비지콜렉팅을 해도 레퍼런스가 삭제되서 메모리 환원이 안되므로 onDestroy안에서 System.gc()를 해줘야한다.
    필요한 메모리 정리를 onDestroy()에서 꼭 해줘야 한다.

---

### 출처
https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko
