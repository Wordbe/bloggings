# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 4



# 함수 (Functions)

<br />

<br />

## 기본값, 이름있는 아규먼트 (Default, Named arguements)

- 파라미터가 기본 값을 가지도록 할 수 있다. (default arguments)
- 이름있는 아규먼트 (named arguments)를 사용하여, 함수를 호출할 때 어떤 파라미터에 어떤 값을 넣는지 명시할 수 있다.
- 파라미터가 여러 개 있을 때 특히 유용하다.

<br />

## varargs

- 여러 아규먼트를 한나의 변수에 받아올 수 있다.
- variable number of arguments 라 한다.

```kotlin
fun sum(vararg nums: Int) : Int {
  var result = 0
  for (num in nums) {
    result += num
  }
  return result
}

sum(1, 2, 3) // 6

val numbers = arrayOf(1, 2, 3)
sum(*numbers) // 6
```

- 만약 Array 를 아규먼트로 바로 넣고 싶다면, 코틀린에 있는 스프레드 연산자 (spread operator) 를 통해 쉽게 넣을 수 있다.

<br />

## 함수 스코프 (Function Scope)

코틀린은 자바와 다르게 top-level 함수 작성이 가능하다. 즉, 클래스 안에 말고, 파일에 그냥 바로 작성할 수 있다.

이와 더불어, 함수안에 함수를 정의할 수 있다. 이를 지역 함수 (local functions) 라고 한다.

지역 함수는 그 바로 바깥 함수에 있는 지역 변수에 접근할 수 있다. (클로저, closure)

```kotlin
fun outer() {
  val outerLocalVariable = 1
  fun inner() {
    if (outerLocalVariable == 1) {
      ...
    }
  }
}
```

<br />

<br />

## 람다 (Lambdas)

### 함수 타입 (Function Types)

- `A -> B` , `(A, B) -> C` 등
- 선택사항으로 파라미터(B)가 담긴 수신자 타입(A)을 명시할 수 있다. `A.(B) -> C`
- 함수 타입에 가명을 만들 수도 있다.
  - `typealias onClick = (Button, ClickEvent) -> Unit`

<br />

### 람다 표현식과 익명 함수 (Lambda expressions and anonymous functions)

- 람다 표현식과 익명 함수는 함수 리터럴이다.
- 함수 리터럴이란, 미리 선언되지 않고 표현문으로써 바로 전달이 가능한 함수를 말한다.

```kotlin
max(strings, { a, b -> a.length < b.length })
```

<br />

### Trailing lambdas

함수 파라미터 마지막에 함수가 온다면, 소괄호 바깥쪽에 람다 표현문을 아규먼트로 전달할 수 있다. 이 부분은 상당히 실용적이다.

즉, 위 max 함수를 아래와 같이 써도 된다.

```kotlin
max(strings) { a, b -> a.length < b.length }
```

<br />

### it

파라미터가 한 개만 있는 람다식은 `->` 를 생략할 수 있고, 그 파라미터를 `{ }` 안에서 `it` 으로 사용할 수 있다.

```kotlin
numbers.filter { it > 0 }
```

<br />

<br />

### 익명 함수

익명 함수는 람다와 같지만, 리턴 타입을 명시해줄 수 있는 장점이 있다. (람다식은 리턴 타입을 자동 추론한다.) 리턴 타입을 꼭 명시하고 싶다면 익명 함수를 사용하자.

```kotlin
fun(x: Int, y: Int): Int {
  return x + y
} 
```

익명 함수도 표현문 바디로 되어있다면 아래처럼 리턴타입을 자동으로 추론해준다.

```kotlin
fun(x: Int, y: Int) = x + y
```



<br />

<br />

<br />

# 널 안정성 (Null Safety)

코틀린은 널을 가질 수 있는 타입과 그렇지 않은 타입을 구분한다. 그리고 널 가능한 타입은 타입 뒤에 `?` 를 붙여서 나타낼 수 있다.

- null 을 관리하는 이유는, 널 참조 에러를 만든 것이 수십억 달러 비용 낭비를 야기했기 때문이다.

널 안정성을 컴파일 레벨에서 관리하게 만들었기 때문에 `NullPointerException` 예외 발생을 제한적으로 일어나게 되고, 개발자들이 예측 가능한 프로그래밍을 할 수 있는데 도움을 준다.

```kotlin
var a: String = "a"
a = null // 컴파일 에러

var b: String? = "b" // nullable type
b = null // 가능
```

<br />

## 안전 호출 (Safe Call)

nullable type 은 `?.` 를 붙여서 해당 타입이 가진 메소드를 안전하게 호출 할 수 있다. 만약, 메소드를 부르는 인스턴스가 `null` 인 경우 에러가 발생하는 대신에 `null` 을 리턴한다.

```kotlin
var a: String? = "a"
print(a?.length) // a

a = null
print(a?.length) // null
```

```kotlin
if (a == null) {
  return null
} else {
  return a.length
}
```

이런 로직이 내부에 있을 것이다.

<br />

safe call 은 연쇄적으로 호출할 때 유용하다.

```kotlin
user?.team?.head?.email
```

유저의 팀이 있다면, 팀을 호출하고, 그 팀의 사장이 있다면 사장을 호출하고, 그 사장의 이메일이 있다면 이메일을 호출 할 것이다.

만약, 하나라도 해당 프로퍼티가 없다면, `null` 을 반환한다. 

safe call 이 없고, 이런 기능을 호출하는 로직이 있다고 생각해보면, 런타임에 쉽게 `NPE` 가 발생할 수 있는 쿼리이다. 이렇게 널 가능한 타입을 통해 개발시점에 미리 해당 부분의 null 처리를 미리할 수 있도록 도와준다.

세이프 콜이 없다면, `if` 로 도배된 분기로직을 작성해야 할 것이다.

<br />

## 엘비스 연산 (Elvis operator)

널 가능한 타입에서 null 일 때 특별한 조치를 취하고 싶으면, `?:` 연산으로 간단하게 처리할 수 있다.

```kotlin
val length: Int = if (a != null) a.length else -1
```

위 로직을 아래와 같이 간단하게 바꿀 수 있다.

```kotlin
val length = a?.length ?: -1
```

`?:` 오른쪽 부분은 왼쪽 부분이 널이 아닐때만 실행된다.

코틀린에서는 `throw`, `return` 이 표현문이기 때문에, 아래와 같이도 사용할 수 있다.

```kotlin
fun add(user: User): String? {
  val team = user.getTeam() ?: return null
  val name = user.getName() ?: throw IllegalArgumentException()
  ...
}
```

<br />

<br />

## !! 연산

널 가능 타입에서 NPE 를 발생시키고 싶다면, `!!` 연산을 사용할 수 있다. 

확실히 널이 아니라고 생각되는 곳에 사용되기도 한다.

```kotlin
val b = a!!.length
```

a 가 널이면 NPE 가 발생한다.

<br />

<br />

## Safe casts

`ClassCastException` 을 방지하고자 할 때 사용할 수 있는 문법이 있다.

클래스 캐스트가 실패하면 `null` 을 리턴한다.

```kotlin
val b: String? = a as? String
```

