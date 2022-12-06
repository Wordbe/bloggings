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
  p := Point{0, 1}
  fmt.Println(p.X) // 0
  pointer := &p
  pointer.X = 1000
  fmt.Println(p) // {1000, 1}
}
```

- `.` 으로 필드에 접근 가능하다. (get, set 가능)
- struct 를 가리키는 포인터로 필드에 접근할 수 있다. `(*pointer).X` 이런 식으로 접근하는 것은 번거로우니 `pointer.X` 를 지원한다.

<br />

# Arrays

```go
func main() {
  var a [2]string
  a[0] = "Hello"
  a[1] = "Go"
  
  primes := [5]int{2, 3, 5, 7, 11} // 배열 크기 5는 생략가능 (Slice literals)
}
```

<br />

## Slices

array 는 슬라이싱이 가능하다.

- `[1:4]` 는 반만 열린 범위를 의미한다. 즉 1을 포함시키고 4는 포함시키지 않는다.

```go
var s []int = primes[1:4] // 3, 5, 7
```

<br />

여러 쌍을 배열에 담는 쉬운 방법을 제공한다.

```go
func main() {
  primes := []struct {
    i int
    b bool
  }{
    {1, false},
    {2, true},
    {3, true},
    {4, false},
    {5, true}, // Go supports the trailing comma
  }
}
```

<br />

- `len()` 은 array 가 포함하는 요소의 갯수를 의미한다.
- `cap()` (capacity) 은 array 가 기본 배열(underlying array)의 요소 수 이다.

## Nil Slices

- 선언되고 할당되지 않은 배열은 빈 배열이 아니라 `nil` 이 된다.
- nil slice 의 length 와 capacity 는 0이 된다.

## Dynamically-sized Arrays

- 내장 함수 `make` 를 이용해서 동적 크기 배열을 만들 수 있다.
- 내장 함수 `append` 를 통해 요소를 더할 수 있다.

<br />

# Range

- for 과 함께 사용할 수 있다. slice 나 map 을 순회할 수 있다.
- 첫 번째 값은 인덱스 이고, 두 번째 값은 해당 인덱스의 요소를 복사한 값이다.

```go
func main() {
  var squares = []int{0, 1, 4, 9, 16, 25}
  
  for i, v := range squares {
    fmt.Printf("%d**%d = %d\n", i, i, v)
  }
}

0**0 = 0
1**1 = 1
2**2 = 4
3**3 = 9
4**4 = 16
5**5 = 25
```

- 두 번째 인자를 생략할 수 있다. 이 때는 인덱스만 사용할 수 있다.
- `_, v`, `i, _` 처럼 필요한 변수만 사용할 수 도 있다.

<br />
