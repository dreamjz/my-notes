# Hello World

Our first program will print the classic “Hello World” message.

Here is the source code.

```go
package main 

import "fmt"

func main() {
    fmt.Println("Hellp World")
}
```

To run the program,put the code in hello_world.go and use go run.

```sh
$go run hello_world.go
Hellp World
```

Sometimes we will want to build our programs into binaries. We can do this use go build.

We can then execute the built binary directly

```sh
± go build ./main.go 
± ls
go.mod  main  main.go
± ./main 
Hello World
```

