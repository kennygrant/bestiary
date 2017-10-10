# Functions & Scope

Go scope is defined by blocks. A block is a possibly empty sequence of declarations and statements within matching brace brackets. The levels of blocks are roughly:

1. Universal block - predeclared identifiers, keywords etc
2. Package block - any top level identifier outside a function
3. Function block - receivers, arguments, return values and identifiers within a function
4. Inner Blocks - identifiers declared within an inner block in a function end at the end of that block

For example package block identifiers:

```
// An exported variable
var Exported = "hello"

// A private variable
var pkgOnly = 1
```

Function block identifiers:

```
func (receiver T)Function(argument int) (returnValue int) {
   var v int
}
```

Block within a function:

```
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

The full rules for scope in Go are set out in the [Language Spec](https://golang.org/ref/spec#Declarations_and_scope). It is broadly similar to other languages in the C family, and there are few surprises here apart from shadowing variables inadvertently \(see below\).

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

## Defer runs at the end of the function

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

## Switch fallthrough

Case statements inside a switch in go **do not fall through by default**, so break is not required. You can use the fallthrough keyword to do so if you require.

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



