(本文来源于 https://github.com/golang/go/wiki/CommonMistakes/_history，鉴于一些不负责的 dev 在未经测试的情况下随意修改，copy 了一份自己维护)

Table of Contents
=================

+ [Introduction](#introduction)
+ [Using goroutines on loop iterator variables](#using-goroutines-on-loop-iterator-variables)

# Introduction

When new programmers start using Go or when old Go programmers start using a new concept, there are some common mistakes that many of them make.  Here is a non-exhaustive list of some frequent mistakes that show up on the mailing lists and in IRC.

# Using goroutines on loop iterator variables 

When iterating in Go, one might atempt to use goroutines to process data in parallel. For example, you might write something like this, using a closure:

```go
for _, val := range values {
	go func() {
		fmt.Println(val)
	}()
}
```

The above for loops might not do what you expect because their ` val ` variable is actually a single variable that takes on the value of each slice element. Because the closures are all only bound to that one variable, there is a very good chance that when you run this code you will see the last element printed for every iteration instead of each value in sequence, because the goroutines will probably not begin executing until after the loop.

The proper way to write that closure loop is:
```go
for _, val := range values {
	go func(val interface{}) {
		fmt.Println(val)
	}(val)
}
```

By adding val as a parameter to the closure, ` val ` is evaluated at each iteration and placed on the stack for the goroutine, so each slice element is available to the goroutine when it is eventually executed.

It is also important to note that variables declared within the body of a loop are not shared between iterations, and thus can be used separately in a closure.  The following code uses a common index variable ` i ` to create separate ` val `s, which results in the expected behavior:

```go
for i := range valslice {
	val := valslice[i]
	go func() {
		fmt.Println(val)
	}()
}
```

Note that without executing this closure as a goroutine, the code runs as expected.  The following example prints out the integers between 1 and 10.

```go
for i := 1; i <= 10; i++ {
	func() {
		fmt.Println(i)
	}()
}
```

Even though the closures all still close over the same variable (in this case, ` i `), they are executed before the variable changes, resulting in the desired behavior.
http://golang.org/doc/go_faq.html#closures_and_goroutines

Another similar situation that you may find like following:

```go
for _, val := range values {
	go val.MyMethod()
}

func (v *val) MyMethod() {
        fmt.Println(v)
}
```

The above example also will print last element of values, unfortunately the reason isn't the same as closure. 
 For each iteration, `val` has been evaluated as the pointer receiver of `MyMethod` after statement `go val.MyMethod()` executed. So When `val.MyMethod` is indeed invoked in goroutine, the `val` pointer point to current iterated value of `values` that you don't expect.

The right way is using value type receiver instead of pointer:

```go
for _, val := range values {
	go val.MyMethod()
}

func (v val) MyMethod() {
        fmt.Println(v)
}
```

Or just declare new variable within the body of the loop:

```go
for _, val := range values {
        newVal := val
	go newVal.MyMethod()
}

func (v *val) MyMethod() {
        fmt.Println(v)
}
```
