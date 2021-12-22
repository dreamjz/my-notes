# Go : Three dots(ellipsis) notaion

The three dots `...` notation is used in four different places in GO :

* Variadic function parameters 
* Arguments to variadic functions
* Array literals
* The go command

## Variadic function parameters

If the last parameter of a function has type `...T`,it can be called with any number of trailling arguments of type T

The actual type of `...T` inside the function is `[]T`

Example: This function can be called with for instance `Sum(1,2,3)` or `Sum()`

```go
func Sum(args ...int) int {
	s := 0
	for _, v := range args {
		s += v
	}
	return s
}
//Sum(1,2,3) return 6
//Sum() 	 return 0
```

## Arguments to variadic functions

You can pass a slice `s` directly to a variadic function by "unpacking" it using `s...` notation. In this case no new slice is created.

Example: Passing a slice to the `Sum` function 

```go
primes:=[]int{2,3,5,7}
fmt.Println(Sum(primes...))//17
```

## Array literals

In an array literal, the `...` notation specifies a length equal to the number of elements in the literal

Example: Array literal with length inferred

 ```go
 numbers:=[...]{1,2,3,4}
 fmt.Prinlf("%T %v")//[4]int [1,2,3,4]
 ```

## The Go command

Three dots are used by the `go` command as a wildcard when describing package lists

Example: Tests all packages in the current directory and its subdirectories:

```sh
$ go test ./...
```

