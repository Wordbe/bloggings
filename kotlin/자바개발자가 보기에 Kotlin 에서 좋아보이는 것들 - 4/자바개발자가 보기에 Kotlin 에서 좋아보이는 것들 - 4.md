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

