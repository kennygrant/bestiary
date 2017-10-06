# Slices

Slices are views into arrays of data, offering an efficient way of working with arrays. They can share the same data, and simply store an offset into it. This is useful if you don't need to mutate the data but can lead to some subtle bugs if you assume every slice is independent. To get to grips with how slices work internally in Go, see this [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals) article from the Go blog, and for an interesting overview of the reasons slices came to their current form, see this [blog post](https://blog.golang.org/slices) by Rob Pike on the Go blog. 

## Nil slices

The zero value of a slice is a nil slice. It has no underlying array, and has 0 length. You can use it just like a normal slice though and start using it without further initialisation \(unlike maps\).

## Slicing

When you use the slice operator, be aware that the backing data might be shared, using the same underlying array. If you need an independent copy of a slice to manipulate the underlying data, take a copy of it first. After you slice, the new slice will be a view onto the same data, but further operations \(like append\) on that new slice may mean it is copied, so it is usually better not to rely on slices sharing data if you wish to mutate that data.

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

## Casting slices of Interface

If you have a slice which contains Interfaces rather than concrete types, you'll need to write a for loop to convert it to a slice of a concrete type, or more efficiently do your type conversions on each element rather than converting the slice.

## Multi-dimensional slices

Go doesn't have multi-dimensional arrays or slices, you have to create them by hand.

