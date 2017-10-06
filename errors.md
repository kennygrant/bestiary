# Errors & Logging

[Errors](https://blog.golang.org/error-handling-and-go) and [logging](https://golang.org/pkg/log/) are one area of Go which does perhaps does deserve the label of simplistic rather than simple. The log package has no levels or interfaces, it simply prints to standard error by default. For many applications, this is enough.

## Either log or return an error

If you're logging an error, you've decided it's not important enough to handle. If you're returning an error \(usually with annotation\), you want the caller to handle it \(which might include logging or reporting it to the user\). 

## Don't use log.Fatalf or log.Panic

For the same reasons, don't use log.Fatalf or log.Panic except in tests or short programs, because they will halt your program without cleanup. Better to explicitly call os.Exit if required, but in almost all cases you can recover gracefully.

## The Error Type

The [error](https://blog.golang.org/error-handling-and-go) type in go is a very simple interface, with one method. Errors offer no introspection into what went wrong or storage of other data. Often errors are nested, so that one error.

```
type error interface {
  Error() string
}
```

The convention is always to return an error as the last argument of a function

You can use your own type for error, as long as it conforms to this interface, in some cases you might want to use a more complex type than the string only errors favoured by the standard library, to record an http status code for example. If you're defining interfaces, prefer requiring the error interface rather than a concrete type. You can use type assertions to determine if an error is of the type you're interested in.

## Stack traces

If you want to get a stack trace at the point of error, you can use the runtime package to determine the caller. You can also use the unofficial [go-errors](https://github.com/go-errors/errors) package to record the stack trace.

## Recovering from a panic

You should use defer to recover from a [panic](https://blog.golang.org/defer-panic-and-recover).

## Recover doesn't catch all panics

If you have panics two go routines deep \(say your web server spawns a goroutine for a handler, which then spawns its own goroutine\), recover at the top level in the web server won't help you. You need a recover for each goroutine spawned.

EXAMPLE

## Don't use panic \(much\)

[Panic](https://blog.golang.org/defer-panic-and-recover) is intended as a mechanism to report exceptional errors which require the program to exit immediately, or to report programmer error which should be fixed. You don't want to see it in production, nor should you use it to try to reproduce exceptions, which were left out of the language for a reason.

Panic is fine for programming errors, or really exceptional situations, but try to avoid using it if you can, especially if you're writing a library. Your users will thank you.

