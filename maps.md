# Maps

Maps in Go are hash tables, which provide fast access to values for a given key, one value per key. You can read more about maps in go in the [Go maps in action](https://blog.golang.org/go-maps-in-action) article on the go blog.

## Maps and Goroutines

Go maps are not thread safe. If you need to access a map across goroutines \(for example in a go server\), you should use a mutex to protect access. 

sync.RWMutex

## Map values

You cannot take the address of map keys or values. You cannot assign to struct fields for structs stored as values in the map, they are immutable. You should instead store pointers in the map, and you can then mutate the structs those pointers refer to.

```go
// Define a type 
type T struct {F int}

// Define a map
m := map[int]T{1:{F:1}}

// Accessing fields on T is not allowed
m[1].F = 4
```

> cannot assign to struct field m\[1\].F in map

## Key order

Maps have no defined order, and keys and values returned are randomised to enforce this.

```go
// Define a map
m := map[int]int{1: 1, 2: 2, 3: 3, 4: 4, 5: 5}
// Range 5 times
for i := 0; i < 5; i++ {
    // The order of keys and values is undefined
    for k, v := range m {
        fmt.Printf("(%d:%d)", k, v)
    }
    fmt.Println("")
}
```

The output or ranging on a map is not deterministic

> \(2:2\)\(3:3\)\(4:4\)\(5:5\)\(1:1\)
>
> \(1:1\)\(2:2\)\(3:3\)\(4:4\)\(5:5\)

To range map keys in order, get the keys,  sort them to some predefined order, then range the keys and get the values.

## Keys not in the map

A map retrieval yields a zero value when the key is not present, so always check a key is present before using it, if this distinction is important to you \(for example if you want to treat an empty string stored in the map differently from a missing value\).

```go
key := "foo"

// You can check whether a value exists in the map
v, ok := m[key]
if ok {
  // do something with v
}

// Or you can simply get the value (a zero value if not present)
v := m[key]
```

## Nil Maps

You cannot add items to a nil map, it causes a panic, so you must always initialise map variables before use. So while you can use a nil slice, you cannot use a nil map.

```go
// Allowed 
var s []int
s = append(s,1)

// You must init a map before use
m := make(map[string]int, 10)

// This is ok
m["key"] = 1

// If you assign to a nil map
var mnil map[string]int

// This panics
mnil["key"] = 1
```

If you assign to a nil map panic ensues

> panic: assignment to entry in nil map

A nil map behaves like an empty map when reading, so if you are sure you should only read from it, it is safe to use.

## Use the right data structure

If you need fast access, use a map. If you need to fast iteration or sorting, use a slice. Iterating a map is significantly slower than iterating over a slice.

