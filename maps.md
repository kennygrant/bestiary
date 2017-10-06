# Maps

Pointers

You cannot take the address of map keys or values. Use a pointer with maps if possible.



## Key order

Maps have no defined order, and keys and values returned are randomised to enforce this. 

 `m := map[string]int{"one":1,"two":2,"three":3,"four":4}`

`    for k,v := range m {`

`        fmt.Println(k,v)`

`    }`

