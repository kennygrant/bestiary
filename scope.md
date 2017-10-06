# Functions & Scope

Scope in go is fairly straightfoward.

## Defers run at the end of the function

The defer statement does not run at the end of a block, but at the end of the function

## Defer arguments evaluated early

The defer statement freezes its arguments the instant it is evaluated \(the line it occurs in the source code\), not when it executes after leaving the containing function. This means any values changed after it occurs do not get passed to defer. 

```go
  s := "hello"
  
  // At point of defer, s is "hello"
  defer fmt.Print(s)
  
  // Not used, even though defer runs after
  s = "hello world"
```

## Switch

Case statements inside a switch in go do not fall through by default, so break is not required.

## Block Scope

```go
// err is created
err := f()
if err != nil {
   // err is recreated - outer err is not updated
   v, err := f()
}
```



