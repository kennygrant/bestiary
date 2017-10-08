# Glossary 

## Channels
Unbuffered channels combine communication — the exchange of a value — with synchronization — guaranteeing that two calculations (goroutines) are in a known state.

## Goroutine
A goroutine is a function executing concurrently with other goroutines in the same address space. It is a lightweight alternative to threads invoked by using the go keyword.  

## Slice
Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data. Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

## Map
Go provides a built-in map type that implements a hash table, which is a data structure offering fast lookups, adds and deletes, but no defined order.