# ![](/assets/bestiary.jpg)

# 

# The Go Bestiary

The Go language is deceptively simple, with only 25 [keywords](https://golang.org/ref/spec#Keywords) and few data structures. The syntax is very like other languages in the C family. This means programmers coming from other languages in this family \(C, C++, Java\) find it easy to get started, and sometimes frustrating at first as they find their way around Go idioms and the rather rigid style guidelines, and feel the lack of familiar tools which Go simply leaves out - inheritance, assertions, exceptions, the ternary operator, enums, and generics are all missing [by design](https://golang.org/doc/faq#Why_doesnt_Go_have_feature_X).

The intention of this book is to let a programmer who has some experience in other languages get up to speed quickly on Go, and be aware of pitfalls and unexpected behaviour which people learning the language often prey into. As the language is relatively simple, there are not too many hidden surprises, and once you have spent some time working with Go you are unlikely to run into problems you don't understand because the core language and standard library is not changing much.

One of the most rewarding aspects of programming in Go is the stability of the language and standard library, encapsulated in the [Go 1 promise](https://golang.org/doc/go1compat) from 2012, or Brad Fitzpatrick's whimsical description of the language as [asymptotically approaching boring](https://golangnews.com/stories/845-video-introducing-go-1.6-asymptotically-approaching-boring-by-brad-fitzpatrick). If you build an app today, in a few years it will probably compile with minimal changes, as long as you have limited dependencies. While being difficult for maintainers, this is a liberating promise for Go programmers as it means you don't have to worry about churn at that level of the ecosystem, only in the libraries above it. If you're looking for a cutting edge  language which explores the limits of computing science, Go is not for you. Go is boring by design.

## Writing Idiomatic Go {#style}

For a guide to idiomatic Go, you can refer to the wiki [Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments), which is a summary of comments made on contributions to the Go project and a lot of good advice on structure and names in Go code.

Go is opinionated about formatting \(as about so much else\) and provides the tool [go fmt](https://blog.golang.org/go-fmt-your-code) to enforce the suggested formatting. There is one standard style for code, including brace positions, spacing etc, enforced by the tools. This has been embraced by users of go, and it turns out it doesn't really matter where you place your braces or whether you use spaces or tabs, so you can now spend your time worrying about more important things, like whether go should have generics.

