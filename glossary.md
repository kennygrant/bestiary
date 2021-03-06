# Glossary

## Channels

Unbuffered channels combine communication — the exchange of a value — with synchronization — guaranteeing that two calculations \(goroutines\) are in a known state.

## Goroutines

A goroutine is a function executing concurrently with other goroutines in the same address space. It is a lightweight alternative to threads invoked by using the go keyword.

## Interfaces

Interfaces in Go provide a way to specify the behaviour of an object by defining a required method set. Commonly used interfaces like io.Writer and io.Reader usually only have very few methods.

## Slices

Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data. Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

## Maps

Go provides a built-in map type that implements a hash table, which is a data structure offering fast lookups, adds and deletes, but no defined order.

## Mutexes

Mutexes provide a locking mechanism to ensure only one goroutine is running on a critical section of code at any one time. Typically they are used to control access to state like a map between goroutines.