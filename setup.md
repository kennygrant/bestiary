

# Setting up Go

To get up and running with Go, you should follow the [Getting Started](https://golang.org/doc/install) instructions on the go website. If you're on Mac or Windows there is an installer which will make life easier for you and is the recommended route to installation. This installs go much like any other project. 

You can find the latest installers at the [download](https://golang.org/dl/) page. 

## Setting GOPATH

`GOPATH`  is the bugbear of many new users of Go. This environment variable was introduced in order to make it simple for the Go tool to install dependencies automatically, and to keep all go code in one place. Since `Go 1.7` this has defaulted to:

```
$HOME/go
```

So you don't need to set it explicitly, but you do need to put all your go code under `$GOPATH/src`  for go tools to work correctly.

## Setting GOROOT

`GOROOT`  needs to be set **only when installing to a custom location**. In most circumstances you can ignore GOROOT, and it is not required for a normal Go setup. 

## Checking your setup 

To test your setup, try the go command 

```
go version
```

You should see output something like this telling you the go version and the installed platform of the go tools:

> go version go1.9 darwin/amd64

Then try a very simple program. Save this file in  `$GOPATH/src/hello.go` :

```
package main 

import "fmt"

func main() {
    fmt.Printf("hello, 世界\n")
}
```

and use the go command to run it:

```
go run $GOPATH/src/hello.go
```

You should see the output:

> hello, 世界

If you don't, your installation is not working and you should recheck the steps above. 

## Cross Compiling 

You can use Go to compile programs not just for the platform you're on, but for another platform, like Windows or Linux if you're working on a mac. This makes it very easy to deploy programs as single binaries, without worrying about the dependencies or building on your server. For example if you want to build the hello.go program above for the Linux platform, you could use:







