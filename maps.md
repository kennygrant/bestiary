# Maps

Maps in Go are hash tables, which provide fast access to values for a given key, one value per key. You can read more about maps in go in the [Go maps in action](https://blog.golang.org/go-maps-in-action) article on the go blog.

## Maps and Goroutines

Go maps are not thread safe. If you need to access a map across goroutines \(for example in a handler\), you should use a [mutex](https://golang.org/pkg/sync/#Mutex) to protect access and make it a private variable protected by that mutex. Any access to the map should lock the mutex first, perform the operation, and then unlock it again. You can also use a RWMutex if you have a lot of reads and few writes.

Be careful when using mutexes or maps not to inadvertently make copies of them \(for example if you passed Data below by value to a function, the mutex would be copied along with the data\). For this reason you will sometimes see mutexes declared as pointers, but the important point is not to copy them \(or their data\).

```go
type Data struct {
    // mu protects access to values
    mu sync.Mutex
    // values stores our data
    values map[int]int
}

func (d *Data)Add(k,v int) {
  d.mu.Lock 
  d.values[k] = v
  d.mu.Unlock
}
```

It is often clearer to put the mutex right next to the data they protect; convention is to name them mu and place them above the field which they protect. You should very rarely need to explicitly initialise a mutex or use a pointer to a mutex, the zero value can be used directly as above. 

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

The output of ranging on a map is not deterministic:

> \(2:2\)\(3:3\)\(4:4\)\(5:5\)\(1:1\)
>
> \(1:1\)\(2:2\)\(3:3\)\(4:4\)\(5:5\)

To range map keys in order, get the keys,  sort them to some predefined order, then range the keys and get the values.

## Keys not in the map

A map retrieval yields a zero value when the key is not present, so always check a key is present before using it if this distinction is important to you \(for example if you want to treat an empty string stored in the map differently from a missing value\).

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

Maps must always be initialised before use, the zero value is not useful.

```go
// Allowed 
var s []int
s = append(s,1)

// You must init a map before use
m := make(map[string]int, 10)

// This is ok
m["key"] = 1

// If you assign to a nil map - panic
var mnil map[string]int
mnil["key"] = 1
```

If you assign to a nil map panic ensues

> panic: assignment to entry in nil map

A nil map behaves like an empty map when reading, so if you are sure you should only read from it, it is safe to use.

## Use the right data structure

If you need fast access, use a map. If you need to fast iteration or sorting, use a slice. Iterating a map is significantly slower than iterating over a slice.

