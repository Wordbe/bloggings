# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 3



# 클래스와 객체에서

## 제네릭(Generics: in, out, where)

여기서는 공변, 반공변에 대한 개념이 조금 필요하다. 혹시 간단하게 알고싶다면 [공변, 반공변이란 무엇일까](https://wordbe.tistory.com/302) 를 참고해보면 좋을 것 같다.

<br />

<br />

### 선언위치 변성 (Declaration-site variance)

```kotlin
interface Factory<out T> {
    fun pop(): T

    fun push(t: T) // compile error
}
```

- `Factory` 라는 클래스는 제네릭 파라미터 T 에 대해 공변(covariant) 임을 의미한다.
- `Factory` 클래스는 T 의 producer 만 가능하다고 생각할 수 있다. consumer 는 될 수 없다. 따라서 메소드 파라미터로 타입이 T 인 변수를 받아오지 못한다. (컴파일 에러가 난다.)
- 따라서 아래 코드처럼 안정성을 확보하면서도, 컴파일에러를 일으키지 않고 잘 사용할 수 있다.
  - 자바에서는 위와 같이 클래스 타입 파라미터에 declarataion-site variance 를 기술할 수 없어, 아래와 같은 코드는 타입이 안정적이지 못하다고 판단하여(그렇지 않음에도 불구하고) 컴파일 에러가 난다.

```kotlin
fun toAnys(strs: Factory<String>): Factory<Any> {
    val anys: Factory<Any> = strs // 정상
    return anys
}
```

<br />

<br />

반공변 예시로서는 아래 예시가 적당하다.

```kotlin
interface Comparable<in T> {
  operator fun compareTo(other: T): Int
}
```

- operator 는 뒤에 설명하겠지만, +, -, *, / 등 과 같은 기호를 재정의할 수 있도록 도와준다. (연산자 오버로딩)
- 아무튼, 타입 파라미터를 반공변(contracovariance)으로 만드려면 `in` 키워드를 붙이면 된다. 이러한 declaration-site variance 가 있는 클래스는 consumer 이며, 해당 타입을 파라미터로 받는 메소드만 만들 수 있고, 리턴하는 메소드는 만들 수 없다.
- `in` 은 `Comparable` 이 consumer 임을 보장하므로 아래 코드가 컴파일 시간에 타입 안정성을 확보할 수 있다.

```kotlin
fun toDouble(num: Comparable<Number>): Comparable<Double> {
    val doubleNum: Comparable<Double> = num
    return doubleNum
}
```

<br />

<br />

### 사용위치 변성 (Use-site variance)

- 코틀린 문서에서는 `use-site variance` 를 `Type projections` 라고 말한다.

당연하게도, 한 클래스에 특정 타입을 반환하는 메소드 또는 특정 타입을 파라미터로 받기만 하는 메소드만 있을 수는 없다. 섞이기 마련이다. (이 때는 선언위치 변성을 사용할 수 없다.) `Array` 가 대표적 예다.

```kotlin
class Array<T>(val size: Int) {
  operator fun get(index: Int): T { ... }
  operator fun set(index: Int, value T) { ... }
}
```

참고)

- get 에 할당된 연산자는 `a[i]` 처럼 사용할 수 이다. 즉, i 번째 원소를 가져오는 것이다.
- set 에 할당된 연산자는 `a[i] = 1` 처럼 사용할 수 있다. 즉, i 번째 배열에 특정 원소를 할당할 수 있다.

이런 클래스가 아래와 같은 상황에 사용된다고 해보자. 

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
  for (i in from.indices) {
    to[i] = from[i]
  }
}
```

위 메소드 `copy` 는 예외가 충분히 발생할 수 있다. 예를 들어 from 에 `Array<String>` 로 들어왔는데, `Array<Int>` 타입인 to 에 할당하려고 하면 `ClassCastException` 이 발생한다. 

<br />

 하지만, 사용자가 정상적으로 사용할 것이라 생각하고 위 메소드를 만들었다고 해보자. 예를 들어 `Array<Int>` 를 받아 `Array<Any>` 에 넣는 것이다. 생각해보면 말이 되지만, 컴파일러는 실행하기도 전에 오류를 탐지한다. 이유는 `Array<Int>` 가 `Array<Any>` 의 공변임을 알 수 없어 타입 안정성을 파악할 수 없다고 판단하기 때문이다.

<br />

따라서 아래처럼 use-site variance 로 타입 파라미터에 변성을 명시해주면 런타임에 예외가 발생하지 않음을 컴파일 시간에 알 수 있어 타입 안정성을 확보할 수 있다. 아래는 from 파라미터가 `Array<Any> `의 공변임을 명시한다. 이 부분은 자바의 `Array<? extends Obejct>` 와 같다.

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```

<br />

`in` 의 사용 예시는 아래와 같다.

```kotlin
fun fill(dest: Array<in String>, value: String) { ... }
```

