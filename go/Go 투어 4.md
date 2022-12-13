# Go 투어 4

# Generics

## Type parameters

- 타입 파라미터는 함수명과 인자 사이에 적으면 된다. 이렇게 제네릭 함수를 만들 수 있다.
- 이 때 T 는 `comparable` 속성을 사용할 수 있다.
  - `==`, `!=` 연산자를 사용해서 해당 타입의 인스턴스들을 비교할 수 있다.

```go
func Index[T comparable](s []T, x T) int {
  for i, v := range s {
    if v == x {
      return i
    }
  }
  return -1
}

func main() {
  si := []int{10, 20, 15, -1}
  fmt.Println(Index(si, 15)) // 2
  
  ss := []string{"a", "b", "c"}
  fmt.Println(Index(ss, "hello")) // -1
}
```

<br />

## Generic types

- 아래처럼 제네릭 타입도 사용할 수 있다.

```go
type List[T any] struct {
  next *List[T]
  val T
}
```

<br />