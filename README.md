# Introduction

### 

### About Go

The Go language spec is deceptively simple to read, with only 25 [keywords](https://golang.org/ref/spec#Keywords) and few data structures.

The syntax is very like other languages in the C family, and programmers coming from other languages with this heritage will find it easy to get started with the language, however it can be difficult to assimilate the culture of radical minimalism. Some programmers feel keenly the lack of familiar tools which Go simply leaves out - inheritance, assertions, exceptions, the ternary operator, enums, and generics are all missing [by design](https://golang.org/doc/faq#Why_doesnt_Go_have_feature_X). In addition, the strict rules on formatting and the culture of limited dependencies and minimal abstraction can feel restrictive, but these very deliberate omissions also provide some concrete advantages. The corollary to this culture of simplicity, and one of the most rewarding aspects of programming in Go, is the accessibility and simplicity of the standard library code and many of the libraries available - Go code is easy to read as well as write. 

Also invaluable is the clarity and stability of the standard library, encapsulated in the [Go 1 promise](https://golang.org/doc/go1compat) made way back in 2012 \(which still holds today\), or Brad Fitzpatrick's whimsical description of the language as [asymptotically approaching boring](https://golangnews.com/stories/845-video-introducing-go-1.6-asymptotically-approaching-boring-by-brad-fitzpatrick). If you build an app today, it will compile with minimal or no changes until Go 2. While being difficult for the maintainers, this is a liberating promise for Go programmers as it makes this level of the ecosystem very stable.

If you're looking for a cutting-edge language which explores recent research, or one which favours terse abstractions, Go is not for you – Go is boring by design.

### Bugs in Go programs

Programs in Go, like those in any other language, are subject to bugs. 

Some are caused by unexpected data - a look at the [Trophy Cabinet](https://github.com/dvyukov/go-fuzz#trophies) of bugs found by [Go Fuzz](https://github.com/dvyukov/go-fuzz) should disabuse us of the notion that most bugs are esoteric or difficult to defend against, or even specific to Go as a language. Often the same errors crop up again and again – bounds not checked or nil pointers. The most common bugs encountered \(even by experienced programmers\) are:

* Index out of range 
* Slice bounds out of range
* Nil pointer dereferences

These are trivial errors, but are easy to make in any sufficiently large program, and are usually due to expectations about data passed in from outside the program which don't hold true \(hence they are easier to find with a fuzzer\). 

There are also a few mistakes which are easy to make when learning Go which a fuzzer might not detect, such as not correctly passing values to a goroutine within a range loop, misuse of an API, or lack of familiarity with the tradeoffs in the design of slices or strings in Go. These are the classes of bugs which this book will hopefully help the reader avoid.

### Why read this book?

Typically Go programmers get up to speed quickly and feel productive within days or weeks, but it can be harder to assimilate the culture and be aware of the subtle problems that can occur. _The Go Bestiary_ provides a quick guide to allow a programmer who has some experience in other languages get started quickly in Go, while being aware of the idioms to use and areas where there might be bugs lurking in their new Go code. The book presents a mix of advice for structuring your Go programs and descriptions of common mistakes to avoid, with liberal code examples, to help ease you into using Go. 

If you're familiar with Go but not an expert, hopefully there will also be a few interesting facts about the language you haven't yet uncovered, and some potential bugs which you might not be aware of. You may want to skip the sections on setup and packages in this case, as they will cover familiar ground.

If you're new to Go, but experienced in other languages, you'll find succinct descriptions of the most important differences in Go which usually trip up learners \(for example declaring strings, strings vs runes, copying slices, using goroutines within a for loop\), descriptions of bugs which beginners often fall prey to, and plenty of detail about unique aspects of the language like goroutines and slices.

