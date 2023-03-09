# Pipelines

A pipeline is a chain of data processing units(a.k.a.*stages*), arranged so the output of each unit is the ouput of the next one.

Pipeline allows to split complex data processing flows in smaller, testable and maintainable logical unit, promoting:

* Separation of Cencenr.
* Stages and functions reusability.
* Single-responsibility principle.

For example, the function f(n) = (n*2)+1 can be reprecented by a pipeline which chain two stages, as follow:

![Pipeline f(n)=(n+1)*2](./img/go-pipelines/pipeline-01.png)

The *stage-1* takes the integer `5` in and calculates *n\*2*, then pass out the result `10` to the next *stage-2* which calculates *n+1* and pass out the final result `11`. Now, a simple implementation written in Golang.


```go
package main

import (
	"fmt"
	"time"
)

func multiply(x, y int) int {
	return x * y
}
func add(x, y int) int {
	return x + y
}
func main() {
	stage1 := func(n int) int {
		return multiply(n, 2)
	}
	stage2 := func(n int) int {
		return add(n, 1)
	}
	pipeline := func(n int) int {
		return stage2(stage1(n))
	}
	inputs := []int{1, 2, 3, 4, 5}
	for _, v := range inputs {
		fmt.Println(pipeline(v))
	}
}
```

Previous code may seems innocent, but in lines 45-47 a new instance of our pipeline is being initialized for each input element. Imagine a scenery of massive amount of inputs (MBs or GBs), it would imply a massive recreation of our pipeline with a massive footprint of memory management in the HEAP.

We can avoid the recreation of our pipeline by implementing batch processing as follows:

```go
package main

import (
	"fmt"
	"time"
)

func multiply(x []int, y int) []int {
	for i, _ := range x {
		x[i] *= y
	}
	return x[:]
}
func add(x []int, y int) []int {
	for i, _ := range x {
		x[i] += y
	}
	return x[:]
}
func main() {
	stage1 := func(n []int) []int {
		return multiply(n, 2)
	}
	stage2 := func(n []int) []int {
		return add(n, 1)
	}
	pipeline := func(n []int) []int {
		return stage2(stage1(n))
	}
	inputs := []int{1, 2, 3, 4, 5}
	for _, v := range pipeline(inputs) {
		fmt.Println(v)
	}
}
```

Previous code just initializes our pipeline once. However, with bash processing we have introduced a new drawback, now all the elements have to be processed entirely for each stage before obtaining the final results.
