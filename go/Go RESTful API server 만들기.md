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

