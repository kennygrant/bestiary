# Slices

Slices are views into arrays of data, offering an efficient way of working with arrays. They can share the same data, and simply store an offset into it. This is useful if you don't need to mutate the data but can lead to some subtle bugs if you assume every slice is independent. To get to grips with how slices work internally in Go, see this [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals) article from the Go blog, and for an interesting overview of the reasons slices came to their current form, see this [blog post](https://blog.golang.org/slices) by Rob Pike on the Go blog.

## Nil slices

The zero value of a slice is a nil slice. It has no underlying array, and has 0 length. You can use it just like a normal slice though and start using it without further initialisation \(unlike maps\).

## Slicing shares data

When you use the slice operator, be aware that the backing data will be shared, using the same underlying array. If you need an independent copy of a slice to manipulate the underlying data, take a copy of it first. After you slice, the new slice will be a view onto the same data, but further operations \(like append\) on that new slice may mean it is copied, so it is usually better not to rely on slices sharing data if you wish to mutate that data.

```go
a := []int{0,1,2,3,4,5}
s := a[:]
s2 := s[1:3]
```

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

## Index out of range

One of the most common slice mistakes is to attempt to access an index past the length of an array or slice, due to programmer error. This will result in a panic:

> panic: runtime error: index out of range

While this may seem like a trivial mistake to make, it causes a panic at runtime, and you should take care to avoid it by always bounding index operations at the length of the slice. To catch this kind of error with varied input you should try running your code through [go-fuzz](https://github.com/dvyukov/go-fuzz#trophies). You can read more about how to use Go Fuzz in this article about [Fuzzing a DNS Parser](https://blog.cloudflare.com/dns-parser-meet-go-fuzzer/). 

## Slicing is limited by cap

Conversely, slicing is limited by 0 and cap, not by len, as stated in the [spec](https://golang.org/ref/spec#Slice_expressions), which may seem a little counter-intuitive, and you probably shouldn't rely on. This means in some circumstances you can retrieve the backing data for a slice even outside its range, as long as it is between len and cap. You shouldn't depend on this behaviour though, and should instead keep a copy of the original slice to make your intent clear. The slice backing data will be retained in memory no matter how much you cut down the window into it in your slice.

## Appending elements

Use the built in Append function to add an element to a slice.

```go
var s []string
s = append(s,"hello")
fmt.Println(s)
```

Appending _may_ create a new backing array, it's best not to rely on the slice argument not changing - the argument may change in length, and the slice returned may not be the slice which goes in. It does not create a copy of the slice and return it for every append, as this would waste a lot of memory. When you definitely need a copy \(for example you need to preserve the original slice as well as mutate it\), take a copy of the slice \(or the part of the slice you need\) first:

```
c := make([]byte, len(s))
copy(c, s)
```

So always assign the result of append to the same slice.

## Appending slices

You can use the append function to concatenate slices, in combination with the variadic operator to turn the second slice argument into an array of elements.

```
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

Slices have been kept intentionally simple, which means you'll be using the for range idiom a lot to perform operations on them. There is no map or forEach, just use a range on the slice.

```
for _, v := range s {
   f(v)
}
```

## Converting slice types

If you have a slice of \[\]T and wish to convert it to a slice of type \[\]interface{}, or vice versa, you will have to do it by hand with a for loop. They do not have the same representation in memory and there is no convenient way to do it.

```
t := []int{1, 2, 3, 4}
s := make([]interface{}, len(t))
for i, v := range t {
    s[i] = v
}
```

## Multi-dimensional slices or maps

Go doesn't have multi-dimensional  slices or maps, you have to create them by hand, initialising each subslice.

