# Structs

A struct in Go is a sequence of fields, each of which has a name and type public and private rules are the same as names in packages - lowercase names are private , names starting with uppercase are exported. Structs can also have methods, which allows them to fulfill interfaces. Structs do not inherit from each other, but they can embed another struct to gain its fields and behaviour. The embedder does not know anything about the struct it is embedded in and cannot access it.

## Inheritance - let it go

_During a Java user group meeting James Gosling \(Java's inventor\) was asked "If you could do Java over again, what would you change?". He replied: "I'd leave out classes"_

Go eschews inheritance in favour of composition. If you're accustomed to building large hierarchies of types as a way of organising your data, you might find this jarring at first. Go interfaces provide polymorphism, so if your function needs something which barks, it simply asks for a Barker. Go embedding provides composition, so if your Barker needs to Yap as well, it can embed a struct to do so. Go provides tools which solve the same problems as inheritance, without most of the downsides. You may find you need it less than you think.

## Composition is not inheritance

Composition can do some of the same things as inheritance, but it is not the same. Inheritance was deliberately left out of Go, so don't try to recreate it by the back door. If you find yourself frustrated that your structs are not able to adjust the fields of their embedder, you're not really composing functionality. Always try to go for the simplest solution first, and only use composition if you definitely need to share behaviour between two different structs.

For example, don't try to do this:

```go
package main

import (
    "fmt"
)

// Type A knows only about A, it cannot call B methods
type A struct{}

func (a A) Foo() { a.Bar() }
func (a A) Bar() { fmt.Println("This is a method on A") }

// Type B knows about A and B
type B struct{ 
  A // Embeds A - note this is not inhertance 
}

// Type B attempts to redefine Bar() but type A doesn't know about it
func (b B) Bar() { fmt.Println("This is a method on B") }

func main() {
    B{}.Foo()
}
```

A.Foo will call A.Bar, not B.Foo as it might if B inherited from A, so it will output:

> This is a method on A

## Don't use this or self

While it is common in other languages, it is frowned upon in Go to use [self or this](https://github.com/golang/go/wiki/CodeReviewComments#Receiver_Names) in receiver names, instead use the first letter of the receiver. Go has no language support for these words, it does not use them as keywords as other languages do and it is therefore confusing to use them in a Go context.

## Pointers vs Values for Methods

You can define methods either on the struct value or struct pointer. You should follow these rules for deciding which to use

* **If in doubt, use a pointer receiver**
* If the method modifies the receiver, it must be on a pointer receiver 
* If the receiver is large, e.g. a big struct, it is cheaper to use a pointer receiver
* If you need any pointer receivers, make them all pointer receivers
* If the type is stored in a map or interface, it is not addressable and T cannot use \*T methods

For small structs you may want to use values for efficiency, but for large structs or structs which modify themselves, you will want to use pointer receivers.

You _normally_ don't have to worry about method sets as the compiler will transform pointers or values to the other in order to use methods defined on the other as a convenience, but this breaks down in some circumstances. The exceptions to this are if a type is stored in a map, or stored in an Interface, or if you want to mutate the value of the receiver within the method. This is a gnarly detail, and effective go is somewhat confusing on this score:

> The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers.

It should probably end with can only be invoked on pointers or addressable values. If you attempt to test this, you'll find that you **can** call pointer methods on struct values in most circumstances:

```go
type T struct{}

func (t T) Foo() { fmt.Println("foo") }
func (t *T) Bar() { fmt.Println("bar") }


func main() {
   // t can call methods Foo and Bar as is is addressable
   // but beware if Bar() changes the receiver changes will be lost
   t := T{}
   t.Foo()
   t.Bar() // compiler converts to (&t).Bar() 

   // tp can call methods Foo and Bar
   tp := &T{}
   t.Foo() 
   tp.Bar()   
}
```

The compiler will try to help you by inserting dereferences for pointers or pointers for types where you try to call a method, but this won't work in all instances, it breaks down if your type is stored in a map, or stored in an interface:

```go
func main() {

    // You can call the pointer method on a pointer entry in a map
    mp := make(map[int]*T)
    mp[1] = tp
    mp[1].Foo() // Allowed
    mp[1].Bar() // Allowed

    // You cannot call the pointer method on a value entry in a map
    m := make(map[int]T)
    m[1] = t
    m[1].Foo() // Allowed
    //m[1].Bar() // Not Allowed - cannot take the address of m[1]

    callFoo(tp) // Allowed
    callFoo(t)  // Allowed

    callBar(tp) // Allowed
    //callBar(t) // Not Allowed - Bar method has pointer receiver

}

func callBar(i I)  { i.Bar() }
func callFoo(i FI) { i.Foo() }
```

You can read more about this on this wiki entry on [Method Sets](https://github.com/golang/go/wiki/MethodSets).

### Why do new and make exist? {#new_and_make}

You can probably get away without using `new` at all - it simple allocates a new instance of a type, and you can use &T{} instead. Perhaps it shouldn't exist?

`make` is required for used with maps, slices and channels to initialise them with a given size, for example:

```
make(map[int]int)    // A map with no entries
make(map[int]int,10) // A map with 10 zero entries
make([]int, 10, 100) // A slice with 10 zero entries, and a capacity of 100 
c := make(chan int)  // An unbuffered channel of integers
```

###  {#new_and_make}



