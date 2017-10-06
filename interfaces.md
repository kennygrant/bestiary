# Interfaces

Int

## Empty Interface

Don't overuse empty interface - it makes no promises about the type passed in. If you find yourself passing in empty interface them switching on type, consider instead defining separate functions which act on different types, or an interface to define your requirements of the types passed in. 



## Define at point of use

Don't define interfaces next to a concrete type. Instead define them at the point of use, beside the functions that use that interface. 



## Accept interfaces, return types

This isn't a hard and fast rule, but it is a good guideline for. 



