# Testing

Go offers fairly simple primitives for testing, and no test framework is particularly popular. You should try to use the tools provided by the standard library before investigating using a framework. Once again, less is more. 

## Unit tests 


## Benchmarks


```go

var result int 

// in foo.go
func Foo() {
    i := 1
    foo := &i
    // store results in a package level variable to avoid elimination
    result = i
    time.Sleep(1*time.Second)
}
```

```go
// in foo_test.go
func BenchmarkFoo(b *testing.B) {
        // run the Foo function b.N times
        for n := 0; n < b.N; n++ {
                Foo()
        }
}
```

Be careful that your machine is under predictable load when testing - this can skew results and make them meaningless. 

Be very careful when writing a benchmark that you don't end up benchmarking nothing, or benchmarking the setup work involved in setting up your tests rather than the work itself. If your code is sufficiently simple, it may get optimised away in strange ways. 

// This pattern is common, how to illustrate this? Show stdlib?
func BenchmarkClient2(b *testing.B)  {  BenchmarkClient(2, b) }
func BenchmarkClient20(b *testing.B)  { BenchmarkClient(20, b) }


Don't use the value of b.N - this is used for looping through tests ,and should not be used in the test itself. 




## Integration tests 

Quick example of testing against a test database. Why you should not mock. 


## Don't mock me

Why mocking is bad and you should feel bad. 


## Table-driven tests

You can use anomyous structs in order to construct tables of tests with results and expected results. Try to name the tables by the functions they are used within, and place them next to those functions. You can see examples of this style of test in the standard library net/http package. 

```go
var theTests = []struct {
        input string
        result   string 
}{
        {"hello", "hell"},
}
```

Simple example trimming strings of o.

## Test fixtures in Go

Use a folder named testdata as it will be ignored by the go tool. Use relative paths:


