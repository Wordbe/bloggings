# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 2







이 글은 이전에 작성했던 **[자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 1](https://wordbe.tistory.com/300)** 에 이어서 연재됩니다.

<br />

<br />

# 클래스와 오브젝트(Classes and objects) 안에서

<br />

<br />

## 클래스(Classes)

### Trailing comma

```kotlin
data class Todo(
    val id: Long,
    val title: String,
    val description: String,
)
```

- 맨 밑에 작성된 프로퍼티 맨 끝에도 , 를 사용할 수 있어서 일관적으로 작성할 수 있다.
- trailing comma 가 없다면 새로운 프로퍼티를 추가할 때, 2줄을 수정해야 해야 한다. 반면에 trailing comma 가 있다면 한 줄만 수정해도 된다.
- 사소하지만 이 부분은 코드리뷰시 수정된 부분만 보여주도록 해서 가독성을 높인다. 

<br />

<br />

## 상속(Inheritance)

- 모든 코틀린 클래스는 `Any` 클래스를 상속한다.
- `Any` 는 `equals()`, `hashCode()`, `toString()` 메소드를 가진다.
- 코틀린 클래스는 기본적으로 상속 불가이고, 상속하려면 `open` 키워드를 앞에 붙여 주어야 한다.
  - 상속은 객체지향에서 안티패턴을 만들 수 있는 경우가 생기므로, 기본적으로 상속을 불가능하게 세팅해놓았다.

<br />

<br />

## 프로퍼티(Properties)

- `var` 은 변할 수 있는 변수(mutable) 
  - 기본 getter, setter 가 생긴다.
- `val` 은 읽기전용 (read-only immutable)
  - 기본 getter 만 생긴다.
- 사용할 때 아래와 같이, 프로퍼티에 바로 접근해서 사용할 수 있다.

```kotlin
class Address {
  var street: String = "Seolleung"
  var city: String = "Gangnam"
}

fun copyAddress(address: Address): Address {
  val result = Address()
  result.name = address.name // result.name(setter), address.name(getter)
  result.city = address.city
  return result
}
```

<br />

<br />

커스텀 getter, setter 표현 방식도 깔끔하다.

```kotlin
class MyString {
	var myString: String = ""
  	get() = this.toString()
	  set(value) {
  	  myString = "[$value]"
  	}
}
```

- `var myString: String = ""` 처럼  initializer 로 초기값을 설정해주고 (혹은 생성자에서 초기화)
- 바로 밑에 gettter 와 setter 를 만들 수 있다.

<br />

참고) **Backing fields**

- 코틀린에서는 값을 메모리에 지정해두고 싶을 때 field 를 사용할 수 있다. 프로퍼티가 이와 같은 `backing field` 가 필요하면 접근자에서 `field` 키워드를 사용해서 사용할 수 있다. `field` 를 사용한 접근자가 하나라도 있을 때 프로퍼티의 `backing field` 가 만들어진다.

```kotlin
var number = 0
	set(value) {
    if (value >= 0)
    	field = value
    	// number = value // 이렇게 하면 StackOverFlow 에러가 나는데, setter 함수가 재귀적으로 무한히 호출되기 때문이다.
  }
```

<br />

<br />

## 함수형 인터페이스 (Functional (SAM) interfaces)

- 함수형 인터페이스는 하나의 추상 메소드SAM (Single Abstract Method)를 가지는 인터페이스이다.
- 코틀린에서는 아래와 같이 `interface` 앞에 `fun` 키워드만 붙여서 간결하게 나타낼 수 있다.
  - 자바에서는 추상메소드가 하나인 인터페이스는 함수형 인터페이스가 될 수 있지만  `@FunctionalInterface` 어노테이션을 붙여 함수형 인터페이스임을 명시해주는 것이 좋다.


```kotlin
fun interface StringPredicate {
    fun accept(s: String): Boolean
}

val isApple = StringPredicate { it == "apple" }
println(isApple.accept("apple")) // true
```

- 함수형 인터페이스로 정의하면, SAM 을 포함하여 다른 abtract 가 아닌 멤버들도 추가할 수 있고, 다른 인터페이스를 구현, 상속할 수도 있다.
- 단, `typealias` 에 비해서 문법적으로나 런타임 시간적으로나 비용이 더 든다. 런타임시 이 인터페이스로 변환이 필요하기 떄문이다.

<br />

**typealias**

- `typealias` 를 통해 간단하게 함수타입을 만들 수 있다. 간단하게 함수타입을 만들 때 정말 좋다.

```kotlin
typealias StringPredicate = (s: String) -> Boolean

val isApple: StringPredicate = { it == "apple" }
println(isApple("apple")) // true
```

- kotlin 에서는 문자열을 `equals() `비교로도 할 수 있지만 `==` 로도 간단하게 가능하다.
- 함수형 인터페이스가 다채로운 기능을 제공하는 반면, `typealias` 는 간단하게 특정 함수 타입을 선언할 때만 사용된다.

<br />

<br />

## 가시성 지정자 (visibility modifier)

