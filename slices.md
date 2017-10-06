# Slices

To get to grips with how slices work internally in Go, see this [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals) article from the Go blog, and for an interesting overview of the reasons slices came to their current form, see this [blog post](https://blog.golang.org/slices) by Rob Pike on the Go blog.

## Casting slices of Interface

If you have a slice which contains Interfaces rather than concrete types, you'll need to write a for loop to convert it to a slice of a concrete type, or more efficiently do your type conversions on each element rather than converting the slice.

## Appending

Use the built in Append function to add an element to a slice.

a = append\(a,b\)

## Concatenating slices

You can use the append function to concatenate slices, in combination with the variadic operator to turn the second slice argument into an array of elements.

a = append\(a,b...\)

## ForEach on slices

There is no ForEach, just use a range on the slice, and perform the action on the elements. You can omit the index if you don't need it.

## Zero Value

The zero value of a slice is equivalent in most cases to an empty slice, but . 

