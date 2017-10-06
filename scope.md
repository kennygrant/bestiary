# Functions & Scope

Scope rules for Go



Defer does not run 

## Block Scope

```go
// err is created
err := f()
if err != nil {
   // err is recreated - outer err is not updated
   v, err := f()
}
```



