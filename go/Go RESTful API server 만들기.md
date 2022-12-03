# Go RESTful API server 만들기





```shell
mkdir jummechu-go
cd jummechu-go

# 의존성을 관리할 module 생성
go mod init jummechu
```

<br />

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
