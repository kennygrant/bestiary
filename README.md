# Go Bugs

### Common Go programming bugs & ways to avoid them

---

### Introducing Go

The intention of this book is to let a programmer who has some experience in other languages get up to speed quickly on Go, and be aware of pitfalls and unexpected behaviour which people learning the language often fall into. As the language is relatively simple, there are not too many hidden surprises, and as it there is an explicit promise not to break existing programs, once you have learned your way around the language, you are unlikely to run into problems you don't understand, because the language is simple.

### The Go 1 promise

One of the most about programming in Go is the stability of the language, encapsulated in the [Go 1 promise](https://golang.org/doc/go1compat), or Brad Fitz's whimsical description of the language as [asymptotically approaching boring](https://golangnews.com/stories/845-video-introducing-go-1.6-asymptotically-approaching-boring-by-brad-fitzpatrick). If you build an app today, in 5 years it will probably compile with minimal changes, as long as you have limited dependencies. While being difficult for maintainers, is a liberating promise for Go programmers as it means you don't have to worry about churn at that level of the ecosystem, only above it.

### Dependencies

> Duplication is far cheaper than the wrong abstraction - Sandi Metz

The biggest problem with dependencies in the long term is change - the more dependencies you have, the greater the chance of changes which break your build \(for security, or api changes, or new features\), and the more painful it becomes to keep up to date with the ecosystem you've bought into. This is why it is useful to limit your dependencies, and explicitly version those you have, and why vendoring \(taking a copy of dependencies frozen at a given version\) has become an accepted solution in the Go community to importing dependencies.