- 자바에서는 접근 지정자(Acess modifier)라고 말한다.
- 코틀린은 `private`, `protected`, `internal`, `public` 가시성이 넓어지며, `public` 이 기본이다.
  - 자바의 기본은 default 즉, 패키지 레벨인 것과는 다르다.
  - 사실 `public` 과 `private` 을 더 자주 사용하므로, 기본으로 `public` 을 사용하자는 아이이더는 좋은 것 같다.
- 여기서 특이한 점은 나머지는 자바와 같지만, `internal` 의 경우 같은 모듈 안에서 가시성을 제공한다는 것이다.
  - 모듈은 함께 컴파일되는 코틀린 파일의 집합인데, 즉 인텔리제이의 모듈개념을 말한다.
  - Maven 은 프로젝트 개념이고, Gradle 은 소스 세트 개념이다.

<br />

<br />

## 확장 (Extensions)

- 이 기능은 꽤나 유용하게 사용된다.
- 기존 클래스 안의 바디를 수정하지 않고, 밖에서 메소드를 추가하거나 프로퍼티의 게터, 세터를 변경할 수 있다.

```kotlin
class Todo(
    val id: Long,
    val title: String,
    val description: String,
)

class TodoResponse (
    val id: Long,
    val title: String,
    val description: String
)

fun Todo.toResponse(): TodoResponse {
    // 여기서는 Todo 의 내부 프로퍼티, 메소드 접근이 가능하다. Todo 자체는 this 로 접근 가능하다.
    return TodoResponse(
        id = id,
        title = title,
        description = description,
    )
}
```

- 이런식으로 `Todo` 클래스의 메소드를 밖에서 (해당 파일 top-level에서) 만들었다.
- 코틀린의 JPA 는 기존 자바 스프링 JPA 의 레포지토리에 `확장함수`를 이용해서 코틀린 전용 새로운 함수를 추가했다.

<br />

<br />

## 데이터 클래스 (Data classes)

```kotlin
data class Todo(val id: Long, val title: String)
```

- 컴파일러는 주생성자에 선언된 모든 프로퍼티에 대해 아래 멤버를 자동으로 생성한다.

  - `equals()`, `hashCode()`
  - `toString()`
  - `componentN()`, `copy()`

- `componentN()` 을 통해 구조 분해된 프로퍼티를 가져오는 함수가 생기면서 편리기능이 생기는데 대표적으로 아래와 같이 구조분해 선언(deconstructing declarations)을 사용할 수 있다.

  ```kotlin
  val (id, title) = Todo(1, "Reading a book")
  ```

- `copy()` 는 다음과 같이 생긴다.

  ```kotlin
  fun copy(val id: Long, val title: String) = Todo(id, title)
  ```

- 보일러플레이트 코드 생성을 위해 사용했던 lombok 을 대체할 수 있다.

개인 의견을 덧붙이자면, `데이터 클래스` 는 데이터를 담는 목적으로 만들어진 객체를 위해 만들어진 것 같은데, 위와 같은 메소드들은 OOP 를 위한 객체들에게 더 필요한 것들이다. 그래서 뭔가 기능적인 측면에서는 `data` 라는 키워드명은 잘 들어맞지 않는 것 같기도하다. 반면에 데이터를 담는 클래스, 즉 데이터 클래스라는 이름을 명시하기에는 적절하다.

코틀린은 이미 `data` 키워드 없이 아래와 같이만 생성해도, 생성자, 게터, 세터를 다 만들어준다. 그리고 이렇게만 해도 데이터를 실어나르는 DTO 역할을 하기에는 충분하다. 

```kotlin
class Todo(val id: Long, val title: String)
```

물론, `data` 를 추가한다면 동등비교나 로그출력, 구조분해가 있는 로직에서 사용하기 더 편리해질 것이다.

<br />

<br />

## 봉인 클래스 (Sealed classes)

- `sealed class` 는 클래스의 계층구조를 제한시켜주어서 상속된 객체들을 다루기 용이해진다.
- `sealed class` 의 모든 하위 클래스는 컴파일 시간에 컴파일러가 알 수 있다.
- `ApplicationException` 을 만들고, 각 에러마다 처리를 해주는 공통 로직이 있다고 가정해보자.

```kotlin
sealed class ApplicationException : RuntimeException()

class InvalidRequestException(val request: String) : ApplicationException()
class DatabaseException(val source: DataSource) : ApplicationException()

class ExceptionHandler {
    
    fun exceptionMessage(e: ApplicationException) = when(e) {
        is InvalidRequestException -> { "잘못된 요청입니다. ${e.request}" }
        is DatabaseException -> { "데이터베이스 에러입니다. ${e.source}" }
    }
}
```

- 위와 같이 `ApplicationException` 의 하위 예외들을 컴파일타임에 잡아줄 수 있다. 따라서 이 때 `when` 절은 `else` 구문이 필수가 아니다.
- 만약 새로운 예외가 `ApplicationException` 하위 클래스로 등록되었다면, `when` 절은 컴파일 에러가 난다.

<br />

<br />

<br />

<br />

<br />
