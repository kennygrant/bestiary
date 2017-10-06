# Functions & Scope

Go scope is defined by blocks, to summarise:

1. Package block - any top level identifier outside a function
2. Function block - receivers, arguments, return values and identifiers within a function
3. Block - identifiers declared within an inner block in a function end at the end of that block

For example package block identifiers:

```
// An exported variable
var Exported  = "hello"

// A private variable
var pkgOnly  = 1
```

Function block identifiers:

```
func (receiver *T)Example(argument int) (returnValue int) {
   var f int
}
```

Block within a function: 

```
func (r *Receiver)Example(argument int) (returnValue int) {
   var f int
   if argument > 0 {
      // Inner block scope 
      var f int
   } // inner no longer valid
   fmt.Printf("%d",f)
}
```



The full rules for scope in Go are set out in the [Language Spec](https://golang.org/ref/spec#Declarations_and_scope). 



The scope of a

1. [predeclared identifier](https://golang.org/ref/spec#Predeclared_identifiers) is the universe block.
2. The scope of an identifier denoting a constant, type, variable, or function \(but not method\) declared at top level \(outside any function\) is the package block.
3. The scope of the package name of an imported package is the file block of the file containing the import declaration.
4. The scope of an identifier denoting a method receiver, function parameter, or result variable is the function body.
5. The scope of a constant or variable identifier declared inside a function begins at the end of the ConstSpec or VarSpec \(ShortVarDecl for short variable declarations\) and ends at the end of the innermost containing block.
6. The scope of a type identifier declared inside a function begins at the identifier in the TypeSpec and ends at the end of the innermost containing block.



In go the scope of function params and return values is the same as the function body.

## Block Scope

Be careful if you use the automatic assignment operator := that you don't accidentally shadow variables from the outer block. In the case below err is created twice, and if you relied on err being set in the outside scope it would not be. In most cases this isn't a problem but it's something to be aware of.

```go
// err is created
err := f()
if err != nil {
   // err is recreated - outer err is not updated
   v, err := f()
}
```

## Defer arguments are frozen

The defer statement freezes its arguments the instant it is evaluated \(the line it occurs in the source code\), not when it executes after leaving the containing function. This means any values changed after it occurs do not get passed to defer.

```go
  s := "hello"

  // At point of defer, s is "hello"
  defer fmt.Print(s)

  // Not used, even though defer runs after
  s = "hello world"
```

## Defers run at the end of the function

The defer statement does not run at the end of a block, but at the end of the containing function.

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

## 



