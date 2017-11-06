# Errors & Logging

[Errors](https://blog.golang.org/error-handling-and-go) and [logging](https://golang.org/pkg/log/) are one area of Go which does perhaps does deserve the label of simplistic rather than simple. The log package has no levels or interfaces, it simply prints to standard error by default. For many applications though, this is enough, and there are various logging packages available for more sophisticated requirements. The error type is very simple and errors are stored as strings.

## The Error Type

> Errors are values
> – Rob Pike

The [error](https://blog.golang.org/error-handling-and-go) type in go is a very simple interface, with one method. Errors offer no introspection into what went wrong or storage of other data. Often errors are nested, as one error may be annotated several times as it passes up the stack. Try to prefer handling an error as close to the site of the error as possible.

```go
type error interface {
  Error() string
}
```

You can use your own type for error, as long as it conforms to this interface, in some cases you might want to use a more complex type than the string only errors favoured by the standard library, to record an http status code for example.

If you're defining interfaces, prefer requiring the error interface rather than a concrete type. You can use type assertions to determine if an error is of the type you're interested in.

## to, err := human\(\)
> to, err := human\(\) – Francesc Campoy

The convention in Go code is always to return an error as the last argument of a function, so if it has multiple arguments. Return an error as the last argument, and try to make it as specific as possible:

```go
func DoSomething() error {
    to, err := human()
    if err != nil [
      // Annotate the error and return
      return fmt.Errorf("pkgname: failed to human %s",err) 
    }
    // no, error, use value to
    ...
    return nil // return nil error
}
```

```go
func DoSomethingElse() (value, value, error) {
  ...
  if err != nil {
  // Annotate the error and return nil value + error
    return nil, nil, fmt.Errorf("pkgname: failed to do something %s",err) 
  }
  ...  
  // Return completed values and nil error
  return v, vv, nil
}
```

The caller should always check for errors before using values. _You should favour returning an error over returning nothing or simply a value_, even if you don't think initially you might encounter many errors, unless your function is just a few lines with no external calls. Functions rarely become simpler over time and it's always better to explicitly report errors rather than silently return zero data.

## _Always_ handle Errors

You should never discard errors using \_ variables in production code. If a function returns an error, make sure you check it before using any values returned by that function. Usually if there is an error returned, values are undefined.

## _Either_ log _or_ return an error

If you're logging an error, you've decided it's not important enough to handle. If you're returning an error \(usually with annotation\), you want the caller to handle it \(which might include logging or reporting it to the user\). Handle the error by logging / reporting, or return it, never both.

**Don't both log and return errors:**

```go
if err != nil {
    // Don't do this, either deal with or return an error
    log.Printf("pkgname: failed to log stats:%s",err)
    return err
}
```

If it's not important, log and move on, if it stops processing in this function, return the error and let the caller decide how to handle it \(usually to report to user and/or log\):

```go
func DoSomething() error {
   // For an error you can handle in this function without telling the user:
   if err != nil {
     // Not important, just log the error and continue with processing
     log.Printf("pkgname: failed to log stats %s",err) 
   }
    ...
    // For an important error return it (usually annotated) and let caller decide how to report
    if err != nil  {
       return fmt.Errorf("pkgname: failed to do something %s",err) 
    }
}
```

Dave Cheney has written a talk about handling errors in go – [Don't just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully), which covers how to handle errors gracefully and avoid falling into the pitfalls outlined above. 

## Stack traces on errors

If you want to get a stack trace at the point of error which is not a panic, you can use the runtime package to determine the caller. You can also use the unofficial [go-errors](https://github.com/go-errors/errors) package, adds stacktrace support to errors, to record the stack trace and understand the state of execution when an error occurred. 

Finally, if you don't mind dumping a stack trace to stderr and terminating the program, you can panic. This is useful for debugging but less so in production programs.

## Interpreting a panic stack trace 

If you don't have much experience with stack traces, the output of a panic may seem inscrutable at first. To generate a stack trace, consider this simple program:

```go
package main

type T struct {
	s string
}

func foo(t *T) *T {
	defer panic("panic in foo")
	return t
}

func main() {
	foo(&T{})
}
```

Which produces the following panic:

```shell
panic: panic in foo

goroutine 1 [running]:
main.foo(0x1042bfa4, 0x0, 0x10410008, 0x0)
	/tmp/sandbox611247315/main.go:13 +0xdb
main.main()
	/tmp/sandbox611247315/main.go:18 +0x40
```

The first line gives the message passed to panic, which should be as informative as possible. 

The lines after detail the crashing go routine (by default this is restricted to just the crashing one since Go 1.6). 
First we encounter the rather inscrutable message:

> main.foo(0x1042bfa4, 0x0, 0x10410008, 0x0)

This details the function with the panic, and may provide some clues if you have a nil pointer exception as to which pointer is nil - it includes the arguments to the function (including the method struct if any, which is always the first argument), and the return values. In this case, because the program uses defer to panic, the return values are filled in before panic, hence the argument and return pointers are the same value of 0x10410008. Without the defer the second two words would both be nil. 

Then the more important line which tells us which file and line the problem occurred at:

> /tmp/sandbox611247315/main.go:13 +0xdb

If nothing else, this is the line you need to read, as it clearly states exactly where the problem occurred. Where you have deliberately panicked using panic() this will normally be obvious anyway, but where you have a nil pointer or index out of range it can be useful for tracking down the bug. 

Remember that methods can be called on nil pointers, which can lead to subtle bugs if the callee never expects this and then tries to use the pointer as if it is initialised. 

If you're debugging complex stack traces featuring many goroutines with similar stack traces you may find the [panicparse](https://github.com/maruel/panicparse) library useful. 


## Recovering from a panic

You should recover within a **deferred function** to recover from a [panic](https://blog.golang.org/defer-panic-and-recover). Without a defer to make sure the recover executes last, recover will return nil and have no other effect. For example this will not work:

```go
func p() {
    // Recover does nothing outwith a defer
    if r := recover(); r != nil {
        fmt.Println("recover", r)
    }
    // Something is wrong, 
    // this panic will end the program
    panic("panic")
}
```

You need to use defer to recover:

```go
func main() {
    // Print calling p
    fmt.Println("calling p")
    // Call p to panic and recover
    p() 
    // Recovered, print panic over
    fmt.Println("panic over")
}

func p() {
    // Defer recover to the end of this function
    defer func() { 
        // Recover from panic
        if r := recover(); r != nil {
            // Print recover
            fmt.Println("recover", r)
        }
    }()

    // Something is wrong, panic
    panic("panic")
}
```

## Recover must be in the same goroutine

You can only recover from panics in the current goroutine. If you have panics two goroutines deep, recovering at the top level won't catch them. For example in a web server handler which spawns a goroutine, you should protect against panics.

```go
func a() {
    panic("panic a") // this panic will crash the program
}

func main() {
    // This recover does nothing
    defer func() {
        if x := recover(); x != nil {
            fmt.Println("catch panic main")
        }
    }()
    fmt.Println("start")
    go a() // use of go means recover above won't work
    time.Sleep(1 * time.Second)
}
```

In order to catch the panic in a, a recover within that goroutine is required. Note this is not to do with function scope \(removing the go before a\(\) would allow the recover in main to work. For more detail see [Handling Panics](https://golang.org/ref/spec#Handling_panics) in the Go spec.

```go
func a() {
    // This recover is required to catch the panic in a
    defer func() { // recovers panic in a
        if x := recover(); x != nil {
            fmt.Println("catch panic a")
        }
    }()
    panic("panic a")
}

func main() {
        // this recover does nothing
    defer func() { // does nothing
        if x := recover(); x != nil {
            fmt.Println("catch panic main")
        }
    }()
    fmt.Println("start")
    go a() // use of go makes the recover above redundant
    time.Sleep(1 * time.Second)
}
```

## Error strings

Error strings may be used inside other error strings, so they should not be capitalized \(unless beginning with proper nouns or acronyms\) or end with punctuation. Make them composable, so that they can be logged or annotated with other messages:

```go
// Annotating an error
return fmt.Errorf("pkgname: failed to read %s: %v", filename, err)

// Returning an error without comment - prefer annotating if possible
return err
```

By convention, errors are annotated with the package name or function involved, to make it easy to find the error. You should try to make your error strings unique so that there is no confusion over where the error may have been emitted. 

## Don't panic

[Panic](https://blog.golang.org/defer-panic-and-recover) is intended as a mechanism to report exceptional errors which require the program to exit immediately, or to report programmer error which should be fixed. You don't want to see it in production, nor should you use it to try to reproduce exceptions, which were left out of the language for a reason. 

In web or api servers, you may never need to use the keyword panic, and should prefer not to. Panic is fine for programming errors, or really exceptional situations \(this should never happen\), but try to avoid using it if you can, especially if you're writing a library. Your users will thank you.

## Don't use log.Fatalf or log.Panic

For the same reasons, don't use log.Fatalf or log.Panic except in tests or short programs, because they will halt your program without cleanup and are equivalent to calling panic. 

In almost all cases you should recover gracefully from errors instead of calling a function which terminates the program.

```go 
    // Don't do this, handle the error
    log.Fatalf("broken:%s",err)
```

## Asserts & Exceptions

Go doesn't provide asserts or exceptions by design. There are reasons given for both decisions in the [FAQ](https://golang.org/doc/faq#exceptions) on the Go website. 

## Go is boring

Hopefully by now it is clear the Go language is deliberately limited and boring. If you want a stable platform on which to build exciting programs, this is a feature, not a bug. The language keeps getting a little better with every iteration (faster, fewer pauses, bugs fixed) without breaking your programs or introducing surprising new idioms. 

Less, in the case of programming languages, is more.