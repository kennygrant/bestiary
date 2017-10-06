# Packages

In go packages are a way of scoping code, and map exactly to folders. Within a package, code can refer to any identifier within that package, public or private, while those importing the package may only reference public identifiers beginning with an uppercase letter.

## Packages vs Folders

Packages are a directory on the file system containing .go files. You cannot have multiple packages within one directory.

## Importing Main

You should never try to import the main package. Always try to import from the top level down, so your lowest level packages know nothing about the levels above. This is not always possible.

## Cyclic dependencies

If you run into cyclic imports, you have a package A which imports package B which then imports package A. To avoid this, try to follow the rules about importing above. The answer to cyclical imports

Go code is organized into packages. Within a package, code can refer to any identifier \(name\) defined within, while clients of the package may only reference the package's exported types, functions, constants, and variables. Such references always include the package name as a prefix: foo.Bar refers to the exported name Bar in the imported package named foo.

## Vendoring

There is no official vendoring tool as yet in Go, though work is progressing on one \(go dep\).

