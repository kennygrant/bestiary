# Slices

Slices are views into arrays of data, offering an efficient way of working with arrays of data. To get to grips with how slices work internally in Go, see this [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals) article from the Go blog, and for an interesting overview of the reasons slices came to their current form, see this [blog post](https://blog.golang.org/slices) by Rob Pike on the Go blog. Slices have been kept intentionally simple, which means you'll be using the for range idiom a lot to perform operations on them. 

## Nil slices

The zero value of a slice is a nil slice. It has no underlying array, and has 0 length. You can use it just like a normal slice though and start using it without further initialisation \(unlike maps\).

## Slicing

When you use the slice operator, be aware that the backing data might be shared. If you need an independent copy of a slice to manipulate the underlying data, take a copy of it first. After you slice, the new slice will be a view onto the same data, but further operations \(like append\) on that new slice may mean it is copied, so it is usually better not to rely on slices sharing data if you wish to mutate that data.

```go
a := []int{0,1,2,3,4,5}
s := a[:]
s2 := s[1:3]
```

## Appending elements

Use the built in Append function to add an element to a slice.

```
s = append(sa,sb)
```

Appending _may_ create a new backing array, it's best to think of it always creating a new slice, and not to rely on the slice s above having the same backing storage as sa after append.

## Appending slices

You can use the append function to concatenate slices, in combination with the variadic operator to turn the second slice argument into an array of elements.

```
s = append(s,b...)
```

## ForEach on slices

There is no ForEach, just use a range on the slice.

```
for _, v := range s {
   f(v)
}
```

## Casting slices of Interface

If you have a slice which contains Interfaces rather than concrete types, you'll need to write a for loop to convert it to a slice of a concrete type, or more efficiently do your type conversions on each element rather than converting the slice.

## Multi-dimensional slices

Go doesn't have multi-dimensional arrays or slices, you have to create them by hand.

