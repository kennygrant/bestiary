# Goroutines

Prefix a function or method call with the `go` keyword to run the call in a new goroutine, asynchronously. The `go` keyword is one of the distinguishing features of Go, but it can lead to subtle bugs if not used carefully.

> Do not communicate by sharing memory; instead, share memory by communicating.
> Rob Pike

## Asynchronous execution

The simplest usage of a goroutine is simply to execute a function without waiting for a response. For example in a web handler to send a mail, you may not want to delay till the mail is sent before reporting back to the user.

```go
// Send a message using the go keyword, without waiting for completion
message := "hello word"
go mail.Send(message, example@example.com) 

// Execution continues on the main goroutine without pausing...
```

## Waiting for go

A common failure when using goroutines in a simple program is to fail to wait for them to finish. When a program's main ends,  all running goroutines that were created by the program will also be stopped. So if your main only spins up some goroutines, those goroutines may not have time to finish, and you're left waiting for godot.

```go
func main() {
    fmt.Println("waiting...")
    go func() {
    fmt.Println("godot") // doesn't arrive
    }()
}
```

> waiting...

Use a wait group to make sure your goroutines are completed before the app terminates.

```go
func main() {
  var wg sync.WaitGroup
  fmt.Println("hello world")
  wg.Add(1)
  go func() {
    defer wg.Done()
    fmt.Println("goodbye, cruel world")
  }()
  wg.Wait()
}
```

> hello world  
> goodbye, cruel world

## Goroutines & range {#goroutines-range}

If you range over values and launch a goroutine within your range, the value v sent to the goroutine will not change each time. . This is because the variable v is reused for each iteration of the for loop. Given a func f that prints the arguments:

```go
func f(s string) {
   fmt.Println(s)
}
```

If we run it in a goroutine, the only value printed \(3 times\), is the last one.

```go
values := []string{"first", "second", "last"}
for _, v := range values {
   go func() { f(v) }()
}
```

> last last last

If instead the value is passed as an argument to the goroutine function, it works as expected:

```go
for _, v := range values {
    go func(val string) { f(val) }(v)
}
```

> first second last

This is probably the most common error when using goroutines, and is easy to make as it's natural to use the values of range within the loop. Always watch for this if using a goroutine inside a for loop - some linters may warn on this.

## Goroutines & Blocking functions

If you provide a blocking function as the argument to a function in a go routine, the blocking run will be run synchronously first, and the realised argument will be sent to the goroutine function.

```go
// if b blocks, b will be called first, then after completion f called with the result
// so b will be called before goroutine is invoked
go f(fb())
```

Instead an anonymous function could be used:

```go
// b will be called in the goroutine
go func() {
 f(fb())
}()
```

## Data Races

If two goroutines access the same memory without access control, this causes a race condition. Race detection is not yet automatic at compile time, but fortunately it's easy to detect at runtime with the [race detector](https://golang.org/doc/articles/race_detector.html).  To find data races, run your program with the -race option and it will warn you if races are detected:

```go
go run -race race.go
```

An example of a data race:

```go
m := make(map[string]string)

// Access OK
m["1"] = "a"

go func() {
  m["1"] = "a" // First conflicting access.
}()

m["2"] = "b" // Second conflicting access.
```

To avoid races, you can pass copies of values, or wrap access with a mutex from the [sync](https://golang.org/pkg/sync/) package.

## `GOMAXPROCS`

`GOMAXPROCS` sets how many processors the Go runtime uses to run goroutines on. Though it was set to 1 by default in early versions of Go for performance reasons, programs usually don't need to set`GOMAXPROCS,` as it is now set intelligently by the [runtime](https://golang.org/pkg/runtime/#GOMAXPROCS) based on the machine. Parallel programs might benefit from a further increase in `GOMAXPROCS` but be aware that [concurrency is not parallelism](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html). You may find this set in older source code, but it is very rarely necessary.

.

