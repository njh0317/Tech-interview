# Kotlin
## 목차
1. [Data Class](#data-class)
    + [특징](#특징)
    + [데이터 분해 및 대입(Destructuring Declarations)](#데이터-분해-및-대입destructuring-declarations)
2. [cope Funcitons](#scope-funcitons)
    + [일반적인 사용 방식](#일반적인-사용-방식)
    + [1. let](#1-let)
    + [2. with](#2-with)
    + [3. run](#3-run)
    + [4. apply](#4-apply)
    + [5. also](#5-also)
3. [초기화 지연 - lateinit, lazy](#초기화-지연---lateinit-lazy)
    + [Late initialization](#late-initialization)
        + [lateinit의 특징](#lateinit의-특징)
        + [lateinit의 동작 원리](#lateinit의-동작-원리)
    + [Lazy initialization](#lazy-initialization)
        + [lazy init의 구현 원리](#lazy-init의-구현-원리)
        + [lazy init의 특징](#lazy-init의-특징)
    + [lateinit vs lazy](#lateinit-vs-lazy)
4. [Null safety](#null-safety)
    + [Nullable과 Non-Nullable 프로퍼티](#nullable과-non-nullable-프로퍼티)
    + [코틀린에서 Null Pointer Exception(NPE)이 발생하는 경우](#코틀린에서-null-pointer-exceptionnpe이-발생하는-경우)
    + [nullable 타입을 non-nullable 타입으로 변경하기](#nullable-타입을-non-nullable-타입으로-변경하기)
    + [안전하게 nullable 프로퍼티 접근하기](#안전하게-nullable-프로퍼티-접근하기)
        + [1. 조건문으로 nullable 접근](#1-조건문으로-nullable-접근)
        + [2. Safe call 연산자로 nullable 접근](#2-safe-call-연산자로-nullable-접근)
    + [안전하게 nullable 프로퍼티 할당](#안전하게-nullable-프로퍼티-할당)
        + [1. if-else](#1-if-else)
        + [2. 엘비스 연산자(Elvis Operation)](#2-엘비스-연산자elvis-operation)
        + [3. Safe Cast](#3-safe-cast)
        + [4. Collection의 Null 객체를 모두 제거](#4-collection의-null-객체를-모두-제거)
5. [Higher-order-function(고차 함수)]()
6. [inline functions](#inline-functions)
    + [고차함수의 Runtime penalties](#고차함수의-runtime-penalties)
    + [inline function 구현 및 동작 원리](#inline-function-구현-및-동작-원리)
    + [noinline keyword](#noinline-keyword)
7. [Collections](#collections)
    + [List](#list)
        + [List : Immutable](#list--immutable)
        + [List : Mutable](#list--mutable)
    + [Set](#set)
        + [Set : Immutable](#set--immutable)
        + [Set : Mutable](#set--mutable)
    + [Map](#map)
        + [Map : Immutable](#map--immutable)
        + [Map : Mutable](#map--mutable)
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
+ 데이터 클래스에 abstract(추상), open(상속), sealed(인터페이스), inner(내부 클래스) 를 붙일 수 없습니다.
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
+ 이 함수를 호출한 객체를 이어지는 함수 블록의 인자로 전달한다.

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
+ 인자로 받은 객체를 이어지는 함수 블록의 리시버로 전달, block 함수의 결과를 반환

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
+ 인자가 없는 익명 함수처럼 사용하는 형태와 객체에서 호출하는 형태 제공, 함수형 인자 block 을 호출하고 결과 반환 또는 호출한 객체를 함수형 인자 block의 리시버로 전달하고 그 결과를 반환한다.

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

![image](https://user-images.githubusercontent.com/33089715/119781488-0b95ae80-bf06-11eb-808a-9f10231d3cd4.png)
출처 : https://medium.com/androiddevelopers/kotlin-standard-functions-cheat-sheet-27f032dd4326


## 초기화 지연 - lateinit, lazy
+ 코틀린에서는 변수 선언을 먼저하고 초기화는 뒤로 미루는 기능들을 제공한다. 
+ 장점 : 사용할지 모르는 데이터를 미리 초기화할 필요가 없어서 성능 향상에 도움이 된다.
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

## Null safety
코틀린은 자바와 다르게 Nullable과 Non-nullable 타입으로 프로퍼티를 선언할 수 있다. Non-nullable 타입으로 선언하면 객체가 null이 아닌 것을 보장하기 때문에 null check 등의 코드를 작성할 필요가 없다.

**`?.` 연산자 : Null Safe Operator**

> 자바의 경우 int, boolean과 같은 primitive type을 제외한 객체들은 항상 null이 될 수 있다. 

### Nullable과 Non-Nullable 프로퍼티
+ 타입을 선언할 때 `?` 를 붙이면 null을 할당할 수 있는 프로퍼티이고, `?` 가 붙지 않으면 null이 허용되지 않는 프로퍼티이다.
```kotlin
var nullable: String? = "nullable"
var nonNullable: String = "non-Nullable"
```
+ nullable 프로퍼티는 null을 할당할 수 있지만, nonNullable에 null을 할당하려고 하면 컴파일 에러가 발생한다.
```kotlin
nullable = null      // 컴파일 성공
nonNullable = null   // 컴파일 에러
```
+ 장점
    + Null Pointer Exception과 같은 예외가 발생하지 않을 수 있다.
    + 자바처럼 try-catch 구문을 많이 쓰지 않아도 된다.

### 코틀린에서 Null Pointer Exception(NPE)이 발생하는 경우
+ 코틀린에서는 NPE가 발생하지 않을 것 같지만 자바의 라이브러리를 쓰는 경우 NPE가 발생할 수 있다. 
+ 자바에서는 non-nullable 타입이 없기 때문에 자바 라이브러리를 사용할 때 nullable 타입으로 리턴된다.

### nullable 타입을 non-nullable 타입으로 변경하기
+ 코틀린에서 아래와 같은 자바 라이브러리를 사용한다고 가정
+ 아래 함수는 String을 리턴하며, 코틀린에서는 이 타입을 nullable인 `String?` 으로 인식한다.
```java
String getString() {
  String str = "";
  ....
  return str;
}
```

+ 코틀린에서 이 함수의 리턴 값을 non-nullable인 `String`으로 변환하고 싶다면?
+ 아래 코드처럼 대입하면 컴파일 에러가 발생한다.
    + 이유? `String?` 타입을 `String` 타입에 할당하려고 했기 때문
```kotlin
var nonNullString1: String = getString()     // 컴파일 에러
```

+ 반면 아래 코드는 컴파일 된다.
    + 이유? `!!` 연산자를 사용했기 때문이다.
    + `!!` 연산자는 객체가 null 이 아닌 것을 보장하고, 만약 null이라면 NPE를 발생시킨다.
```kotlin
var nonNullString2: String = getString()!!   // 컴파일 성공
```

### 안전하게 nullable 프로퍼티 접근하기
#### **1. 조건문으로 nullable 접근**
if-else를 이용하는 방법, nullable 접근 전에 if로 null을 체크한다.
```kotlin
val b: String? = "Kotlin"
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```
+ 장점 : 구현이 쉽다.
+ if-else 루프가 반복되는 경우 가독성을 해칠 수 있다.(if-else 루프 지옥)

#### **2. Safe call 연산자로 nullable 접근**
Safe call은 객체를 접근할 떄 `?.`로 접근하는 방법을 말한다.
+ 예를 들어 아래 코드에서 `b?.length`를 수행할 때 `b`가 null이라면 `length`를 호출하지 않고 null을 리턴한다. 그렇기 때문에 NPE가 발생하지 않는다.
```kotlin
val a: String = "Kotlin"
val b: String? = null
println(b?.length)
println(a?.length) // Unnecessary safe call
```
+ 위의 코드에서 a?.length는 불필요하게 Safe call을 사용하고 있다. a는 Non-nullable이기 때문
+ 아래 여러 객체로 둘러 쌓인 String에 접근하는 코드. 
    + 이 객체들 중 null이 있으면 null을 리턴한다.
```kotlin
println(a?.b?.c?.d?.length)
```

### 안전하게 nullable 프로퍼티 할당
어떤 프로퍼티를 다른 프로퍼티에 할당할 때, 객체가 null인 경우 default 값을 할당하고 싶을 수 있다. 
+ 자바에서는 삼항연산자를 사용하여 아래 코드처럼 객체가 null인 경우 default값을 설정해줄 수 있다.
```java
String b = null;
int l =  b != null ? b.length() : -1;
```
+ 하지만 코틀린은 삼항 연산자를 지원하지 않는다.

#### **1. if-else**
자바와는 다르게 코틀린은 한줄로 if-else를 쓸 수 있다. 
+ 다음은 if-else를 사용하여 삼항연산자와 동일한 내용을 구현한 코드
```kotlin
val l = if (b != null) b.length else -1
```
#### **2. 엘비스 연산자(Elvis Operation)**
엘비스 연산자는 `?:`를 말한다. 삼항연산자와 비슷한데 `?:` 왼쪽의 객체가 `null`이 아니면 이 객체를 리턴하고 null이라면 `?:`의 오른쪽 객체를 리턴한다. 
+ 다음은 위의 if-else 예제를 엘비스 연산자를 사용하여 구현한 코드입니다.
```kotlin
val l = b?.length ?: -1
```

#### **3. Safe Cast**
코틀린에서 형변환할 때 Safe Cast를 이용하면 안전하다.
+ 아래 코드에서 string은 문자열이지만 Any타입이다.
+ as?를 이용하여 String과 Int로 형변환을 시도하고 있다. 
+ String은 가능하기 때문에 성공하였고, Int는 타입이 맞지 않기 때문에 null을 리턴

```kotlin
val string: Any = "AnyString"
val safeString: String? = string as? String
val safeInt: Int? = string as? Int
println(safeString)
println(safeInt)
```
+ 실행 결과
    + safeInt는 캐스팅이 실패하여 null이 할당
```
AnyString
null
```

#### **4. Collection의 Null 객체를 모두 제거**
Collection에 있는 Null 객체를 미리 제거할 수 있는 함수 제공
+ 다음은 List에 있는 null 객체를 `filterNotNull` 메소드를 이용하여 삭제하는 코드
```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
println(intList)
```
+ 실행결과
    + null이 제거된 나머지 아이템들만 출력

```
[1, 2, 4]
``` 

## Higher-order-function(고차 함수)
Higher-order-function이란 다음 2가지 중 하나 이상을 만족하는 함수를 말한다.
+ 함수를 파라미터로 전달 받는 함수
+ 함수를 리턴하는 함수

**예시**
```kotlin
object Main {
    @JvmStatic
    fun main(args: Array<String>) {
        println(simpleHigherOrderFunction({ x, y -> x + y }, 10, 20)) //30
    }

    fun simpleHigherOrderFunction(sum: (Int, Int) -> Int, a: Int, b: Int): Int = sum(a, b)
}
```
+ simpleHigherOrderFunction의 파라미터 : sum 이라는 함수 하나, Int 값 2개 전달 받는다.
+ sum 파라미터는 Int 값 2개를 전달 받아 Int 값을 반환하는 함수
+ 위의 예시는 함수를 파라미터로 전달받았기 때문에 **Higher-order-function** 이다.

### 관련 질문
#### **Lambda Function과 High Order Function에 대해 설명해보세요**
Kotlin에서는 함수형 프로그래밍을 지원합니다.
High Order Function(고차 함수)란, 함수를 인수로 취하거나 함수를 결과로 반환할 수 있는 함수입니다. Android Studio에서 자주 사용하는 Call-Back Method 등이 고차함수입니다.
이러한 고차함수에서 매개변수로 주어지는 식을 Lambda Expression(람다 표현식)이라고 부릅니다.

## inline functions
코틀린에서 고차함수를 사용하면 추가적인 메모리 할당 및 함수 호출로 Runtime overhead가 발생한다. inline functions는 내부적으로 함수 내용을 호출되는 위치에 복사하며, Runtime overhead를 줄여준다.

### 고차함수의 Runtime penalties
+ kotlin은 다음과 같이 함수를 인자로 전달하는 Lambda expression을 정의할 수 있다.
```kotlin
fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}

fun main(args: Array<String>) {
    var result = someMethod(2, {println("Just some dummy function")})
    println(result)
}
```
+ 위의 코드를 다음과 같이 자바로 변환해보면, `someMethod`메소드 호출을 위해 객체를 생성한다.
```java
public final class InlineFunctions {
   public static final InlineFunctions INSTANCE;

   public final int someMethod(int a, @NotNull Function0 func) {
      func.invoke();
      return 2 * a;
   }

   @JvmStatic
   public static final void main(@NotNull String[] args) {
      int result = INSTANCE.someMethod(2, (Function0)null.INSTANCE);
      System.out.println(result);
   }

   static {
      InlineFunctions var0 = new InlineFunctions();
      INSTANCE = var0;
   }
}
```
+ 내부적으로 객체 생성과 함수 호출을 하도록 구현이 되어있으며, 이런 부분이 성능을 떨어뜨릴 수 있다.

### inline function 구현 및 동작 원리
+ 다음처럼 함수 앞에 inline 키워드를 붙이면 inline function이 된다.
```kotlin
inline fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}
```
+ 위의 코드가 컴파일될 때, 컴파일러는 함수 내부의 코드를 호출하는 위치에 복사한다.
+ 컴파일 되는 바이트코드의 양은 많아지지만, 함수 호출을 하거나 추가적인 객체 생성하는 부분은 없다.
+ 위의 코드를 자바로 컴파일 해보면, 다음과 같이 코드가 복사된 것을 알 수 있다.
```java
public final class InlineFunctions {

   @JvmStatic
   public static final void main(@NotNull String[] args) {
      int a = 2;
      int var5 = false;
      String var6 = "Just some dummy function";
      System.out.println(var6);
      int result = 2 * a;
      System.out.println(result);
   }

}
```

+ 일반 함수보다 성능이 좋다.
+ 하지만 inline function은 내부적으로 코드를 복사하기 때문에, 인자로 전달받은 함수는 다른 함수로 전달되거나 참조될 수 없다.
```kotlin
inline fun newMethod(a: Int, func: () -> Unit, func2: () -> Unit) {
    func()
    someMethod(10, func2)
}

fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}

fun main(args: Array<String>) {
    newMethod(2, {println("Just some dummy function")},
            {println("can't pass function in inline functions")})
}
```
+ 위의 코드에서 `newMothod()`는 inline으로 선언되었으며, `someMethod()`를 호출하며 함수를 인자로 전달한다.
+ 이 코드를 컴파일해보면 `someMethod(10, func2)` 때문에 컴파일이 실패한다. inline 함수에서 인자로 전달받은 함수는 참조할 수 없기 때문에 전달하는 것이 불가능 하다.

```
Error:(9, 24) Kotlin: Illegal usage of inline-parameter 'func2' in 'public final inline fun newMethod(a: Int, func: () -> Unit, func2: () -> Unit): Unit defined in example.InlineFunctions'. Add 'noinline' modifier to the parameter declaration
```

### noinline keyword
+ 위의 예제처럼 모든 인자를 inline으로 처리하고 싶지 않은 경우 인자 앞에 noinline 키워드를 사용하면 그 인자는 inline에서 제외된다. 즉, 인자를 다른 함수의 인자로 전달할 수 있다.
+ 아래 코드는 위의 코드에서 `newMethod()`의 인자 `func2` 앞에 noinline 키워드를 붙였다.
    + 이 키워드가 붙은 함수만 제외하고 모두 inline으로 처리된다.
```kotlin
inline fun newMethod(a: Int, func: () -> Unit, noinline func2: () -> Unit) {
    func()
    someMethod(10, func2)
}

fun someMethod(a: Int, func: () -> Unit):Int {
    func()
    return 2*a
}

@JvmStatic
fun main(args: Array<String>) {
    newMethod(2, {println("Just some dummy function")},
            {println("can't pass function in inline functions")})
}
```
+ 위의 코드를 자바로 변환한 것, `func2()`를 제외한 나머지 코드들은 모두 inline으로 처리되었다.
```java
public final void newMethod(int a, @NotNull Function0 func, @NotNull Function0 func2) {
   func.invoke();
   this.someMethod(10, func2);
}

public final int someMethod(int a, @NotNull Function0 func) {
   func.invoke();
   return 2 * a;
}

@JvmStatic
public static final void main(@NotNull String[] args) {
   String var6 = "Just some dummy function";
   System.out.println(var6);
   this_$iv.someMethod(10, func2$iv);
}
```
## Collections
자바의 List, Map, Set 등을 Collection이라고 하며, 코틀린의 Collections은 기본적으로 **Mutable과 Immutable을 별개로 지원한다**.
+ Mutable로 생성하면 추가, 삭제가 가능하지만, Immutable로 생성하면 수정이 안된다.

![image](https://user-images.githubusercontent.com/33089715/119791326-b1015000-bf0f-11eb-950f-dd092c933e81.png)
출처 : kotlinlang.com

### List
List는 데이터가 저장하거나 삭제될 때 순서를 지키는 Collection으로, Mutable과 Immutable을 모두 지원한다.

#### List : Immutable
+ `listOf<타입>(아이템, )`로 Immutable List를 생성 및 초기화를 할 수 있다.(타입 생략 가능)
+ Immutable이기 때문에 get만 가능하다. 
    + List의 getter는 자바처럼 get(index)도 지원하고 배열처럼 [index]도 지원한다.

```kotlin
val fruits= listOf<String>("apple", "banana", "kiwi", "peach")
// val fruits= listOf("apple", "banana", "kiwi", "peach") -> 타입 생략 가능
println("fruits.size: ${fruits.size}")
println("fruits.get(2): ${fruits.get(2)}")
println("fruits[3]: ${fruits[3]}")
println("fruits.indexOf(\"peach\"): ${fruits.indexOf("peach")}")

//실행 결과
//fruits.size: 4
//fruits.get(2): kiwi
//fruits[3]: peach
//fruits.indexOf("peach"): 3

```

#### List : Mutable
+ 수정가능한 List는 `mutableListOf`로 선언
+ listOf와 대부분 비슷하지만, 추가 및 삭제가 가능하다.
+ remove, add, addAll, removeAt, replace, replaceAll, contains, forEach 등

```kotlin
val fruits= mutableListOf<String>("apple", "banana", "kiwi", "peach")
fruits.remove("apple")
fruits.add("grape")
println("fruits: $fruits")

fruits.addAll(listOf("melon", "cherry"))
println("fruits: $fruits")
fruits.removeAt(3)
println("fruits: $fruits")

// 실행 결과

// fruits: [banana, kiwi, peach, grape]
// fruits: [banana, kiwi, peach, grape, melon, cherry]
// fruits: [banana, kiwi, peach, melon, cherry]
```

### Set
Set은 동일한 아이템이 없는 Collection, 아이템 순서는 특별히 정해져 있지 않다.
+ null 객체 갖고 있을 수 있다.(1개만)
+ immutable과 Mutable을 별개로 지원

#### Set : Immutable
+ `setOf<타입>(아이템들)`로 객체를 생성

```kotlin
val numbers = setOf<Int>(33, 22, 11, 1, 22, 3)
println(numbers)
println("numbers.size: ${numbers.size}")
println("numbers.contains(1): ${numbers.contains(1)}")
println("numbers.isEmpty(): ${numbers.isEmpty()}")

// 실행 결과
// [33, 22, 11, 1, 3]
// numbers.size: 5
// numbers.contains(1): true
// numbers.isEmpty(): false

```
#### Set : Mutable
+ Mutable은 `mutableSetOf<타입>(아이템들)` 로 생성할 수 있다. 
+ Mutable이기 때문에 추가, 삭제가 가능하다. 
+ List와 비슷한 메소드들을 지원

```kotlin
val numbers = mutableSetOf<Int>(33, 22, 11, 1, 22, 3)
println(numbers)
numbers.add(100)
numbers.remove(33)
println(numbers)
numbers.removeIf({ it < 10 }) // 10 이하의 숫자를 삭제
println(numbers)

// 실행 결과
// [33, 22, 11, 1, 3]
// [22, 11, 1, 3, 100]
// [22, 11, 100]

```

### Map
Map은 key와 value를 짝지어 저장하는 Collection으로, Map의 key는 유일하기 때문에 동일한 이름의 key는 허용되지 않는다. Immutable과 Mutable을 별개로 지원한다.

#### Map : Immutable
+ Map은 `mapOf<key type, value type>(아이템)`로 생성할 수 있다. 
+ 아이템은 Pair객체로 표현하며, Pair에 key와 value를 넣을 수 있다.
+ Pair(A, B)는 A to B로 간단히 표현이 가능하다. 이런 문법이 가능한 것은 to가 `Infix`이기 때문이다.

```kotlin
val numbersMap = mapOf<String, String>(
    "1" to "one", "2" to "two", "3" to "three")
println("numbersMap: $numbersMap")
val numbersMap2 = mapOf(Pair("1", "one"), Pair("2", "two"), Pair("3", "three"))
println("numbersMap2: $numbersMap2")

// 실행해보면 모두 동일한 값을 갖고 있습니다.
// numbersMap: {1=one, 2=two, 3=three}
// numbersMap2: {1=one, 2=two, 3=three}
```
+ 데이터를 읽는 것도 다른 Collection과 유사하다. get(index), [index] 모두 지원한다.
+ `keys`와 `values`는 key와 value 만으로 구성된 Set을 리턴

```kotlin
val numbersMap = mapOf<String, String>(
    "1" to "one", "2" to "two", "3" to "three")
println("numbersMap.get(\"1\"): ${numbersMap.get("1")}")
println("numbersMap[\"1\"]: ${numbersMap["1"]}")
println("numbersMap[\"1\"]: ${numbersMap.values}")
println("numbersMap keys:${numbersMap.keys}")
println("numbersMap values:${numbersMap.values}")

for (value in numbersMap.values) {
    println(value)
}

// 실행결과
// numbersMap.get("1"): one
// numbersMap["1"]: one
// numbersMap["1"]: [one, two, three]
// numbersMap keys:[1, 2, 3]
// numbersMap values:[one, two, three]
// one
// two
// three
```

#### Map : Mutable
+ Mutable은 mutableMapOf<key type, value type>(아이템)로 생성
+ 객체 추가는 put 메소드이며, Pair를 사용하지 말고 인자로 key와 value를 넣어주면 된다.
+ put도 배열 방식을 지원
+ 그 외에 자바의 Map과 유사

```kotlin
val numbersMap = mutableMapOf<String, String>(
        "1" to "one", "2" to "two", "3" to "three")
println("numbersMap: $numbersMap")

numbersMap.put("4", "four")
numbersMap["5"] = "five"
println("numbersMap: $numbersMap")

numbersMap.remove("1")
println("numbersMap: $numbersMap")

numbersMap.clear()
println("numbersMap: $numbersMap")

// 실행결과
// numbersMap: {1=one, 2=two, 3=three}
// numbersMap: {1=one, 2=two, 3=three, 4=four, 5=five}
// numbersMap: {2=two, 3=three, 4=four, 5=five}
// numbersMap: {}
```
---
### 출처
https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html

https://codechacha.com/ko/kotlin-late-init/

https://wsx2792.tistory.com/16

https://codechacha.com/ko/kotlin-inline-functions/