# Channels

Channels are a queue of values of a specific type which can be used to share information safely between goroutines. You can read more about the uses of channels with goroutines in the [Pipelines](https://blog.golang.org/pipelines) article on the go blog. Before using channels, consider other options, particularly if you just need to control access to shared state. You can use mutexes to control access to state across goroutines, which is much simpler than channels. Use channels to orchestrate more complex behaviour like signalling between goroutines, passing ownership of data, or distributing units of work.

## Nil Channels

Sending to a nil channel blocks forever:

```go
func main() {
    var c chan string
    // send to nil channel blocks forever
    c <- "test" 
}
```

Receive on a nil channel blocks forever:

```go
func main() {
    var c chan string
    // receive on nil blocks forever
    fmt.Println(<-c) 
}
```

## Closed Channels

Sending to a closed channel causes a panic

```go
package main

import (
    "fmt"
)

func main() {
    output := make(chan int, 1)
    write(output, 2) // send 
    close(output)    // close 
    write(output, 3) // send after close = panic
}
```

A receive from a closed channel returns the zero value immediately

```go
package main

import "fmt"

func main() {
    c := make(chan int, 2)
    c <- 1
    c <- 2
            c <- 3
    close(c)
    for i := 0; i < 5; i++ {
       fmt.Printf("%d ", <-c) 
    }
}
```

outputs the channel values then the zero value

> 1 2 0 0 0

Instead you can use a range loop to just get the values

```go
func main() {
    c := make(chan int, 2)
    c <- 1
    c <- 2
    close(c)
    for v := range c {
        fmt.Printf("%d ", v)
    }
}
```

## Stopping a goroutine

You can use a channel to stop a goroutine by using the channel to signal completion.

```go
// create a signal channel 
done := make(chan bool)

// launch the goroutine 
go func() {

    // listen for signals
    for {
        select {
        case <- done:
            return
        default:
            // Do something
            // ...
        }
    }
}()

// Do something
// ... 

// Stop the goroutine
done <- true
```

## Deadlocks

Go channels created with make\(chan T\) without a size are not buffered. An unbuffered channel is synchronous, you can only send when there is a receiver. If the reads don't match the writes, the anon goroutines deadlock.

```go
func main() {
    channel := make(chan string)
    done_channel := make(chan bool)
    go func() {
        channel <- "value" // write 1
        channel <- "value" // write 2
        done_channel <- true
    }()
    variable := <-channel // read 1    
    ok <-done_channel
    fmt.Println(variable,ok)
}
```

> fatal error: all goroutines are asleep - deadlock!

```go
func main() {
    channel := make(chan string)
    done_channel := make(chan bool)
    go func() {
        channel <- "write1" // write 1
        channel <- "write2" // write 2
        done_channel <- true
    }()
    variable := <-channel // read 1
    variable = <-channel  // read 2 required to finish
    ok := <-done_channel
    fmt.Println(variable, ok)
}
```

> write2 true

## Counting channel elements

If you want to know how many elements are in a channel, you can just use len\(channel\)

```go
func main() {
        c := make(chan int, 100)
        for i := 0; i < 34; i++ {
                c <- 0
        }
        fmt.Println(len(c))
}
```



