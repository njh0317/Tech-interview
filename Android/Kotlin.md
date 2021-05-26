# Kotlin
## 목차
1. [Data Class](#)
    + [특징](#)
    + [데이터 분해 및 대입(Destructuring Declarations)](#)
2. [cope Funcitons](#)
    + [일반적인 사용 방식](#)
    + [1. let](#)
    + [2. with](#)
    + [3. run](#)
    + [4. apply](#)
    + [5. also](#)
3. [초기화 지연 - lateinit, lazy]()
    + [Late initialization](#)
        + [lateinit의 특징](#)
        + [lateinit의 동작 원리](#)
    + [Lazy initialization](#)
        + [lazy init의 구현 원리](#)
        + [lazy init의 특징](#)
    + [lateinit vs lazy](#)
## Data Class
데이터 클래스(Data class)는 데이터 보관 목적으로 만든 클래스를 의미한다. 데이터 클래스는 프로퍼티에 대한 toString(), hashCode(), equals(), copy() 메소드를 자동으로 만들어 준다. 그래서 boilerplate code를 만들지 않아도 된다.

데이터 클래스는 클래스 앞에 data를 붙여줍니다. 예를 들어, 다음과 같이 데이터 클래스를 정의할 수 있습니다.

```kotlin
data class Site(val url: String, val title: String) {
    val description = ""
}
```
+ 자바로 변환해서 보면, 기본적인 toString(), hashCode(), equals(), copy() 메소드가 구현된 것을 볼 수 있다.(컴파일 시점에)

```java
public final class Site {
   ....

   public Site(@NotNull String url, @NotNull String title) {
      ....
   }

   @NotNull
   public final Site copy(@NotNull String url, @NotNull String title) {
     return new Site(url, title);
   }

   @NotNull
   public String toString() {
      return "Site(url=" + this.url + ", title=" + this.title + ")";
   }

   public int hashCode() {
      return (this.url != null ? this.url.hashCode() : 0) * 31 + (this.title != null ? this.title.hashCode() : 0);
   }

   public boolean equals(@Nullable Object var1) {
      if (this != var1) {
         if (var1 instanceof Site) {
            Site var2 = (Site)var1;
            if (Intrinsics.areEqual(this.url, var2.url) && Intrinsics.areEqual(this.title, var2.title)) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }
}
```
### 특징
+ 데이터 클래스의 생성자(primary constructor)는 1개 이상의 프로퍼티를 선언되어야 합니다.
+ 데이터 클래스의 생성자 프로퍼티는 val 또는 var으로 선언해야 합니다.
+ 데이터 클래스에 abstract, open, sealed, inner 를 붙일 수 없습니다.
+ 클래스에서 toString(), hashCode(), equals(), copy()를 override하면, 그 함수는 직접 구현된 코드를 사용합니다.
+ 데이터 클래스는 상속받을 수 없습니다.

### 데이터 분해 및 대입(Destructuring Declarations)
+ Site객체의 내부 변수를 다른 변수에 대입하려면 일반적으로 다음과 같이 해야한다.
```kotlin
val site1 = Site("kotlinlang.com", "Kotlin New Features (1)")
val url = site1.url
val title = site1.title
```

+ 하지만 데이터 클래스는 Destructuring Declarations를 지원하기 때문에, 아래 코드처럼 한줄로 표현할 수 있다.
```kotlin
val site1 = Site("kotlinlang.com", "Kotlin New Features (1)")

val (url, title) = site1
// 각 변수에 다음 값들이 대입됨
// url = "Kotlin New Features (1)"
// title = "kotlinlang.com"
```


## Scope Funcitons
*let, with, run, apply, also*

코틀린에는 이렇게 생긴 확장함수들이 있다. 객체를 사용할 때 명령문들을 블럭{} 으로 묶어서 간결하게 사용할 수 있게 해주는 함수들이다. 

**객체를 사용할 때 Scope(범위, 영역) 를 일시적으로 만들어서 속성(property)나 함수를 처리하는 용도로 사용되는 함수**이다. 간편한 코드 사용과 가독성, 빌더 패턴의 이용, 부가적인 후처리 등을 하는 데에 도움을 준다.

1. let
2. with
3. run
4. apply
5. also

### 일반적인 사용 방식
일반적으로 객체를 변경하려면 이런 방식을 사용한다.

```kotlin
data class Person(var name: String, var age: Int)

val person = Person("", 0)
person.name = "James"
person.age = 56
println("$person")

// Person(name=James, age=56)
```

### 1. let
```kotlin
fun <T, R> T.let(block: (T) -> R): R
```
+ `let` 함수는 매개변수화된 타입 T의 확장 함수이다.(extension) 
+ 자기 자신을 받아서 R을 반환하는((T) -> R) 람다 식을 입력으로 받고, 블럭 함수의 반환값 R을 반환한다. 
+ 여기서는 Person 클래스의 확장 함수로 사용되어 person.let 의 형태가 가능해진다.

```kotlin
val person = Person("", 0)
val resultIt = person.let {
    it.name = "James"
    it.age = 56
    it // (T)->R 부분에서의 R에 해당하는 반환값.
}

val resultStr = person.let {
    it.name = "Steve"
    it.age = 59
    "{$name is $age}" // (T)->R 부분에서의 R에 해당하는 반환값.
}

val resultUnit = person.let {
    it.name = "Joe"
    it.age = 63
    // (T)->R 부분에서의 R에 해당하는 반환값 없음
}

println("$resultIt")
println("$resultStr")
println("$resultUnit")


// Person(name=James, age=56)
// Steve is 59
// kotlin.Unit
```
+ 블럭의 마지막 return 값에 따라 let의 return 값 형태도 달라지는 모습을 보인다.
+ 또한 `T?.let { }` 형태에서의 let 블럭 안에는 non-null 만 들어올 수 있어 non-null 체크 시에 유용하게 쓸 수 있다. 객체를 선언하는 상황일 경우에는 elvis operator(`?:`) 를 사용해서 기본값을 지정해줄 수도 있다.
```kotlin
val nameStr = person?.let { it.name } ?: "Defalut name"
```

### 2. with
```kotlin
fun <T, R> with(receiver: T, block: T.() -> R): R
```
+ `with`는 일반 함수이기 때문에 객체 receive를 직접 입력받고, 객체를 사용하기 위한 두 번째 파라미터 블럭을 받는다. 
+ `T.()`를 람다 리시버라고 하는데, 입력을 받으면 함수 내에서 `this`를 사용하지 않고도 입력받은 객체(receiver)의 속성을 변경할 수 있다.

+ 즉, 아래 예제에서 with(T) 타입으로 Person을 받으면 {} 내의 블럭 안에서 곧바로 name 이나 age 프로퍼티에 접근할 수 있다. 
+ with는 non-null의 객체를 사용하고 블럭의 return 값이 필요하지 않을 때 사용한다. 그래서 주로 객체의 함수를 여러 개 호출할 때 그룹화하는 용도로 활용된다.

```kotlin
val person = Person("James", 56)
with(person) {
    println(name)
    println(age)
    //자기자신을 반환해야 하는 경우 it이 아닌 this를 사용한다
}

// James
// 56
```

### 3. run
두 가지 형태로 선언

**첫 번째**
```kotlin
fun <T, R> T.run(block: T.() -> R): R
```
+ run은 with처럼 인자로 람다 리시버를 받고, 반환 형태도 비슷하게 생겼지만 T의 확장함수라는 점에서 차이가 있다. 
+ 확장함수이기 때문에 safe call(.?)을 붙여 non-null 일 때에만 실행할 수 있다. 
+ 어떤 값을 계산할 필요가 있거나 여러 개의 지역변수 범위를 제한할 때 사용한다.

``` kotlin
val person = Person("James", 56)
val ageNextYear = person.run {
    ++age
}

println("$ageNextYear")

// 57

```

**두 번째**
``` kotlin
fun <R> run(block: () -> R): R
```
+ 이 run은 확장 함수가 아니고, 블럭에 입력값도 없다. 따라서 객체를 전달받아서 속성을 변경하는 형식에 사용되는 함수가 아니다. 
+ 이 함수는 어떤 객체를 생성하기 위한 명령문을 블럭 안에 묶음으로써 가독성을 높이는 역할을 한다.

```kotlin
val person = run {
    val name = "James"
    val age = 56
    Person(name, age)
}

```
### 4. apply
```kotlin
fun <T> T.apply(block: T.() -> Unit): T
```
+ apply는 T의 확장 함수이고, 블럭 함수의 입력을 람다 리시버로 받았기 때문에 블럭 안에서 객체의 프로퍼티를 호출할 때 it이나 this를 사용할 필요가 없다. 
+ run과 유사하지만 블럭에서 return 값을 받지 않으며 자기 자신인 T를 반환한다는 점이 다르다.

```kotlin
val person = Person("", 0)
val result = person.apply {
    name = "James"
    age = 56
}

println("$person")

//Person(name=James, age=56)
```
+ 앞에서 살펴 본 let, with, run 은 모두 맨 마지막 반한되는 값은 R이었다. 하지만 apply와 아래에서 살펴볼 also는 T를 반환한다.

    + 예시로, let에서는 resultUnit의 블럭 내에서 반환되는 값이 없어 최종적으로 Unit 타입이 되었지만, apply에서는 마지막에는 객체 자기 자신(T)을 반환하기 때문에 result는 Person 타입이 되었다. 그래서 println을 실행했을 때에 person의 내용이 콘솔에 찍히는 것을 확인할 수 있다.

+ 객체의 함수를 사용하지 않고 자기 자신을 다시 반환할 때에 사용되기 때문에, 객체의 초기화나 변경 시에 주로 사용된다.

### 5. also
```kotlin
fun <T> T.also(block: (T) -> Unit): T
```
+ also는 T의 확장함수이고, 블럭 함수의 입력으로 람다 리시버를 받지 않고 this로 받았다. 
+ apply와 마찬가지로 T를 반환한다.

```kotlin
val person = Person("", 0)
val result = person.also {
    it.name = "James"
    it.age = 56
}

println("$person")

//Person(name=James, age=56)
```
+ 블럭 함수의 입력으로 T를 받았기 때문에 it을 사용해 프로퍼티에 접근하는 것을 볼 수 있다. 
+ 그래서 객체의 속성을 전혀 사용하지 않거나 변경하지 않고 사용하는 경우에 also를 사용한다.

+ 객체의 데이터 유효성을 확인하거나, 디버그, 로깅 등의 부가적인 목적으로 사용할 때에 적합하다.

```kotlin
val numbers = arrayListOf("one", "two", "three")
numbers
    .also { println("add 하기 전에 print: $it") }
    .add("four")
```

## 초기화 지연 - lateinit, lazy
+ 코틀린에서는 변수 선언을 먼저하고 초기화는 뒤로 미루는 기능들을 제공한다. 
+ 장점 : 사용할지 모르는 데이터를 미리 초기화할 필요가 없어서 성능 향상에 도움이 됩니다. 
    + 예를 들어, Rest API로 GitHub의 데이터를 가져오는 기능이 있는데, 앱이 실행했을 때 미리 가져오는 것보다 데이터를 화면에 보여줄 때 가져오는 것이 CPU 자원도 아끼고, 네트워크 자원도 아낄 수 있다.

코틀린에서 제공하는 초기화 지연

+ Late initialization : 필요할 때 초기화하고 사용할 수 있음. 초기화하지 않고 쓰면 Exception 발생
+ Lazy initialization : 변수를 선언할 때 초기화 코드도 함께 정의. 변수가 사용될 때 초기화 코드 동작하여 변수가 초기화됨

### Late initialization
**late initialization**은 `var` 앞에 `lateinit`을 붙여 변수를 선언한다. 
```kotlin
class Rectangle {
    lateinit var area: Area
    fun initArea(param: Area): Unit {
        this.area = param
    }
}

class Area(val value: Int)

fun main() {
    val rectangle = Rectangle()
    rectangle.initArea(Area(10))
    println(rectangle.area.value)
}
```
+ lateinit var area로 선언하고 초기값을 지정하지 않았다.
+ 이 변수는 nullable이 아니기 때문에 초기값을 할당하지 않으면 에러가 발생한다. 하지만 lateinit를 붙여 나중에 초기화하겠다고 컴파일러에게 말했기 때문에 컴파일 에러가 발생하지 않는다.
+ main에서 rectangle.initArea(Area(10))로 늦게 변수를 초기화하고 그 다음 줄에 Area 객체를 사용

#### **lateinit의 특징**
+ var 에만 사용 가능
+ primitive type에 적용 불가. 
    + primitive type은 Int, Boolean, Double 등의 코틀린에서 제공하는 기본적인 타입
+ 따라서, 아래처럼 val을 사용하거나, Int를 사용한 코드는 컴파일 에러가 발생
``` kotlin
lateinit val area : Area    // compile error
lateinit var width : Int    // compile error
```
+ Custom getter/setter를 만들 수 없다.
+ Non-null 프로퍼티만 사용 가능
+ 따라서 아래 코드들은 컴파일 에러
```kotlin
lateinit var area: Area?    // compile error
lateinit var area: Area     // compile error
  get() {
      area;
  }
```

#### **lateinit의 동작 원리**
+ 자바로 디컴파일된 코드
```java
public final class Area {
   private final int value;

   public final int getValue() {
      return this.value;
   }

   public Area(int value) {
      this.value = value;
   }
}

public final class Rectangle {
   @NotNull
   public Area area;

   @NotNull
   public final Area getArea() {
      Area var10000 = this.area;
      if (var10000 == null) {
         Intrinsics.throwUninitializedPropertyAccessException("area");
      }

      return var10000;
   }

   public final void setArea(@NotNull Area var1) {
      Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
      this.area = var1;
   }
      public final void initArea(@NotNull Area param) {
      Intrinsics.checkParameterIsNotNull(param, "param");
      this.area = param;
   }
}

public static final void main() {
   Rectangle rectangle = new Rectangle();
   rectangle.initArea(new Area(10));
   int var1 = rectangle.getArea().getValue();
   System.out.println(var1);
}

```
+ public Area area로 변수가 초기화되지 않은 상태로 선언
+ 중요 코드는 다음 코드
    ```java
    public final Area getArea() {
        Area var10000 = this.area;
        if (var10000 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("area");
        }

        return var10000;
    }
    ```
+ Area.getArea()으로 Area 객체를 가져올 때 null check를 하고 있다.
    + null이면 Exception을 던진다. Area를 가져올 때는 모두 getArea()를 사용하기 때문에 초기화 전에 쓰려고 시도하면 예외가 발생하는 것을 알 수 있다. 이 때문에 custom getter/setter가 제한하였다는 것을 추측해볼 수 있다. 
+ 또한 코틀린에서 nullable 프로퍼티를 사용하지 못한 것은 객체가 null로 초기화되면, null check로 초기화가 되었는지 구분을 하지 못하기 때문이다.

### Lazy initialization
**Lazy initialization**은 프로퍼티를 정의할 때 초기화 코드도 함께 정의한다. 그리고 프로퍼티가 처음 사용될 때 초기화 구문이 실행되면서 초기값이 할당된다.

+ 아래 코드처럼 lazy init을 사용할 수 있다. 
+ 초기값 대신에 by lazy { ... }를 입력한다.
    + { ... } 부분은 변수가 처음 사용될 때 한번 호출되며 마지막의 값이 초기값으로 할당된다.
+ 아래 예제에서는 balance에 100이 할당된다.
```kotlin
val balance : Int by lazy {
    println("Setting balance!")
    100
}
```
+ {...}는 처음 사용할 때 한번만 호출되고 두번째 사용할 때는 호출되지 않는다. 
+ 다음은 Account.balance를 두번 출력한 코드와 결과
```kotlin
class Account() {
    val balance : Int by lazy {
        println("Setting balance!")
        100
    }
}

fun main() {
    val account = Account()
    println(account.balance)
    println(account.balance)
}
```
```
Setting balance! //한번 출력
100
100
```

#### **lazy init의 구현 원리**
+ 코틀린 코드를 자바로 디컴파일한 결과입니다. 
+ getBalance()가 처음 불릴 때 초기화 코드가 동작하여 할당
```kotlin
public final class Account {
   @NotNull
   private final Lazy balance$delegate;

   public Account() {
      this.balance$delegate = LazyKt.lazy((Function0)null.INSTANCE);
   }

   public final int getBalance() {
      Lazy var1 = this.balance$delegate;
      return ((Number)var1.getValue()).intValue();
   }
}

public static final void main() {
   Account account = new Account();
   int var1 = account.getBalance();
   System.out.println(var1);
   var1 = account.getBalance();
   System.out.println(var1);
}
```
+ getBalance()에서 ((Number)var1.getValue()).intValue()가 balance의 값을 리턴하는 것으로 보이며, 초기값이 할당되지 않았을 때 내부에서 초기화 코드를 호출하는듯..?

#### **lazy init의 특징**
+ val 프로퍼티만 사용할 수 있음
+ primitive type(Int, Boolean 등)도 사용 가능
+ Non-null, Nullable 모두 사용 가능

    + private final Lazy으로 객체를 선언하고 초기화도 한 상태이다. 이 때문에 변하지 않는 val만 사용가능.
    + 또, 그 객체의 메소드가 어떤 값을 리턴하기 때문에 primitive type도 사용

### lateinit vs lazy
+ lazy는 val 프로퍼티에만 사용할 수 있지만, lateinit은 var에만 사용할 수 있다.
그렇기 때문에 lateinit은 immutable(불변) 프로퍼티가 아니다.
+ lateinit은 nullable 또는 primitive type의 프로퍼티를 사용할 수 없다. 반면에 lazy는 모두 가능하다.
+ lateinit은 직접적으로 프로퍼티를 갖고 있는 구조지만(자바에서 field를 갖고 있음), lazy는 Lazy라는 객체 안에 우리가 선언한 field를 갖고 있다. 그래서 lazy의 프로퍼티를 직접 변경할 수 없다.

---
### 출처
https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html

https://codechacha.com/ko/kotlin-late-init/