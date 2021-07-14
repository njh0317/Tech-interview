# Dependency Injection(의존성 주입)
의존성 주입은 두 개 이상의 모듈 간 결합도를 낮추기 위해 **외부에서 객체를 생성하고 주입하는 것**이다.
+ 장점 : 개발 진행 시 독립된 모듈에 개발 테스트 자유도가 보장
+ 즉, 개발 진행시 모듈간 의존성은 유지하며 모듈간 독립성을 확보하기 위해 의존성 주입 사용

### 안드로이드 대표 의존성 주입 콘퍼넌트
+ dagger
+ Kodein

---

## 의존성 주입
**예시**

+ 클래스 `Computer`, `CPU`가 있다고 하자, 클래스 `Computer`가 `CPU`를 멤버로 가지고 있으면 이런 관계에서 `Computer`가 `CPU`에 대해 의존성을 가지고 있다고 말한다. 

```kotlin
class Computer {
    private val cpu = CPU() //내부에서 생성
}

class Computer(private val cpu: CPU) //외부에서 생성
```
+ `CPU` 멤버를 클래스 내부에서 생성하는 것이 아니라 외부에서 생성해서 멤버로 세팅하는 방법을 의존성 주입이라 한다. 
+ `CPU`를 생성해서 멤버로 세팅되는 시점에 따라 `Computer`가 생성될 때 되는지 혹은 Computer의 다른 멤버 함수를 호출할 때 되는지에 따라 *생성자 주입* 또는 *메소드 주입* 등으로 구분을 하기도 한다.

**특징**
+ 클래스간 관계를 느슨하게 한다.
+ 비슷하지만 다른 동작을 하는 멤버를 런타임에서 결정할 수 있도록 해준다.
+ 설계과정에서 미리 멤버의 인터페이스만 정해두고 실제 구현은 다른 팀원이 하도록 분업 할 수 있다.


---
## Kodein
Kodein은 의존성을 담는 일종의 컨테이너, 모듈간 binding 처리 및 의존성 주입을 도와주는 도구이다.

안드로이드 java 환경에 의존성 주입의 경우 Dagger를 많이 사용하지만 Kotlin 환경에서 Kodein은 Dagger에 비해 가볍고 배우기도 사용하기도 쉽고 좀 더 직관적이다.

```kotlin
interface CPU
class DualCoreCPU : CPU

interface RAM
class DDR4RAM : RAM

val kodein = Kodein { // 의존성과 그 바인딩 방법 선언
  bind<CPU>() with singleton { DualCoreCPU() }
  bind<RAM>() with singleton { DDR4RAM() }
}

class Computer(val dependencies: Kodein) {
  val cpu: CPU = dependencies.instance()  // 바인딩
  val ram: RAM = dependencies.instance()
}

val computer = Computer(kodein) // 의존성 주입
```
+ kodein 블럭 안에 bind로 시작하는 것들이 의존성, 어떤 방법으로 생성해서 바인딩 시킬지 선언
+ 그 이후 computer 객체를 생성하면서 주입
+ Computer 클래스 내부에서 타입 추론을 통해 적절한 의존성을 찾아 바인딩 된다.

### 바인딩 방법
**Singleton binding**
+ 최초 바인딩에서 생성된 객체가 계속 이용, 의존성을 받는 객체의 생명주기에서 단 하나의 객체만 사용할 경우 유용하다.

    ```kotlin
    val kodein = Kodein {
        bind<CPU>() with singleton { DualCoreCPU() }
    }

    class Computer {
        val cpu: CPU = kodein.instance()  // 여러번 바인딩해도 같은 객체가 나옴
    }
    ```

**Provider binding**
+ 호출할 때마다 매번 새로운 객체를 생성
    ```kotlin
    val kodein = Kodein {
        bind<RAM>() with singleton { DDR4RAM() }
    }

    class Computer {
        val provideRAM: () -> RAM = kodein.provider()  // 이후 provideRAM() 을 호출
    }
    ```

**Factory binding**
+ 인자 하나를 받아 객체를 생성하는 함수를 생성한다.
+ 특정 조건에 따라 다른 객체를 만드는 팩토리 패턴을 이용할 때 유용하다.

    ```kotlin
    val kodein = Kodein {
        bind<RAM>() with factory { type: Int -> when(type) {
            3 -> DDR3RAM()
            4 -> DDR4RAM()
            else -> RAM()
        }
    }

    class Computer {
        val createRAM: (Int) -> RAM = kodein.factory()  // 이후 createRAM(3) 이런식으로 호출

        val ram: RAM = kodein.with(3).instance()  // Singleton binding 처럼 사용 가능
        val provideRAM: () -> RAM = kodein.with(4).provide()  // Provider binding 처럼 사용 가능
    }
    ```

**Tagged binding**
+ 만약 같은 타입에 대해 여러가지 바인딩을 한다고 하면 이를 구분해 주기 위해 bind 함수의 파라미터로 태그를 넣어서 사용할 수 있다.
    ```kotlin
    val kodein = Kodein {
        bind<RAM>() with singleton { RAM() }
        bind<RAM>("DDR4") with singleton { DDR4RAM() }
        bind<RAM>("DDR3") with singleton { DDR3RAM() }
    }

    class Computer {
        val ram0: RAM = kodein.instance()
        val ram1: RAM = kodein.instance("DDR3")
        val ram2: RAM = kodein.instance("DDR4")
    }
    ```

**Constant binding**
+ 단순 상수를 바인딩 할 때 사용
+ 설정값 같은 것들을 지정할 때 유용하다.
    ```kotlin
    val kodein = Kodein {
        constant("NAME") with "my super computer"
    }

    class Computer {
        val name: String = kodein.instance("NAME")
    }
    ```

**의존성의 의존성 제공하기**
+ 의존성이 또 다른 의존성을 가지고 있을 경우 의존성을 받아올 때의 방법과 동일하게 선언하면 된다.
    ```kotlin
    class GPU
    class GraphicCard(val gpu: GPU)

    val kodein = Kodein {
        bind<GPU>() with singleton { GPU() }  // 이줄을 나중에 선언해도 잘 동작함
        bind<GraphicCard>() with singleton { GraphicCard(instance()) }
    }

    class Computer {
        val graphicCard: GraphicCard = kodein.instance()
    }
    ```

### KodeinAware

---
### 출처

https://cchcc.github.io/blog/Kodein%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-Kotlin-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85/

https://bravenamme.github.io/2019/11/13/kotlin-kodein/