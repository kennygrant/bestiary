# Goroutines

The `go` keyword is one of the distinguishing features of Go, but it can lead to some bugs if not used carefully.

## Goroutines not completing

When a program's main ends,  all running goroutines that were created by the program will also be stopped. So if your main only spins up some goroutines, those goroutines may not have time to finish. Use a wait group to make sure your goroutines are completed before the app terminates. 

Example. 



## ListenAndServe

After spawning a server with http.ListenAndServe, don't expect control to return to your main function until the end of the program.

Example 

## Range & `go`

If you range over values and launch a goroutine within your range, the value v sent to the goroutine will not change each time.

```go
for _, v := range values {
        go func() { f(v) }()
}
```

## Blocking functions & `go`

If you provide a blocking function as the argument to a function in a go routine, the blocking run will be run synchronously first, and the realised argument will be sent to the goroutine function.

```go
// if b blocks, b will be called first, then after completion f called with the result
go f(fb())
```





