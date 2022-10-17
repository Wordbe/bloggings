# Kotlin 서버 개발하기 2



<br />

# Pomodoro

최근에 알게 된 공부/일 집중 방법으로 '뽀모도로' 기법이 있습니다.

쉽게 말해 25분 공부하고, 5분 쉬는 것인데요.

실제 타이머로 25분을 재면서 어떤 일에 몰두하면, 생각보다 집중이 잘 되서 효과를 볼 수 있습니다.

특히, 25분이 끝나올 때 다급해지는 마음에 더 집중이 잘 되기도 합니다.

뽀모도로 기법은 이 포모도르를 4번 시행하면 1시간이 채워지는데, 이 때는 5분보다 좀 더 긴 휴식 시간을 가지길 권장합니다.

이렇게 한 싸이클이 끝나면, 다음 목표를 향해 다음 싸이클을 반복하면 됩니다.

지금 이 블로그 글쓰기 역시 뽀모도로 기법을 통해 집중해서 진행하고 있습니다.

<br />

![](https://blog.kakaocdn.net/dn/bu73eZ/btrNvxKUTEh/O4nrz633b91CaFncDKpi11/img.png)

이번 프로젝트는 뽀모도로 기법을 통한 자기 관리 앱을 만들 것 입니다.

<br />

<br />

## Domain

먼저 할일을 담을 객체(`Todo`)가 있어야 합니다.

 ```kotlin
 class Todo(
     val id: Long,
     val title: String,
     val content: String,
 ) {
 }
 ```

<br />

이 글은 코틀린 자체에 대한 이야기를, 프로젝트를 만들어보면서 하므로 사소한 문법도 간단하게 같이 설명합니다.

**변수**

- 코틀린에서 val 키워드는 변수가 변경되지 않음을 나타냅니다. (불변)
- 자바의 final 키워드와 비슷합니다.
- 이와 반대로 가변 변수를 만들고 싶다면, var 키워드를 붙입니다.

참고)
- val : value, 불변 값을 뜻함
- var : variable, 변수를 뜻함

<br />

**클래스**

- 코틀린에서 클래스 다음 ( ) 소괄호 안에 필드를 넣으면, 필드와 동시에 생성자를 만들 수 있습니다.
- { } 중괄호 안에는 아래와 같은 요소를 만들 수 있고, 아무 것도 없으면 생략가능합니다.
  - 필드
  - 다른 생성자
  - 메소드
  - `companion object` 등 (자바의 static 변수, 메소드를 코틀린에서 사용하는 방법)



<br />

<br />

위와 같이 생성한 코틀린 파일은 컴파일하여 자바로 역컴파일하면 아래와 같은 결과가 나옵니다.

```java
...
public final class Todo {
   private final long id;
   @NotNull
   private final String title;
   @NotNull
   private final String content;

   public final long getId() {
      return this.id;
   }

   @NotNull
   public final String getTitle() {
      return this.title;
   }

   @NotNull
   public final String getContent() {
      return this.content;
   }

   public Todo(long id, @NotNull String title, @NotNull String content) {
      Intrinsics.checkNotNullParameter(title, "title");
      Intrinsics.checkNotNullParameter(content, "content");
      super();
      this.id = id;
      this.title = title;
      this.content = content;
   }
}
```

즉 getter 메소드와 모든 필드를 인자로 받는 생성자가 자동으로 만들어집니다.

- 롬복의 `@Getter`, `@AllArgsConstructor` 를 붙인 결과와 같습니다.
- 코틀린은 이런 bolierplate code 를 줄여주는 기능을 언어적 차원에서 제공한다는 장점이 있습니다.

<br />

<br />

## Repository

이제 우리는 R2DBC 를 통해서 도메인 엔티티와 DB 테이블을 연결해주려고 합니다.

어노테이션을 통해 매핑해 봅시다.

**Todo.kt**

```kotlin
@Table(name = "todo")
class Todo(
    @Id
    var id: Long? = null,
    val title: String,
    val description: String,
)
```

