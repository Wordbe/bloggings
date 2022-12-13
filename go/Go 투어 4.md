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

<br />

# 동시성(Concurrency)

## Goroutine

- goroutine 은 go 런타임에 관리되는 경량 쓰레드(lightweight thread)이다.
- `go` 키워드로 새로운 고루틴을 시작할 수 있다.

## Channel

- 채널은 값을 전송하고 받을 수 있는 `typed conduit` (타입화된 도관)이다.

- make 함수로 채널을 만들 수 있다.

  ```go
  ch := make(chan int) // int 형 채널
  
  ch <- 1
  ch <- 2
  fmt.Println(<-ch) // 1
  fmt.Println(<-ch) // 2
  ```

- 채널은 버퍼 크기를 가질 수 있다. 선언할 때 두 번째 인자로 버퍼 길이를 전달한다.

  ```go
  ch := make(chan int, 10)
  ```

- 채널은 보내는쪽 (sender) 에서 닫을 수 있다. `close` 사용

- 또한 받는쪽 (receiver) 에서는 `for i := range c` 처럼 자연스럽게 range 로 꺼내어 사용할 수 있다. 채널이 closed 될 때까지 반복하여 안에 있는 값을 꺼내게 된다.

  ```go
  func fibonacci(n int, c chan int) {
  	x, y := 0, 1
  	for i := 0; i < n; i++ {
  		c <- x
  		x, y = y, x+y
  	}
  	close(c)
  }
  
  func main() {
  	c := make(chan int, 10)
  	go fibonacci(cap(c), c)
  	for i := range c {
  		fmt.Println(i)
  	}
  }
  ```

<br />

<br />

## Select

- `select` 문으로 여러 커뮤니케이션 연산에서 고루틴을 기다리게 할 수 있다.
- `select` 는 `case` 중 실행가능한 상태가 있을 때까지 block 하고, 그 case 를 실행한다. `case`가  여러 개가 가능하다면 랜덤으로 선택한다.

```go
func fib(c, quit chan int) {
  x, y := 0, 1
  for {
    select {
      case c <- x:
      	x, y = y, x+y
      case <-quit:
	      fmt.Println("quit")
      	return
    }
  }
}

func main() {
  c := make(chan int)
  quit := make(chan int)
  go func() {
    for i := 0; i < 10; i++ {
      fmt.Println(<-c)
    }
    quit <- 0
  }()
  fib(c, quit)
}

0
1
1
2
3
5
8
13
21
34
quit
```

<br />

<br />

## Exercise - Equivalent Binary Tree

```go
// Walk walks the tree t sending all values from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	var walk func(t *tree.Tree)
	walk = func(t *tree.Tree) {
		if t == nil {
			return
		}
		walk(t.Left)
		ch <- t.Value
		walk(t.Right)
	}
	walk(t)
	close(ch)
}

// Same determines whether the trees t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
	ch1, ch2 := make(chan int), make(chan int)

	go Walk(t1, ch1)
	go Walk(t2, ch2)

	for {
		v1, ok1 := <-ch1
		v2, ok2 := <-ch2
		if v1 != v2 || ok1 != ok2 {
			return false
		}

		if !ok1 {
			break
		}
	}
	return true
}

func main() {
	fmt.Println(Same(tree.New(1), tree.New(1)))
	fmt.Println(Same(tree.New(1), tree.New(2)))
}
```

<br />

<br />

## sync.Mutex

- `sync.Mutex` 는 하나의 고루틴만 한 시점에 하나의 변수에 접근하도록 보장하는 방법을 제공한다.
- Mutex(Mutual Exclusion) 알고리즘을 제공해주어 `Lock`, `Unlock` 이 가능하다.

```go
import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 3; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}

```

<br />

## Exercise: Web Crawler

- 앞에서 배웠던 `SafeCounter` 와 `sync.Mutex` 를 응용해서 만들면 쉽다.

```go
package main

import (
	"fmt"
	"sync"
)

type SafeCache struct {
	mu sync.Mutex
	v  map[string]bool
}

func (s SafeCache) visited(url string) bool {
	s.mu.Lock()
	defer s.mu.Unlock()
	_, ok := s.v[url]
	if !ok {
		s.v[url] = true
		return false
	}
	return true
}

var sc SafeCache = SafeCache{v: make(map[string]bool)}

type Fetcher interface {
	// Fetch returns the body of URL and a slice of URLs found on that page.
	Fetch(url string) (body string, urls []string, err error)
}

// Crawl uses fetcher to recursively crawl pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher) {
	// This implementation doesn't do either:
	if depth <= 0 {
		return
	}

	if sc.visited(url) {
		return
	}

	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)
	for _, u := range urls {
		Crawl(u, depth-1, fetcher)
	}
	return
}

func main() {
	Crawl("https://golang.org/", 4, fetcher)
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}

```



<br />

<br />

<br />

<br />

<br />