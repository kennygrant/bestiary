# Functions & Scope

In go the scope of function params and return values is the same as the function body.

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

## Block Scope

```go
// err is created
err := f()
if err != nil {
   // err is recreated - outer err is not updated
   v, err := f()
}
```

## Enums

There are no first class enums in Go. You can use the keyword iota to increment constants from a known base, so the closest to an enum is a set of constants in a file:

```go
// Describe the constants here
const (
   RoleAnon = iota 
   RoleReader
   RoleAdmin
)
```



