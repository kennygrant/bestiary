# Packages

In go packages are a way of scoping code, and map exactly to folders. Within a package, code can refer to any identifier within that package, public or private, while those importing the package may only reference public identifiers beginning with an uppercase letter. Each go file in a package begins with a declaration at the top of the file

```
package x
```

where x is the package name. The name should be short, explicit and avoid punctuation - see [Package Names](https://blog.golang.org/package-names) on the Go blog.

## Packages vs Dirs

Packages are a directory on the file system containing .go files. You cannot have multiple packages within one directory. By convention, the package name is the same as the directory name, and the directory should contain a file of the package name with the .go suffix which contains the primary exports. You can if necessary rename packages on import.

## Don't import main

You should never try to import the main package. Always try to import from the top level down, so your lowest level packages know nothing about the levels above. This is not always possible but is a good guideline to limit fragile web of dependencies.

## Cyclic dependencies

If you run into cyclic imports, you have a package A which imports package B which then imports package A. To avoid this, try to keep your imports in one direction, importing small packages into the top level one. For example for a git command line tool you might have a main command, which imported a package which contains all the git specific structures and could be used as a library by that command, a server or any other type of app. 

If you structure your apps this way you will avoid cyclic imports and you should consider them a hint that something is wrong with your design. 

## Vendoring

There is no official vendoring tool as yet in Go, though work is progressing on one \(go dep\).