- DB 안에 있는 todo 테이블에 대응되는 엔티티가 됩니다.
- id 변수의 타입은 nullable 한 Long 타입으로 설정했습니다. 그 이유는 Todo 객체를 생성할 때 id 없이 만든 상태로 repository 에게 넘겨주기 때문입니다. 이 프로젝트에서는 H2 DB 에서 auto increment 로 id 값을 자동으로 1개씩 올려 생성해주는데, 우리는 이 값을 그대로 받아올 것입니다.
- 코틀린에서는 이렇게 `?` 문법을 통해서  nullable 타입을 not nullable 타입과 구분해서 선언할 수 있는데, 이는 자바에서 개발자가 생각하지못해 자주 발생하는 `NPE(NullPointerException)` 예외를 컴파일타임에 쉽게 잡아줄 수 있습니다. 실제로 코틀린에서 자바보다 NPE 발생률이 현저하게 감소했다는 통계 결과가 있습니다.
- 또한 id 만 `val` 에서  `var` 로 변경시켜, `getId()` 와 더불어  `setId()` 메소드도 생기도록 만들었습니다. 레포지토리에서 save를 사용하는 구조상 id 를 나중에 변경해야하는 부분이다보니 id 에 대해서는 가변 변수로 만들어주었습니다.



<br />

혹은 클래스 자체를 data class 로 만들면 val 로 만들어도 동작이 잘되기도 합니다. 이 부분은 r2dbc 에서 불변 객체를 사용할 수 있도록 하면서도, 엔티티로서 사용할 수 있게끔 내부적으로 id 변경을 허용하는 로직을 담아 만들어져 있을거라 추측하고 있습니다.

```kotlin
@Table(name = "todo")
data class Todo(
    @Id
    val id: Long? = null,
    val title: String,
    val description: String,
)
```

코틀린은 이렇게 data class 를 사용해서, equals(), hashcode(), toString(), component(), copy() 함수를 컴파일타임에 자동으로 만들어줍니다.

말그대로 data 를 옮겨 주고 받는 DTO(Data Transfer Object) 등에 잘 어울릴 수 있는 기능이라고 생각합니다.

반면, 위와 같이 객체 지향적으로 사용하고 싶은 엔티티에서도 사용할 수 있을 것 같기도 한게 Todo 엔티티를 객체로써 잘 사용하고자 하면, 위 메소드들은 꼭 필요하기 때문입니다.

이 프로젝트에서는 일단 엔티티에서는 data class 를 사용하지 않고 진행하도록 하겠습니다.



<br />

이제 repository 를 선언해봅시다.

지금 프로젝트는 Spring Reactive Web + R2DBC 를 사용하고 있으므로,  `R2dbcRepository` 인터페이스를 상속받아 레포지토리를 만듭니다.

```kotlin
interface TodoRepository : R2dbcRepository<Todo, Long>
```

이렇게 되면 코드에 리액티브 스트림의 Publisher 구현체인 Mono, Flux 가 많이 보이게될 것입니다.

<br />

하지만 우리는 코틀린의 코루틴을 통한 리액티브 앱을 만들 계획이므로 `CoroutineCrudRepository` 를 상속하려고 합니다.

**TodoRepository.kt**

```kotlin
interface TodoRepository : CoroutineCrudRepository<Todo, Long>
```

마치 Spring Data JPA 에서 JpaRepository 를 상속받는 것과 형식이 비슷하고, 인터페이스도 거의 똑같으니 기존에 사용하는 것과 큰 괴리 없이 사용할 수 있습니다.

위 레포지토리는 자동으로 스프링 빈으로 등록됩니다.

<br />

<br />



## Service

**TodoService.kt**

```kotlin
@Service
class TodoService(
    val todoRepository: TodoRepository
) {

    suspend fun create(todoRequest: TodoRequest): Todo {
        return todoRepository.save(todoRequest.toEntity())
    }
}
```

위에서 만들었던 `TodoRepository` 를 주입해서 사용하려고 합니다. 의존성 주입(DI) 는 스프링에게 맡기고, `TodoService` 는 생성자에 `TodoRepository` 빈을 받아오면 됩니다. 

필드를 선언하면서 생성자를 만들 수 있다는 코틀린 문법의 장점 덕분에 코드가 깔끔해졌습니다.

- Todo 를 리턴하는데 함수 앞에 `suspend` 키워드는 이 함수를 언제든 중단할 수 있고, 나중에 다른 곳에서 재개할 수 있도록 만든 것입니다. 코루틴 활용과 연관있습니다.

<br />

<br />

## Controller

**TodoController.kt**

```kotlin
@RestController
class TodoController(
    val todoService: TodoService
) {

    @PostMapping("/api/v1/todos")
    suspend fun create(@RequestBody todoRequest: TodoRequest) =
        TodoResponse(todoService.create(todoRequest))
}
```

