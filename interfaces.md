# Interfaces

An interface in go is a contract specifying which method signatures a type must have. Crucially, it is specified by the user of the interface, not the code which satisfies it.

## Accept interfaces, return types

Interfaces are a way of avoiding tight coupling between different packages, so they are most useful when defined at their point of use, and only used there. If you export an interface as a return type, you are forcing others to use this interface only forever, or to attempt to cast your interface back into a concrete type.

Do not design interfaces for mocking, design them for real world use, and don't add methods to them before you have a concrete use for the methods.

## Keep interfaces simple

Interfaces are at their most powerful when they express a simple contract that any type can easily conform to. If they start to demand a laundry list of functions \(say over around 5\), they have very little advantage over a concrete type as an argument, because the caller is not going to be able to create an alternative type without substantially recreating the original.

## Avoid the empty Interface 

The empty interface is written like this, but unlike interfaces it requires nothing:

```
interface{}
```

Don't overuse empty interface - it means nothing. If you find yourself using empty interface and then switching on type, consider instead defining separate functions which operate on concrete types. Don't try to use empty interface as a poor man's generics.

## Don't use pointers to interface

You probably meant to use a pointer to your real type, or just a plain old Interface.

## Interfaces are not pointers

An interface will only be nil when both their type and value fields are nil, so comparing interface to nil can have unexpected results.

