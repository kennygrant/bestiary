# Interfaces

An interface in go is a contract specifying which method signatures a type must have. Crucially, it is specified by the user of the interface, not the code which satisfies it.

## Don't use pointers to interface

You probably meant to use a pointer to your real type. This will usually cause a compile time error. 

## Interfaces are not pointers

An interface will only be nil when both their type and value fields are nil, so comparing interface to nil can have unexpected results.

## Empty Interface

_If you stare long enough into the Abyss, the Abyss stares back into you – Nietzsche_

Don't overuse empty interface - it means nothing.

If you find yourself using empty interface and then switching on type, consider instead defining separate functions which operate on concrete types. Don't try to use empty interface as a poor man's generics, it won't end well.

# Define at point of use

Don't define interfaces next to a concrete type. Instead define them at the point of use, beside the functions that use that interface. That way you'll avoid having to import a package just to use its interfaces.

## Accept interfaces, return types

This isn't a hard and fast rule, but it is a good guideline for.