- 컨트롤러에서는 서비스를 주입받습니다.
- 코틀린은 위처럼 함수의 반환 문법으로 `{ return value }` 대신 `= value ` 로 간단하게 표현이 가능합니다.
  - 코틀린은 타입을 추론할 수 있다면 생략이 가능한데, 개인적으로는 타입이 눈에 보이면 코드를 보기 쉬워서, 리턴 타입이 명확히 보이는 표현을 사용하면 좋을 것 같습니다.

<br />

추가로 여기서 활용되는 Dto 에 주목해보겠습니다.

**TodoDto.kt**

```kotlin
data class TodoRequest(
    val title: String,
    val description: String = ""
) {
    fun toEntity(): Todo {
        return Todo(
            title = title,
            description = description
        )
    }
}

data class TodoResponse(
    val id: Long,
    val title: String,
    val description: String
) {
    companion object {
        operator fun invoke(todo: Todo): TodoResponse {
            return TodoResponse(
                id = todo.id!!,
                title = todo.title,
                description = todo.description
            )
        }
    }
}
```

코틀린은 이와 같이 **하나의 파일에 여러 클래스를 구성**할 수 있습니다. Dto 같이 관련있는 객체를 한 곳에 모아 관리하기 좋습니다.

<br />

엔티티와 Dto 를 변환하는 로직을 Dto 안에 넣고 싶다면, 내부 메소드를 생성해 로직을 만들 수도 있습니다.

특히, 인스턴스를 생성하지 않고 클래스를 통해 바로 호출할 수 있는 정적변수와 메소드는 `companion object` 안에서 만들 수 있습니다.

여기서는 추가적으로 `operator fun invoke` 를 통해 생성자를 연산자 오버로딩 했습니다.

- operator 키워드는 연산자를 오버로딩 한다는 뜻이고, 생성자호출에 대한 메소드이름이 invoke 입니다.

- Todo 객체를 파라미터로 받는 생성자를 하나 만들었다고 생각하시면 편합니다. 그러면 호출하는 쪽에서 생성자를 호출하듯이 깔끔하게 사용할 수 있을 것입니다.

- 아래 코드와 똑같이 동작합니다.

  코틀린에서 생성자를 만드는 문법도 간략하게 주석으로 설명해놓았습니다.

  ```kotlin
  data class TodoResponse ( // primary constructor (주생성자)
      val id: Long,
      val title: String,
      val description: String
  ) {
    	// secondary constructor (부생성자)
      constructor(todo: Todo) : this( // delegation to the primary constructor (주생성자있는 경우 주생성자에게 위임해야 함)
          id = todo.id!!,
          title = todo.title,
          description = todo.description
      )
  }
  ```

<br />

자바에서 자주사용하는 static of 메소드 처럼 만들고 싶을 경우 아래와 같이 가능합니다.

```kotlin
data class TodoResponse (
    val id: Long,
    val title: String,
    val description: String
) {
    companion object {
        fun of(todo: Todo): TodoResponse {
            return TodoResponse(
                id = todo.id!!,
                title = todo.title,
                description = todo.description
            )
        }
    }
}
```

또한 마지막으로 한 가지 다른 방법이 있는데, 확장함수를 이용하는 것입니다. Todo 의 확장함수로 TodoResponse 를 반환하는 함수를 생성했습니다.

```Kotlin
data class TodoResponse (
    val id: Long,
    val title: String,
    val description: String
)

fun Todo.toResponse(): TodoResponse {
    return TodoResponse(
        id = id!!,
        title = title,
        description = description
    )
}
```

팀에서는 사용하기 편한 방법을 약속해서 사용하면 좋을 것 같습니다. 혹은 다른 Mapper 계층이 있어도 좋겠습니다.



<br />

## 실행

마지막으로, 지금까지 만들어놓은 애플리케이션을 실행해보겠습니다.

인텔리제이에서 HTTP 요청을 편하게 보낼 수 있도록 문법을 지원합니다.

![](https://blog.kakaocdn.net/dn/ckeEPx/btrNwCLd790/JSuZRpRnibNiUoAeXpn0Kk/img.png)

<br />

아래와 같이 Todo 생성 요청을 작성해 봅시다.

**generated-requests.http**

```http
###
# @no-log
POST http://localhost:8080/api/v1/todos
Content-Type: application/json

{
  "title": "블로그 작성하기",
  "description": "1) 노트북을 편다. 2) 글을 작성한다."
}
```

![](https://blog.kakaocdn.net/dn/lmiGD/btrNyGzAE4W/HaSIZNaaw8jptJmWtVIk31/img.png)

성공적으로 Todo 를 생성하여 데이터베이스에 저장한 것을 확인했습니다.

<br /><br /><br /><br /><br />