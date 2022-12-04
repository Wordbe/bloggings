# Go 투어



# Packages, variables, and functions

## Exported Names

- 대문자로 시작하면 외부로 노출된다.
- exported 되지 않은 이름들은 외부 패키지에서 접근이 불가능하다.

<br />

## Functions continued

- 2개 이상 파라미터가 타입이 같다면, 타입을 마지막에 하나만 써주어도 된다.

```go
func add(x, y int) int {
  return x + y
}
```

<br />

## Multiple Results

- 함수는 반환값 여러 개를 반환할 수 있다.

```go
func swap(x, y int) (int, int) {
  return y, x
}

func main() {
  a, b := swap(1, 2) // 2, 1
}
```

<br />

## Short variable declarations

```go
func main() {
  var i, j int = 1, 2
  k := 3
  a, b, c := true, false, ""
}
```

<br />

## Basic types

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

- int, uint, uintptr 타입은 32bit 시스템에서는 32bit 로 사용되고, 64bit 시스템에서는 64bit 로 사용된다. 

<br />

# Flow control statements

<br />

- while 은 없고 같은 문법인데 for 로 사용한다.

```go
sum := 0
for sum < 10 {
  sum += 1
}
fmt.Println(sum)
```

<br />

## 연습문제, for문으로 제곱근 구해보기

- Newton's method 로 구해본다.
- 예를 들어 2의 제곱근을 구하려면 x^2 = 2 인 x 를 구하면 된다.
- 다시 생각하면 f(x) = x^2 - 2 라고 놓고, f(x) = 0 이 되는 x, 즉 f(x) 의 해를 구하면 된다.
- 그런데 이런 이차함수의 해는 뉴턴 메소드로 쉽게 구할 수 있다. (여러 번의 대입을 통해 근사치를 찾아가는 알고리즘이다.)
- 위키에서 examples > square root 를 검색해서 보면 이해가 쉽다. https://en.wikipedia.org/wiki/Newton%27s_method
- 아래와 같이 코드를 구현해본다.

```go
package main

import (
	"fmt"
)

func Sqrt(x float64) float64 {
	z := 1.0
	for i := 0; i < 10; i++ {
		z -= (z*z - x) / (2*z)
	}
	return z
}

func main() {
	fmt.Println(Sqrt(2))
}

// 1.41421356237309
```

<br />

## Switch

- go 의 switch 문은 case 조건에 들어맞으면, 해당 블럭의 로직을 실행하고 자동으로 break 가 된다. (C, java 와는 다르게)
- case 에는 int 형 뿐만아니라 string 등 다양한 타입이 비교될 수 있다.

<br />

## Defer

- `defer` 키워드가 붙은 statement 는 해당 절을 감싸는 함수 (surrounding function) 이 끝나기 전까지 지연된다. (실행되지 않는다.)
- 다만 함수의 인자(arguments) 는 바로 실행된다.(evaluate)

```go
func main() {
  defer fmt.Println(1 + 1)
  
  fmt.Println("1 + 1 = ")
}

1 + 1 = 
2
```

<br />

## Stacking defers

- `defer` 된 함수는 stack 에 담겨서, 여러 개의 deferred function 이 있을 경우 나중에 들어온 것이 먼저 실행된다. (LIFO)

```go
func main() {
  for i := 0; i < 3; i++ {
    defer fmt.Println(i)
  }
  
  fmt.Println("start")
}

start
2
1
0
```







