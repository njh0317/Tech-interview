# Coroutine

Coroutine은 안드로에드에서 백그라운드 스레드에서 코드를 처리할 때 사용하는 하나의 방법이다.

Coroutine을 사용하는 경우, 즉 백그라운드 태스크가 필요한 대표적인 경우는 
1. 네트워크 리퀘스트(Retrofit, Volley)
2. 내부 저장소 접근(Room, SQLite)

## Coroutine-Thread 차이
**Coroutine** : 하나의 실행-종료되어야 하는 일(Job)

**Thread** : 그 일이 실행되는 곳

따라서 Thread안에서 여러 개의 Coroutine 이 실행가능하다.

![image](https://user-images.githubusercontent.com/33089715/119599979-03177800-be21-11eb-82d0-2cadee378f17.png)
이미지 출처 - https://www.youtube.com/watch?v=F63mhZk-1-Y

+ Main Thread 이외에 Sub Thread가 있다면 이 둘을 동시에 병행 실행하는 개념
+ Thread는 하나의 Thread가 끝날 때 까지 계속되는 것과는 달리 코루틴은 실행 중간에 다른 작업을 하러 갔다가 돌아와서 작업을 다시 시작할 수 있다.

## 구현
### Dependency 추가
```
dependencies {
    ...
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.5"
}
```

### Coroutine Scope
`Coroutine Scope` 는 새로운 코루틴을 생성함과 동시에 실행되어야 할 Job을 그룹핑한다. 그래서 하나의 작업이 끝나고 다른 작업을 호출하다가 실패하게 된다면 전체가 취소 처리 된다.

```kotlin
CoroutineScope(Main).launch {
    // do something
}

CoroutineScope(IO).launch {
    // do something
}

CoroutineScope(Default).launch {
    // do something
}
```

코루틴 컨텍스트 `CoroutineContext` 에는 `Main`, `IO`, `Default`의 세 가지가 있다.
+ Main은 말 그대로 메인 쓰레드에 대한 Context이며 UI 갱신이나 Toast 등의 View 작업에 사용된다.
+ IO는 네트워킹이나 내부 DB 접근 등 백그라운드에서 필요한 작업을 수행할 때 사용된다.
    + Blocking IO 용 공유 쓰레드 풀에서 동작하도록 지정한 것이다.
+ Default는 크기가 큰 리스크를 다루거나 필터링을 수행하는 등 무거운 연산이 필요한 작업에 사용된다.

### suspend
`suspend` 함수는 그 함수가 비동기 환경(Asynchronous)에서 사용될 수 있음을 의미한다.
+ 비동기 함수인 suspend 함수는 다른 suspend 함수, 혹은 코루틴 내에서만 호출할 수 있고, 아닌 곳에서 그냥 호출하려고 하면 warning이 뜨면서 이런 메세지가 나온다.
    + Suspend function (FUNCTION_NAME) should be called only from a coroutine or another suspend function

+ 코루틴의 컨텍스트를 사용해 함수를 실행하려면 suspend 를 붙여주어야 한다. suspend를 붙인 함수는 코루틴 스코프 안에서 실행이 가능하다.

### delay
`delay`는 코루틴에서 정의된 suspend function이다. 즉 코루틴이나 다른 suspend 함수 안에서만 수행될 수 있다.

+ 괄호 안의 ms만큼 실행을 멈춘다. Thread.sleep(1000)와 거의 비슷하게 느껴질 것이다. 하지만 위에서 설명한 것처럼 코루틴은 쓰레드 안에서 돌아가는 하나의 Job이며 그 쓰레드 안에 여러 개의 코루틴이 실행되고 있을 수 있다.
+ 따라서 delay는 코루틴 하나만 멈추게 되지만, Thread.sleep은 해당 쓰레드 안에 있는 코루틴을 다 멈추게 된다. 코루틴 안에서 쓰레드 슬립을 호출하지 않는 편이 좋다.

### withContext
+ 아래의 코드는 크래시가 일어난다.
    ```kotlin
    CoroutineScope(IO).launch {
        val resultStr = getResultFromApi() 
        //resultStr = "ok"
        textView.text = resultStr
    }
    ```
+ 이유? IO context에서 네트워크 통신을 한 것 까지는 좋았지만 텍스트를 설정하는 부분은 메인 쓰레드에서 작업해야 하기 때문이다. 따라서 text를 세팅하는 부분은 Main 에서 실행해준다.
    ```kotlin
    CoroutineScope(IO).launch {
        val resultStr = getResultFromApi() //resultStr = "ok"

        CoroutineScope(Main).launch {
            textView.text = resultStr
        }
    }

    // ok
    ```
+ 코루틴 안에 Main context를 가지는 또다른 코루틴을 생성해 처리해주었다. 하지만 한 눈에 들어오지도 않고 코루틴을 하나 더 생성해야 해서 리소스 낭비가 있다. 이럴 때 사용할 수 있는 것이 바로 withContext 이다.
    ```kotlin
    CoroutineScope(IO).launch {
        val resultStr = getResultFromApi() //resultStr = "ok"

        withContext(Main) {
            textView.text = resultStr
        }
    }
    ```

+ 코루틴은 **쓰레드로부터 독립적(Thread independent)** 이다. MainThread에서 하나의 코루틴을 시작하고, 이것을 다른 SubThread1로 보내고, 또 SubThread2로 보냈다가 다시 MainThread 에서 작업을 하는게 가능하다. 
+ 이렇게 컨텍스트 스위칭을 해주는 것이 `withContext`의 역할이다.

### withTimeoutOrNull
네트워크 타임아웃 처리는 `withTimeoutOrNull(timeMillis)` 를 이용하면 손쉽게 처리할 수 있다. 밀리세컨드를 초과하는 시간이 걸리는 경우 null을 반환한다.
``` kotlin
CoroutineScope(IO).launch {
    val resultStr = withTimeoutOrNull(10000) {
        getResultFromApi()
    }

    if (resultStr != null) {
        withContext(Main) {
            textView.text = resultStr
        }
    }
}
```

### async
Coroutine 초기 예제에서도 확인 했던 것 처럼 Coroutine 작업들은 비동기적으로 처리하는 것이 가능하다. 비동기로 처리하려는 Coroutine들은 async { } 로 작업을 선언하면 된다.
+ async Coroutine에 딜레이 3초와 리턴값을 넣고 마지막에 각 Coroutine 결과의 총합을 출력하는 코드

    ```kotlin
    CoroutineScope(Dispatchers.IO).launch { 
        printLog("Coroutine Start") 
        val deferred1 = async { 
            delay(3000) 
            printLog("1") 
            return@async 1 
        } 

        val deferred2 = async { 
            delay(3000) 
            printLog("2") 
            return@async 2
        }

        val deferred3 = async { 
            delay(3000) 
            printLog("3") 
            return@async 3 
        } 
        printLog("total sum: " + (deferred1.await() + deferred2.await() + deferred3.await())) 
    }
    ```
    ```
    2020-04-15 22:18:29.090 3877-3990/? D/CoroutineSample: Coroutine Start 
    2020-04-15 22:18:32.108 3877-3996/kwony.study D/CoroutineSample: 2 
    2020-04-15 22:18:32.109 3877-3993/kwony.study D/CoroutineSample: 3 
    2020-04-15 22:18:32.112 3877-3991/kwony.study D/CoroutineSample: 1 
    2020-04-15 22:18:32.125 3877-3996/kwony.study D/CoroutineSample: total sum: 6
    ```
    + Coroutine 의 로그 출력 순서는 실행할 때마다 변경되는 반면 마지막에 Coroutine 반환 값의 합을 출력하는 부분은 항상 마지막에 출력되는데 이는 각 async 객체에 **`await()`** 함수를 사용해서 리턴값을 반환했기 때문이다. `await()` 함수는 async 함수가 모두 종료될 때 까지 구문 실행을 기다리도록 하는 함수다. 합을 출력하는 부분은 세개의 Coroutine의 연산결과를 모두 기다려야하기 때문에 제일 마지막에 실행 될 수 밖에 없다.

    + 또 주목할 만한 점은 모든 Coroutine에 3초씩 딜레이를 주었는데 각 Coroutine의 로그 출력 시간은 3초씩 차이가 나지 않는다. 이는 각각의 Coroutine이 Sequential하게 동작하지 않고 **Parallel**하게 동작했기 때문이다. Coroutine의 로그 출력 순서가 바뀌는 이유도 Parallel 하게 동작했기 때문이다.

---
### 출처
https://blog.yena.io/studynote/2020/04/26/Android-Kotlin-Coroutine.html

https://selfish-developer.com/entry/Kotlin-Coroutine [아는 개발자]