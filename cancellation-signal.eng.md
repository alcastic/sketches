# Cancellation Signal 

Golang's goroutines can not force the termination of anothers goroutines, since that would leave all its shared variables in undefined/inconsistent states.

Next main goroutine example creates a `chan int` and spawns a goroutine in order to stream integers into the channel. Notice how the parent main goroutine has no mechanism to instruct the child goroutine to stop streaming data into the channel.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	stream := make(chan int)

	go func() {
		for i := 0; ; i++ {
			stream <- i
			time.Sleep(100 * time.Millisecond)
		}
	}()

	for v := range stream {
		fmt.Println(v)
	}
}
```

How can we make the previous code ends up what is doing? Let's first see a bad practice to achieve it:

```go
package main

import (
	"fmt"
	"os"
	"time"
)

func main() {
	go func() {
		time.Sleep(5 * time.Second)
		os.Exit(0)
	}()

	stream := make(chan int)

	go func() {
		for i := 0; ; i++ {
			stream <- i
			time.Sleep(100 * time.Millisecond)
		}
	}()

	for v := range stream {
		fmt.Println(v)
	}
}
```

The previous code spawns a new goroutine which sleep for 5 seconds before exiting the entire application. It is a poor bad practice since in can lead to inconsistencies and our `stream` channel has not been closed. Could you imagine the consequences of doing so in a scenario with thousands of gorutines?

A good practice is to allow the *parent goroutine* to **send a cancellation signal** to its *child goroutine* in order to instruct them to stop what it is doing and avoiding memory leaks. The signal will be the closing of *done channel* (tipically of type `chan struct{}`, but the channel's type is nor relevant). Notice how by closing the `done channel` the other goroutine is instruct to end its work.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	done := make(chan struct{})
	go func() {
		defer close(done)
		time.Sleep(5 * time.Second)
	}()

	stream := make(chan int)

	go func() {
		defer close(stream)
		for i := 0; ; i++ {
			select {
			case <-done:
				return
			case stream <- i:
				time.Sleep(100 * time.Millisecond)
			}
		}
	}()

	for v := range stream {
		fmt.Println(v)
	}
}
```

The Golang's context package provides a handy implementation to implement cancellation signals. The context package is out of the scope of this documentation, but here is the implementation of our code using the context package

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	stream := make(chan int)

	go func() {
		defer close(stream)
		for i := 0; ; i++ {
			select {
			case <-ctx.Done():
				return
			case stream <- i:
				time.Sleep(100 * time.Millisecond)
			}
		}
	}()

	for v := range stream {
		fmt.Println(v)
	}
}
```

Finally, a refactor of our code:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func intStream(done <-chan struct{}) <-chan int {
	stream := make(chan int)
	go func() {
		defer close(stream)
		for i := 0; ; i++ {
			select {
			case <-done:
				return
			case stream <- i:
				time.Sleep(100 * time.Millisecond)
			}
		}
	}()
	return stream
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	for v := range intStream(ctx.Done()) {
		fmt.Println(v)
	}
}
```
