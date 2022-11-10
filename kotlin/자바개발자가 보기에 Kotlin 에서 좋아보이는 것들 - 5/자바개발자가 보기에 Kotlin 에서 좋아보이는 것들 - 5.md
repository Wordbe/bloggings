# 자바개발자가 보기에 Kotlin 에서 좋아보이는 것들 - 5

<br />

# 비동기 프로그래밍 기술 (Asynchronous Prgramming Techniques)

블로킹은 사용자들을 기다리게 만들고, 애플리케이션을 확장하는데 병목 현상을 초래한다.

애플리케이션에서 블로킹을 어떻게 막을 것인지 여러 방법이 있다.

<br />

<br />

## 쓰레딩 Threading

블로킹을 막기 위한 가장 잘 알려진 방법. 한 메인 프로그램에서 여러 쓰레드가 실행되며 동시 처리한다.

```kotlin
fun postItem(item: Item) {
  val token = preparePost()
  val post = submitPost(token, item)
  processPost(post)
}

fun preparePost(): Token {
  // 요청을 보내 토큰을 받음
  return token
}
```

하지만 단점이 있다.

- 쓰레드는 비싸다. 컨텍스트 스위칭 비용이 크다.
- 쓰레드는 쓰레드풀 안에서 갯수가 제한적이다. 이 때문에 서버 애플리케이션에서 병목현상을 일으킨다.
- 쓰레드를 지원하지 않는 플랫폼도 있다. (예. JavaScript)
- 쓰레드는 어렵다. 멀티쓰레드 프로그래밍에서 race condition 을 막고, 쓰레드를 디버깅하는게 어렵다.

<br />

<br />

## 콜백 Callbacks

콜백 함수를 실행 시키고자하는 다른 함수의 파라미터로 넘기고, 처리가 완료되면 넘겨준 콜백 함수를 실행시키는 방법이다.

```kotlin
fun postItem(item: Item) {
  preparePostAsync { token -> 
    submitPostAsync(token, item) { post ->
      processPost(post)
    }
  }
}

fun preparePostAsync(callback: (Token) -> Unit) {
  // 요청을 보내 토큰을 받고 바로 return
  // 나중에 콜백 처리
}
```

<br />

우아한 해결책처럼 보이지만, 콜백도 몇몇 이슈가 있다.

- 계속해서 안에 담겨지게 될 수 있는(nested) 콜백 처리가 어렵다. 가독성이 안좋다. 이를 콜백 헬 또는 `titled chrismas tree` 라 부른다. (모양이 트리처럼 생김)
- 에러 처리가 복잡하다.

콜백은 JavaScript 같은 이벤트루프 아키텍처에서 주로 사용되지만, 그런 구조에서조차 promise 나 reactive extensions 로 옮겨가고 있다.

<br />

<br />

## 퓨처, 프로미스 등 (Futures, Promises, and others)

퓨처와 프로미스는 같은 용어이다. (언어, 플랫폼에 따라 다르게 부른다.)

우리가 어떤 함수를 호출하면 언젠가 `Promise` 라는 객체를 받기를 약속받고, 우리는 그 객체를 사용할 수 있다.

```kotlin
fun postItem(item: Item) {
  preparePostAsync()
    .thenCompose { token -> 
      submitPostAsync(token, item)
    }
    .thenAccept { post -> 
      processPost(post)
    }
}

fun preparePostAsync(): Promise<Token> {
  // 요청을 보내 토큰을 받고 Promise 로 감싸서 리턴
  return promise
}
```

<br />

이런 방식은 우리가 프로그래밍하는 방식을 좀 다르게 한다.

- callback 을 사용하던 환경은 탑다운 형식의 절차적 접근이었다면, promise 는 체이닝 콜을 가진 구성적 모델(compositional model) 이다.
- `thenCompose`, `thenAccept` 등 새로운 API 를 배워야 한다. 이는 또 플랫폼마다 다르다.
- `Promise` 같은 새로운 반환 타입을 받고, 이를 우리가 원하는 방식으로 적용해야 한다.
- 에러 처리가 복잡하다.

<br />

<br />

## 리액티브 확장 (Reactive Extensions)

리액티브 확장(Rx, Reactive Extensions)는 C# 에서 도입됐는데, `.NET` 플랫폼에서 명확히 사용된 이후로, 넷플릭스가 java 에 도입하여 RxJava 를 만들고, 이후 RxJS (JavsScript) 등 다른 언어에도 구현체가 만들어졌다.

Rx 의 아이디어는 **데이터를 스트림 (무한한 양의 데이터) 이라 생각하고, 관측가능하다고 생각하여** 옵저버 패턴으로 이 데이터를 관리하는 것이다.

Rx 는 정해진 인터페이스하에 다른 언어에서도 일관성있는 API 를 제공하는 것이 큰 장점이다. 그리고 에러 처리에도 훌륭하게 수행한다.

<br />

<br />

## 코루틴 (Coroutines)

코틀린의 비동기 코드에 대한 접근 방식은 코루틴이다. 아이디어는 함수를 잠시 중단(suspend)해 놓았다가 나중에 다시 재개(resume)하는 것이다.

```kotlin
fun postItem(item: Item) {
  launch {
    val token = preparePost()
    val post = submitPost(token, item)
    processPost(post)
  }
}

suspend fun preparePost(): Token {
  // 요청을 보내 토큰을 받고, 코루틴을 suspend
  return suspendCoroutine { ... }
}
```

<br />

코루틴의 큰 장점은 블로킹 코드를 작성하는 것처럼 논블로킹 코드를 작성할 수 있다는 것이다.

- 함수 앞에 `suspend` 키워드가 붙는 것을 제외하면 거의 비슷하다. `launch` 블록 등 다른 처리도 있음.
- 새로운 API 를 배울 필요 없이, 프로그래밍 모델과 API 는 기존과 같다.
- 플랫폼과는 독립적이다. JVM 이든, JavaScript 또는 다른 플랫폼이든 코드는 같다. 
- 코루틴 개념은 새로운 개념은 아니고 이전에 `Go` 에서도 `goroutine` 으로 사용되었다. 코루틴의 장점은 `suspend` 를 제외하면 대부분 기능은 라이브러리에 위임돼있다. 따라서 적용하기 쉽다.



<br />

<br />

<br />

<br />

<br />

