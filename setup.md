# Setting up Go

To get up and running with Go, you should follow the [Getting Started](https://golang.org/doc/install) instructions on the go website. If you're on Mac or Windows there is an installer which will make life easier for you and is the recommended route to installation. This installs go much like any other project. You can find the latest installers at the [download](https://golang.org/dl/) page. If you have already written and compiled Go code you can probably skip this section.

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

## Cross Compiling

You can use Go to compile programs not just for the platform you're on, but for another supported platform, like Windows or Linux if you're working on a mac. This makes it very easy to deploy programs as single binaries, without worrying about the dependencies or building on your server. For example if you want to build the hello.go program above for the Linux platform, you could use:

```
GOOS=linux GOARCH=arm go build -o hello-linux $GOPATH/src/hello.go
```

This will give you a binary which runs on linux arm \(or any other platform you choose\), without any extra fuss.

# Editors

Some of the most widely used editors for Go programmers, roughly in order of popularity are: [VSCode](https://code.visualstudio.com/), Vim \(with [vim-go](https://github.com/fatih/vim-go)\), [IntelliJ IDEA](https://www.jetbrains.com/idea/), and [Atom](https://atom.io/). Since go just requires text files and doesn't require any special support, you can easily switch between editors, but make sure your editor has support for running go fmt on save, and ideally support for looking up definitions from within the editor. Another useful tool for browsing go source code is [sourcegraph](https://sourcegraph.com) \(which happens to be written in Go\).

You should always run [go fmt](https://blog.golang.org/go-fmt-your-code) on your code, and ideally a linter like gometalinter too. Try to set up your editor initially you run these on save. Almost all public go code is run through go fmt, and if interacting with other go programmers, this is taken as a given. Most editors have a go plugin like [vscode-go](https://github.com/Microsoft/vscode-go/) or [vim-go](https://github.com/fatih/vim-go) which will take care of this for you.

#### Linter Warnings

If you pay attention to linter warnings, and make sure you fix them all, you'll find your go code is more idiomatic and you will avoid common errors like executing goroutines with values from a range loop.

## Dependencies

There is a culture of limiting dependencies in Go, unlike some other ecosystems, the design of Go does not encourage importing libraries for trivial functions.

> Duplication is far cheaper than the wrong abstraction - Sandi Metz

The biggest problem with dependencies in the long term is change - the more dependencies you have, the greater the chance of changes which break your build \(for security, or api changes, or new features\), and the more painful it becomes to keep up to date with the ecosystem you've bought into. This is why it is useful to limit your dependencies, and explicitly version those you have, and why vendoring \(taking a copy of dependencies frozen at a given version\) has become an accepted solution in the Go community to importing dependencies.

You should not assume that if dependencies change and your code compiles everything is working. A change to a dependency might make subtle changes to defaults or values which while they still compile result in the wrong behaviour. The only solution to managing dependencies is to freeze them at the import version and inspect changes carefully before upgrading.

## Go Playground

The Go Playground is a web service that runs on golang.org's servers. The service receives a Go program, compiles, links, and runs the program inside a sandbox, then returns the output. The intention is for go playground links to last forever \(in internet time that's at least a few years\).

The playground has certain limitations, mostly for security reasons it restricts certain operations like writing files or accessing the network, and the time is the same for each run. There are also limits on execution time and on CPU and memory usage. See [Inside the Go playground](https://blog.golang.org/playground) for more details.

## Using Stack Overflow

You can find the answer to many questions about Go on [stackoverflow](https://stackoverflow.com/questions/tagged/go), in particular small questions of grammar. Be sure to search for related questions before you post your own. Try to use the right tags and post a full explanation of your problem with code to get a good response.

## Resources

You should start by working through the[ Go Tour](https://tour.golang.org/welcome/1)

You should then skim [Effective Go](https://golang.org/doc/effective_go.html) \(you will want to come back to this later\) 

You should read the [docs](https://golang.org/pkg/) from the standard library, which contain lots of examples 

You might want to refer to [Go By Example](https://gobyexample.com/) - a set of code snippets

## Community

There are many partially overlapping online communities for Go which you should be aware of.

[The Go Forum](https://forum.golangbridge.org/) is a friendly place to ask beginner questions

The [Go Time](https://changelog.com/gotime) podcast interviews luminaries in the Go community

[Golang News](https://golangnews.com) provides links to articles, videos etc. about Go

The Go [subreddit](https://www.reddit.com/r/golang/) is another source of links and news

The [Gopher Slack ](https://gophers.slack.com)has an active community of slackers

The [Golang Nuts](https://groups.google.com/forum/#!forum/golang-nuts) mailing list has a lot of activity if you prefer email

There are many videos from Go conferences at [GopherVids](https://gophervids.appspot.com/)

[Just for func](https://www.youtube.com/channel/UC_BzFbxG2za3bp5NRRRXJSw) is a great podcast on Go programming by the inimitable [@francesc](https://twitter.com/francesc)

## Writing Idiomatic Go {#style}

For a guide to idiomatic Go, you can refer to the wiki [Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments), which is a summary of comments made on contributions to the Go project and a lot of good advice on structure and names in Go code.

Go is opinionated about formatting \(as about so much else\) and provides the tool [go fmt](https://blog.golang.org/go-fmt-your-code) to enforce the suggested formatting. There is one standard style for code, including brace positions, spacing etc, enforced by the tools. This has been embraced by users of go, and it turns out it doesn't really matter where you place your braces or whether you use spaces or tabs, so you can now spend your time worrying about more important things, like whether go should have generics.

