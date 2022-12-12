# Go 투어 3

# Methods and Interfaces

- go 는 클래스가 없다.
- 대신에 type 에 메소드를 정의할 수 있다.
- 메소드는 특별한 수신자(receiver) 인자를 받는 함수이다.

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Abs(v.X*v.X + v.Y + v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

<br />

- method 는 non-struct type 에도 사용할 수 있다.

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
```

<br />

## Pointer Receiver

- 수신자를 포인터로 받는 method 도 만들 수 있다.
- 아래 `Scale` 메소드는 포인터를 받아왔기 때문에, 변경된 값을 리턴하지 않고도, 참조한 v 를 바꿀 수 있다.
- 16번째 줄에서 의아한 것은, `v` 가 포인터 수신자가 아닌데 `v.Scale(2)` 함수가 잘 작동되는 것이다. 이는 go 가 사용의 편의성을 위해 자동으로 `(&v).Scale(2)` 로 해석하여 컴파일하기 때문에 가능한 것이다.

```go
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(2)
	ScaleFunc(&v, 10)

	p := &Vertex{4, 3}
	p.Scale(2)
	ScaleFunc(p, 10)

	fmt.Println(v, p)
}
```

- 역으로 포인터 수신자를 받아 만든 메소드만 있는 상황에서, 일반 수신자를 통해 메소드를 호출하면, 포인터 수신자를 받아 만든 메소드가 실행된다.
- 포인터 p 가 있을 때 `p.Abs()` 대신에 `(*p).Abs()` 로 해석하여 컴파일된다.



포인터 수신자가 존재하는 이유는 1) 메소드가 수신가 가리키는 값을 변경하기 위함과 2) 갑 복사를 함수 호출에서 피하기 위해서이다. 값 복사는 수신자가 큰 구조체라면 효율적이지 못하다.

일반적으로, 동일한 타입의 메소드는 모두 값이거나 모두 포인터 수신를 사용한 메소드이어야 한다.

<br />

## Interface

go 의 인터페이스 타입은 메소드 시그니처의 집합이다.

- 인터페이스를 구현하려면 인터페이스를 먼저 선언한 뒤, 이 인터페이스 변수에 다른 구현체를 할당해주면 된다.

<br />

**MyFloat.go**

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
```

**Vertex.go**

```go
type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

**Abser.go**

```go
type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
  
	f := MyFloat(-math.Sqrt2)
	a = f  // a MyFloat implements Abser
	fmt.Println(a.Abs()) // 1.4142135623730951
	
  v := Vertex{3, 4}
	a = &v // a *Vertex implements Abser
	fmt.Println(a.Abs()) // 5
}
```

<br />

## Interface Values

- 인터페이스 값은 `(value, type)` 으로 된 쌍이 튜플이다.
- 인터페이스는 등록된 구현체의 값과, 구현체 타입을 가지고 있는다.

```go
a = f
fmt.Printf("(%v %T)\n", a, a)

a = &v
fmt.Printf("(%v %T)\n", a, a)

(-1.4142135623730951 main.MyFloat)
(&{3 4} *main.Vertex)
```

<br />

## Type Assertion, Type switch

문법은 `i.(T)` 와 같다.

인터페이스에서 특정 타입이 구현되어 있고, 등록되어 있는지 확인한다. 또한 인터페이스에 존재하는 타입이라면, case 문에서는 자동으로 value 를 해당 타입으로 전환시켜 준다.

```go
// 인터페이스 i 가 T 타입을 가지면 t 에 T 타입의 값을 할당한다.
t := i.(T)

// 인터페이스 i 가 T 타입을 가지면 t 에 T 타입의 값을 할당하고, ok=true
// 가지지 않으면, t 는 T 타입의 zero value, ok=false
t, ok := i.(T)
```

```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

<br />

## Exercise: Stringers

- `IPAddr` 타입은 `[4]byte` 타입인데, 여기 시그니처가 String() string 인 메소드를 구현하면 된다.

`fmt` 패키지에는 아래 인터페이스가 있고, `fmt` 에서 프린트할 때 이 인터페이스를 사용하기 때문에, 이를 구현해주면 원하는대로 출력할 수 있다. 마치 `ToString()` 같은 메소드

```go
type Stringer interface {
    String() string
}
```

<br />

```go
package main

import (
	"fmt"
	"strconv"
	"strings"
)

type IPAddr [4]byte

func (ip IPAddr) String() string {
	s := make([]string, len(ip))
	for i, v := range ip {
		s[i] = strconv.Itoa(int(v))
	}
	return strings.Join(s, ".")
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}

loopback: 127.0.0.1
googleDNS: 8.8.8.8
```

<br />

## Errors

go 에서는 error 라는 내장된 인터페이스가 있다. `Stringer` 처럼 fmt 가 출력시 참고하는 인터페이스이다.

```go
type error interface {
    Error() string
}
```



error 를 반환하는 함수가 있는데, error 가 `nil` 인지 검사후 에러처리를 해주면 된다.

```go
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

<br />

예제)

```go
type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s", e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

<br />

<br />

## Readers

- `io` 패키지는 `io.Reader` 인터페이스를 제공한다. 

```go
func (T) Read(b []byte) (n int, err error)
```

- byte 배열(문자열) 을 받아서 몇 개의 문자를 받아왔는지(populate)와, 에러가 있다면 err 를 반환한다.

<br />

## Images

```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```

<br />

<br />

<br />

<br />
