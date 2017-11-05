# Setting up Go

##### If you have already written and compiled Go code before you can probably skip this section.

To get up and running with Go, you should follow the [Getting Started](https://golang.org/doc/install) instructions on the go website. If you're on Mac or Windows there is an installer which will make life easier for you and is the recommended route to installation – you can find the latest installers at the [download](https://golang.org/dl/) page.

You can use homebrew to install go on macs, but you're probably better sticking with the official installer package available on golang.org. Do not use both.

## Setting GOPATH

`GOPATH`  is the bugbear of many new users of Go, but it is simply a place to store go code and binaries. This environment variable was introduced in order to make it simple for the Go tool to install dependencies automatically, and to keep all go code in one place. Since `Go 1.7` this has defaulted to:

```
$HOME/go
```

`GOPATH` contains three directories:

* bin - for installed binaries
* src - for go source code
* pkg - a cache for intermediate build products

You may want to add your `$GOPATH/bin` folder to your path so that installed go tools are found automatically.

So you don't need to set it explicitly, but you do need to put all your go code under `$GOPATH/src`  for go tools to work correctly. This is where code will be installed when you run go get, and this is where your dependencies will be stored. There is no concept of versioning in the go get tool, so it will fetch the latest master of any dependencies.

While it can be frustrating for newcomers, you should accept that all go code will live in your `GOPATH` – there are workarounds to attempt to keep code elsewhere, but the standard tools at present assume you use `GOPATH` for your code. You may find projects sometimes use folders called pkg or src internally inside their main go gettable folder, but this is unrelated to `GOPATH` and is just a way of organising their code. The only special src folder is that at the root of your `GOPATH`.

The similarly named ENV variable `GOROOT`  need only be set** if installing to a custom location**. In most circumstances you can ignore `GOROOT`, as it is not required for a normal Go setup. 

## Checking your setup

To test your setup and confirm go is installed correctly, try the go command:
```bash
go version
```

You should see output something like this telling you the go version and the installed platform of the go tools:

> go version go1.9 darwin/amd64

Then try a very simple program to check that your go setup is working. Save this file in  `$GOPATH/src/hello.go` :

```go
package main 

import (
    "fmt"
)

// Main is the entry point of your program
func main() {

    // Print to stdout
    fmt.Printf("hello, 世界\n")

}
```

and use the go command to run it:

```bash
go run $GOPATH/src/hello.go
```

You should see the output:

> hello, 世界

If you don't, your installation is not working and you should recheck the steps above and check the [Installation](https://golang.org/doc/install) instructions on the Go website.

## Cross Compiling

You can use Go to compile programs not just for the platform you're on, but for another supported platform, like Windows or Linux if you're working on a mac. This makes it very easy to deploy programs as single binaries, without worrying about the dependencies or building on your server. For example if you want to build the hello.go program above for the Linux platform, you could use:

```bash
GOOS=linux GOARCH=arm go build -o hello-linux $GOPATH/src/hello.go
```

This will give you a binary which runs on linux arm \(or any other platform you choose\), without any extra fuss.

# Editors

Some of the most widely used editors for Go programmers, roughly in order of popularity are: [VSCode](https://code.visualstudio.com/), Vim \(with [vim-go](https://github.com/fatih/vim-go)\), [IntelliJ IDEA](https://www.jetbrains.com/idea/), Emacs, and [Atom](https://atom.io/). Since go just requires text files and doesn't require any special support, you can easily switch between editors, but make sure your editor has support for running go fmt on save, and ideally support for looking up definitions from within the editor. Another useful tool for browsing go source code is [sourcegraph](https://sourcegraph.com) \(which happens to be written in Go\).

You should always run [go fmt](https://blog.golang.org/go-fmt-your-code) on your code, and ideally a linter like gometalinter too. Try to set up your editor initially you run these on save. Almost all public go code is run through go fmt, and if interacting with other go programmers, this is taken as a given. Most editors have a go plugin like [vscode-go](https://github.com/Microsoft/vscode-go/) or [vim-go](https://github.com/fatih/vim-go) which will take care of this for you.

#### Linter Warnings

If you pay attention to linter warnings, and make sure you fix them all, you'll find your go code is more idiomatic and you should avoid common errors like executing goroutines with values from a range loop.

## Dependencies

> Duplication is far cheaper than the wrong abstraction
> Sandi Metz

There is a culture of limiting dependencies in Go, unlike some other ecosystems, the design of Go does not encourage importing libraries for trivial functions. If you're coming from node this will come as a shock, but you should try to adapt to the very different culture, as there are some advantages.

The biggest problem with dependencies in the long term is entropy over time - the more dependencies you have, the greater the chance of changes which break your build \(for security, or api changes, or new features\), and the more painful it becomes to keep up to date with the ecosystem you've bought into. This is why it is useful to limit your dependencies, and explicitly version those you have, and why vendoring \(taking a copy of dependencies frozen at a given version\) has become an accepted solution in the Go community to importing dependencies.

Although Go is statically typed which offers some protection, you should not assume that if dependencies change and your code compiles everything is working. A change to a dependency might make subtle changes to defaults or values which while they still compile result in the unexpected behaviour. The only solution to managing dependencies is to freeze them at the import version and inspect changes carefully before upgrading.

## Resources

You should start by working through the[ Go Tour](https://tour.golang.org/welcome/1), you should then skim [Effective Go](https://golang.org/doc/effective_go.html) \(you will want to come back to this later\). You should read the [docs](https://golang.org/pkg/) from the standard library, which contain lots of examples, and finally you might want to refer to [Go By Example](https://gobyexample.com/) - a set of code snippets.

### Go Playground

The Go Playground is a web service that runs on golang.org's servers. The service receives a Go program, compiles, links, and runs the program inside a sandbox, then returns the output. The intention is for go playground links to last forever \(in internet time that's at least a few years\).

The playground has certain limitations, mostly for security reasons it restricts certain operations like writing files or accessing the network, and the time is the same for each run. There are also limits on execution time and on CPU and memory usage. See [Inside the Go playground](https://blog.golang.org/playground) for more details.

### Stack Overflow

You can find the answer to many questions about Go on [stackoverflow](https://stackoverflow.com/questions/tagged/go), in particular small questions of grammar. A few tips for using this resource:

* Be sure to search for related questions with the go tag before you post your own. 
* Try to post a link to a reproduction of the problem on play.golang.org
* Include the full code and an explanation of the problem
* Accepted answers on stack overflow are sometimes wrong

## Community

There are many partially overlapping online communities for Go which vary greatly in style and content.

[The Go Forum](https://forum.golangbridge.org/) is a friendly place to ask beginner questions

The [Go Time](https://changelog.com/gotime) podcast interviews luminaries in the Go community and beyond with a focus on Go

[Golang News](https://golangnews.com) provides links to articles, videos etc. about Go

The Go [subreddit](https://www.reddit.com/r/golang/) is another source of links and news

[Women Who Go](http://www.womenwhogo.org/) organises meetups for women and gender minorities who use Go all over the world

The [Gopher Slack ](https://gophers.slack.com)has an active community of slackers

The [Golang Nuts](https://groups.google.com/forum/#!forum/golang-nuts) mailing list has a lot of activity if you prefer mailing lists

There are many videos from Go conferences at [GopherVids](https://gophervids.appspot.com/)

[Just for func](https://www.youtube.com/channel/UC_BzFbxG2za3bp5NRRRXJSw) is a great podcast on Go programming by the inimitable [@francesc](https://twitter.com/francesc)

There are a huge number of meetups and conferences about Go, you can find more on the [Community](https://github.com/golang/go/wiki#the-go-community) page of the wiki.

## Writing Idiomatic Go {#style}

For a guide to idiomatic Go, you can refer to the wiki [Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments), which is a summary of comments made on contributions to the Go project and a lot of good advice on structure and names in Go code.

Go is opinionated about formatting \(as about so much else\) and provides the tool [go fmt](https://blog.golang.org/go-fmt-your-code) to enforce the suggested formatting. There is one standard style for code, including brace positions, spacing etc, enforced by the tools. This has been embraced by users of go, and it turns out it doesn't really matter where you place your braces or whether you use spaces or tabs, so you can now spend your time worrying about more important things, like whether go should have generics.

