 # Import Cycles in Golang: How To Deal With Them

As a Golang developer, you probably have encountered import cycles. Golang do not allow import cycles. Go throws a compile-time error if it detects the import cycle in code. In this post, let's understand how the import cycle occurs and how you can deal with them.

## Import cycles

Let's say we have two packages, `p1` and `p2`. When package `p1` depends on package `p2` and package `p2` depends on package `p1`, it creates a cycle of dependency. Or it can be more complicated than this e.g. package `p2` does not directly depend on package `p1` but `p2` depends on package `p3` which depends on `p1`, again it is cycle.

![](./image/import_cycle.png)

Let's understand it through some example code.

*package 1*:

```go
package p1

import (
	"fmt"
	"import-cycle-example/p2"
)

type PP1 struct {}

func New() *PP1 {
	return &PP1{}
}

func (p *PP1) HelloFromP1(){
	fmt.Println("Hello from p1")
}

func (p *PP1) HelloFromP2Side(){
	pp2 := p2.New()
	pp2.HelloFromP2()
}
```

*package 2*:

```go
package p2

import (
	"fmt"
	"import-cycle-example/p1"
)

type PP2 struct{}

func New() *PP2 {
	return &PP2{}
}

func (p *PP2) HelloFromP2() {
	fmt.Println("Hello from p2")
}

func (p *PP2) HelloFromP1Side() {
	pp1 := p1.New()
	pp1.HelloFromP1()
}
```

On building, compiler returns the error:

```
package import-cycle-example/p1
        imports import-cycle-example/p2
        imports import-cycle-example/p1: import cycle not allowed
```

## Import Cycles Are Bad Design

Go is highly focused on faster compile time rather than speed of execution (even willing to sacrifice some run-time performance for faster build). The Go compiler doesn't spend a lot of time trying to generate the most efficient possible machine code, it cares more about compiling lots of code quickly.

Allowing cyclic/circular dependencies would **significantly increase compile times** since the entire circle of dependencies would need to get recompiled every           time one of the dependencies changed. It also increases the link-time cost and makes it hard to test/reuse things independently(more difficult to unit test because they cannot be tested in isolation from one another). Cyclic dependencies can result in infinite recursions sometimes.

Cyclic dependencies may also cause memory leaks since each object holds on to the other, their reference counts will never reach zero and hence will never become candidates for collection and cleanup.

[Robe Pike, replying to proposal for allowing import cycles in Golang](https://github.com/golang/go/issues/30247#issuecomment-463940936), said that, this is one area where up-font simplicity is worthwhile. Import cycles can be convenient but their cost can be catastrophic. They should continue to be disallowed.

## Debugging Import Cycles 

 The worst thing about the import cycle error is, Golang doesn’t tell you source file or part of the code which is causing the error. So it becomes tough to figure out when the codebase is large. You would be wondering around different files/packages to check where actually the issue is. Why golang do not  show the cause that causing the error? Because there is not a singe culprit source file in the cycle.

But it does show the packages which are causing the issue. So you can look into those packages and fix the problem.

To visualize the dependencies in your project, you can use [godepgraph](https://github.com/kisielk/godepgraph), a Go dependency graph visualization tool. It can be installed by running:

```
go get github.com/kisielk/godepgraph
```



