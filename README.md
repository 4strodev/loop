# Loop

Loop is a package that provides commonly used functions
for ranging.

⚠️ This package currently relies on the experimental
[range-over-function iterators](https://tip.golang.org/wiki/RangefuncExperiment) 
feature. You can use this package without it, however it will likely not be as
an enjoyable experience.

## Requirements

This package requires Go 1.22 with the `GOEXPERIMENT=rangefunc` env
var enabled.

## Usage

The package provides a number of different functions for ranging.

### Parallel

A commonly used pattern in Go is to iterate over a slice of elements in parallel with a 
wait group.

The parallel iterator provides this functionality in an easy to use interface

```go
package main

import "github.com/dreamsofcode-io/loop"

func main() {
    xs := []int{1,2,3,4,5}
    squares := make([]int{}, len(xs))

    // Each iteration runs in a goroutine
    for i, x := range loop.Parallel(xs) {
        // Simulate a long running task
        time.Sleep(time.Second)
        squares[i] = x * x
    }

    fmt.Println(squares) // [2, 4, 9, 16, 25]
}
```

The above task will run in parallel, which means the total operation will only take 1 second, 
instead of the 5 it would take otherwise. 

⚠️ One thing to be aware of is that teach iteration runs in a separate goroutine. Therefore
you'll want to make sure you are performing thread safe operations.

The parallel task won't speed up any compute heavy operations, in that case, you're better
off using a normal loop. However, in the event of performing network requests or async
tasks, then using loop.Parallel will improve performance.

```go
import (
    "slog"
    "net/http"

    "github.com/dreamsofcode-io/loop"
)

func main() {
    colors := []string{"green", "yellow", "blue"}

    results := make([]*http.Response{}, len(colors))
    for _, color := range loop.Parallel(colors) {
        _, err := http.Post("http://example.com/colors", "text/plain", strings.NewReader(color))
        if err != nil {
            slog.Error("oops", slog.Any(err))
        }
    }
}
```

### Pool
The pool function is very similar to `loop.Parallel`, however it allows to caller to set the
concurrency amount with the second argument.

This is useful in the event you want bounded concurrency.

```go
package main

import "github.com/dreamsofcode-io/loop"

func main() {
    xs := []int{1,2,3,4,5}

    // Each iteration runs in a goroutine
    for _, x := range loop.Pool(xs, 2) {
        // Simulate a long running task
        time.Sleep(time.Second)
    }
}
```

In the above example, only 2 elements will be performed at a time.

### Batch

The Batch function provides the ability to range over elements in batches. The size of each batch
is decided by the given size argument, in which a batch will either be the same size or less than.

The `loop.Batch` method runs in a single goroutine

```go
import "github.com/dreamsofcode-io/loop"

func main() {
    nums := []int{1, 2, 3, 4, 5}

    for i, batch := range loop.Batch(nums, 2) {
        fmt.Println(i, batch)
    }
}
```

The above code will print the following output:

```golang
0 [1, 2]
1 [3, 4]
2 [5]
```

If a batch size of 0 is passed in, then no iterations of the loop are performed. This behavior
may change instead to panic as it's effectively a divide by 0.

