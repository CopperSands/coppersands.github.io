# What is Go?
Golang or Go is a language first created by Google. It is a memory-safe language, compiled, and advertised as easy to learn. It is also a curly braced language that shares similarities with C and C++. Additionally, statements do not need to be semicolon terminated. If you are familiar with other curly braced languages and using packages, Go code should make sense for the most part.


The official documentation for Go is actually really good. I have stepped through the get started tutorial and I would highly recommend it. You can find a link to the official get started guide [here](https://go.dev/doc/tutorial/getting-started).  This guide does expect that you have decent exposure to other programming languages. Understanding object oriented programming, and if statements will help you understand the tutorials. 

#  Choosing a Development Platform

Go can be installed on Linux, Mac, and Windows. Since Go is a compiled language it will compile an executable or binary for the specific operating system (OS) and CPU architecture you are developing on. There are ways to build for another OS/architecture but that topic will not be covered here. 

If you like developing in Linux I recommend that you use the `golang` Docker image. It is updated frequently and it makes it easy to spin up a testing environment. You can pull the latest version  of the image with `docker pull golang`, assuming your Docker Desktop or Docker daemon is running.

To create a container with a shell you can run `docker run -it --name golangtest golang bash`. You can check out the documentation on the image [here](https://hub.docker.com/_/golang).

To start the container again you can run `docker start golangtest` followed by `docker exec -it `

# Go Structure

Go uses packages and every function, struct, variable must be in a package. Multiple files can be part of the same package. This means that you can separate functions `a` and `b` between files, but they will still be able to call each other if they are in the same package. 

Before you start creating packages you should create a module to track dependencies in your project. The module name can also provide a way to export your package for others to use, see this [link](https://go.dev/doc/modules/developing). If you are unfamiliar with how to create a module and start coding I highly recommend using the [getting-started guide](https://go.dev/doc/tutorial/getting-started). You can initiate a go module with `go mod ini github.com/reponame/pathto/modulename`. This is assuming you are using a GitHub repo. You can also use the module path `example/modulename` to play around with Go. 

To start your first project you can use this example code from step 5 of the official [getting-started guide](https://go.dev/doc/tutorial/getting-started). Go files end in `.go` for example, `example.go`. The `main` package will be the main package of your Go program. The `main()` function will be called by default when you run the program. `fmt` is a package that is part of the default Go library and controls I/O.
```Go
package main

import "fmt"

func main() {
	fmt.Println("Hello World!")
}
```

To run your go program use `go run .` or you can replace `.` with the name of the go file. `go run` will compile and run the program. `go run` will not create an executable. To create an executable you will need to use `go build`.

# Declaring a Variable

If you are familiar with declaring variables in other languages, declaring variables in Go will seem backwards for you. 

To declare a variable you would write `var` the name of the variable and the the type. In the example below we have declared a new variable `example` to be a string.

```Go
var example string
```

Assigning the variable `example` a value is easy and typical of other languages. For example `example = "this is a string"` will work. But there is a short-hand way of declaring a variable and assigning it a value in Go `:=`. The following code block declares a new variable and assigns the value. The `:=` takes the type of the variable on the right and sets it as the type for the variable on the left. 

```Go
example := "this is a string"
```

## Variable Data Types

There are a few basic data types in Go. These are:
- `bool`
- `string`
- `int` 
- `byte`
- `rune`
- `float32` `float64`
`bool` stands for boolean and contains a value `true` or `false`. `string` is a string. It is important to note that the value of a string must be in double quotes. 

There are various forms of `int`: `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`. According to Go [documentation](https://go.dev/tour/basics/11), the default bit size of `int`, `uint`, and `uintptr` is based on the CPU bit size (32 or 64 ). It is also recommended to just use `int` unless there is a specific need for another bit size. `byte` is an alias for `uint8`. `rune` is an alias for `int32` and can be used to represent Unicode.

`float32` and `float64` are the two float representations. The first represents a float with 32 bits while the latter represents a float with 64 bits. You can read more about it [here](https://www.w3schools.com/go/go_float_data_type.php).

There are also 2 other data types `complex64` and `complex128`. 
These two types are meant to handle imaginary numbers. You can learn more about them [here](https://how.dev/answers/what-is-type-complextype-in-golang).

# Declare a Function

Function declaration in Go has similarities to other languages but is a bit different.

First to declare a function you must use the key word `func` followed by the name of the function, then the parameters in a set of parenthesis, and finally the return type or types.  The following are examples.

```Go
func foo( var1 int, var2 string) bool {
	return true
}

func foo2 ( var1 string) {
// A return type does not have to be declared
// If the return type is not declared it will not return
// there can be a space between the method name and params
}

func foo3 (var1 int) (int, bool){
// you can return more than one value
	myBool := true
	return var1, myBool
}

```

There are a lot of other things you can do with functions. I found this [site](https://www.w3schools.com/go/go_function_returns.php) helpful to understand the different ways to write functions.

## Named Return Values and Naked Returns
One of the other cool ways to declare a function in Go is using named return values in the function header. This negates the need to declare the variable in the function later. This also means that you can perform a naked return. Naked returns involve simply using the `return` key word to end the function. Since the return variable is know, it will be returned when the function ends. It is not recommended to use naked returns for long functions because of [readability](https://go.dev/tour/basics/7). Below are a few examples.

```Go
func foo4()(value int){
// named return value with naked return
	value = 5
	return
}

func foo5()(value2 int){
// example of a named return value
	value2 = 6
	return value2
}
```

## Export Names
Export names are a type of function in Go that can be called outside of the package in which it is declared. Export names resemble regular functions but the first letter of the function is capitalized. You can learn more about export names [here](https://go.dev/tour/basics/3). A example of an export name is below.

```Go
func Foo6(var1 int) (int){
// This is an export name. Export names are the 
// functions that are availble when a package is imported
	return 1
}
```

I hope this post has been a good starting point in your new journey with Go. There will be more posts in the future on this subject.
