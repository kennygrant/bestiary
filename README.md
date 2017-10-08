# The Go Bestiary

The Go language is deceptively simple, with only 25 [keywords](https://golang.org/ref/spec#Keywords) and few data structures. The syntax is very like other languages in the C family, and programmers coming from other languages with this heritage will find it easy to get started with the language, but sometimes frustrating to get to grips with the culture of radical minimalism. Often programmers from other languages keenly feel the lack of familiar tools which Go simply leaves out - inheritance, assertions, exceptions, the ternary operator, enums, and generics are all missing [by design](https://golang.org/doc/faq#Why_doesnt_Go_have_feature_X). Sometim es they will chafe also at the strict rules on formatting or the culture of limited dependencies and lack of abstraction.

The corollary to this culture of simplicity, and one of the most rewarding aspects of programming in Go, is the stability of the language and standard library, encapsulated in the [Go 1 promise](https://golang.org/doc/go1compat) made way back in 2012 \(which still holds today\), or Brad Fitzpatrick's whimsical description of the language as [asymptotically approaching boring](https://golangnews.com/stories/845-video-introducing-go-1.6-asymptotically-approaching-boring-by-brad-fitzpatrick). If you build an app today, it will probably compile with minimal or no changes until Go 2. While being difficult for maintainers, this is a liberating promise for Go programmers as it means you don't have to worry about churn at that level of the ecosystem, only in the libraries above it. So if you're looking for a cutting edge language which explores the limits of computing science, or favours terse abstrations, Go is not for you – Go is boring by design.

### Bugs

Though it aspires to simplicity and  Go code, . A look at the [Trophy Cabinet](https://github.com/dvyukov/go-fuzz#trophies) of bugs found by [Go Fuzz](https://github.com/dvyukov/go-fuzz) should disabuse us of the notion that most bugs are esoteric or difficult to defend against, or even specific to Go as a language. Often the same errors crop up again and again - of bounds not checked or nil pointers. The most common bugs encountered \(even from experienced programmers\) are:

* Index out of range 
* Slice bounds out of range
* Nil pointer dereferences
* Failing to check errors

These are trivial errors, but are easy to make in a sufficiently large program, and are usually due to expectations about data passed in from outside the program which don't hold true \(hence they are easier to find with a fuzzer\). There are also some mistakes which are easy to make in go which a fuzzer might not detect, like not correctly passing values to a goroutine within a range loop.

### Why read this book?

The intention of this book is to let a programmer who has some experience in other languages get up to speed quickly on Go, while being aware of the idioms and possible bugs lurking in their new Go code. The book therefore presents a mix of advice for structuring your Go programs and descriptions of common mistakes to avoid, with liberal code examples.

If you're familiar with Go but not an expert, hopefully there will also be a few interesting facts about the language you haven't yet uncovered, and some potential bugs which you might not be aware of. You may want to skip the sections on setup and packages for example if you are already familiar with Go.

If you're new to Go, but experienced in other languages, you'll find succinct descriptions of the most important differences in Go which usually trip up learners \(for example declaring strings, strings vs runes, using goroutines within a for loop\), descriptions of bugs which beginners often fall prey to, and plenty of links to in depth articles about unique aspects of the language like goroutines and slices.

