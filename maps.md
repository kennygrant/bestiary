# Maps

## Pointers

You cannot take the address of map keys or values. Use a pointer with maps if possible.

### Changing map values

You cannot assign to struct fields for structs stored as values in the map, they are immutable. You should instead store pointers in the map, and you can then mutate the structs those pointers refer to.

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

## Nil Maps

You cannot add items to a nil map, it causes a panic, so you must always initialise map variables before use.

## 



