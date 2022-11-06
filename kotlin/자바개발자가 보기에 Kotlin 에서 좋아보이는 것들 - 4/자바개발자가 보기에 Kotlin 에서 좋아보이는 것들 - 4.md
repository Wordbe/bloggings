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





