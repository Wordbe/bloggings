<br />

<br />

# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 1



# 1) 기본 타입(Basic Types)

<br />

## 숫자(Numbers)

- Byte(8 bits), Short(16), Int(32), Long(64)
- Float(32), Double(64)
  - 자바에서 숫자 타입은 primitive, wrapper 타입으로 나뉘어있다. 보통은 primitive 를 쓰고, 콜렉션이나 제네릭 안에 사용되는 타입으로 wrapper 타입을 많이 사용한다. 혹은 null 필요할 경우도 wrapper 를 사용한다. (혹은 그냥 모두 wrapper 로 통일한다.)
  - 코틀린에서는 이런 고민없이 타입이 모두 wrapper 이다.
- 컴파일러가 가장작은 범위의 타입으로 값을 자동으로 암시해준다.

```kotlin
val three = 3 // Int
val threeBillion = 3_000_000_000 // Long
val one = 1L // Long
```

<br />

## 문자열(Strings)

### String Template

```kotlin
val name = "whale"
val wholeName = "blue $whale"
// val wholeName = "blue ${whale}"
```

- 이 문법은 사용하기 참 간편해서 좋다.

<br /><br />



---

# 2) 제어 흐름(Control flow)

## 조건문과 반복문(Conditions and loops)

<br />

### If

- if 가 표현문(expression) 이다. 즉, 값을 리턴한다. (그래서 대신 삼항연산자는 없다.)

```kotlin
val max = if (a > b) a else b
```

- if 를 표현문으로 사용할 경우, 위와 같이 else 는 반드시 작성해주어야 한다.

<br />

### when

- switch case 구문과 비슷하다.
- when 도 표현문 (expression) 이어서 값을 리턴한다.
- enum, sealed class 처럼 모든 하위 요소가 포함된 클래스와 같이 사용하면 좋다. 
  - 이 때 하위 요소 별로 분기를 모두 만들었다면, else 를 사용하지 않아도 된다.
  - else 를 사용하면, 만약 새로운 하위 요소가 생겼을 때 when 절의 로직을 고치지 않으면 논리적으로 잘못 될 가능성이 있다. 따라서 else 를 사용하지 않는 것이 좋고, 이 때 새로운 하위 요소가 추가되었다면 when 문에서 컴파일 에러가 난다. (새로운 케이스를 다뤄야 한다고 알려준다.)
- 또한 is 와 함께쓰면, smart cast 의 도움으로 해당 타입의 메소드와 프로퍼티에 바로 접근할 수 있다.

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

- 리턴된 변수와 함께 사용할 수 있다.

```kotlin
fun Request.getBody() =
    when (val response = executeRequest()) {
        is Success -> response.body
        is HttpError -> throw HttpException(response.status)
    }
```

<br />

### Exception

- 모든 코틀린 예외는 `Throwable` 클래스를 상속한다.
- `if ~ else`, `when` 과 마찬가지로 `try ~ catch, finally` 또한 표현문(expression) 이다. 값을 리턴한다.

코틀린은 체크드 예외(Checked exception) 가 없다! 

- 이와 관련해서는 여러 논쟁이 있을 수 있지만, 코틀린은 체크드 예외를 없애는 것으로 동의했고, 이에 따라 강제로 예외처리를 할 필요가 없다. 
- 큰 소프트웨어에서는 발생할 수 있는 예외를 조사하는게 생산성을 감소시키고, 코드 질도 감소시킬 수 있다. (Bruce Eckel)
- 하지만 만약 다른 언어(Java 등)의 코드에게 코틀린 예외를 전달해주고 싶다면, `@Throws` 어노테이션을 사용할 수 있다.



<br />

<br />

# 3) 패키지와 임포트(Packges and imports)



## Imports as

- 이름이 겹치고, 패키지가 다른 클래스를 하나의 파일에서 사용할 때, 가명을 만들 수 있다.

```kotlin
import co.wordbe.pomodoro.client.todo.TodoClient
import co.wordbe.pomodoro.client.todo.Todo as ExternalTodo
...

@Service
class TodoGatheringService(
    private val todoRepository: TodoRepository,
    private val todoClient: TodoClient,
) {

    fun gatherTitles(id: Long): List<String> {
        val todo: Todo = todoRepository.findById(id) ?: throw NotFoundException();
        val externalTodo: ExternalTodo = todoClient.getById(id)

        return listOf(todo.title, externalTodo.title)
    }
}
```

<br />

<br /><br /><br /><br /><br />

