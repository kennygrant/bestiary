# Functions & Scope

Go scope is defined by blocks. A block is a possibly empty sequence of declarations and statements within matching brace brackets. The levels of blocks are roughly:

1. Universal block - predeclared identifiers, keywords etc
2. Package block - any top level identifier outside a function
3. Function block - receivers, arguments, return values and identifiers within a function
4. Inner Blocks - identifiers declared within an inner block in a function end at the end of that block

For example package block identifiers include variables and functions within the package:

```go
// An exported variable
var Exported = "hello"

// A private variable
var pkgOnly = 1
```

Function block identifiers include receivers, arguments, return values and identifiers within a function:

```go
// Functions define a block
func (receiver T)Function(argument int) (returnValue int) {
   var v int
}
```

A block within a function:

```go
func (receiver T) Function(argument int) (returnValue int) {
    var v int

    // Inner block begins
    if argument > 0 {
        var v int // shadows outer v
        v = 4 + v
    } 
    // block ends 

    fmt.Printf("v=%d\n", f)
    return v
}

func main() {
    T{}.Function(1)
}
```

The full rules for scope in Go are set out in the [Language Spec](https://golang.org/ref/spec#Declarations_and_scope). Blocks all create a new scope, so an if statement will start a new block, and variables declared within it will go out of scope when it ends. In contrast with Java, blocks protect their variables from the outer scope, you can overwrite a variable in the outer scope by declaring a new one in an inner scope. If required declare the variable before an if block and use it within. Otherwise Go is broadly similar to other languages in the C family, and there are few surprises here apart from shadowing variables inadvertently \(see below\).

## Shadowing

Be careful when using the automatic assignment operator := that you don't accidentally shadow variables from the outer block. In the case below err is created twice, and if you relied on err being set in the outside scope it would not be. In most cases this isn't a problem but it's something to be aware of. Most linters warn about any dangerous instances of shadowing so if you use a linter it is not usually a problem in practice.

```go
// err is created
err := f()
if err != nil {
   // err is recreated - outer err is not updated
   v, err := f()
}
```

## Defer runs after the function

The defer statement does not run at the end of a block, but at the end of the containing function.

## Defer arguments are frozen

The defer statement freezes its arguments the instant it is evaluated \(the line it occurs in the source code\), not when it executes after leaving the containing function. This means any values changed after it occurs do not get passed to defer.

```go
  s := "hello"

  // At point of defer, s is "hello"
  defer fmt.Print(s)

  // Not used, even though defer runs after
  s = "hello world"
```

## Switch fall through

Case statements inside a switch in go **do not fall through by default**, so break is not required. You can use the fallthrough keyword to do so if you require. This should be used sparingly and carefully annotated because of the difference with C - programmers not familiar with Go may be tripped up by it so use this sparingly if at all.

```go
switch(i) {
case 0:
// do something
case 1:
// do something
case 3:
   fallthrough // the fallthrough keyword is required to fall through explicitly
default:
// do something
}
```

## Naked Returns

In go, naked returns \(the use of the keyword return without parameters\) will return the current state of the named return values. This is sometimes used as a shortcut to avoid specifying what is returned.  Try to avoid using naked returns, they are unclear, particularly within or at the end of a large function. Instead specify exactly what will be returned, and use nil for values when returning an error.

## Prefer synchronous functions

Try to write synchronous functions, which can then be transformed by use of the go keyword into asynchronous ones. Do not attempt to make your API async by default by using go keywords within library functions. It is easy to add concurrency with the go keyword, but impossible to remove it if a function uses it internally.

## Prefer functions over methods

Coming from an object-oriented background, many programmers reach for structs and methods first. Before using a method, you should consider whether you could instead use a function. Functions are independent of the data they work with, and ideally use their inputs and no other state, so that the output is predictable. Use a method where you need to reflect the state of the type the method is attached to.

