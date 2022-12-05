# Go 투어 2



# Pointer

- 포인터는 값에 대한 메모리 주소를 가진다.
- 아무 것도 할당되지 않으면 기본값(zero value)은 `nil` 이다.

```go
var p *int
```

`&` 연산자는 특정 변수에 대한 포인터를 만들어낸다.

```go
i := 10
p = &i
```

`*` 연산자는 포인터가 가리키는 값을 나타낸다. 이를 deferencing (역참조) 또는 indirecting (간접 참조)라 한다.

```go
*p // 10 : 포인터 p 를 통해 i 를 읽음
*p = 20 // 포인터 p 를 통해 i 를 세팅함
```

<br />

<br />

# Struct

struct 는 필드의 집합이다.

```go
type Point struct {
  X int
  Y int
}

func main() {
  fmt.Println(Point{0, 1})
}
```

`.` 으로 필드에 접근 가능하다. (get, set 가능)

