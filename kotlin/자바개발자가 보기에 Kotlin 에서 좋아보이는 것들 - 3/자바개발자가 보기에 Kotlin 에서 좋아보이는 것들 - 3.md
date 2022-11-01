# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 3



# 클래스와 객체에서

## 제네릭(Generics: in, out, where)

여기서는 공변, 반공변에 대한 개념이 조금 필요하다. 혹시 간단하게 알고싶다면 [공변, 반공변이란 무엇일까](https://wordbe.tistory.com/302) 를 참고해보면 좋을 것 같다.



### 선언 방식 변성 (Declaration-site variance)

```kotlin
interface Factory<out T> {
    fun pop(): T

    fun push(t: T) // compile error
}
```

- `Factory` 라는 클래스는 제네릭 파라미터 T 에 대해 공변(covariant) 임을 의미한다.
- `Factory` 클래스는 T 의 producer 만 가능하다고 생각할 수 있다. consumer 는 될 수 없다. 따라서 메소드 파라미터로 타입이 T 인 변수를 받아오지 못한다. (컴파일 에러가 난다.)
- 따라서 아래 코드처럼 안정성을 확보하면서도, 컴파일에러를 일으키지 않고 잘 사용할 수 있다.
  - 자바에서는 선언형 방식으로 variance 를 기술할 수 없어, 아래와 같은 코드는 타입이 안정적이지 못하다고 판단하여(그렇지 않음에도 불구하고) 컴파일 에러가 난다.

```kotlin
fun toAnys(strs: Factory<String>) {
  val anys: Factory<Any> = strs
}
```









### Star-projections

- 타입 아규먼트에 대해 아는 정보가 없지만, 제네릭을 안전한게 사용하고 싶을 때 사용한다.
- `Foo<out T : TUpper>`  라고 정의된 클래스 (T 는 TUpper 를 상속하는 공변인 파라미터이다.) 에서 `Foo<*>` 는 `Foo<out TUpper` 과 같다. 즉, T 를 모르는 상태에서도 `Foo<*>` 로부터 `TUpper` 의 값을 안전하게 읽어낼 수 있다.
- `Foo<in T>` 라고 정의된 클래스 (T 는 반공변인 파라미터이다.)에서 `Foo<*>` 는 `Foo<in Nothing>` 과 같다. 즉, T를 모르는 상태에서 `Foo<*>` 에 안전하게 쓸 수 있는게 없다.
- `Foo<T : TUpper>` 라고 정의된 클래스 (T는 TUpper 를 상속하는 불변인 파라미터이다.)에서 `Foo<*>` 는 값을 읽을 때는  `Foo<out TUpper>` 과 같고, 값을 쓸 때는 `Foo<in Nothing>` 과 같다.



### Upper bounds

- 특정 파라미터가 두 개 이상의 클래스를 상속받은 한정된 클래스임을 나타내고 싶다면 `where` 을 사용할 수 있다.

```kotlin
fun <T> foo(list: List<T>, threshold: T): List<String>
  where T : CharSequence,
  			T : Compareable<T> {
  return list.filter { it > threshold }.map { it.toString() }
}
```

이 때 T 타입은 반드시 명시된 두 개의 인터페이스를 구현한 클래스여야 한다.



### Type erasure

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



