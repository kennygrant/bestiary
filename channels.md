# Channels

Channels are a queue of values of a specific type which can be used to share information safely between goroutines. You can read more about the uses of channels with goroutines in the [Pipelines](https://blog.golang.org/pipelines) article on the go blog.

## Stopping a goroutine

Stopping a goroutine using a channel to signal completion.

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

```go
package main

import "fmt"

func main() {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
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

## Nil Channels

Sending to a nil channel blocks forever

```go
func main() {
    var c chan string
    c <- "test" // send to nil channel blocks forever
}
```

Receive on a nil channel blocks forever

```go
func main() {
    var c chan string
    fmt.Println(<-c) // receive on nil blocks forever
}
```

## Deadlocks

Go channels created with make\(chan T\) without a size are not buffered. An unbuffered channel is synchronous, you can only send when there is a receiver. If the reads don't match the writes, the anon goroutines deadlock.

```go
package main

import "fmt"

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
package main

import "fmt"

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

```
package main

import "fmt"

func main() {
        c := make(chan int, 100)
        for i := 0; i < 34; i++ {
                c <- 0
        }
        fmt.Println(len(c))
}
```

## 



