# Setting up Go

To get up and running with Go, you should follow the [Getting Started](https://golang.org/doc/install) instructions on the go website. If you're on Mac or Windows there is an installer which will make life easier for you and is the recommended route to installation. This installs go much like any other project. You can find the latest installers at the [download](https://golang.org/dl/) page.

## Setting GOPATH

`GOPATH`  is the bugbear of many new users of Go. This environment variable was introduced in order to make it simple for the Go tool to install dependencies automatically, and to keep all go code in one place. Since `Go 1.7` this has defaulted to:

```
$HOME/go
```

So you don't need to set it explicitly, but you do need to put all your go code under `$GOPATH/src`  for go tools to work correctly.

You can use go run on code anywhere, but if you use go get to fetch code, it will be placed in src, under a path corresponding to the path on the server. For example code fetched from GOPATH

You may find projects sometimes use folders called pkg or src internally inside their main go gettable folder, but this is unrelated to gopath and is just a way of organising their code.

The similarly named ENV variable `GOROOT`  need only be set** if installing to a custom location**. In most circumstances you can ignore `GOROOT`, as it is not required for a normal Go setup. You may find some old instructions which reference it, but you can safely ignore it. So for a new version of Go install don't need to set `GOROOT`  or `GOPATH` .

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

## Go fmt

You should always run [go fmt](https://blog.golang.org/go-fmt-your-code) on your code, and ideally a linter like gometalinter too. Try to set up your editor initially yo run these on save. Almost all public go code is run through go fmt, and if interacting with other go programmers, this is taken as a given. Most editors have a go plugin like [vscode-go](https://github.com/Microsoft/vscode-go/) or [vim-go](https://github.com/fatih/vim-go) which will take care of this for you.

## Cross Compiling

You can use Go to compile programs not just for the platform you're on, but for another supported platform, like Windows or Linux if you're working on a mac. This makes it very easy to deploy programs as single binaries, without worrying about the dependencies or building on your server. For example if you want to build the hello.go program above for the Linux platform, you could use:

```
GOOS=linux GOARCH=arm go build -o hello-linux $GOPATH/src/hello.go
```

This will give you a binary which runs on linux arm \(or any other platform you choose\), without any extra fuss.

## Go Playground

The Go Playground is a web service that runs on golang.org's servers. The service receives a Go program, compiles, links, and runs the program inside a sandbox, then returns the output. The intention is for go playground links to last forever \(in internet time that's at least a few years\).

The playground has certain limitations, mostly for security reasons it restricts certain operations like writing files or accessing the network, and the time is the same for each run. There are also limits on execution time and on CPU and memory usage. See [Inside the Go playground](https://blog.golang.org/playground) for more details.

## Using Stack Overflow

You can find the answer to many questions about Go on [stackoverflow](https://stackoverflow.com/questions/tagged/go), in particular small questions of grammar. Be sure to search for related questions before you post your own. Try to use the right tags and post a full explanation of your problem with code to get a good response.

