# Kotlin

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

---
### 출처
https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html