- dest 파라미터가 `Array<String>` 의 반공변임을 명시하여, 타입 안정성을 확보한다.
- 예를 들어 `Array<Any>` 인 파라미터에도 `String` 타입의 value 를 채워넣는 것이 가능하다. 

<br />

<br />

<br />

### 스타 프로젝션 (Star-projections)

- 타입 아규먼트에 대해 아는 정보가 없지만, 제네릭을 안전한게 사용하고 싶을 때 사용한다.
- `Foo<out T : TUpper>`  라고 정의된 클래스 (T 는 TUpper 를 상속하는 공변인 파라미터이다.) 에서 `Foo<*>` 는 `Foo<out TUpper` 과 같다. 즉, T 를 모르는 상태에서도 `Foo<*>` 로부터 `TUpper` 의 값을 안전하게 읽어낼 수 있다.
- `Foo<in T>` 라고 정의된 클래스 (T 는 반공변인 파라미터이다.)에서 `Foo<*>` 는 `Foo<in Nothing>` 과 같다. 즉, T를 모르는 상태에서 `Foo<*>` 에 안전하게 쓸 수 있는게 없다.
- `Foo<T : TUpper>` 라고 정의된 클래스 (T는 TUpper 를 상속하는 불변인 파라미터이다.)에서 `Foo<*>` 는 값을 읽을 때는  `Foo<out TUpper>` 과 같고, 값을 쓸 때는 `Foo<in Nothing>` 과 같다.

<br />

### 상한클래스(Upper bounds) 선언

- 상한선할 때 상한이다.
- 특정 파라미터가 두 개 이상의 클래스를 상속받은 한정된 클래스임을 나타내고 싶다면 `where` 을 사용할 수 있다.

```kotlin
fun <T> foo(list: List<T>, threshold: T): List<String>
  where T : CharSequence,
  			T : Compareable<T> {
  return list.filter { it > threshold }.map { it.toString() }
}
```

이 때 T 타입은 반드시 명시된 두 개의 인터페이스를 구현한 클래스여야 한다.

<br />

### 타입 지워짐 (Type erasure)

- 런타임에는 제네릭타입의 인스턴스가 타입 아규먼트의 실제정보를 가지고 있지 않다. 이를 지워졌다(erased)고 표현하는데 `Foo<Bar>` 은 런타임에 `Foo<*>` 로 지워진다.
- 이 때문에 제네릭 타입의 인스턴스가 특정 타입 아규먼트로 생성되었는지 런타임에 확인 할 방법이 없다. 따라서 컴파일 시 아래와 같은 `is` 사용을 금지한다.
  - `ints is List<Int>`, `list is T` 불가능
- 하지만 코틀린에서는 star-projected 된 타입은 인스턴스 확인이 가능하다.
  - `something is List<*>` 가능

<br />

제네릭 함수의 타입 아규먼트도 컴파일 시점에만 확인이 가능하다. 하지만 `reified` 파라미터를 사용한 `inline` 함수에서는 타입 파라미터로 타입 확인과 타입 캐스팅이 가능하다. 이렇게 선언된 함수는 호출시 실제 타입 아규먼트가 그 때마다 보전되어 처리된다. (inline 된다)

```kotlin
inline fun <reified A, reified B> Pair<*,*>.of(): Pair<A,B> {
  if (first !is A || second !is B) return null
  return first as A to second as B
}
```

<br />

<br />

---

## Object 선언

- 코틀린에서는 `object` 키워드로 싱글톤 패턴을 만들기 쉽다.
- `object declaration` 의 초기화는 객체에 첫번째로 접근할 때 생성되고 (지연초기화), 스레드 세이프하다.

```kotlin
object StringUtils {
  fun isBlank(s: String): Boolean {
    ...
  }
}

StringUtils.isBlank("   ")
```

<br />

### Companioin object

- `object declaration` 은 클래스 안에서도 사용될 수 있는 데 이때 `companion` 키워드와 함께 사용해주어야 한다.
- 컴패년 오브젝트는 이를 포함하는 클래스가 로드될 때 초기화된ㄷ. 자바의 static 초기화와 같다.

```kotlin
class Coin {
  companion object Factory {
    fun create(): Coin = Coin()
  }
}

val coin = Coin.create()
```

`companion object` 는 Factory 라고 이름을 작성할 수도 있고, 생략할 수도 있다. 사용방법은 클래스 이름에 바로 메소드 (또는 필드) 를 접근하여 사용하면 된다. 마치 자바의 `static` 문법과 흡사하다.

하지만 `companion object` 는 런타임에도 생성된 객체의 인스턴스 멤버이며, 따라서 인터페이스도 구현할 수 있다.

```kotlin
interface Factory<T> {
  fun create(): T
}

class Coin {
  companion object : Factory<Coin> { // object 이름 생략, 인터페이스 구현
    override fun create(): Coin = Coin()
  }
}

val coin = Coin.create()
```



