# Errors & Logging

[Errors](https://blog.golang.org/error-handling-and-go) and [logging](https://golang.org/pkg/log/) are one area of Go which does perhaps does deserve the label of simplistic rather than simple. The log package has no levels or interfaces, it simply prints to standard error by default. For many applications though, this is enough, and there are various logging packages available for more sophisticated requirements. The error type is very simple and errors are stored as strings. 

## The Error Type

The [error](https://blog.golang.org/error-handling-and-go) type in go is a very simple interface, with one method. Errors offer no introspection into what went wrong or storage of other data. Often errors are nested, as one error may be annotated several times as it passes up the stack. Try to prefer handling an error as close to the site of the error as possible.

```go
type error interface {
  Error() string
}
```

You can use your own type for error, as long as it conforms to this interface, in some cases you might want to use a more complex type than the string only errors favoured by the standard library, to record an http status code for example.

If you're defining interfaces, prefer requiring the error interface rather than a concrete type. You can use type assertions to determine if an error is of the type you're interested in.

## to, err := human\(\)

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
// Don't do this
if err != nil {
// Don't do this
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

## Stack traces on errors

If you want to get a stack trace at the point of error which is not a panic, you can use the runtime package to determine the caller. You can also use the unofficial [go-errors](https://github.com/go-errors/errors) package to record the stack trace. Finally, if you don't mind dumping a stack trace to stderr and terminating the program, you can panic.

## Recovering from a panic

You should recover within a **deferred function** to recover from a [panic](https://blog.golang.org/defer-panic-and-recover). Without a defer to make sure the recover executes last, recover will return nil and have no other effect. If the current goroutine is in a panic, a call to recover will capture the value given in panic, and swallow it. For example this will not work:

```go
func p() {
    // Recover does nothing outwith a defer
    if r := recover(); r != nil {
        fmt.Println("recover", r)
    }

    // Something is wrong, this panic will end the program
    panic("panic")
}
```

You need to use defer to recover:

```go
func main() {
        // Start
    fmt.Println("calling p")

    // Call p to panic and recover
    p()

    // Recovered
    fmt.Println("panic over")
}

// p panics and recovers
func p() {
    // Defer recover to the end of this function
    defer func() {
        // Recover from panic
        if r := recover(); r != nil {
            fmt.Println("recover", r)
        }
    }()

    // Something is wrong
    panic("panic")
}
```

## Recover must be in the same goroutine

You can only recover from panics in the current goroutine. If you have panics two goroutines deep, recovering at the top level won't catch them \(for example in a web server handler which then spawns another goroutine, you must protect against panics within the goroutine spawned\).

```go
// if a is run in a goroutine, this panic will crash the program
func a() {
    panic("panic a")
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
// if a is run in a goroutine, this panic will not crash the program
func a() {
    // This recover is required to catch the panic in a
    defer func() {
        if x := recover(); x != nil {
            fmt.Println("catch panic a")
        }
    }()
    panic("panic a")
}

func main() {
        // this recover does nothing
    defer func() {
        if x := recover(); x != nil {
            fmt.Println("catch panic main")
        }
    }()
    fmt.Println("start")
    go a() // use of go makes the recover above redundant
    time.Sleep(1 * time.Second)
}
```

## Don't panic

[Panic](https://blog.golang.org/defer-panic-and-recover) is intended as a mechanism to report exceptional errors which require the program to exit immediately, or to report programmer error which should be fixed. You don't want to see it in production, nor should you use it to try to reproduce exceptions, which were left out of the language for a reason. In servers, you may never need to use the keyword panic, and should prefer not to.

Panic is fine for programming errors, or really exceptional situations \(this should never happen\), but try to avoid using it if you can, especially if you're writing a library. Your users will thank you.

## Don't use log.Fatalf or log.Panic

For the same reasons, don't use log.Fatalf or log.Panic except in tests or short programs, because they will halt your program without cleanup and are equivalent to calling panic.In almost all cases you can recover gracefully from errors.

## Asserts & Exceptions

Go doesn't provide asserts or exceptions by design. Go is boring.

