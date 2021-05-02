# Java

[JVM](#jvm)

1. [JVM의 기능은?](#jvm의-기능은)

2. [자바 컴파일 과정을 설명해보세요.](#자바-컴파일-과정을-설명해보세요)

3. [JVM 메모리 영역을 설명해보세요.](#jvm-메모리-영역을-설명해보세요)

[Garbage Collection](#garbage-collection)

1. [Garbage Collection이란?](#garbage-collection이란)
2. [Garbage Collection의 동작 방식은?](#garbage-collection의-동작-방식은)
3. [Minor GC가 무엇이고, 어떻게 동작하는지?](#minor-gc가-무엇이고-어떻게-동작하는지)
4. [Major GC가 무엇이고, 어떻게 동작하는지?](#major-gc가-무엇이고-어떻게-동작하는지)

## JVM

### JVM의 기능은?
JVM은 Java Virtual Machine, 자바 가상 머신으로, Stack기반의 가상머신입니다. 

주요 기능으로는 자바 애플리케이션을 클래스 로더를 통해 읽어 들여 자바 AOI와 함께 실행하는 것입니다. 또, 자바 프로그램이 어느 기기나 운영체제 상에서도 독립적으로 실행될 수 있도록 하고, 프로그램 메모리를 관리하고 최적화합니다.

</br>

### 자바 컴파일 과정을 설명해보세요.
1. 자바 소스코드(.java)를 자바 컴파일러가 읽어들여 자바 바이트 코드(.class)로 변환시킵니다.
2. 컴파일된 바이트 코드를 JVM 클래스 로더(Class Loader)에게 전달합니다.
3. 클래스 로더는 동적 로딩(Dynamic Loading)을 통해 필요한 클래스들을 로딩 및 링크하여 런타임 데이터 영역(Runtime Data Area), 즉 JVM의 메모리에 올립니다.
4. 실행 엔진(Excution Engine)은 JVM 메모리에 올라온 바이트 코드들을 명령어 단위로 하나씩 가져와서 실행합니다. 이 때 실행 엔진은 두 가지 방식으로 변경합니다.

    1. 인터프리터 : 바이트 코드 명령어를 하나씩 읽어서 해석하고 실행합니다. 전체적인 실행 속도가 느리다는 단점이 있습니다.
    2. JIT 컴파일러(Just in time compiler) : 바이트 코드 전체를 컴파일하여 바이너리 코드로 변경하고 이 후에는 해당 메서드를 더이상 인터프리팅 하지 않고, 바이너리 코드로 직접 실행하는 방식입니다. 전체적인 실행 속도는 인터프리팅 방식보다 빠릅니다.  

</br>

### JVM 메모리 영역을 설명해보세요.
Runtime Data Area로 프로그램을 수행하기 위해 OS에서 할당받은 메모리입니다. PC Register, JVM Stack영역, Native Method Stack, Method Area, Heap영역으로 구성되어 있습니다.

<img src="https://user-images.githubusercontent.com/33208360/116801888-10568500-ab49-11eb-9497-979053d8e60c.png" alt="image" style="zoom: 50%;" />

먼저, **PC Register**는 Thread가 생성될 때마다 생성되는 공간으로, Thread마다 하나씩 존재합니다. thres가 어떤 부분의 명령을 실행해야할 지에 대한 기록을 하는 부분으로 현재 수행 중인 JVM명령의 주소를 갖습니다.

두 번째로, **JVM Stack**영역은 프로그램 실행과정에서 임시로 할당되었다가 메소드를 빠져나가면 바로 소멸되는 특성의 데이터를 저장하기 위한 영역입니다. 각종 형태의 변수나 임시 데이터, 스레드나 메소드의 정보를 저장합니다.

(:heavy_plus_sign: 특징: 메소드를 호출할 때마다 그 메소드만을 위한 공간으로 각각의 Stack 프레임이 생성됩니다. 메소드 수행이 끝나면 프레임별로 삭제됩니다.

또, 호출된 메소드의 매개변수, 메소드 안에서 사용되는 값인 지역변수, 리턴 값 및 연산시 일어나는 값 등을 임시로 저장합니다. ) 

세 번째로, **Native Method Stack**은 자바 프로그램이 컴파일되어 생성되는 바이트 코드가 아닌 실제 실행할 수 있는 기계어로 작성된 프로그램을 실행시키는 영역입니다. Java가 아닌 다른 언어로 작성된 코드를 위한 공간으로, 일반 프로그램처럼 커널이 스택을 잡아 독자적으로 프로그램을 실행시키는 영역입니다.

네 번째로, **Method Area**(=Class Area = Stack Area)는 클래스 정보를 처음 메모리 공간에 올릴 때 초기화되는 대상을 저장하기 위한 메모리 공간입니다.

마지막으로, **Heap**영역은 객체를 저장하는 가상 메모리 공간으로, `new` 연산자로 생성된 객체와 배열을 저장합니다. (Class Area영역에 올라온 클래스들만 객체로 생성할 수 있다.)

Heap은 세 부분으로 나뉘는데, Permanent 영역, New/Young영역, Old영역으로 나뉩니다.

1. Permanent 영역은 생성된 객체 정보의 주소값이 저장되는 공간입니다. Class Loader에 의해 로드되는 class, Method 등에 대한 메타정보가 저장되는 영역이고, JVM에 의해 사용됩니다. (+ 또, Reflection을 사용하여 동적으로 클래스가 로딩되는 경우에 사용됩니다. 내부적으로 Reflection기능을 자주 사용하는 Spring프레임워크를 이용할 경우 이 영역에 대한 고려가 필요.)
2. New/Young 영역은 Eden과 Survivor 0/1공간이 있는데, Eden은 객체들이 최초로 생성되는 공간이며, Survivor0/1 은 Eden에서 참조되는 객체들이 저장되는 공간입니다.
3. Old영역은 New Area에서 일정 시간 참조되고 있는, 살아남은 객체들이 저장되는 공간입니다.

(:heavy_plus_sign: 인스턴스는 소멸 방법과 소멸 시점이 자역변수와 다르기 때문에 Heap이라는 별도의 영역에 할당됩니다. )

**:heavy_plus_sign: Java의 Heap 영역**

Heap 영역은 처음 설계될 때,

1. 대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.
2. 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

이 2가지를 전제로 설계되었습니다. 즉, 객체는 대부분 일회성이며, 메모리에 오랫동안 남아있는 경우가 드물다는 것을 의미합니다. 그렇기 때문에 객체의 생존기간에 따라 물리적인 Heap 영역을 나누게 되었는데, 이에 따라 Young, Old 총 2가지 영역으로 설계되었습니다.

**1) Young 영역**

새롭게 생성된 객체가 할당(Allocation)되는 영역으로, 대부분의 객체가 금방 Unreachable 상태가 되기 때문에, 많은 객체가 Young영역에 생성되었다가 사라집니다. 이 Young 영역에 대한 Garbage Collection을 **Minor GC**라고 부릅니다. 

**2) Old 영역**

Young 영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역으로, 복사되는 과정에서 대부분 Young 영역보다 크게 할당되고 크기가 큰 만큼 Garbage는 적게 발생합니다. 이 Old 영역에 대한 Garbage Collection을 **Major GC** (Full GC)라고 부릅니다. 

**+) Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우**

이 경우를 대비해 Old 영역에는 512바이트의 덩어리로 되어있는 카드 테이블이 존재합니다. 카드 테이블에는 Old영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 그에 대한 정보가 표시됩니다. Young 영역에서 Minor GC가 실행될 때, 모든 Old 영역에 존재하는 객체를 검사하여 참조되지 않는 Young영역의 객체를 식별하는 것은 비효율적이기 때문입니다. 그렇기 때문에 Minor GC가 진행될 때, 카드 테이블만 조회하여 GC의 대상인지 식별할 수 있도록 하고 있습니다. 

</br>

## Garbage Collection

### Garbage Collection이란?

프로그램을 개발하다보면 유효하지 않은 메모리가 발생하게 되는데, JVM의 Garbage Collector가 불필요한 메모리를 알아서 정리해주는 과정을 말합니다. 그래서 Java를 이용해 개발하면 개발자가 메모리를 직접 해제하지 않아도 되기 때문에 개발효율을 높일 수 있습니다. 

Heap영역의 Young영역에서 일어나는 Minor GC와 Old영역에서 일어나는 Major GC가 있습니다.

(:heavy_plus_sign: Java나 Kotlin에서는 메모리 누수를 방지하기 위해 Garbage Collector가 주기적으로 검사하여 메모리를 청소해줍니다.)

(:heavy_plus_sign: Java에서 `System.gc()`를 이용해 호출할 수 있지만, 해당 메소드를 호출하는 것은 시스템의 성능에 매우 큰 영향을 미치므로 절대 호출해서는 안됨. )

### Garbage Collection의 동작 방식은?

기본적으로 'Stop the World', 'Mark ans Sweep' 2가지의 공통적인 단계를 따릅니다.

먼저, **Stop The World**는 GC를 실행하기 위해 JVM이 애플리케이션의 실행을 멈추는 작업입니다. GC가 실행될 대는 GC를 실행하는 thread를 제외한 모든 thread들의 작업이 중단되고, GC가 완료되면 작업이 재개됩니다. 

다음으로, **Mark and Sweep**과정으로, GC는 Stack의 모든 변수 또는 Reachable 객체를 스캔하면서 각각이 어떤 객체를 참고하고 있는지를 탐색합니다. 그리고 사용되고 있는 메모리를 식별하는데 이러한 과정을 Mark라고 합니다. 이후에 Mark되지 않은 객체들을 메모리에서 제거하는데, 이러한 과정을 Sweep이라고 합니다.

### Minor GC가 무엇이고, 어떻게 동작하는지?

Heap의 Young 영역에 대한 Garbage Collection으로, Young영역은 1개의 Eden영역과 2개의 Survivor영역으로 총 3가지로 나뉘어지는데, Eden영역은 새로 생성된 객체가 할당되는 영역이고, Survivor은 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역입니다.

Minor 동작 방식은 

1. 먼저 새로 생성된 객체가 Eden 영역에 할당됩니다.

2. 객체가 계속 생성되어 Eden영역이 꽉 차게 되고, Minor GC가 실행됩니다.

   Eden영역에서 사용되지 않는 객체의 메모리는 해제되고, Eden영역에서 살아남은 객체는 1개의 Survivor영역으로 이동됩니다.

3. 1~2번의 과정이 반복되다가, Survivor 영역이 가득 차게 되면 Survivor영역의 살아남은 객체를 다른 Survivor영역으로 이동시킵니다.

4. 이러한 과정을 반복하여 계속 살아남은 객체는 Old 영역으로 이동(Promotion)됩니다.

이때, 객체의 생존 횟수를 카운트하기 위해 Minor GC에서 살아남은 횟수를 의미하는 **Age**를 Object Header에 기록합니다. 그리고 기록된 **Age**를 보고 Promotion여부를 결정합니다.

(:heavy_plus_sign: Survivor 영역 중 1개는 반드시 사용되어야 합니다. 두 영역 모두 데이터가 존재하거나, 두 영역의 사용량이 모두 0이라면 현재 시스템이 정상적인 상황이 아님을 파악할 수 있습니다. )

### Major GC가 무엇이고, 어떻게 동작하는지?

Heap의 Old 영역에 대한 Garbage Collection입니다. Young영역에서 오래 살아남은 객체가 Old 영역으로 Promotion되는데, 객체들이 계속해서 Promotion되어 Old영역의 메모리가 부족해지면 Major GC가 발생하게 됩니다.

(:heavy_plus_sign: Young영역은 일반적으로 Old영역보다 크기가 작기 때문에 GC가 빠르게 진행됩니다. 그렇기 때문에 Minor GC는 애플리케이션에 크게 영향을 주지않습니다. 하지만 Old영역은 Young영역보다 크며, Young영역을 참조할 수도 있기 때문에 Major GC는 일반적으로 비교적 시간이 오래 걸립니다.)