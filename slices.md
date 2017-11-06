# Slices

Slices are views into arrays of data, offering an efficient way of working with arrays. They can share the same data, and simply store an offset into it. This is useful if you don't need to mutate the data but can lead to some subtle bugs if you assume every slice is independent. To get to grips with how slices work internally in Go, see [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals), and for an interesting overview of the reasons slices came to their current form, see [Slices](https://blog.golang.org/slices) by Rob Pike on the Go blog.

## Nil slices

The zero value of a slice is a nil slice. It has no underlying array, and has 0 length. It can be used without initialisation \(unlike maps\).

## Slicing shares data

When you use the slice operator, be aware that the backing data will be shared, using the same underlying array. If you need an independent copy of a slice to manipulate the underlying data, take a copy of it first. After you slice, the new slice will be a view onto the same data, but further operations \(like append\) on that new slice may mean it is copied, so it is usually better not to rely on slices sharing data if you wish to mutate that data. This sharing of a backing store can be very useful if you don't need to change data but just use parts of it as it avoids extra allocations.

```go
a := []int{0,1,2,3,4,5}
s := a[:]
s2 := s[1:3]
```

This can be confusing though if you don't recognise what is happening. For example if you take two slices of data, then append to one of them, the results for the other view into that data are unpredictable and depend on the length of data:

```go
	// start with an array
	data := []int{0, 1, 2}

	// Get two views on data, showing positions 0 and 1
	a := data[0:1] // 0
	b := data[1:2] // 1

	a = append(a, 3, 4) // append two numbers to a, which overwrites b
	//	a = append(a, 3, 4, 5) // append three numbers to a, growing the slice, leaving b alone

    // the new slice a is as expected
    println(a[0]) 
    // but the values in b may surprise
	println(b[0]) 
```

For this reason if mutating a slice with append, you may need to take a copy of data before using it elsewhere or face unexpected side effects. This bug may hit you if you use [bytes.Split](https://golang.org/pkg/bytes/#Split) or similar functions to return sections of data from a slice of bytes.


## Slicing retains the original

Slicing a slice doesn't copy the underlying array, so the full array will be kept in memory until it is no longer referenced. If you're just using a small part of a large slice, consider copying that new slice into another shorter one so that the original data can be released.

```go
// readHeader reads the file header
func readHeader(file string) {

    // Read data at file path
    b, _ := ioutil.ReadFile(file)

    // slice the data to truncate
    s := b[:12]

    // Copy the data we want to a new shorter slice
    c := make([]byte, len(s))
    copy(c, s)

    return c   
}
```

## Slicing is limited by cap

Conversely to indexes \(from 0 to len\), slicing is limited by cap, not by len, as stated in the [spec](https://golang.org/ref/spec#Slice_expressions), which may seem a little counter-intuitive, and you probably shouldn't rely on. This means in some circumstances you can retrieve the backing data for a slice even outside its range, as long as it is between len and cap. If you need to see the original data it is better to keep a copy of the original slice to make the intent clear.


## Index out of range

One of the most common slice mistakes is to attempt to access an index past the length of an array or slice, due to programmer error. This will result in a panic at runtime:

> panic: runtime error: index out of range

While this may seem like a trivial mistake to make, it causes a panic at runtime, and you should take care to avoid it by always bounding index operations at the length of the slice. To catch this kind of error with varied input you should try running your code through [go-fuzz](https://github.com/dvyukov/go-fuzz#trophies). You can read more about how to use Go Fuzz in [Fuzzing a DNS Parser](https://blog.cloudflare.com/dns-parser-meet-go-fuzzer/).

## Copying slices 

To copy a slice and duplicate the backing data, use the built-in copy function. The destination comes first in the arguments to copy. Beware when copying - the minimum of len(dst) and len(src) is chosen as the length of the new slice, so the destination will need enough space to fit src. Do not attempt to copy into an empty slice as it will not expand automatically. 

```go
// Make a new slice to copy
src := []int{1,2,3}
// Make sure dst has enough capacity for src
dst := make([]int, len(src))
// Copy from src to dst, n is the no of elements copied
n := copy(dst, src)
```

If you try to copy a src of lenght 100 into a destination slice of length 1, only 1 entry will be copied into the destination.

## Appending elements

Use the built in append function to add an element to a slice.

```go
var s []string
s = append(s,"hello")
fmt.Println(s)
```

Appending _may_ create a new backing array, it's best not to rely on the slice argument not changing - the argument may change in length, and the slice returned may not be the slice which goes in. It does not create a copy of the slice and return it for every append, as this would waste a lot of memory. When you definitely need a copy \(for example you need to preserve the original slice as well as mutate it\), take a copy of the slice \(or the part of the slice you need\) first:

```go
// Make a new slice
c := make([]byte, len(s))

// Take a copy
copy(c, s)
```

Because of this behaviour it is best to always assign the result of append to the same slice.

## Appending slices

You can use the append function to concatenate slices, in combination with the variadic operator to turn the second slice argument into an array of elements.

```go
s = append(s,b...)
```

## Sorting

To sort a slice you can use [sort.Slice](https://golang.org/pkg/sort/#Slice) from the sort package. This package contains other helpers for operations such as stable sorts, binary search, and reversing slices.

```go
people := []struct {
        Name string
        Age  int
    }{
        {"Gopher", 7},
        {"Alice", 55},
        {"Vera", 24},
    }
sort.Slice(people, func(i, j int) bool { 
  return people[i].Name < people[j].Name 
})
```

## Map on slices

Slices have been kept intentionally simple, which means you'll be using the for range idiom a lot to perform operations on them. There is no map or forEach function, just use a range on the slice.

```go
for _, v := range s {
   f(v)
}
```

## Converting slice types

If you have a slice of \[\]T and wish to convert it to a slice of type \[\]interface{}, or vice versa, you will have to do it by hand with a for loop. They do not have the same representation in memory and there is no convenient way to do it.

```go
t := []int{1, 2, 3, 4}
s := make([]interface{}, len(t))
for i, v := range t {
    s[i] = v
}
```

## Passing slices to functions

Be aware when passing a slice to a function that while the slice itself is passed by value, it points to a backing array which does not change, even if the slice is copied, so modifying the elemnts of the slice passed in will modify elements of the original backing array. 

## Multi-dimensional slices

Go doesn't have multi-dimensional  slices or maps, you have to create them by hand, initialising each subslice.