<br />

<br />

---

## 위임 (Delegation)

위임 패턴은 이미 상속, 구현의 좋은 대안으로 증명이 되었다. 코틀린은 위임 패턴에서 필요한 보일러플레이트 코드를 자동으로 작성해주는 문법을 지원한다.

이런 패턴을 문법 속에 녹여든게 정말 좋은 것 같다. 개발자에게는 코드를 짜는데 더 효율적이고 깔끔한 방법을 구현할 수 있는 다양한 선택지가 주어진 셈이다. 아래와 같이 `by` 키워드를 사용해 위임 패턴을 사용할 수 있다.

```kotlin
interface Bill {
  fun print()
}

class ShopBill(val x: Int) : Base {
  override fun print() { print(x) }
}

class Counter(bill: Bill) : Bill by b

fun main() {
  val bill = ShopBill(100)
  Counter(bill).print() // 100
}
```

<br />

<br />

---

## 위임 프로퍼티 (Delegated Properties)

이 기능은 유용하게 쓸 수 있을 것 같다. 예를 들어 getter 로 프로퍼티 조회 후 (또는 setter 로 프로퍼티 변경 후) 무언가를 하고 싶다면 위임을 사용하면 좋다.

```kotlin
class Bill {
  var title: String by OrderAlarm
}

class OrderAlarm {
  operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
    return "$thisRef, 계산서 ${property.name} 이 호출되었습니다."
  }
  
  operator fun getValue(thisRef: Any?, property: KProperty<*>, value: String) {
    println("$thisRef, 계산서 ${property.name} 에 $value 가 입력되었습니다.")
  }
}

val bill = Bill()
bill.title // "Bill@12ab34412, 계산서 title 이 호출되었습니다."
bill.title = "Pizza" // "Bill@12ab34412, 계산서 title 에 Pizza 가 입력되었습니다."
```

- 위처럼 위임 프로퍼티를 사용하면 이벤트 처리 기능 같이 특정 프로퍼티가 호출되었을 때 어떤 기능이 동작되는 것을 만들 수 있다.

위임을 하면 컴파일러에 의해 숨겨진 프로퍼티가 (아래코드에서 `prop$delegate`) 생성된다.

```kotlin
class C {
    var prop: Type by MyDelegate()
}

// this code is generated by the compiler instead:
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

gettter 에서는 위임 클래스의 `getValue()` 가, setter 에서는 위임 클래스의 `setValue()` 가 호출되는 것을 알 수 있다.

<br />

### Standard delegates

위임된 프로퍼티를 사용하는 표준 delegates 를 살펴보자.

<br />

**Lazy properties**

지연 초기화를 위한 `by lazy` 문법 또한 위임 프로퍼티의 일부이다. `lazy()` 는 람다를 받아 `Lazy<T>` 를 반환하는 함수인데 프로퍼티를 지연 초기화하는데 사용된다. 람다 로직은 최초 한번만 실행되고 이 때 결과값을 기억해서 다음번 호출때 이 값을 바로 반환한다.

```kotlin
val price: Int by lazy {
  println("가격 입력")
  1000
}

fun main() {
  println(price)
  println(price)
}

가격 입력
1000
1000
```

<br />

**Observable properties**

`Delegates.observable()` 을 활용해보자. 프로퍼티에 값이 할당되면 handler 가 실행되도록 구성할 수 있다.

```kotlin
class Bill {
  val price: Int by Delegates.observable(0) {
    prop, old, new -> println("$old -> $new")
	}
}

fun main() {
  val bill = Bill()
  bill.price = 1000
  bill.price = 2000
}

0 -> 1000
1000 -> 2000
```

<br />

Map 을 사용할 때는 유연하게 값을 프로퍼티에 매핑할 수 있다. Json 을 받아서 바로 객체에 넣는다면 아래 문법은 매우 유용할 것이다.

```kotlin
class Bill(val map: Map<String, Any?>) {
  val title: String by map
  val price: Int by map
}

val bill = Bill(mapOf("title" to "orange", "price" to 2000))
println(bill.title) // orange
println(bill.price) // 2000
```







---

## 타입 가명 (Type aliases)

기존에 만든 사용하는 타입의 이름이 너무 길 때 유용하다.

```kotlin
typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

<br />

함수의 타입을 정할 수도 있다. 이름을 지어준다.

```kotlin
typealias MyPredicate<T> = (T) -> Boolean
```

<br />

Nested class 를 사용하면 자연스럽게 이름이 길어지는데, 이것도 가명을 부여할 수 있다.

```kotlin
class MusicalInstrument {
  inner class Violin
}

typealias MIViolin = MusicalInstrument.Violin
```

type aliases 는 새로운 타입을 만드는 것이 아니라 단지 원래 있는 타입을 참조하는 가명을 만드는 것이다.
