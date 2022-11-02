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
